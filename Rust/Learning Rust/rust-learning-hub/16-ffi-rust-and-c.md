---
id: zrjvts0nzd
title: "16 FFI: Rust and C"
sort: 5000
version: 1
---

<callout background="red" icon="🔌">

**The foreign function interface.** C is the lingua franca: every OS API, most native libraries, and Python/Node/Go all speak the C ABI. Learn to call C from Rust, call Rust from C, and keep ownership straight across the boundary.

</callout>

**Level:** 🔴 Expert · **Needs:** [15 Unsafe Rust](page:fnp814n35n)

## Calling C from Rust

Declare the foreign functions in an `extern` block. In edition 2024 the block itself is `unsafe extern`, because *you* are vouching that the signatures are correct; items you know are always safe to call can be marked `safe`.

```rust
use std::ffi::{c_char, c_int, CStr, CString};

unsafe extern "C" {
    safe fn abs(x: c_int) -> c_int;          // no preconditions: callable without unsafe
    fn strlen(s: *const c_char) -> usize;     // needs a valid NUL-terminated pointer
}

fn main() {
    println!("{}", abs(-7));
    let s = CString::new("hello").unwrap();   // adds the trailing NUL
    let n = unsafe { strlen(s.as_ptr()) };
    println!("{n}");
}
```

### Rust ↔ C type map

| C | Rust |
| --- | --- |
| `int`, `unsigned int`, `long` | `c_int`, `c_uint`, `c_long` (from `std::ffi`) |
| `int32_t`, `uint64_t`, `size_t` | `i32`, `u64`, `usize` |
| `double`, `float` | `f64`, `f32` |
| `bool` (C99) | `bool` |
| `char *` (owned or borrowed string) | `*const c_char` / `*mut c_char`; wrap with `CStr` / `CString` |
| `T *` that can be NULL | `*const T`, or `Option<&T>` / `Option<NonNull<T>>` |
| `struct S {...}` | `#[repr(C)] struct S {...}` |
| `enum` | `#[repr(C)]` or `#[repr(i32)]` enum, or plain integer constants (safer: C can send any value) |
| function pointer | `Option<extern "C" fn(...)>` |
| opaque `struct handle *` | `#[repr(C)] struct Handle { _private: [u8; 0] }` and `*mut Handle` |

<callout background="yellow" icon="⚠️">

**Strings:** `String`/`&str` are UTF-8 with a length and **no NUL**. C strings are NUL-terminated bytes with no length. `CString` = owned, for passing *to* C. `CStr` = borrowed view of a C string you *received*. Never pass `my_string.as_ptr()` directly.

</callout>

### Building up to 16.1

**The goal in plain words:** your computer already has a C library (libc) built in, full of useful functions. Rust can call them directly, as long as you tell it each function's exact signature. You'll call four of them, each harder than the last: one with numbers, one taking a string, one *returning* a string that might be missing, and one that calls *your* Rust function back.

**Step 1: find the C signature.** `man 3 strlen` (or search "man getenv") shows the C declaration. Translate it with the type map above:

| C declaration | Rust declaration |
| --- | --- |
| `pid_t getpid(void);` | `fn getpid() -> c_int;` (`pid_t` is an `int` on Linux) |
| `size_t strlen(const char *s);` | `fn strlen(s: *const c_char) -> usize;` |
| `char *getenv(const char *name);` | `fn getenv(name: *const c_char) -> *mut c_char;` |

They all go in one `unsafe extern "C" { … }` block. libc is linked automatically, so no extra setup is needed.

**Step 2: Rust string → C string.** C wants a NUL-terminated pointer. `CString::new("HOME")` makes one (and fails if your text contains a NUL byte), and `.as_ptr()` lends it to C. Keep the `CString` in a variable while C uses the pointer:

```rust
let name = CString::new("HOME").unwrap();
let p = unsafe { getenv(name.as_ptr()) };   // name is still alive here: good
```

