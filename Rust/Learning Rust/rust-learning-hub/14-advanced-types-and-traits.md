---
id: olti30ae4p
title: "14 Advanced Types and Traits"
sort: 7000
version: 1
---

<callout background="orange" icon="🧬">

**Using the type system as a design tool.** Associated types, operator overloading, trait objects, newtypes, `PhantomData`, typestate, GATs and higher-ranked bounds: the patterns behind most well-designed crates.

</callout>

**Level:** 🟠 Advanced · **Needs:** [07 Generics, Traits and Lifetimes](page:8s3iybt7d1)

## Associated types vs generic parameters

```rust
trait Container {
    type Item;                         // one Item per implementing type
    fn first(&self) -> Option<&Self::Item>;
}

trait ConvertTo<T> {                   // many T per implementing type
    fn convert(&self) -> T;
}
```

Use an **associated type** when there's exactly one sensible choice per type (`Iterator::Item`, `Deref::Target`). Use a **generic parameter** when a type can implement the trait many times (`From<T>`, `Add<Rhs>`).

## Operator overloading

Operators are traits in `std::ops`: `Add`, `Sub`, `Mul`, `Neg`, `Index`, `AddAssign`, … and comparisons in `std::cmp`.

```rust
use std::ops::Add;

#[derive(Debug, Clone, Copy, PartialEq)]
struct Vec2 { x: f32, y: f32 }

impl Add for Vec2 {
    type Output = Vec2;
    fn add(self, o: Vec2) -> Vec2 { Vec2 { x: self.x + o.x, y: self.y + o.y } }
}
```

### Building up to 14.1

**The goal in plain words:** a `Vec2` is a 2D vector (a position or a velocity in a game). You want to write maths with it the way you would on paper: `a + b`, `a - b`, `-a`, `v * 2.0`, `2.0 * v`, `pos += vel`, `v[0]`. In Rust each operator is a trait, so "make `+` work" means "implement `Add`". You'll implement one trait per operator.

**Step 1: operator = trait = method.** The compiler rewrites operators into method calls:

| You write | Rust calls | Trait to implement |
| --- | --- | --- |
| `a + b` | `a.add(b)` | `Add` |
| `a - b` | `a.sub(b)` | `Sub` |
| `-a` | `a.neg()` | `Neg` (only one input) |
| `v * 2.0` | `v.mul(2.0)` | `Mul<f32> for Vec2` |
| `2.0 * v` | `2.0.mul(v)` | `Mul<Vec2> for f32` |
| `a += b` | `a.add_assign(b)` | `AddAssign` (takes `&mut self`, returns nothing) |
| `v[0]` | `*v.index(0)` | `Index<usize>` (returns a reference) |

**Step 2: copy the `Add` impl for `Sub`.** Same shape, `-` instead of `+`. `Neg` has no second argument: `fn neg(self) -> Vec2`.

**Step 3: the generic parameter is the right-hand side.** `Mul<f32>` means "multiply by an `f32`". For `2.0 * v`, the left side is an `f32`, so the impl goes on `f32`:

```rust
impl std::ops::Mul<Vec2> for f32 {
    type Output = Vec2;
    fn mul(self, v: Vec2) -> Vec2 { Vec2 { x: self * v.x, y: self * v.y } }
}
```

**Step 4: `Index` returns a reference.**

```rust
impl std::ops::Index<usize> for Vec2 {
    type Output = f32;
    fn index(&self, i: usize) -> &f32 {
        match i { 0 => &self.x, 1 => &self.y, _ => panic!("Vec2 index {i} out of range") }
    }
}
```

**Step 5: `dot` and `length`** are plain methods: dot = `x₁x₂ + y₁y₂`, length = `√(x² + y²)`, which is `self.dot(self).sqrt()`.

**Step 6: `Sum`.** `.sum()` on an iterator needs to know how to add up *your* type: start from zero and add each item. `fold(start, |acc, item| …)` is exactly that, and the second hint shows it.

<callout background="blue" icon="🏋️">

**Exercise 14.1 — Vec2 operator overloading**

Give `Vec2`: `Add`, `Sub`, `Neg`, `Mul<f32>` (scale), `Mul<Vec2> for f32` (so `2.0 * v` works too), `AddAssign`, and `Index<usize>` (0 → x, 1 → y, panic otherwise). Add `dot` and `length`. Then implement `Sum` so `vec_of_vec2.iter().copied().sum::<Vec2>()` works.

</callout>

<toggle>

Hints

- `impl Mul<Vec2> for f32 { type Output = Vec2; ... }` is allowed: `Mul` is foreign but `Vec2` (a type parameter of it) is local.
- `impl std::iter::Sum for Vec2 { fn sum<I: Iterator<Item = Vec2>>(iter: I) -> Vec2 { iter.fold(Vec2 { x: 0.0, y: 0.0 }, |a, b| a + b) } }`

