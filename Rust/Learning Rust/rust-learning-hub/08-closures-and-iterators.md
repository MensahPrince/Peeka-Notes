---
id: ch1buo23yl
title: "08 Closures and Iterators"
sort: 13000
version: 1
---

<callout background="yellow" icon="🔁">

**Functional Rust with zero overhead.** Closures are anonymous functions that capture their environment; iterators are lazy chains of transformations that compile down to the same loop you'd write by hand.

</callout>

**Level:** 🟡 Intermediate · **Needs:** [07 Generics, Traits and Lifetimes](page:8s3iybt7d1)

## Closures

```rust
let factor = 3;
let times = |x: i32| x * factor;        // captures `factor` by reference
println!("{}", times(5));                // 15

let add = |a, b| a + b;                  // types inferred from first use
let mut count = 0;
let mut incr = || count += 1;            // captures by &mut, so the closure is `mut`
incr();
incr();
```

### How they capture, and the three traits

The compiler picks the least it needs: borrow, mutable borrow, or move.

| Trait | Can be called | Captures | Example |
| --- | --- | --- | --- |
| `Fn` | many times, even concurrently | by `&` | `\|x\| x * factor` |
| `FnMut` | many times, not concurrently | by `&mut` | `\|\| count += 1` |
| `FnOnce` | once | moves out of a capture | `\|\| drop(name)` |

Every `Fn` is also `FnMut`, and every `FnMut` is also `FnOnce`. Accept the **most general** bound you can: `FnOnce` if you call it once, `FnMut` if many times.

```rust
fn apply_twice<F: FnMut()>(mut f: F) { f(); f(); }
```

`move` forces captures by value, which you need when the closure outlives the current scope (threads, returned closures):

```rust
let name = String::from("Ada");
let greet = move || println!("hi {name}");   // greet now owns name
```

## Iterators

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // + 70 provided methods built on next()
}
```

Three ways to iterate a collection:

| Call | Yields | Collection after |
| --- | --- | --- |
| `v.iter()` / `for x in &v` | `&T` | untouched |
| `v.iter_mut()` / `for x in &mut v` | `&mut T` | modified in place |
| `v.into_iter()` / `for x in v` | `T` | consumed |

### Adapters (lazy) and consumers (do the work)

```rust
let v = vec![1, 2, 3, 4, 5, 6];

let evens_squared: Vec<i32> = v.iter()
    .filter(|&&x| x % 2 == 0)     // lazy
    .map(|&x| x * x)               // lazy
    .collect();                    // runs the chain: [4, 16, 36]