**Step 3: C string → Rust, maybe missing.** `getenv` returns NULL if the variable isn't set. Check that first. Otherwise, `CStr::from_ptr` reads up to the NUL, and `to_string_lossy` turns the bytes into Rust text:

```rust
if p.is_null() {
    None
} else {
    Some(unsafe { CStr::from_ptr(p) }.to_string_lossy().into_owned())
}
```

**Step 4: C calls Rust back.** `qsort` sorts any array, so it asks *you* how to compare two elements. You give it a function written with C's calling convention (`extern "C" fn`). It receives untyped pointers (`*const c_void`) that you cast back to `*const i32`, and returns negative, zero or positive. The hint shows both halves. Check that the array prints sorted afterwards.

<callout background="blue" icon="🏋️">

**Exercise 16.1 — Call libc**

Without the `libc` crate, declare and call: `getpid`, `strlen`, `getenv` (convert the returned `*mut c_char` to a Rust `Option<String>` via `CStr`, handling NULL), and `qsort` on a `[i32]` with a Rust `extern "C" fn` comparator.

</callout>

<toggle>

qsort hint

```rust
use std::ffi::{c_int, c_void};

unsafe extern "C" {
    fn qsort(base: *mut c_void, n: usize, size: usize,
             cmp: extern "C" fn(*const c_void, *const c_void) -> c_int);
}

extern "C" fn cmp_i32(a: *const c_void, b: *const c_void) -> c_int {
    let (a, b) = unsafe { (*(a as *const i32), *(b as *const i32)) };
    a.cmp(&b) as c_int
}
```

`Ordering` is `#[repr(i8)]` with -1/0/1, so `as c_int` works. Call with `v.as_mut_ptr().cast()`, `v.len()`, `size_of::<i32>()`.

</toggle>

## Building C code with your crate