</toggle>

## Trait objects and dyn compatibility

`dyn Trait` erases the concrete type; calls go through a vtable (see [17 Memory Layout and Performance](page:ap65audlmt)). A trait can be used as `dyn` only if it's **dyn compatible** (formerly "object safe"):

- no generic methods (add `where Self: Sized` to exclude one)
- no methods returning `Self` or taking `Self` by value without `where Self: Sized`
- no associated consts, no `Sized` supertrait

```rust
trait Shape {
    fn area(&self) -> f64;
    fn scaled(&self, k: f64) -> Self where Self: Sized;   // excluded from the vtable
}

let shapes: Vec<Box<dyn Shape>> = vec![/* … */];
```

<columns>

<column>

### Generics (`impl Trait`, `T: Trait`)

- Static dispatch, inlined, fastest
- One copy of code per type (bigger binary)
- All items one concrete type

</column>

<column>

### Trait objects (`dyn Trait`)

- Dynamic dispatch through a vtable
- One copy of code
- Mixed types in one collection; plugin systems

</column>

</columns>

## Small type tools

<toggle>

Supertraits, fully qualified syntax, blanket impls

```rust
trait Named { fn name(&self) -> String; }
trait Greeter: Named {                       // every Greeter must be Named
    fn greet(&self) -> String { format!("Hi, {}", self.name()) }
}

// Two traits with the same method name: disambiguate.
<Dog as Animal>::baby_name();
Trait::method(&value);

// Blanket impl: every Display type gets ToString this way in std.
impl<T: std::fmt::Display> MyExt for T { /* … */ }
```

</toggle>

<toggle>

Newtypes, type aliases, the never type, DSTs

- **Newtype** `struct Meters(f64);`: a distinct type at zero cost. Stops mixing up units, and lets you implement foreign traits on foreign types.
- **Type alias** `type Result<T> = std::result::Result<T, MyError>;`: just a name, not a new type.
- **Never type `!`**: the type of expressions that never finish (`panic!`, `loop {}`, `return`, `continue`). It coerces to any type, which is why `let x: u32 = match o { Some(v) => v, None => return };` works.
- **Dynamically sized types** (`str`, `[T]`, `dyn Trait`) only live behind a pointer. Generics have an implicit `T: Sized`; write `T: ?Sized` to accept them: `fn show<T: ?Sized + Debug>(t: &T)`.

</toggle>

## Encoding rules in types

### Typestate

Make invalid state transitions **fail to compile** by giving each state its own type:

```rust
use std::marker::PhantomData;

struct Open;
struct Closed;

struct Door<State> { name: String, _s: PhantomData<State> }

impl Door<Closed> {
    fn open(self) -> Door<Open> { Door { name: self.name, _s: PhantomData } }
}
impl Door<Open> {
    fn close(self) -> Door<Closed> { Door { name: self.name, _s: PhantomData } }
    fn walk_through(&self) { println!("walked through {}", self.name); }
}
```

`closed_door.walk_through()` is now a compile error, and because `open(self)` consumes the old value, you can't keep using a stale state.

### Building up to 14.2

**The goal in plain words:** in 4.4 a wrong move (approving a draft) was caught *while the program ran* and returned as an `Err`. Here, wrong moves don't even compile. A locked door has no `open` method at all, so writing `locked_door.open()` is a compile error. Then you'll use the same trick on a builder that refuses to `send()` until it has everything it needs.

**Step 1: read the `Door` example as "one type per state".** `Door<Open>` and `Door<Closed>` are different types, and each `impl` block gives methods to only one of them. `Open` and `Closed` are empty structs used only as labels. `PhantomData<State>` is how a struct "uses" a type parameter without storing anything.

**Step 2: add `Locked`.** One more empty struct, plus two methods:

- in `impl Door<Closed>`: `fn lock(self, key: &str) -> Door<Locked>`
- a new `impl Door<Locked>` with `fn unlock(self, key: &str) -> Door<Closed>`