let total: i32 = v.iter().sum();
let any_big = v.iter().any(|&x| x > 5);
let pos = v.iter().position(|&x| x == 4);       // Some(3)
let (small, big): (Vec<i32>, Vec<i32>) = v.iter().partition(|&&x| x < 4);
```

<toggle>

The adapters you'll use weekly

- **Transform:** `map`, `filter`, `filter_map`, `flat_map`, `flatten`, `enumerate`, `zip`, `chain`, `rev`, `inspect`
- **Limit:** `take`, `skip`, `take_while`, `skip_while`, `step_by`, `peekable`
- **Group:** `windows(n)` and `chunks(n)` (on slices), `scan`
- **Consume:** `collect`, `sum`, `product`, `count`, `fold`, `reduce`, `min`/`max`, `min_by_key`, `max_by_key`, `find`, `any`, `all`, `last`, `nth`, `for_each`
- `collect` can build any `FromIterator`: `Vec`, `String`, `HashMap` (from pairs), and even `Result<Vec<T>, E>` (stops at the first error)
- The `itertools` crate adds `sorted`, `group_by`, `dedup`, `join`, `tuple_windows` …

</toggle>

<callout icon="💡">

**Zero-cost:** an iterator chain is typically compiled to the same machine code as a hand-written loop, often better, because the bounds checks disappear. You'll verify this yourself in [18 Inline Assembly](page:nes55qk0m4).

</callout>

### Building up to 8.1

**The goal in plain words:** five small tasks you could solve with a `for` loop and a `mut` variable. Instead, write each as one chain: a **source** of items, some **adapters** that change or filter them, and one **consumer** that produces the answer. It's a new way of thinking, so here's how to translate.

**Step 1: spot the three parts of a loop.** Almost every loop that builds an answer looks like this:

```rust
let mut total = 0;                 // ← consumer: what you're building (here: a sum)
for n in 1..10 {                   // ← source: where the items come from
    if n % 3 == 0 {                // ← adapter: filter
        total += n * 2;            // ← adapter: map (n → n * 2), then add up
    }
}
```

and becomes:

```rust
let total: i32 = (1..10).filter(|n| n % 3 == 0).map(|n| n * 2).sum();
```

| In the loop | In the chain |
| --- | --- |
| `for x in …` | the source: a range, `v.iter()`, `s.chars()`, `s.lines()` |
| `if cond { … }` | `.filter(\|x\| cond)` |
| compute something from `x` | `.map(\|x\| …)` |
| `total += …` | `.sum()` |
| `if x > best { best = x }` | `.max()` (returns an `Option`) |
| `out.push(…)` | `.collect()` |
| "try something, skip it if it fails" | `.filter_map(\|x\| …)`, where the closure returns an `Option` |
| needs the index too | `.enumerate()` gives `(index, item)` |

**Step 2: hints for each task.**

1. The source is `1..1000`. Then filter odd numbers, map to squares, and sum.
1. The source is `words.iter()`. Map each word to its `len()`, then `max()`. An empty list gives `None`, so finish with `.unwrap_or(0)`.
1. Each line might fail to parse, so use `filter_map`. Inside the closure, `?` works on `Option`: `let (name, age) = line.split_once(',')?;` then `age.trim().parse::<u32>().ok()?`, and return `Some((name.to_string(), age))`.
1. You need each number *and the one before it*. `nums.windows(2)` gives overlapping pairs `[a, b]`: `[1, 3, 2]` → `[1, 3]`, `[3, 2]`. Add `enumerate()`, keep the pairs where `w[1] > w[0]`, and map to the index of the second number (`i + 1`).
1. Split into words, change each word, then join them back with `" "`. To capitalise one word, take its first `char` with `chars().next()`, uppercase it, and add the rest of the chars.

**Step 3: the `&&x` in the examples above.** `v.iter()` gives `&i32`, and `filter` passes a reference *to that*, so the closure gets `&&i32`. Writing `|&&x|` unwraps both references. You can also write `|x| **x % 2 == 0`, or just `|x| *x % 2 == 0` and let auto-deref do the rest. When in doubt, the compiler tells you the type.

<callout background="blue" icon="🏋️">

**Exercise 8.1 — Loops to iterator chains**

Rewrite each as a single iterator expression with no `mut`:

1. Sum of squares of odd numbers below 1000.
1. Given `words: Vec<&str>`, the length of the longest word.
1. Given `lines: Vec<&str>` like `"alice,30"`, a `Vec<(String, u32)>`, skipping lines that don't parse.
1. Given `nums: &[i32]`, the indices where a number is bigger than the one before it.
1. Turn `"hello world"` into `"Hello World"`.

</callout>

<toggle>

Solutions

```rust
let a: u32 = (1..1000).filter(|n| n % 2 == 1).map(|n| n * n).sum();

let b = words.iter().map(|w| w.len()).max().unwrap_or(0);

let c: Vec<(String, u32)> = lines.iter()
    .filter_map(|l| {
        let (name, age) = l.split_once(',')?;
        Some((name.to_string(), age.trim().parse().ok()?))
    })
    .collect();

let d: Vec<usize> = nums.windows(2).enumerate()
    .filter(|(_, w)| w[1] > w[0])
    .map(|(i, _)| i + 1)
    .collect();

