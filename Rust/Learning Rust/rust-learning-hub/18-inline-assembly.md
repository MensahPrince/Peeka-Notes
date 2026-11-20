---
id: nes55qk0m4
title: "18 Inline Assembly"
sort: 3000
version: 1
---

<callout background="red" icon="⚙️">

**The bottom of the stack.** First learn to *read* the assembly rustc produces (that's 95% of the value), then write your own with `asm!`, `global_asm!` and naked functions for the few things the compiler can't express.

</callout>

**Level:** 🔴 Expert · **Needs:** [15 Unsafe Rust](page:fnp814n35n), [17 Memory Layout and Performance](page:ap65audlmt) · Examples are **x86-64 Linux** (Intel syntax, which is Rust's default) unless noted.

## Part 1 · Reading the compiler's output

Ways to see the assembly for your code:

| Tool | How |
| --- | --- |
| Compiler Explorer | [godbolt.org](https://godbolt.org), language Rust, flags `-C opt-level=3` |
| cargo-show-asm | `cargo install cargo-show-asm`, then `cargo asm --lib my_crate::my_fn` |
| rustc directly | `cargo rustc --release -- --emit asm` then look in `target/release/deps/*.s` |
| Disassemble the binary | `objdump -d -M intel target/release/app` |

Functions must be `pub` (or `#[inline(never)]` and used) or they get inlined away and you see nothing.

```rust
pub fn add(a: u32, b: u32) -> u32 {
    a + b
}
```

```asm
add:
    lea eax, [rdi + rsi]   ; eax = a + b  (lea is a cheap adder)
    ret                    ; return value lives in eax/rax
```

### x86-64 crash course

<columns>

<column>

**Registers** (64-bit / 32-bit low half)

- `rax`/`eax`: return value
- `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`: first six integer args (System V ABI: Linux, macOS)
- `rsp`: stack pointer, `rbp`: frame pointer
- `rbx`, `rbp`, `r12`–`r15`: callee-saved
- `xmm0`–`xmm15`: floats and SIMD

</column>

<column>

**Instructions you'll see constantly**

- `mov dst, src`: copy
- `lea dst, [addr]`: compute an address (often used as maths)
- `add`, `sub`, `imul`, `shl`, `shr`, `and`, `or`, `xor`
- `cmp a, b` + `jl`/`jg`/`je`/`jne`: branch
- `cmovcc`: branch-free conditional move
- `call`, `ret`, `push`, `pop`
- `ud2`: deliberate trap (Rust uses it for `unreachable`)

</column>

</columns>

### Building up to 18.1

**The goal in plain words:** the compiler turns your Rust into machine instructions. You'll look at those instructions for three tiny functions, and answer questions you can't answer from the Rust alone: is the iterator version slower than the loop? Where does the bounds check actually happen? Did my `if` become a jump?

**Step 1: set up Compiler Explorer.** Go to [godbolt.org](https://godbolt.org) and choose **Rust** as the language. In the compiler options box, type `-C opt-level=3`. Paste one function at a time. It must be `pub`, or it's optimised away and the right-hand pane stays empty.

**Step 2: read the pane.** Hovering over a line of Rust highlights the instructions it produced (same colour). Right-click an instruction and choose the documentation option to see what it does. For a short function, read top to bottom:

```asm
example::add:          ; the function's name
    lea eax, [rdi + rsi]   ; arguments arrive in rdi, rsi (first, second)
    ret                    ; the answer is in eax/rax
```

**Step 3: what to look for in each task.**

1. **Loop vs iterator.** Put both functions in the same pane and compare the instruction lists. Instructions on `xmm`/`ymm` registers with names like `paddd` mean **SIMD**: adding 4 or 8 numbers in one instruction.
1. **Bounds check.** Look for a `cmp` between `i` (in `rsi`) and the length (in `rdx`, because a slice is passed as pointer + length), followed by a jump to a block that calls something containing `panic_bounds_check`.
1. **Branch or not.** A *branch* is a conditional jump (`jg`, `jl`, `jle` …) that skips over code. A *branch-free* version computes both possibilities and picks one with `cmov` ("conditional move"). Which do you see?

**Step 4: tell the compiler about your CPU.** Add `-C target-cpu=native` to the options, which says "use every instruction this CPU has". Compare the sum functions again: do the register names change from `xmm` to `ymm`? (On godbolt, "native" is the server's CPU, which usually has AVX2.)

<callout background="blue" icon="🏋️">

**Exercise 18.1 — Read the compiler's assembly**

Put these in Compiler Explorer with `-C opt-level=3` and explain each output in your own words:

1. `pub fn sum_loop(v: &[u32]) -> u32` with a manual `for i in 0..v.len()` loop, and `pub fn sum_iter(v: &[u32]) -> u32 { v.iter().sum() }`. Are they different? Spot the SIMD (`paddd`, `xmm`/`ymm`).
1. `pub fn get(v: &[u32], i: usize) -> u32 { v[i] }`. Find the bounds check and the call to the panic function.
1. `pub fn abs_diff(a: i32, b: i32) -> i32 { if a > b { a - b } else { b - a } }`. Is there a branch?

Then add `-C target-cpu=native` and compare 1 again.

</callout>

<toggle>

What you should find

1. Usually **identical** or nearly so: the loop is auto-vectorised either way, since the indexing bounds check is proven redundant by `i < v.len()`. Wrapping arithmetic means the sum can be reordered, so vector adds are legal. With `target-cpu=native` on an AVX2 machine, `xmm` (128-bit) becomes `ymm` (256-bit).
1. A `cmp` of `i` against the length (in `rsi`), a jump to a cold block that calls `core::panicking::panic_bounds_check`. Using `v.get(i)` or proving `i < len` earlier removes it.
1. Usually no branch: a `sub` and a `cmov`, or `neg` + `cmov`.

</toggle>

## Part 2 · The `asm!` macro

```rust
use std::arch::asm;

let i: u64 = 3;
let o: u64;
unsafe {
    asm!(
        "mov {0}, {1}",
        "add {0}, 5",
        out(reg) o,
        in(reg) i,
    );
}
assert_eq!(o, 8);
```

The template is a string; `{0}`, `{1}` or `{name}` are replaced by **registers the compiler picks** for each operand.

### Operands

| Operand | Meaning |
| --- | --- |
| `in(reg) x` | put `x` in some register before the asm |
| `out(reg) y` | asm writes it; read into `y` after. Never shares a register with an input |
| `lateout(reg) y` | like `out`, but may reuse an input's register (only safe if all inputs are read before it's written) |
| `inout(reg) x` | read and written; `x` is updated |
| `inout(reg) a => b` | starts as `a`, result goes to `b` |
| `in("rax") x` | a **specific** register |
| `out("rcx") _` | the asm clobbers `rcx`; discard it |
| `sym my_fn` | the symbol name of a function or static |
| `const 42` | an immediate value |
| `clobber_abi("C")` | everything a C call clobbers (use when you `call` a function) |

### Options (tell the compiler what you *don't* do)

| Option | Promise |
| --- | --- |
| `pure` | no side effects; output depends only on inputs (lets it dedupe or delete the asm) |
| `nomem` | doesn't read or write memory |
| `readonly` | reads memory but doesn't write |
| `nostack` | doesn't push onto the stack (no red-zone worries) |
| `preserves_flags` | doesn't change the flags register |
| `noreturn` | never returns |
| `att_syntax` | AT&T syntax instead of Intel |

<callout background="yellow" icon="⚠️">

Every option is a **promise the compiler trusts blindly**. Saying `nomem` when you touch memory, or forgetting to list a clobbered register, is undefined behaviour that shows up as "impossible" bugs elsewhere in the function. Also: `rbx` (and `rbp`, `rsp`) can't be used as operands on x86-64 because LLVM reserves them; save and restore them yourself if an instruction like `cpuid` writes `rbx`.

</callout>

### Example: reading the timestamp counter

```rust
use std::arch::asm;

fn rdtsc() -> u64 {
    let lo: u32;
    let hi: u32;
    unsafe {
        asm!("rdtsc", out("eax") lo, out("edx") hi, options(nomem, nostack));
    }
    ((hi as u64) << 32) | lo as u64
}
```

### Building up to 18.2

**The goal in plain words:** `Instant::now()` measures nanoseconds. The CPU also counts its own clock ticks ("cycles"), and the `rdtsc` instruction reads that counter directly, which is finer and cheaper than asking the operating system. You'll build a cycle timer from it, measure three snippets, and learn why one measurement is never enough.

**Step 1: read the `rdtsc` example operand by operand.** `rdtsc` puts a 64-bit count into two 32-bit registers, the low half in `eax` and the high half in `edx`. `out("eax") lo` says "after the asm, copy `eax` into `lo`". The last line glues the halves together:

```text
hi = 0x0000_0002, lo = 0x8000_0000  →  (hi << 32) | lo = 0x0000_0002_8000_0000
```

`options(nomem, nostack)` promises that the instruction doesn't touch memory or the stack, which is true for `rdtsc`.

**Step 2: a timer that takes a closure.** Read the counter, run the work, and read it again:

```rust
fn cycles<F: FnOnce()>(f: F) -> u64 {
    let start = rdtsc();
    f();
    rdtsc() - start
}

let c = cycles(|| { std::hint::black_box(Vec::<u8>::with_capacity(1000)); });
```

`black_box` stops the optimiser from noticing the `Vec` is unused and deleting it, which would leave you timing nothing.

**Step 3: repeat and take the minimum.** One run includes random noise (an interrupt, another program, a cold cache). Run each snippet 100 times:

```rust
let best = (0..100).map(|_| cycles(|| { /* work */ })).min().unwrap();
```

The "empty closure" result is the timer's own overhead. Subtract it from the others.

**Step 4: the intrinsic.** `core::arch::x86_64::_rdtsc()` is the same instruction as a normal (unsafe) function call, with no `asm!`. Swap it in, check that you get the same kind of numbers, and compare the two versions in godbolt.

**Step 5: why the minimum?** Noise only ever *adds* time: nothing can make the code run faster than it really does. Use that to explain why the minimum is closest to the true cost, and the mean isn't.

<callout background="blue" icon="🏋️">

**Exercise 18.2 — rdtsc cycle timer**

Write `fn cycles<F: FnOnce()>(f: F) -> u64` using `rdtsc` before and after. Time an empty closure, a `Vec::with_capacity(1000)`, and summing 1 million `u32`s. Run each 100 times and print the minimum. Then compare with `core::arch::x86_64::_rdtsc()` (the intrinsic: no `asm!` needed) and think about why the minimum, not the mean, is the number to report.

</callout>

<toggle>

Hints

- Wrap the measured work in `std::hint::black_box(...)` so the optimiser can't delete it.
- `rdtsc` isn't serialising: the CPU may reorder it around your code. For more honest numbers use `lfence; rdtsc` or `rdtscp`.
- The minimum is the run with the least noise (interrupts, cache misses from other processes); everything above it is interference.

</toggle>

### Example: a Linux system call with no libc

```rust
use std::arch::asm;

fn sys_write(fd: usize, buf: &[u8]) -> isize {
    let ret: isize;
    unsafe {
        asm!(
            "syscall",
            inlateout("rax") 1isize => ret, // syscall number 1 = write
            in("rdi") fd,
            in("rsi") buf.as_ptr(),
            in("rdx") buf.len(),
            lateout("rcx") _, // the syscall instruction clobbers rcx and r11
            lateout("r11") _,
            options(nostack, readonly),
        );
    }
    ret
}
```

### Building up to 18.3

**The goal in plain words:** `println!` eventually asks the operating system to write bytes to the screen. That request is a **system call**: a special instruction that hands control to the kernel. You'll skip `println!` and libc entirely and make the system call yourself. Then you'll build a program that has *no* libc at all.

**Step 1: what a system call is.** On Linux x86-64, the rules for a system call are fixed:

| Register | Holds |
| --- | --- |
| `rax` | **which** system call (1 = `write`, 60 = `exit`) |
| `rdi`, `rsi`, `rdx` | its first three arguments |
| `rax` (afterwards) | the result: bytes written, or a negative error code |
| `rcx`, `r11` | overwritten by the `syscall` instruction itself |

`write(fd, buf, len)` therefore needs `rax = 1`, `rdi = 1` (stdout), `rsi` = pointer to the bytes, `rdx` = how many. That's exactly what the `sys_write` operands above say. Read them against this table.

**Step 2: call it.**

```rust
let msg = b"hello from the kernel boundary\n";   // b"…" is a &[u8; N]
let n = sys_write(1, msg);
assert_eq!(n, msg.len() as isize);              // all bytes written
```

Try `fd = 99` (not an open file) and print what comes back. Negative numbers are error codes: `-9` is `EBADF`, "bad file descriptor".

**Step 3: find other syscall numbers.** `grep -E '__NR_(write|exit) ' /usr/include/asm/unistd_64.h` lists them. For `exit`, the code goes in `rdi`. It never returns, so the function returns `!` and the asm gets `options(noreturn)`.

**Step 4: a program without libc.** Normally libc sets up your program and then calls `main`. Without it, you provide the entry point the kernel jumps to, `_start`, yourself. Do this part after reading module 19, which explains each line:

```rust
#![no_std]
#![no_main]

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! { sys_exit(1) }

#[unsafe(no_mangle)]
pub extern "C" fn _start() -> ! {
    sys_write(1, b"no libc here\n");
    sys_exit(0)
}
```

Use `core::arch::asm` instead of `std::arch::asm`. Tell the linker not to add libc's startup code with `rustflags = ["-C", "link-arg=-nostartfiles"]` in `.cargo/config.toml`. In `Cargo.toml`, set `panic = "abort"` for both `[profile.dev]` and `[profile.release]`. Check the result with `ls -l` (tiny) and `ldd` ("not a dynamic executable" or "statically linked").

<callout background="blue" icon="🏋️">

**Exercise 18.3 — Hello world via raw syscall**

Use `sys_write` to print "hello from the kernel boundary" to stdout (fd 1) and check the return value. Then add `sys_exit(code)` (syscall 60, `options(noreturn)`) and combine it with [19 no\_std and Bare Metal](page:qa2zkcsrl1) to make a `no_std` Linux binary with no libc at all. Look up syscall numbers in `/usr/include/asm/unistd_64.h`.

</callout>

<toggle>

Solution sketch for sys\_exit

```rust
fn sys_exit(code: i32) -> ! {
    unsafe {
        asm!("syscall", in("rax") 60, in("rdi") code, options(noreturn, nostack));
    }
}
```

For the libc-free binary: `#![no_std]`, `#![no_main]`, a `#[panic_handler]`, `#[unsafe(no_mangle)] pub extern "C" fn _start() -> !` that calls `sys_write` then `sys_exit(0)`, and link with `-C link-arg=-nostartfiles` (in `.cargo/config.toml` under `rustflags`).

</toggle>

### Example: popcount, three ways

```rust
use std::arch::asm;

// 1. Plain Rust: the compiler picks the best instruction for the target.
fn pop_rust(x: u64) -> u32 {
    x.count_ones()
}

// 2. Inline asm.
fn pop_asm(x: u64) -> u64 {
    let r: u64;
    unsafe {
        asm!("popcnt {r}, {x}", x = in(reg) x, r = lateout(reg) r,
             options(pure, nomem, nostack));
    }
    r
}

// 3. Intrinsic, gated on a CPU feature.
#[target_feature(enable = "popcnt")]
fn pop_intrinsic(x: u64) -> i32 {
    unsafe { core::arch::x86_64::_popcnt64(x as i64) }
}
```

### Building up to 18.4

**The goal in plain words:** "popcount" counts the 1 bits in a number (`0b1011` → 3). Modern CPUs do it in one instruction, `popcnt`. The question is how to get that instruction: let the compiler choose, write it in `asm!`, or call an intrinsic? You'll measure all three, and see why the compiler often wins once you tell it which CPU it's building for.

**Step 1: 10 million random numbers.** No crate needed, a quick generator is enough:

```rust
let mut x: u64 = 0x9E37_79B9_7F4A_7C15;
let data: Vec<u64> = (0..10_000_000)
    .map(|_| { x ^= x << 13; x ^= x >> 7; x ^= x << 17; x })   // xorshift
    .collect();
```

**Step 2: time each version the same way.** Sum the results so the work can't be optimised away:

```rust
let start = std::time::Instant::now();
let total: u64 = data.iter().map(|&v| pop_rust(v) as u64).sum();
println!("rust: {total} in {:?}", std::hint::black_box(start.elapsed()));
```

Do the same with `pop_asm` and `pop_intrinsic`. Every total must match.

**Step 3: the CPU check.** `#[target_feature(enable = "popcnt")]` lets that one function use `popcnt` even if the rest of the program can't. Calling it on a CPU *without* `popcnt` would crash, so it's `unsafe` to call, and you check first:

```rust
if is_x86_feature_detected!("popcnt") {
    let n = unsafe { pop_intrinsic(42) };
}
```

**Step 4: build twice.** `cargo run --release`, then `RUSTFLAGS="-C target-cpu=native" cargo run --release`. Write the three timings down for each build.

**Step 5: explain it with the assembly.** Look at `pop_rust` with `cargo asm` or godbolt, with and without `-C target-cpu=native`. One build shows a single `popcnt` and the other a longer sequence of shifts and masks. That difference explains your timings.

<callout background="blue" icon="🏋️">

**Exercise 18.4 — popcnt three ways**

Benchmark the three versions over 10 million random `u64`s, with and without `-C target-cpu=native`. Look at the assembly of `pop_rust` in both cases. Call `pop_intrinsic` only after checking `is_x86_feature_detected!("popcnt")`.

</callout>

<toggle>

What you should see

Without `target-cpu=native`, the baseline x86-64 target doesn't assume `popcnt` exists, so `count_ones` compiles to a bit-twiddling sequence (or a SIMD equivalent). With it, `count_ones` becomes a single `popcnt` and ties the asm version. Lesson: **reach for `asm!` last**. Intrinsics and target features usually get you the instruction while keeping the optimiser informed.

</toggle>

## Part 3 · Whole functions in assembly

**`global_asm!`** emits assembly at the top level of the object file, e.g. a bootloader entry or an interrupt trampoline:

```rust
use std::arch::global_asm;

global_asm!(
    ".global add_asm",
    "add_asm:",
    "    lea rax, [rdi + rsi]",
    "    ret",
);

unsafe extern "C" {
    fn add_asm(a: u64, b: u64) -> u64;
}
```

**Naked functions** (stable since Rust 1.88) have no compiler-generated prologue or epilogue; the body is a single `naked_asm!`:

```rust
#[unsafe(naked)]
pub extern "sysv64" fn add_naked(a: u64, b: u64) -> u64 {
    core::arch::naked_asm!(
        "lea rax, [rdi + rsi]",
        "ret",
    )
}
```

Use them for interrupt handlers, context switches and trampolines, where you must control every instruction.

## Part 4 · SIMD without writing asm

```rust
#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "avx2")]
fn add8(a: &[f32; 8], b: &[f32; 8]) -> [f32; 8] {
    use std::arch::x86_64::*;
    let mut out = [0.0f32; 8];
    unsafe {
        let va = _mm256_loadu_ps(a.as_ptr());
        let vb = _mm256_loadu_ps(b.as_ptr());
        _mm256_storeu_ps(out.as_mut_ptr(), _mm256_add_ps(va, vb));
    }
    out
}

// Calling a #[target_feature] fn from code without that feature is unsafe:
// you must check the CPU first.
if is_x86_feature_detected!("avx2") {
    let r = unsafe { add8(&a, &b) };
}
```

<toggle>

ARM64 (Apple Silicon, Raspberry Pi) equivalents

- Registers `x0`–`x30` (64-bit), `w0`–`w30` (32-bit). Args in `x0`–`x7`, return in `x0`.
- `add x0, x0, x1` / `ret`. Linux syscalls: number in `x8`, `svc #0`.
- SIMD is NEON in `std::arch::aarch64` (`vaddq_f32`, …), always available on aarch64.
- `asm!` works the same; only the template and register names change. Use `#[cfg(target_arch = "aarch64")]`.

</toggle>

## Checklist

- [ ] I can find a function's assembly with godbolt or `cargo asm`
- [ ] I can read a short x86-64 function and name the args and return register
- [ ] I can write `asm!` with `in`/`out`/`lateout`, a clobber and correct options
- [ ] I know why `rbx` is special and what `nomem` promises
- [ ] I prefer `count_ones` → intrinsic → `asm!`, in that order

---

← [17 Memory Layout and Performance](page:ap65audlmt) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [19 no\_std and Bare Metal](page:qa2zkcsrl1)