Store the key in the door (for example a `key: String` field set when it's made), and compare it in `unlock`. A wrong key is a run-time problem, so `unlock` can return `Result<Door<Closed>, Door<Locked>>`, handing the door back still locked. Then check that `Door<Locked>` has no `open`: write `locked.open()` and read the error.

**Step 3: a builder, checking one requirement at a time.** A builder collects settings step by step and then does something with them. Start with just the URL:

```rust
struct Missing;
struct Set;

struct Request<U> { url: Option<String>, _u: PhantomData<U> }

impl Request<Missing> {
    fn new() -> Self { Request { url: None, _u: PhantomData } }
    fn url(self, u: &str) -> Request<Set> { Request { url: Some(u.into()), _u: PhantomData } }
}
impl Request<Set> {
    fn send(self) { println!("GET {}", self.url.unwrap()); }
}

Request::new().url("https://x.io").send();   // ✅
// Request::new().send();                    // ❌ no method `send` on Request<Missing>
```

**Step 4: two requirements, in any order.** Give it two labels, `Request<U, M>`, one for the URL and one for the method. `url()` turns `U` from `Missing` into `Set` and leaves `M` as it was, and `method()` does the same for `M`. Writing `impl<M> Request<Missing, M>` means "for any `M`", which is what makes the order not matter. `send` goes in `impl Request<Set, Set>` only.

<callout background="blue" icon="🏋️">

**Exercise 14.2 — Typestate door**

Extend the door with a `Locked` state: `Closed → lock(key) → Locked`, `Locked → unlock(key) → Closed`. A `Door<Locked>` can't be opened. Then build an HTTP request builder where `send()` only exists after `.url(..)` and `.method(..)` have both been called, in any order.

</callout>

<toggle>

Builder hint

Use two type parameters, `RequestBuilder<U, M>`, where each is either `Missing` or `Set`. `url()` is implemented for `RequestBuilder<Missing, M>` and returns `RequestBuilder<Set, M>`; same for `method()`. `send()` is implemented only for `RequestBuilder<Set, Set>`.

</toggle>

### PhantomData

`PhantomData<T>` is a zero-sized field that tells the compiler "act as if I own/use a `T`". It's used for typestate and units, and in unsafe containers to get variance and drop-check right.

### Building up to 14.3

**The goal in plain words:** in 1999 a Mars probe was lost because one team's software used feet and another's used metres. Here you'll make the compiler stop that mistake. A length *carries its unit in its type*, so adding metres to feet is a compile error, and converting between them is an explicit call.

**Step 1: the unit is a label, not data.** Every length stores just an `f64`. The unit exists only in the type, as in the door example:

```rust
use std::marker::PhantomData;

struct Meters;
struct Feet;
struct Length<U> { value: f64, _u: PhantomData<U> }

let a: Length<Meters> = Length { value: 3.0, _u: PhantomData };
let b: Length<Feet>   = Length { value: 10.0, _u: PhantomData };
```

A small `fn new(value: f64) -> Self` in `impl<U> Length<U>` saves typing `PhantomData` every time.

**Step 2: `+` only for the same unit.** Implement `Add` for `Length<U>` with `Output = Length<U>` and the right-hand side also `Length<U>` (the default `Rhs = Self`). Since both sides must have the same `U`, `a + b` above doesn't compile. Try it and read the error: it says `expected Length<Meters>, found Length<Feet>`.

**Step 3: attach a number to each unit with an associated constant.** A trait can require a constant, and each unit type supplies its own:

```rust
trait Unit { const PER_METER: f64; }          // how many of this unit make one metre
impl Unit for Meters { const PER_METER: f64 = 1.0; }
impl Unit for Feet   { const PER_METER: f64 = 3.280_84; }

println!("{}", Feet::PER_METER);   // 3.28084
```

**Step 4: convert.** `fn to<V: Unit>(self) -> Length<V>` goes through metres: divide by your own unit's `PER_METER` (giving metres), then multiply by the target's:

```text
10 ft → 10 / 3.28084 = 3.048 m → 3.048 × 1.0 = 3.048 m
```

Inside the method, `U::PER_METER` and `V::PER_METER` give the two numbers. Call it with `b.to::<Meters>()`, then `a + b.to::<Meters>()` compiles.

<callout background="blue" icon="🏋️">

**Exercise 14.3 — Units of measure with PhantomData**

Make `struct Length<U> { value: f64, _u: PhantomData<U> }` with unit types `Meters` and `Feet`. `Length<Meters> + Length<Meters>` works; `Length<Meters> + Length<Feet>` doesn't compile. Add `fn to<V>(self) -> Length<V>` that converts using a `trait Unit { const PER_METER: f64; }`.

</callout>

<toggle>

Solution sketch

```rust
trait Unit { const PER_METER: f64; }
struct Meters; impl Unit for Meters { const PER_METER: f64 = 1.0; }
struct Feet;   impl Unit for Feet   { const PER_METER: f64 = 3.280_84; }

impl<U: Unit> Length<U> {
    fn to<V: Unit>(self) -> Length<V> {
        Length { value: self.value / U::PER_METER * V::PER_METER, _u: PhantomData }
    }
}

impl<U> std::ops::Add for Length<U> {
    type Output = Length<U>;
    fn add(self, o: Self) -> Self { Length { value: self.value + o.value, _u: PhantomData } }
}
```

</toggle>

## Generic associated types (GATs)

An associated type can take its own generic parameters, usually a lifetime. That lets a trait return items that **borrow from the iterator itself**, which `Iterator` can't express:

```rust
trait LendingIterator {
    type Item<'a> where Self: 'a;
    fn next(&mut self) -> Option<Self::Item<'_>>;
}
```

### Building up to 14.4

**The goal in plain words:** `slice.windows(2)` gives overlapping *read-only* pairs: `[1, 2, 3]` → `[1, 2]`, `[2, 3]`. You want the same thing, but where you can *change* each window. With that you can turn `[1, 2, 3, 4]` into running sums in place, `[1, 3, 6, 10]`, by adding each element to the next. The standard `Iterator` trait can't do this, and working out why is the real lesson.

**Step 1: why `Iterator` refuses.** Overlapping windows share elements: `[1, 2]` and `[2, 3]` both contain the `2`. If you could hold two of them at once, you'd have two `&mut` to the same number, which Rust forbids. `Iterator` can't stop you holding two items at once: `.collect()` keeps all of them. So no safe `Iterator` can hand out overlapping `&mut` windows.

**Step 2: the fix is "lend one at a time".** If each window borrows *the iterator*, you have to drop it before calling `next` again, so there's never more than one. That is what this signature says:

```rust
fn next(&mut self) -> Option<Self::Item<'_>>;
//      ^^^^^^^^^                       ^^^ the item borrows from this &mut self
```

`type Item<'a>` is an associated type that takes a lifetime, a **GAT** (generic associated type). For windows it's `type Item<'a> = &'a mut [T] where Self: 'a;`.

**Step 3: getting one mutable window.** `get_mut(range)` returns `Some(&mut slice[range])`, or `None` if the range runs past the end. That `None` is your stopping signal:

```rust
let mut data = [1, 2, 3, 4];
if let Some(w) = data.get_mut(1..3) {   // &mut [2, 3]
    w[1] += w[0];                         // data is now [1, 2, 5, 4]
}
```

**Step 4: `next`.** Take the window at `self.pos..self.pos + self.size` with `?` (which stops at the end), add 1 to `pos`, and return it.

**Step 5: using it.** `for` loops need `Iterator`, so use `while let` instead:

```rust
let mut it = WindowsMut { slice: &mut data, size: 2, pos: 0 };
while let Some(w) = it.next() {
    w[1] += w[0];
}
```

Try saving two windows (`let a = it.next(); let b = it.next();` and then using `a`), and read the borrow error. That error is the whole point of the exercise.

<callout background="blue" icon="🏋️">

**Exercise 14.4 — Lending iterator with GATs**

Implement `LendingIterator` for `WindowsMut<'s, T> { slice: &'s mut [T], size: usize, pos: usize }` yielding overlapping **mutable** windows `&mut [T]`. Use it to turn `[1, 2, 3, 4]` into running pair sums in place. Explain why a normal `Iterator` can't yield overlapping `&mut` windows.

</callout>

<toggle>

Solution sketch and the why

```rust
impl<'s, T> LendingIterator for WindowsMut<'s, T> {
    type Item<'a> = &'a mut [T] where Self: 'a;
    fn next(&mut self) -> Option<&mut [T]> {
        let w = self.slice.get_mut(self.pos..self.pos + self.size)?;
        self.pos += 1;
        Some(w)
    }
}

while let Some(w) = it.next() { w[1] += w[0]; }
```

A normal `Iterator::next(&mut self) -> Option<Self::Item>` returns items unconnected to the `&mut self` borrow, so you could `collect()` all windows and hold two overlapping `&mut` at once. With the GAT, each window borrows the iterator, so you must drop it before calling `next` again.

</toggle>

## Higher-ranked trait bounds

`for<'a>` says "for every lifetime":

```rust
fn apply<F>(f: F) where F: for<'a> Fn(&'a str) -> &'a str {
    let s = String::from("hi");
    println!("{}", f(&s));   // works for this local borrow, whatever its lifetime
}
```

You mostly meet these in error messages; `Fn(&str) -> &str` is sugar for the same thing.

## Checklist

- [ ] I can choose between an associated type and a generic parameter
- [ ] I can overload operators, including the "scalar on the left" case
- [ ] I know the rules for dyn compatibility and when to choose `dyn`
- [ ] I can make invalid states unrepresentable with typestate
- [ ] I can read a GAT and a `for<'a>` bound

---

← [13 Macros](page:4s3fizqndu) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [15 Unsafe Rust](page:fnp814n35n)
