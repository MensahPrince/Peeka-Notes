---
id: fnp814n35n
title: "15 Unsafe Rust"
sort: 6000
version: 1
---

<callout background="red" icon="☢️">

**`unsafe` doesn't turn off the rules. It means *you* now promise the rules hold**, in places the compiler can't check. Safe Rust can never cause undefined behaviour (UB); unsafe Rust can, if you break a promise. The skill is building small, audited unsafe cores with safe APIs on top.

</callout>

**Level:** 🔴 Expert · **Needs:** all of Basics and Intermediate, especially [03 Ownership and Borrowing](page:acpvbrr41r) and [09 Smart Pointers and Interior Mutability](page:p7tycay9d0)

## The five superpowers

Inside an `unsafe` block you may additionally:

1. **Dereference a raw pointer** (`*const T`, `*mut T`)
1. **Call an `unsafe fn`** (including foreign functions and many intrinsics)
1. **Access or modify a `static mut`**
1. **Implement an `unsafe trait`** (`Send`, `Sync`, `GlobalAlloc`)
1. **Read a field of a `union`**

That's all. The borrow checker, type checker and lifetimes all still run inside `unsafe`.

## Undefined behaviour: what you're promising to avoid

UB doesn't mean "crashes". It means the compiler assumed it can't happen and optimised on that assumption, so **anything** can result, often far from the bug.

<columns>

<column>

### Memory

- Dereferencing a dangling, null or misaligned pointer
- Reading or writing out of bounds
- Use after free, double free
- Reading uninitialised memory (as anything but `MaybeUninit`)

</column>

<column>

### Aliasing and values

- A `&mut T` that aliases another live reference
- Mutating through a `&T` (outside `UnsafeCell`)
- Data races
- Invalid values: a `bool` that's 2, a `char` above `0x10FFFF`, an enum with an unknown tag, a null `&T`
- Unwinding into code that can't unwind

</column>

</columns>

## Raw pointers

```rust
let mut x = 10;
let p = &raw mut x;          // *mut i32, created without making a reference
let q = &raw const x;        // *const i32

unsafe {
    *p += 1;                 // deref needs unsafe
    println!("{}", *q);      // 11
}
```

Creating raw pointers is safe; **using** them is the unsafe part. Raw pointers can be null, dangling, unaligned and aliased; they carry no lifetime.

Useful tools: `ptr.add(n)` / `offset`, `ptr::read` / `ptr::write` (move without dropping), `ptr::copy_nonoverlapping` (memcpy), `ptr::null_mut()`, `NonNull<T>` (non-null, covariant), `slice::from_raw_parts`.

## Writing unsafe code responsibly

```rust
/// Returns the element at `i` without bounds checking.
///
/// # Safety
/// `i` must be less than `v.len()`.
pub unsafe fn get_unchecked_demo(v: &[u32], i: usize) -> u32 {
    // SAFETY: the caller guarantees i < v.len(), so the pointer is in bounds
    // and points to an initialised u32.
    unsafe { *v.as_ptr().add(i) }
}
```

- An `unsafe fn` has a **`# Safety`** doc section: the contract callers must meet.
- Every `unsafe {}` block has a **`// SAFETY:`** comment: why the contract is met here.
- Edition 2024 requires an explicit `unsafe {}` *inside* an `unsafe fn` too (lint `unsafe_op_in_unsafe_fn`), so each operation is justified.
- Keep unsafe blocks **tiny**, and wrap them in a type whose safe methods maintain the invariants.

## A safe abstraction over unsafe code

You can't take two `&mut` borrows of the same slice, even of disjoint halves. The standard library's `split_at_mut` uses unsafe to do it, behind a safe signature:

```rust
fn split_at_mut<T>(s: &mut [T], mid: usize) -> (&mut [T], &mut [T]) {
    let len = s.len();
    assert!(mid <= len);
    let ptr = s.as_mut_ptr();
    // SAFETY: both halves are within the original slice and don't overlap,
    // and the original borrow is held for the returned lifetime.
    unsafe {
        (
            std::slice::from_raw_parts_mut(ptr, mid),
            std::slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

The `assert!` is what makes this sound: without it, a caller in safe code could create out-of-bounds slices. **A safe function must be sound for every possible input.**

### Building up to 15.1

**The goal in plain words:** the borrow checker won't let you hold two `&mut` into the same slice, even when you *know* they point at different elements. You'll use `unsafe` to make two non-overlapping mutable views, and wrap it in a safe function, so callers can't misuse it whatever they pass in. Then you'll check your work with Miri, a tool that catches unsafe mistakes.

**Step 1: see the problem in safe code.**

```rust
let mut v = [1, 2, 3, 4];
let a = &mut v[0];
let b = &mut v[3];   // ❌ E0499: v is already borrowed mutably
*a += *b;
```

The compiler doesn't track *which* elements a borrow covers, only that it borrows `v`.

**Step 2: a slice is a pointer plus a length.** `s.as_mut_ptr()` gives the raw pointer to element 0. `ptr.add(n)` moves it `n` elements along. `slice::from_raw_parts_mut(ptr, len)` builds a `&mut [T]` from the two again, and it's `unsafe` because *you* promise the pointer and length are valid and that nothing else is using that memory.

**Step 3: type `split_at_mut` from memory.** Work out each piece before you write it: the left half starts at `ptr` and has `mid` elements, and the right half starts at `ptr.add(mid)` and has `len - mid`. Which line stops a caller passing `mid = 100` on a 4-element slice?

**Step 4: `split_first_last`, safety first.** Before writing any `unsafe`, list the lengths a caller might pass in and what should happen for each:

| `s.len()` | First and last are… | Return |
| --- | --- | --- |
| 0 | there aren't any | `None` |
| 1 | the **same** element: two `&mut` to it would be UB | ? |
| 2 or more | different elements | `Some((first, last))` |

Then `&mut *p` turns a raw pointer back into a reference (the element at index 0), and `&mut *p.add(n - 1)` gives the last one.

**Step 5: run it under Miri.** Miri runs your tests in a checking interpreter that reports undefined behaviour:

```bash
rustup +nightly component add miri
cargo +nightly miri test
```

Write a test calling both functions, including the length-1 and empty cases. To see what a Miri report looks like, temporarily make the length-1 case return two references to the same element.

<callout background="blue" icon="🏋️">

**Exercise 15.1 — Implement split\_at\_mut**

Type the function above from memory. Then write `fn split_first_last<T>(s: &mut [T]) -> Option<(&mut T, &mut T)>` returning mutable refs to the first and last elements (what if `len == 1`?). Test both under Miri.

</callout>

<toggle>

Solution sketch

```rust
fn split_first_last<T>(s: &mut [T]) -> Option<(&mut T, &mut T)> {
    if s.len() < 2 {
        return None; // len 1 would give two &mut to the same element: UB
    }
    let p = s.as_mut_ptr();
    let n = s.len();
    // SAFETY: n >= 2, so index 0 and n-1 are distinct and in bounds.
    unsafe { Some((&mut *p, &mut *p.add(n - 1))) }
}
```

Safe alternative worth knowing: `let (first, rest) = s.split_first_mut()?; let last = rest.last_mut()?;`.

</toggle>

## Miri: a UB detector

Miri interprets your program and checks every memory operation against Rust's rules, including the aliasing model (**Stacked Borrows**, or **Tree Borrows** with `-Zmiri-tree-borrows`).

```bash
rustup +nightly component add miri
cargo +nightly miri test
cargo +nightly miri run
```

### Building up to 15.2

**The goal in plain words:** four programs that compile and *seem* to work, but each breaks one of the promises from the UB list above. Undefined behaviour often doesn't crash. It just quietly does the wrong thing sometimes. Miri catches it reliably. Run each snippet under Miri, work out which rule it breaks, and fix it.

**Step 1: one snippet per run.** Put each one in its own test (`#[test] fn a() { … }`), and run them one at a time: `cargo +nightly miri test a`.

**Step 2: read the report.** A Miri error looks like this (from a different bug):

```text
error: Undefined Behavior: out-of-bounds pointer use: alloc1234 has size 12, so pointer at offset 16 is out-of-bounds
  --> src/lib.rs:7:20
   |
7  |     unsafe { *p.add(4) }
   |              ^^^^^^^^^ out-of-bounds pointer use
```

The first line names the kind of UB. Match it to the list near the top of this page.

**Step 3: the rule behind each snippet.** Use these to know what to look for:

| Snippet | Look at | The rule |
| --- | --- | --- |
| a | when `v` is dropped vs when `p` is read | a pointer is only valid while the thing it points to is alive |
| b | the order: `a` is made, then `r`, then both are used | making a new `&mut` invalidates older pointers to the same thing |
| c | which values a `bool` may hold | only `0` (false) and `1` (true): anything else is an invalid value |
| d | where a `u32` may be stored | a `u32` must be at an address divisible by 4, and a `[u8; 4]` might not be |

**Step 4: fix the smallest thing.** Keep the program's intent: move a `drop`, reorder two lines, replace `transmute` with a safe comparison, or use a method that allows unaligned writes (`write_unaligned`). Re-run Miri until it's quiet.

<callout background="blue" icon="🏋️">

**Exercise 15.2 — Catch UB with Miri**

Each snippet compiles and probably "works". Run each under Miri, read the report, and fix it:

```rust
// a
let v = vec![1, 2, 3];
let p = v.as_ptr();
drop(v);
println!("{}", unsafe { *p });

// b
let mut x = 5;
let a = &mut x as *mut i32;
let r = &mut x;
unsafe { *a = 6; }
*r += 1;

// c
let b: bool = unsafe { std::mem::transmute(2u8) };

// d
let mut arr = [0u8; 4];
let p = arr.as_mut_ptr() as *mut u32;
unsafe { *p.add(0) = 1; }   // hint: alignment
```

</callout>

<toggle>

What Miri says

- **a**: use after free. Keep `v` alive while `p` is used.
- **b**: aliasing violation. Creating `r` (a new `&mut x`) invalidates `a`. Use only the raw pointer, or create `r` after the last use of `a`.
- **c**: invalid value for `bool`. `transmute` is the most dangerous function in Rust; prefer `u8 != 0`.
- **d**: `[u8; 4]` has alignment 1, but writing a `u32` needs alignment 4. Use `p.write_unaligned(1)` or `u32::to_ne_bytes`.

</toggle>

## Uninitialised memory

```rust
use std::mem::MaybeUninit;

let mut buf: [MaybeUninit<u32>; 4] = [MaybeUninit::uninit(); 4];
for (i, slot) in buf.iter_mut().enumerate() {
    slot.write(i as u32 * 10);
}
// SAFETY: every element was written above.
let buf: [u32; 4] = unsafe { std::mem::transmute(buf) };
```

`MaybeUninit<T>` is the only legal way to hold memory that may not be a valid `T` yet. `mem::uninitialized()` and `mem::zeroed()` on types where zero isn't valid (references, `NonNull`, many enums) are instant UB.

## Unsafe traits: Send and Sync

- `Send`: safe to **move** to another thread. `Sync`: safe to **share** `&T` between threads.
- They're auto-implemented when all fields are. Raw pointers are neither, so a type containing one opts out.
- `unsafe impl Send for MyType {}` is you promising the compiler it's thread-safe.

### Building up to 15.3

**The goal in plain words:** `Vec<T>` is ordinary Rust code built on a few unsafe operations. You'll write your own: ask the allocator for a block of memory, put values into it, make it bigger when it fills up, and give everything back at the end, without leaking anything or dropping anything twice. It's the best way to understand what a `Vec` really is.

**Step 1: the three fields.** A `Vec` is a pointer to a heap buffer, how many slots the buffer has, and how many are filled:

```text
ptr ──► [ "a" | "b" | "c" | ░░░ | ░░░ ]     cap = 5 (slots)
                                            len = 3 (filled: 0..3 hold real values)
```

`NonNull<T>` is a raw pointer that's never null. Start with `NonNull::dangling()` (a placeholder that's never read) and `cap = 0`, so an empty vector allocates nothing.

**Step 2: asking for memory.** `Layout` describes a block of memory, and `alloc` hands one out:

```rust
use std::alloc::{alloc, dealloc, Layout};

let layout = Layout::array::<u64>(4).unwrap();   // room for 4 u64s: 32 bytes, aligned to 8
unsafe {
    let p = alloc(layout) as *mut u64;           // may return null if memory is out
    p.write(7);                                  // put a value in slot 0
    println!("{}", p.read());                    // 7
    dealloc(p as *mut u8, layout);               // give it back, with the SAME layout
}
```

