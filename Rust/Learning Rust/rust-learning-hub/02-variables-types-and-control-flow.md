---
id: 9ll61je1wh
title: "02 Variables, Types and Control Flow"
sort: 19000
version: 1
---

<callout background="green" icon="🔤">

**The building blocks every program uses.** Immutable-by-default variables, a small set of precise number types, functions where almost everything is an expression, and loops that can return values.

</callout>

**Level:** 🟢 Basics · **Needs:** [01 What Is Rust](page:yfq54za15s)

## Variables

```rust
let x = 5;              // immutable by default
// x = 6;               // ❌ cannot assign twice to immutable variable
let mut y = 5;          // opt in to mutation
y += 1;

let x = x * 2;          // shadowing: a NEW variable with the same name
let spaces = "   ";
let spaces = spaces.len();   // shadowing can even change the type

const MAX_POINTS: u32 = 100_000;        // compile-time constant, type required
static GREETING: &str = "hello";        // one fixed address for the whole program
```

<callout icon="💡">

**Why immutable by default?** Most variables never change. Making mutation explicit (`mut`) means when you see `mut`, it's a signal worth paying attention to, and the compiler can reason more about your code.

</callout>

## Types

Rust is **statically typed** but infers most types. Annotate when it can't guess: `let n: u64 = "42".parse().unwrap();`

### Scalars

| Kind | Types | Notes |
| --- | --- | --- |
| Signed ints | `i8` , `i16`, `i32` ,`i64` ,`i128` ,`isize` | `i32` is the default |
| Unsigned ints | `u8`, `u16` ,`u32` ,`u64` ,`u128`, `usize` | `usize` is for indexes and lengths |
| Floats | `f32` ,`f64` | `f64` is the default |
| Boolean | `bool` | `true` / `false` |
| Character | `char` | 4 bytes, any Unicode scalar: `'a'`, `'é'`, `'🦀'` |

Literals: `1_000_000`, `0xff`, `0o77`, `0b1010`, `b'A'` (a `u8`), `2.5e3`, `10u8`.

### Compound

```rust
let tup: (i32, f64, char) = (500, 6.4, 'z');
let (a, b, c) = tup;                 // destructure
let first = tup.0;

let arr = [1, 2, 3, 4, 5];           // fixed length, type [i32; 5]
let zeros = [0u8; 256];              // 256 zeros
let third = arr[2];                  // out of bounds → panic, never silent garbage
```

### Integer overflow

In **debug** builds, overflow **panics**. In **release** builds it wraps silently. When overflow is possible, say what you want:

```rust
let a: u8 = 250;
a.checked_add(10);      // None
a.wrapping_add(10);     // 4
a.saturating_add(10);   // 255
a.overflowing_add(10);  // (4, true)
```

Conversions are explicit: `x as u64` (can truncate silently), or `u8::try_from(x)` (returns a `Result`).

## Functions

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b            // no semicolon: this expression is the return value
}

fn print_sum(a: i32, b: i32) {      // returns () ("unit") when no type is given
    println!("{}", add(a, b));
}
```

**Expressions vs statements:** an expression produces a value; adding `;` turns it into a statement that produces `()`. Blocks are expressions too:

```rust
let y = {
    let x = 3;
    x + 1            // the block's value: 4
};
```

## Control flow

```rust
// if is an expression
let kind = if n % 2 == 0 { "even" } else { "odd" };

// loop forever, break with a value
let mut i = 0;
let found = loop {
    i += 1;
    if i * i > 50 { break i; }
};

// while
while i > 0 { i -= 1; }

// for over a range or a collection (the usual choice)
for n in 1..=5 { print!("{n} "); }          // 1 2 3 4 5
for (idx, ch) in "rust".chars().enumerate() { println!("{idx}: {ch}"); }
for n in (1..4).rev() { print!("{n} "); }   // 3 2 1

// labels to break out of nested loops
'outer: for x in 0..10 {
    for y in 0..10 {
        if x * y == 42 { println!("{x} * {y}"); break 'outer; }
    }
}
```

`match` (covered fully in module 04) is the multi-way branch:

```rust
let msg = match n {
    0 => "zero",
    1..=9 => "small",
    _ => "big",
};
```

### Building up to 2.1

**The goal in plain words:** print a conversion table, Celsius on the left and Fahrenheit on the right, with the numbers lined up in neat columns:

```text
   -40 °C =  -40.0 °F
   -30 °C =  -22.0 °F
   ...
   100 °C =  212.0 °F
