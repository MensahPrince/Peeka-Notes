---
id: 4s3fizqndu
title: "13 Macros"
sort: 8000
version: 1
---

<callout background="orange" icon="🪄">

**Code that writes code.** Declarative macros (`macro_rules!`) match on syntax patterns; procedural macros are Rust functions that take tokens and return tokens. Reach for them to remove boilerplate that functions and generics can't.

</callout>

**Level:** 🟠 Advanced · **Needs:** [12 Testing and Tooling](page:m4szna8wnq) (you'll want `cargo expand`)

## Why macros at all?

Macros run at compile time on **syntax**, so they can do what functions can't:

- take a variable number of arguments (`println!`, `vec!`)
- generate items: structs, impls, tests (`#[derive(Debug)]`)
- create small DSLs (`html!`, `sqlx::query!` which checks SQL against your database at compile time)

Prefer a function or generic when one will do: macros are harder to read, document and debug.

## Declarative macros: `macro_rules!`

```rust
macro_rules! square {
    ($x:expr) => {
        $x * $x
    };
}

let n = square!(2 + 3);   // 25, not 11: $x is a whole expression node
```

### Fragment specifiers

| Specifier | Matches | Example |
| --- | --- | --- |
| `expr` | an expression | `2 + 3`, `foo()` |
| `ident` | an identifier | `my_var`, `Point` |
| `ty` | a type | `Vec<u8>`, `&str` |
| `pat` | a pattern | `Some(x)`, `1..=5` |
| `tt` | a single token tree | anything; the escape hatch |
| `literal` | a literal | `42`, `"hi"` |
| `block` | a `{ … }` block |  |
| `item` | an item | `fn f() {}`, `struct S;` |
| `path`, `stmt`, `lifetime`, `vis`, `meta` | as named | `std::io`, `'a`, `pub(crate)` |

### Repetition

`$( ... ),*` means "zero or more, separated by commas"; `+` is one or more; `?` is optional.

```rust
macro_rules! my_vec {
    ( $( $x:expr ),* $(,)? ) => {{
        let mut v = Vec::new();
        $( v.push($x); )*
        v
    }};
}

let v = my_vec![1, 2, 3,];
```

<callout icon="💡">

**Hygiene:** the `v` inside the macro can't clash with a `v` at the call site; local variables in a macro live in their own scope. Items and types are *not* hygienic, so refer to them by full path: `::std::vec::Vec`, and `$crate::helper` for your own crate.

</callout>

### Building up to 13.1

**The goal in plain words:** Rust has `vec![1, 2, 3]` for building a `Vec` in one go, but nothing like it for a `HashMap`. You'll write your own `hashmap!` macro, so that this:

```rust
let m = hashmap! { "a" => 1, "b" => 2 };
```

turns into this before the compiler sees it:

```rust
let m = {
    let mut m = ::std::collections::HashMap::new();
    m.insert("a", 1);
    m.insert("b", 2);
    m
};
```

**Step 1: a macro rule is "pattern ⇒ code".** The left side describes what the call looks like, and `$name:kind` captures a piece of it. The right side is the code to produce:

```rust
macro_rules! pair {
    ($a:expr => $b:expr) => { ($a, $b) };   // matches  pair!(1 => 2)
}
let p = pair!(1 => 2);   // (1, 2)
```

`=>` inside the pattern is just a token the caller has to type. You get to invent the syntax.

**Step 2: repetition.** `$( … ),*` in the pattern means "this, zero or more times, separated by commas". In the output, `$( … )*` repeats once for each match. That's exactly how `my_vec!` above works: compare its pattern and output line by line, then do the same with `$k:expr => $v:expr` as the repeated part.

**Step 3: the details from the exercise.**

| Requirement | How |
| --- | --- |
| trailing comma allowed | add `$(,)?` at the end of the pattern: "an optional comma" |
| zero entries | `*` already allows zero repetitions, so `hashmap!{}` gives an empty map |
| works in any module | write `::std::collections::HashMap` in full, so the caller needn't `use` it |
| usable from other crates | `#[macro_export]` above `macro_rules!` |
| double braces `{{ … }}` | the outer pair belongs to the macro rule and the inner pair makes a block, so the `let` stays inside it |

**Step 4: pre-allocating capacity (the tricky bit).** `HashMap::with_capacity(n)` needs the number of entries, but macros can't count. The trick in the solution turns every key into `()`, puts the `()`s in an array, and takes the array's length. It's fine to skip this at first and use `HashMap::new()`.

Check your output with `cargo expand`, which shows the code your macro produced.

<callout background="blue" icon="🏋️">

**Exercise 13.1 — hashmap! macro**

Write `hashmap!{ "a" => 1, "b" => 2 }` that builds a `HashMap`, allows a trailing comma, works with zero entries, and pre-allocates the right capacity. Use full paths so it works in any module.

</callout>

<toggle>

Solution

```rust
#[macro_export]
macro_rules! hashmap {
    ( $( $k:expr => $v:expr ),* $(,)? ) => {{
        let mut m = ::std::collections::HashMap::with_capacity(
            <[()]>::len(&[ $( $crate::hashmap!(@unit $k) ),* ])
        );
        $( m.insert($k, $v); )*
        m
    }};
    (@unit $_t:tt) => { () };
}
```

The `@unit` rule is the counting trick: replace each key with `()` and take the array's length, which is a constant. (Simpler is fine too: skip the capacity.)

</toggle>

### Recursive macros and internal rules

Macros can call themselves to process input one piece at a time ("tt munchers"). Rules starting with a marker like `@unit` or `@step` are a convention for internal helpers.

### Building up to 13.2

**The goal in plain words:** writing a getter method for every field of a struct is boring and repetitive, which makes it a job for a macro. You'll write the struct *inside* a macro call, and the macro gives back the same struct plus a method for each field, so `person.name()` and `person.age()` work without you writing them.

```rust
struct_with_getters! {
    struct Person { name: String, age: u32 }
}
// becomes:
#[derive(Debug)]
struct Person { name: String, age: u32 }
impl Person {
    pub fn name(&self) -> &String { &self.name }
    pub fn age(&self) -> &u32 { &self.age }
}
```

**Step 1: match the struct's outline.** The pattern copies what a struct looks like, with captures for the parts that change: the struct's name (`ident`), and for each field its name (`ident`) and type (`ty`):

```rust
(struct $name:ident { $( $field:ident : $ty:ty ),* $(,)? }) => { … }
```

Read it as: the word `struct`, a name, `{`, then any number of `field: Type` separated by commas, `}`.

**Step 2: output the struct again.** Write the same shape back with the captures filled in: `struct $name { $( $field: $ty ),* }`, with `#[derive(Debug)]` above it.

**Step 3: one method per field.** In the `impl` block, repeat a method for each `$field`. The captured field name becomes the **method** name, which is something a function could never do:

```rust
impl $name {
    $( pub fn $field(&self) -> &$ty { &self.$field } )*
}
```

**Step 4: look at the result.** `cargo install cargo-expand`, then `cargo expand` prints your program with every macro expanded. Find `Person` in the output and check it matches the example above.

**Extras** (as in the solution): `$vis:vis` to allow `pub struct`, and `$(#[$meta:meta])*` to pass the caller's own attributes through.

<callout background="blue" icon="🏋️">

**Exercise 13.2 — Getter-generating macro**

Write `struct_with_getters! { struct Person { name: String, age: u32 } }` that emits the struct (with `#[derive(Debug)]`) and an `impl` with a `fn name(&self) -> &String` and `fn age(&self) -> &u32` for every field. Inspect the result with `cargo expand`.

</callout>

<toggle>

Solution

```rust
macro_rules! struct_with_getters {
    (
        $(#[$meta:meta])*
        $vis:vis struct $name:ident { $( $fvis:vis $field:ident : $ty:ty ),* $(,)? }
    ) => {
        $(#[$meta])*
        #[derive(Debug)]
        $vis struct $name { $( $fvis $field: $ty ),* }

        impl $name {
            $( pub fn $field(&self) -> &$ty { &self.$field } )*
        }
    };
}
```

</toggle>

## Procedural macros

Three kinds, all defined in a separate crate with `proc-macro = true`:

| Kind | Looks like | Gets |
| --- | --- | --- |
| Derive | `#[derive(MyTrait)]` | the item; **adds** new items after it |
| Attribute | `#[route(GET, "/")] fn index() {}` | the attribute args and the item; **replaces** the item |
| Function-like | `sql!(SELECT * FROM t)` | the tokens inside |

```mermaid
flowchart LR
  A[Your source tokens] --> B["syn: parse into a syntax tree"] --> C[Your logic] --> D["quote!: build output tokens"] --> E[Compiler continues]
```

```toml
# describe-derive/Cargo.toml
[lib]
proc-macro = true

[dependencies]
syn = { version = "2", features = ["full"] }
quote = "1"
proc-macro2 = "1"
```

```rust
// describe-derive/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;
    quote! {
        impl #name {
            pub fn hello() { println!("Hello from {}", stringify!(#name)); }
        }
    }
    .into()
}
```

### Building up to 13.3

**The goal in plain words:** `#[derive(Debug)]` is a procedural macro: a small Rust program the compiler runs on your struct, which writes code for it. You'll write your own derive, `Describe`. Putting `#[derive(Describe)]` on a type gives it a `describe()` function that returns a text description of the type's shape:

```rust
#[derive(Describe)]
struct Point { x: i32, y: i32 }
Point::describe()          // "Point { x: i32, y: i32 }"

#[derive(Describe)]
enum Dir { Up, Down }
Dir::describe()            // "Dir { Up, Down }"
```

**Step 1: why two crates?** A proc macro has to be compiled *before* the code that uses it, so it lives in its own crate marked `proc-macro = true`. The trait goes in a normal crate:

```text
describe-ws/
├── Cargo.toml            [workspace] members = ["describe", "describe-derive", "demo"]
├── describe/             pub trait Describe { fn describe() -> String; }
├── describe-derive/      #[proc_macro_derive(Describe)]  (the example above, adapted)
└── demo/                 depends on both; uses #[derive(Describe)]
```

**Step 2: get the example above working first.** Rename it to `Describe` and generate `impl Describe for #name { fn describe() -> String { … } }` returning just the name. Seeing `Point::describe()` print `"Point"` proves the whole pipeline works.

**Step 3: what you get as input.** `parse_macro_input!(input as DeriveInput)` gives a parsed syntax tree. The parts you need:

| Field | Holds |
| --- | --- |
| `ast.ident` | the type's name (`Point`) |
| `ast.data` | `Data::Struct(…)`, `Data::Enum(…)` or `Data::Union(…)` |
| struct → `.fields.named` | each field's `ident` (name) and `ty` (type) |
| enum → `.variants` | each variant's `ident` |

`match &ast.data { … }` picks the case.

**Step 4: build the string at compile time.** For each field, `quote!(#n: #t).to_string()` turns tokens back into text such as `"x : i32"`. Join the pieces with `", "`, then put the finished string into the output with `quote! { … #text … }`.

**Step 5: a clear error for unions.** Instead of panicking, return a compile error pointing at the user's code:

```rust
return syn::Error::new_spanned(&ast, "Describe can't be derived for unions")
    .to_compile_error()
    .into();
```

**Step 6: test that the error appears.** `trybuild` compiles small files that are *meant* to fail and compares the error output with a saved `.stderr` file. Add it as a dev-dependency of `demo`, then write `tests/ui/union.rs` with the bad code and a test calling `trybuild::TestCases::new().compile_fail("tests/ui/*.rs")`. On the first run it writes the `.stderr` file for you to check.

<callout background="blue" icon="🏋️">

**Exercise 13.3 — Derive macro Describe**

Create a workspace with `describe` (the trait: `fn describe() -> String`) and `describe-derive` (the proc macro). `#[derive(Describe)]` on `struct Point { x: i32, y: i32 }` should return `"Point { x: i32, y: i32 }"`; on an enum it lists the variants. On a union, emit a proper `compile_error!` with a span pointing at the item. Test with `trybuild` for the error case.

</callout>

<toggle>

Hints

- Match `ast.data` against `syn::Data::Struct`, `Data::Enum`, `Data::Union`.
- For fields: `fields.named.iter().map(|f| { let n = f.ident.as_ref().unwrap(); let t = &f.ty; quote!(#n: #t).to_string() })`.
- Errors: `return syn::Error::new_spanned(&ast, "Describe can't be derived for unions").to_compile_error().into();`
- Generics: use `ast.generics.split_for_impl()` to get `impl_generics`, `ty_generics`, `where_clause`.

</toggle>

## Debugging macros

- `cargo install cargo-expand` then `cargo expand` shows the code after macro expansion
- `trace_macros!(true);` on nightly prints each expansion step
- `compile_error!("message")` inside a rule gives users a clear error for bad input

## Checklist

- [ ] I can write a `macro_rules!` with repetition and a trailing comma
- [ ] I know which fragment specifier to use and what hygiene covers
- [ ] I can write a derive macro with `syn` and `quote`
- [ ] I use `cargo expand` when a macro misbehaves

---

← [12 Testing and Tooling](page:m4szna8wnq) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [14 Advanced Types and Traits](page:olti30ae4p)