`realloc(old_ptr, old_layout, new_size_in_bytes)` grows a block, moving it if needed.

**Step 3: `write` and `read`, never `=`.** The slots past `len` hold garbage. `*p = value` would first *drop* the garbage that's there, which is UB for something like a `String`. So:

- `ptr::write(slot, value)` puts a value in without touching what was there (for `push`).
- `ptr::read(slot)` copies the value out and leaves the slot to be treated as empty (for `pop`).

**Step 4: build it in this order, testing after each.**

1. `new`, `len`, and a `grow` that doubles `cap` (with a minimum of 4).
1. `push` (grow if `len == cap`, write into slot `len`, add 1 to `len`) and `pop` (subtract 1 from `len`, read slot `len`).
1. `Drop`: pop until empty (that drops each element), then `dealloc` if `cap > 0`.
1. `Deref<Target = [T]>` using `slice::from_raw_parts(ptr, len)`. Indexing, `iter()` and `len()` on slices then work for free.
1. The edge cases: `Layout::array` returns `Err` on capacity overflow, and a zero-sized `T` (like `()`) needs no memory at all.

**Step 5: test with `String` under Miri.** Numbers can't show a missing drop. With `MyVec<String>`, push a few strings, pop one, let the rest drop, and run `cargo +nightly miri test`. Miri reports leaks (memory never freed) and double frees.

<toggle>

Other unsafe corners worth knowing

- **`static mut`**: taking references to it is denied in edition 2024 (`static_mut_refs`). Use an atomic, `Mutex`, `OnceLock`/`LazyLock`, or `&raw mut STATIC` with care.
- **`union`**: like a C union; reading a field reinterprets the bytes. Used for FFI and very compact data.
- **`Pin` and self-referential structs**: projecting through `Pin` manually is unsafe; use `pin-project-lite`.
- **Variance and `PhantomData`**: a raw pointer `*mut T` is invariant over `T`; `NonNull<T>` is covariant. Getting this wrong in a container type is unsound. See the Rustonomicon's chapter on subtyping.
- **Drop check**: `#[may_dangle]` and why `Vec<T>` can drop `T`s that borrow shorter-lived data.

</toggle>

<callout background="blue" icon="🏋️">

**Exercise 15.3 — Build MyVec\<T> from scratch**

Implement a growable `MyVec<T>` with `std::alloc::{alloc, realloc, dealloc, Layout}` and `NonNull<T>`: `new`, `push`, `pop`, `len`, `Index`/`IndexMut` via `Deref<Target = [T]>`, and `Drop` that drops every element *then* frees the buffer. Handle zero-sized types (`MyVec<()>`) and capacity overflow. Test it under Miri, including with `T = String` to catch leaks and double drops.

</callout>

<toggle>

Roadmap and hints

1. Fields: `ptr: NonNull<T>`, `cap: usize`, `len: usize`. Start with `NonNull::dangling()` and `cap = 0`.
1. `grow`: new cap = `max(4, cap * 2)`; `Layout::array::<T>(new_cap)` (returns `Err` on overflow); `alloc` the first time, `realloc` after. Check for null and call `std::alloc::handle_alloc_error`.
1. `push`: grow if `len == cap`, then `ptr::write(self.ptr.as_ptr().add(self.len), value)` (not `*p = value`, which would drop garbage).
1. `pop`: `len -= 1`, then `ptr::read`.
1. `Drop`: `while self.pop().is_some() {}` (or `ptr::drop_in_place` on the slice), then `dealloc` if `cap != 0`.
1. ZSTs: `size_of::<T>() == 0` means never allocate and treat capacity as `usize::MAX`.

The Rustonomicon's **"Implementing Vec"** chapter walks through exactly this; read it *after* your attempt.

</toggle>

## Checklist

- [ ] I can list the five unsafe superpowers and a dozen kinds of UB
- [ ] Every unsafe fn I write has `# Safety`; every unsafe block has `// SAFETY:`
- [ ] I run Miri on code with unsafe in it
- [ ] I can explain why `split_at_mut` is sound and what would make it unsound

---

← [14 Advanced Types and Traits](page:olti30ae4p) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [16 FFI: Rust and C](page:zrjvts0nzd)