let e: String = "hello world".split(' ')
    .map(|w| {
        let mut cs = w.chars();
        match cs.next() {
            Some(f) => f.to_uppercase().chain(cs).collect::<String>(),
            None => String::new(),
        }
    })
    .collect::<Vec<_>>()
    .join(" ");
```

</toggle>

## Writing your own iterator

```rust
struct Countdown(u32);

impl Iterator for Countdown {
    type Item = u32;
    fn next(&mut self) -> Option<u32> {
        if self.0 == 0 { return None; }
        self.0 -= 1;
        Some(self.0 + 1)
    }
}

let v: Vec<u32> = Countdown(3).collect();   // [3, 2, 1]
```

Implement only `next`, and every adapter is yours for free.

### Building up to 8.2

**The goal in plain words:** in 2.3 you walked the Fibonacci numbers with a hand-written loop. Now make a type `Fib` that *is* an iterator of Fibonacci numbers, so you can use `.take_while()`, `.filter()`, `.sum()` and the rest on it. It should end cleanly when the next number wouldn't fit in a `u64`, rather than crash.

**Step 1: the state.** `Fib { a, b }` holds the next two numbers. Each call to `next` hands out `a` and shifts along:

```text
start:   a=0  b=1
next() → 0,  then a=1  b=1
next() → 1,  then a=1  b=2
next() → 1,  then a=2  b=3
```

**Step 2: `next` is the shift from 2.3, plus one line.** Use the `Countdown` above as a template. Save `self.a` to return it, compute `self.a.checked_add(self.b)`, move the values along, and return `Some(saved)`.

**Step 3: stopping on overflow.** `checked_add` returns an `Option`, and `next` returns an `Option` too, so `?` does the stopping for you:

```rust
let next = self.a.checked_add(self.b)?;   // None → next() returns None → iteration ends
```

**Step 4: ask the three questions with iterator methods.**

| Question | Methods |
| --- | --- |
| sum of the even ones below 4 million | `take_while(\|&x\| x < 4_000_000)`, `filter(…even…)`, `sum()` |
| first one with 10 digits | `find(\|&x\| x >= 1_000_000_000)` |
| how many in total | `count()` |

Each question needs a fresh `Fib { a: 0, b: 1 }`, because an iterator is used up as you go.

**Bonus: `successors`.** `std::iter::successors(first, f)` builds an iterator from a starting value and a function that makes the next value from the current one, stopping when `f` returns `None`:

```rust
let powers: Vec<u32> = std::iter::successors(Some(1u32), |&x| x.checked_mul(10)).collect();
// [1, 10, 100, …, 1_000_000_000]: stops when ×10 would overflow
```

For Fibonacci the "value" has to be the pair `(a, b)`, followed by `.map(|(a, _)| a)`.

<callout background="blue" icon="🏋️">

**Exercise 8.2 — Fibonacci iterator**

Write `struct Fib { a: u64, b: u64 }` implementing `Iterator<Item = u64>` that **stops** (returns `None`) instead of overflowing. Then use it: the sum of even Fibonacci numbers below 4 million (Project Euler 2), the first one with 10 digits, and how many there are in total before `u64` overflows. Bonus: write the same thing with `std::iter::successors` in one expression.

</callout>

<toggle>

Solution

```rust
impl Iterator for Fib {
    type Item = u64;
    fn next(&mut self) -> Option<u64> {
        let out = self.a;
        let next = self.a.checked_add(self.b)?;  // ? on Option: stop on overflow
        self.a = self.b;
        self.b = next;
        Some(out)
    }
}

let euler2: u64 = Fib { a: 1, b: 2 }.take_while(|&x| x < 4_000_000).filter(|x| x % 2 == 0).sum();

let fib = std::iter::successors(Some((0u64, 1u64)), |&(a, b)| Some((b, a.checked_add(b)?))).map(|p| p.0);
```

`euler2` is 4613732. (This version stops a couple of numbers early, when computing the *next* one would overflow; fixing that is a nice extra.)

</toggle>

## Returning closures

Each closure has its own unnamed type, so you return them as `impl Fn` (one concrete type) or `Box<dyn Fn>` (any of several).

```rust
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n
}