```

**Step 1: a function that does maths.** The formulas are `F = C × 9 / 5 + 32` and `C = (F − 32) × 5 / 9`. A function takes `f64` and returns `f64`, and its last line (no `;`) is the answer:

```rust
fn double(x: f64) -> f64 {
    x * 2.0          // write 2.0, not 2: Rust won't mix f64 and integers
}
```

Write `c_to_f` and `f_to_c` the same way, with `9.0`, `5.0` and `32.0`.

**Step 2: count in steps of 10.** `-40..=100` is every integer from -40 to 100, and `.step_by(10)` keeps every 10th one:

```rust
for c in (-40..=100).step_by(10) {
    println!("{c}");   // -40, -30, …, 100
}
```

**Step 3: turn the integer into a float.** `c` is an `i32` here, but `c_to_f` wants an `f64`, and Rust never converts silently. Write `c as f64`.

**Step 4: line the columns up.** Inside `{}` you can add a format spec after a `:`:

```rust
println!("[{:>6}]", 42);       // [    42]   right-aligned in 6 characters
println!("[{:>8.1}]", 3.14159); // [     3.1]  8 wide, 1 decimal place
println!("[{:<6}]", 42);       // [42    ]   left-aligned
```

Put the steps together, one `println!` per row. Then check that `c_to_f(-40.0)` gives `-40.0`.

<callout background="blue" icon="🏋️">

**Exercise 2.1 — Temperature converter**

Write `c_to_f` and `f_to_c` (`f64 → f64`). In `main`, print a table from -40 °C to 100 °C in steps of 10, with both columns aligned using format specs (`{:>8.1}`). Check that -40 is the same in both scales.

</callout>

<toggle>

Hint

`F = C × 9/5 + 32`. Loop with `for c in (-40..=100).step_by(10)` and convert with `c as f64`. Format: `println!("{:>6} °C = {:>6.1} °F", c, c_to_f(c as f64));`

</toggle>

### Building up to 2.2

**The goal in plain words:** count from 1 to 100, saying "Fizz" instead of numbers divisible by 3, "Buzz" instead of numbers divisible by 5, and "FizzBuzz" for numbers divisible by both (like 15). The rule for this version: no `if`, only one `match`.

**Step 1: "divisible by" is a remainder of 0.** `%` gives the remainder:

```rust
9 % 3    // 0  → 9 is divisible by 3
10 % 3   // 1  → it isn't
```

**Step 2: a tuple holds two values together.** `(n % 3, n % 5)` for `n = 15` is `(0, 0)`, for `n = 9` it's `(0, 4)`, and for `n = 7` it's `(1, 2)`.

**Step 3: `match` can check both parts of a tuple at once.** `_` means "anything here":

```rust
let point = (0, 7);
match point {
    (0, 0) => println!("at the origin"),
    (0, _) => println!("on the y axis"),     // first is 0, second is anything
    (_, 0) => println!("on the x axis"),
    _      => println!("somewhere else"),   // everything else
}
```

The arms are tried **top to bottom**, and the first match wins. That's why the "both" case has to come first. FizzBuzz is this exact shape, with `(n % 3, n % 5)` in place of `point`.

<callout background="blue" icon="🏋️">

**Exercise 2.2 — FizzBuzz with match**

Print 1 to 100, but "Fizz" for multiples of 3, "Buzz" for 5, "FizzBuzz" for both. Do it with a single `match` on the tuple `(n % 3, n % 5)`, no `if`.

</callout>

<toggle collapsed=true>

Solution

```rust
for n in 1..=100 {
    match (n % 3, n % 5) {
        (0, 0) => println!("FizzBuzz"),
        (0, _) => println!("Fizz"),
        (_, 0) => println!("Buzz"),
        _ => println!("{n}"),
    }
}
```

</toggle>

### Building up to 2.3

**The goal in plain words:** Fibonacci numbers grow fast (0, 1, 1, 2, 3, 5, 8, 13 …, each the sum of the two before it). Every integer type has a maximum, so at some point the next number no longer fits. Find that point for `u32`, `u64` and `u128`, and see what Rust does when you go past it.

**Step 1: walk the sequence with two variables.** Keep the last two numbers and move them along:

```rust
let (mut a, mut b): (u32, u32) = (0, 1);   // F(0), F(1)
for _ in 0..10 {
    let next = a + b;
    a = b;
    b = next;
}
println!("{a}");   // F(10) = 55
```

**Step 2: ask before adding.** `checked_add` gives `Some(sum)` if the result fits and `None` if it would overflow:

```rust
let big: u32 = 4_000_000_000;
big.checked_add(1);              // Some(4000000001)
big.checked_add(500_000_000);    // None: past u32::MAX (4,294,967,295)
```

**Step 3: stop at the first `None`.** Use a `loop` that `break`s when the add fails. `Some(next)` in a `match` arm takes the value out:

```rust
match big.checked_add(1) {
    Some(next) => println!("fits: {next}"),
    None => println!("would overflow"),
}
```

Keep a counter `i` next to `a` and `b`, so that when the loop stops you know which F(i) was the last one that fit.

**Step 4: repeat for each type.** Change `u32` to `u64`, then to `u128`. Finally, swap `checked_add` for a plain `a + b` in a loop that never stops, and run it twice: `cargo run`, then `cargo run --release`. Compare what happens.

<callout background="blue" icon="🏋️">

**Exercise 2.3 — Find where Fibonacci overflows**

Compute Fibonacci numbers in a `loop` using `u32`, then `u64`, then `u128`, with `checked_add`. For each type, print the index and value of the last Fibonacci number that fits. What happens if you use plain `+` in a debug build? In a release build (`cargo run --release`)?

</callout>

<toggle>

Answers to check against

- `u32`: F(47) = 2,971,215,073
- `u64`: F(93) = 12,200,160,415,121,876,738
- `u128`: F(186)
- Plain `+` in debug: `panicked at 'attempt to add with overflow'`. In release: it wraps and you silently get nonsense. That's why `checked_*` exists.

</toggle>

## Checklist

- [x] I know the difference between `let`, `let mut`, shadowing, `const` and `static`
- [x] I can pick integer types and handle overflow on purpose
- [x] I understand that blocks, `if`, `match` and `loop` are expressions
- [x] I use `for` over ranges and iterators instead of index loops

---

← [01 What Is Rust](page:yfq54za15s) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [03 Ownership and Borrowing](page:acpvbrr41r)