A `build.rs` runs before your crate compiles. The [`cc`](https://crates.io/crates/cc) crate compiles C files and links them:

```rust
// build.rs
fn main() {
    cc::Build::new().file("c/vec3.c").compile("vec3");
    println!("cargo::rerun-if-changed=c/vec3.c");
}
```

For a big header, generate the `extern` block with [`bindgen`](https://rust-lang.github.io/rust-bindgen/) instead of writing it. Convention: a `foo-sys` crate holds the raw bindings, and a `foo` crate wraps them in a safe API.

### Building up to 16.2

**The goal in plain words:** you'll write a small library in C (a stack of integers), have Cargo compile it along with your Rust code, and then wrap it in a Rust type that's impossible to misuse. Users of your `Stack` never write `unsafe`, never free memory by hand, and get an `Option` back instead of C's "out-pointer plus bool".

**Step 1: the C side.** Keep it simple, with a fixed-size array. The `struct` is defined only in the `.c` file, so to Rust it's just "a pointer to something":

```c
// c/stack.c
#include <stdbool.h>
#include <stdlib.h>

typedef struct { int items[256]; int len; } Stack;

Stack *stack_new(void)               { return calloc(1, sizeof(Stack)); }
void   stack_push(Stack *s, int v)   { if (s->len < 256) s->items[s->len++] = v; }
bool   stack_pop(Stack *s, int *out) {
    if (s->len == 0) return false;
    *out = s->items[--s->len];
    return true;
}
void   stack_free(Stack *s)          { free(s); }
```

`stack_pop` can't return "nothing", so it returns `false` and puts the value into `*out` when there is one. That's C's version of `Option`.

**Step 2: compile it with `build.rs`.** `cargo add --build cc`, then the three-line `build.rs` above with the file renamed. Cargo runs it before compiling your crate and links the result in.

**Step 3: declare it to Rust.** Rust can't see the struct's fields, and doesn't need to. The "opaque type" row of the type map is the pattern: `#[repr(C)] struct RawStack { _private: [u8; 0] }`, then write each function in an `unsafe extern "C"` block using `*mut RawStack`.

**Step 4: wrap it so `unsafe` stays inside.** One rule per C promise:

| C needs | The Rust wrapper |
| --- | --- |
| `stack_new` may return NULL | check it once in `new` and store a `NonNull` |
| every stack must be freed exactly once | `impl Drop` calls `stack_free`, and nothing else can |
| `pop` uses an out-pointer + bool | pass `&mut out`, turn `true`/`false` into `Some(out)`/`None` |
| no use after free | `Stack` owns the pointer, so Rust's ownership rules apply |

**Step 5: check it.** Write a `main` that pushes 1, 2, 3 and pops until `None`. Users of `Stack` should need no `unsafe` at all. Then try copying a `Stack` (`let b = a;`, then use `a`), and see that ownership prevents a double free.

<callout background="blue" icon="🏋️">

**Exercise 16.2 — Build and call a C library**

Write a tiny C library `stack.c` with `stack_new`, `stack_push(int)`, `stack_pop(int *out) -> bool` and `stack_free`. Compile it with `cc` in `build.rs`, declare the bindings by hand, then write a **safe** `Stack` type in Rust whose `Drop` calls `stack_free` and whose `pop` returns `Option<i32>`. No `unsafe` should be visible to users of `Stack`.

</callout>

<toggle>

Shape of the safe wrapper

```rust
#[repr(C)]
struct RawStack { _private: [u8; 0] }

unsafe extern "C" {
    fn stack_new() -> *mut RawStack;
    fn stack_push(s: *mut RawStack, v: i32);
    fn stack_pop(s: *mut RawStack, out: *mut i32) -> bool;
    fn stack_free(s: *mut RawStack);
}

pub struct Stack(std::ptr::NonNull<RawStack>);

impl Stack {
    pub fn new() -> Self {
        let p = unsafe { stack_new() };
        Stack(std::ptr::NonNull::new(p).expect("stack_new returned NULL"))
    }
    pub fn push(&mut self, v: i32) { unsafe { stack_push(self.0.as_ptr(), v) } }
    pub fn pop(&mut self) -> Option<i32> {
        let mut out = 0;
        unsafe { stack_pop(self.0.as_ptr(), &mut out) }.then_some(out)
    }
}

impl Drop for Stack {
    fn drop(&mut self) { unsafe { stack_free(self.0.as_ptr()) } }
}
```

`NonNull` makes `Stack` neither `Send` nor `Sync` automatically, which is right until you know the C code is thread-safe.

</toggle>

## Calling Rust from C

```rust
// Cargo.toml: [lib] crate-type = ["cdylib", "staticlib"]

#[unsafe(no_mangle)]
pub extern "C" fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

- `#[unsafe(no_mangle)]` keeps the symbol name `add` (edition 2024 marks it unsafe because a clashing symbol is UB).
- `extern "C"` uses the C calling convention.
- [`cbindgen`](https://github.com/mozilla/cbindgen) generates the `.h` header from your Rust code.

### Ownership across the boundary

**Whoever allocates, frees.** Rust's allocator and C's `malloc` aren't interchangeable. The pattern: hand C an opaque pointer from `Box::into_raw`, and give C a `_free` function that rebuilds the `Box`.

```rust
pub struct Counter { n: u64 }

#[unsafe(no_mangle)]
pub extern "C" fn counter_new() -> *mut Counter {
    Box::into_raw(Box::new(Counter { n: 0 }))
}

/// # Safety
/// `c` must come from `counter_new` and not be freed yet.
#[unsafe(no_mangle)]
pub unsafe extern "C" fn counter_incr(c: *mut Counter) -> u64 {
    let c = unsafe { &mut *c };
    c.n += 1;
    c.n
}

/// # Safety
/// `c` must come from `counter_new` (or be NULL), and not be used afterwards.
#[unsafe(no_mangle)]
pub unsafe extern "C" fn counter_free(c: *mut Counter) {
    if !c.is_null() {
        drop(unsafe { Box::from_raw(c) });
    }
}
```

### Panics at the boundary

A panic must not unwind into C. Since Rust 1.81, a panic escaping an `extern "C"` function **aborts** the process. If you need errors to cross, catch them with `std::panic::catch_unwind` and return an error code. Use `extern "C-unwind"` only when both sides are built to unwind through each other.

### Building up to 16.3

**The goal in plain words:** the other direction. Compile Rust into a library file that a C program can link and call, just as it would any C library. You'll write the C program yourself. Then you'll see what happens when the Rust side panics, and stop that panic from killing the C program.

**Step 1: make Cargo produce a C-style library.** A normal Rust library can only be used by Rust. `crate-type = ["cdylib"]` makes a shared library (`libcounter.so` on Linux) that anything can load:

```toml
[lib]
crate-type = ["cdylib"]
```

Paste the `Counter` code above into `src/lib.rs` and `cargo build --release`. Check that `target/release/libcounter.so` exists.

**Step 2: a header file for C.** C needs declarations of the functions before it can call them. `cargo install cbindgen`, then:

```bash
cbindgen --lang c --output counter.h
```

Open `counter.h` and compare it with your Rust signatures.

**Step 3: the C program.**

```c
// main.c
#include <stdio.h>
#include "counter.h"

int main(void) {
    Counter *c = counter_new();
    for (int i = 0; i < 3; i++) counter_incr(c);
    printf("%llu\n", (unsigned long long)counter_incr(c));   // 4
    counter_free(c);
    return 0;
}
```

**Step 4: compile, link, run.** `-L` says where to find the library, and `-lcounter` links `libcounter.so`. At run time the system also has to find it, which is what `LD_LIBRARY_PATH` is for:

```bash
cc main.c -L target/release -lcounter -o main
LD_LIBRARY_PATH=target/release ./main
```

**Step 5: panic across the boundary.** Add `if c.n == 3 { panic!("too many"); }` to `counter_incr`, rebuild, and run `./main`. The process **aborts**, because a panic must not travel into C code.

**Step 6: catch it.** `std::panic::catch_unwind(|| …)` runs a closure and returns `Err` if it panicked, instead of letting the panic escape:

```rust
let result = std::panic::catch_unwind(|| {
    // the body that might panic, returning the new count
});
result.unwrap_or(0)          // 0 tells C "something went wrong"
```

The closure uses a `&mut` from a raw pointer, so the compiler may ask for `std::panic::AssertUnwindSafe(|| …)`. That's you promising it's fine to continue after a panic here.

<callout background="blue" icon="🏋️">

**Exercise 16.3 — Expose Rust to C**

Build the `Counter` library as a `cdylib`, generate `counter.h` with cbindgen, and write `main.c` that creates a counter, increments it 3 times, prints the result and frees it. Compile with `cc main.c -L target/release -lcounter` and run it (set `LD_LIBRARY_PATH`). Then make `counter_incr` panic when `n` reaches 3 and observe the abort; fix it with `catch_unwind` returning `0` on error.

</callout>

<toggle>

Beyond C

The same skills power: **PyO3** + maturin (Python extensions), **napi-rs** (Node.js), **wasm-bindgen** (browser), **cxx** (safe C++ interop), **uniffi** (Swift/Kotlin for mobile). They generate the `extern "C"` layer for you.

</toggle>

## Checklist

- [ ] I know why `extern` blocks are `unsafe extern` in edition 2024
- [ ] I can convert between `String`, `CString`, `&CStr` and `*const c_char`
- [ ] I use `#[repr(C)]` for every struct that crosses the boundary
- [ ] I follow "whoever allocates, frees", with `into_raw` / `from_raw`
- [ ] I never let a panic unwind into C

---

← [15 Unsafe Rust](page:fnp814n35n) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [17 Memory Layout and Performance](page:ap65audlmt)