fn make_op(op: char) -> Box<dyn Fn(i32, i32) -> i32> {
    match op {
        '+' => Box::new(|a, b| a + b),
        '*' => Box::new(|a, b| a * b),
        _ => Box::new(|_, _| 0),
    }
}
```

### Building up to 8.3

**The goal in plain words:** functions that *make* functions. Each part gives back a closure that you store in a variable and call later. Three closures with different jobs: one remembers a count, one glues two functions together, and one is looked up by name from a table.

**Part 1: a closure with a memory.** A closure can own a variable it captured and change it on every call. That's the counter:

```rust
let mut total = 0;
let mut add = move |x: u32| { total += x; total };   // `add` now owns its own copy of total
println!("{}", add(5));   // 5
println!("{}", add(5));   // 10: it remembered
```

Changing a captured variable makes it `FnMut`. To return one from a function, the variable must be created *inside* the function and moved into the closure with `move`, because the function's local variables are gone once it returns. The return type is `impl FnMut() -> u32`. Call it with `let mut c = counter(); c(); c();`, and note that `c` needs `mut`.

**Part 2: composing.** "Compose f and g" means "a new function that does f, then g on the result":

```rust
let add_one = |x: i32| x + 1;
let double = |x: i32| x * 2;
let both = |x| double(add_one(x));   // both(3) = double(4) = 8
```

Your `compose(f, g)` returns that last closure. Read the generics as types flowing through: `f` turns an `A` into a `B`, `g` turns a `B` into a `C`, so the result turns an `A` into a `C`. It needs `move`, because it must own `f` and `g` after `compose` returns.

**Part 3: a table of named functions.** Each closure has its own type, so they can't share a `HashMap` as they are. `Box<dyn Fn(f64) -> f64>` is "any closure from `f64` to `f64`, on the heap", which gives them one common type:

```rust
use std::collections::HashMap;

let mut ops: HashMap<&str, Box<dyn Fn(f64) -> f64>> = HashMap::new();
ops.insert("double", Box::new(|x| x * 2.0));
ops.insert("negate", Box::new(|x| -x));

let f = &ops["double"];
println!("{}", f(3.0));   // 6
```

**The REPL.** For the input `square double 3`, split it into words. The last word is the number (`parse::<f64>()`), and the others are names. "Right to left" means apply `double` first, then `square`: (3 × 2)² = 36. So walk the names with `.rev()`, look each one up with `ops.get(name)` (unknown names give `None`), and apply it to the running value. Wrap it in a loop over stdin lines, like the game loop in 4.3.

<callout background="blue" icon="🏋️">

**Exercise 8.3 — Return closures**

1. `fn counter() -> impl FnMut() -> u32` returning 1, 2, 3, … on each call.
1. `fn compose<A, B, C>(f: impl Fn(A) -> B, g: impl Fn(B) -> C) -> impl Fn(A) -> C`.
1. A `HashMap<&str, Box<dyn Fn(f64) -> f64>>` of named functions (`"double"`, `"square"`, `"negate"`) and a tiny REPL that applies `square double 3` right-to-left.

</callout>

<toggle>

Solutions for 1 and 2

```rust
fn counter() -> impl FnMut() -> u32 {
    let mut n = 0;
    move || { n += 1; n }
}

fn compose<A, B, C>(f: impl Fn(A) -> B, g: impl Fn(B) -> C) -> impl Fn(A) -> C {
    move |x| g(f(x))
}
```

</toggle>

## Checklist

- [ ] I know which of `Fn`, `FnMut`, `FnOnce` a closure implements and why
- [ ] I can use `move` and know when it's required
- [ ] I write iterator chains instead of index loops by default
- [ ] I can implement `Iterator` and return closures

---

← [07 Generics, Traits and Lifetimes](page:8s3iybt7d1) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [09 Smart Pointers and Interior Mutability](page:p7tycay9d0)
