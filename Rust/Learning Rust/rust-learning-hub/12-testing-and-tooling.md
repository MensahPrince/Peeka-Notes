---
id: m4szna8wnq
title: "12 Testing and Tooling"
sort: 9000
version: 1
---

<callout background="orange" icon="🧪">

**Cargo ships with a test runner, a doc generator, a formatter and a linter.** Learn them properly and add the next ring of tools (property tests, fuzzing, benchmarks, Miri) so your code stays correct as it grows.

</callout>

**Level:** 🟠 Advanced · **Needs:** [06 Modules, Crates and Cargo](page:x4l9tlfa1i)

## Three kinds of tests

<columns>

<column>

### Unit tests

In the same file, can test private functions.

```rust
pub fn add(a: i32, b: i32) -> i32 { a + b }

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn adds() {
        assert_eq!(add(2, 2), 4);
    }

    #[test]
    #[should_panic(expected = "overflow")]
    fn overflows() {
        let _ = add(i32::MAX, 1);
    }
}
```

</column>

<column>

### Integration tests

In `tests/`, each file a separate crate using only your **public** API.

```rust
// tests/api.rs
use my_crate::add;

#[test]
fn public_api() {
    assert_eq!(add(1, 2), 3);
}
```

### Doc tests

Examples in `///` comments are compiled and run.

````rust
/// ```
/// assert_eq!(my_crate::add(1, 1), 2);
/// ```
pub fn add(a: i32, b: i32) -> i32 { a + b }
````

</column>

</columns>

<callout icon="💡">

`should_panic(expected = "overflow")` passes in debug builds because overflow panics there; in `--release` it wraps and the test fails. Tests that rely on overflow checks should use `checked_add`.

</callout>

### Running them

```bash
cargo test                        # everything
cargo test parse                  # only tests whose name contains "parse"
cargo test -- --nocapture         # show println! output
cargo test -- --test-threads=1    # run serially
cargo test --doc                  # only doc tests
cargo test -- --ignored           # run #[ignore]d slow tests
```

Tests can return `Result<(), E>` so you can use `?` inside them.

### Building up to 12.1

**The goal in plain words:** your config parser from 5.2 works when you try it by hand. Tests make it *keep* working: small functions that call your code with known inputs and check the answers. `cargo test` runs all of them in a second, every time you change something. You'll write all three kinds of test, then measure how much of your code they actually run.

**Step 0: the layout you'll end up with.** If you did 6.1, the parser is already a library:

```text
config-parser/
├── src/
│   ├── lib.rs          parse, Config, ConfigError (+ doc test on parse)
│   └── parser.rs       … + #[cfg(test)] mod tests { … }   ← unit tests
└── tests/
    ├── config.rs       ← integration test
    └── fixtures/
        └── good.conf   ← a sample file the test reads
```

**Step 1: one unit test per error variant.** Each test feeds in text that should trigger exactly one error. Make a list first:

| Variant | Input that triggers it |
| --- | --- |
| `MissingEquals` | `"host localhost"` (no `=`) |
| `BadNumber` | `"port = abc"`, then `get_u32("port")` |
| `MissingKey` | any config, then `get_u32("nope")` |
| `Io` | `Config::load("no/such/file")`, if you wrote `load` |

Then write each one as a test. Errors usually can't be compared with `==`, so check the *shape* with `matches!`:

```rust
#[test]
fn missing_equals_reports_the_line() {
    let err = parse("# comment\nhost localhost").unwrap_err();   // unwrap is fine in tests
    assert!(matches!(err, ConfigError::MissingEquals { line: 2 }));
}
```

Also add a "happy path" test that parses a good config and checks a couple of values.

**Step 2: a doc test.** Put a usage example in a `///` comment above `pub fn parse`, inside a code fence. `cargo test` compiles and runs it, so your documentation can't go out of date:

````rust
/// Parses `key = value` lines.
///
/// ```
/// let cfg = config_parser::parse("port = 80").unwrap();
/// assert_eq!(cfg.get_u32("port").unwrap(), 80);
/// ```
pub fn parse(text: &str) -> Result<Config, ConfigError> { … }
````

A doc test uses your crate from outside, so name it by the crate name.

**Step 3: an integration test with a fixture.** Files in `tests/` are compiled as separate programs that can only use your `pub` API. A *fixture* is a sample input file kept for tests. Tests run from the package root, but `CARGO_MANIFEST_DIR` is the safe way to build the path:

```rust
// tests/config.rs
#[test]
fn reads_the_good_fixture() {
    let path = concat!(env!("CARGO_MANIFEST_DIR"), "/tests/fixtures/good.conf");
    let text = std::fs::read_to_string(path).unwrap();
    let cfg = config_parser::parse(&text).unwrap();
    assert_eq!(cfg.get_u32("port").unwrap(), 8080);
}
```

**Step 4: coverage.** `cargo install cargo-llvm-cov`, then `cargo llvm-cov --open` shows your source with every line that no test ran highlighted. Write tests until the error branches are covered too.

<callout background="blue" icon="🏋️">

**Exercise 12.1 — Test the config parser**

Take your config parser from exercise 5.2. Write unit tests for every error variant, a doc test on the public `parse` function, and an integration test that reads a fixture file from `tests/fixtures/`. Aim for every branch covered: check with `cargo llvm-cov` (install `cargo-llvm-cov`).

</callout>

<toggle>

Hint

Use `matches!` for errors that carry data you don't care about: `assert!(matches!(parse("x"), Err(ConfigError::MissingEquals { line: 1 })));`. For the fixture path use `concat!(env!("CARGO_MANIFEST_DIR"), "/tests/fixtures/good.conf")`.

</toggle>

## Property-based testing

Instead of hand-picked examples, state a **property** that must hold for *all* inputs and let the library generate hundreds of cases and shrink failures to a minimal one.

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn reverse_twice_is_identity(v: Vec<i32>) {
        let mut w = v.clone();
        w.reverse();
        w.reverse();
        prop_assert_eq!(v, w);
    }
}
```

Good properties: round trips (`decode(encode(x)) == x`), invariants (sorted output is sorted and has the same elements), comparing against a simple slow reference implementation.

### Building up to 12.2

**The goal in plain words:** run-length encoding compresses text by counting repeats: `"aaab"` → `"3a1b"` (three a's, one b). You'll write the encoder and the decoder, then, instead of testing a few hand-picked strings, let proptest throw hundreds of random strings at them. You'll probably get a bug you never thought to test for.

**Step 1: encode by walking runs.** Keep the current character and how many times you've seen it in a row. When the character changes, write out the run:

```text
"aaab"
 a → current = a, count = 1
 a → same, count = 2
 a → same, count = 3
 b → different: write "3a", current = b, count = 1
end → write "1b"                                 result: "3a1b"
```

Loop over `s.chars()` (not bytes, so `é` stays whole), and `push_str(&format!("{count}{c}"))` for each run.

**Step 2: decode by reading digits, then a character.** Collect digit characters into a number until you meet a non-digit. That non-digit is the character to repeat:

```rust
let mut n = 0;
for c in encoded.chars() {
    if let Some(d) = c.to_digit(10) {
        n = n * 10 + d as usize;           // "12" → 1, then 12
    } else {
        out.extend(std::iter::repeat_n(c, n));   // c repeated n times
        n = 0;
    }
}
```

**Step 3: test the round trip by hand first.** `assert_eq!(decode(&encode("aaab")), "aaab");` for a few strings, as in 12.1.

**Step 4: the property test.** `cargo add --dev proptest`. Inside `proptest! { … }`, a test's arguments are *generated*. `s: String` means "try many random strings, including weird ones":

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn round_trip(s: String) {
        prop_assert_eq!(decode(&encode(&s)), s);
    }
}
```

When it fails, proptest **shrinks** the input to the smallest string that still fails, and prints it. Look at that string and work out what went wrong. Ask yourself how your decoder can tell a digit that's a count from a digit that's part of the text.

<callout background="blue" icon="🏋️">

**Exercise 12.2 — Property-test a round trip**

Write a run-length encoder `encode(&str) -> String` (`"aaab"` → `"3a1b"`) and `decode`. Property-test that `decode(encode(s)) == s` for any string. Proptest will very likely find a bug in your first version: what is it?

</callout>

<toggle>

The usual bug

Digits in the input. `"11"` encodes to `"21"`, which decodes as "one `1`, repeated 2 times"? or "21 of something"? The format is ambiguous. Fix it by escaping, by restricting the input alphabet (`"[a-z]*"` as a proptest strategy), or by a different format. Also check multi-byte characters such as `"é"`: iterate `chars()`, not bytes.

</toggle>

## Lint, format, document

| Tool | Command | What it does |
| --- | --- | --- |
| rustfmt | `cargo fmt` | standard formatting; `cargo fmt --check` in CI |
| Clippy | `cargo clippy` | 750+ lints for bugs, style, performance |
| rustdoc | `cargo doc --open` | HTML docs from `///` and `//!` comments |
| cargo-expand | `cargo expand` | show code after macros |
| cargo-llvm-cov | `cargo llvm-cov` | code coverage |
| cargo-nextest | `cargo nextest run` | faster, nicer test runner |
| cargo-audit / cargo-deny | `cargo audit` | known vulnerabilities, licences |
| Miri | `cargo +nightly miri test` | detects undefined behaviour (module 15) |
| cargo-fuzz | `cargo fuzz run target` | coverage-guided fuzzing with libFuzzer |
| criterion / divan | `cargo bench` | statistically sound benchmarks |

```rust
// at the top of lib.rs: opt into more lints
#![warn(clippy::pedantic, missing_docs)]
```

### Building up to 12.3

**The goal in plain words:** Clippy is a program that reads your code and points out things that are probably mistakes, slow, or not the usual Rust way. Its "pedantic" level is very picky on purpose. You'll run it on a real exercise, go through every warning, and decide for each one whether to fix it or deliberately ignore it.

**Step 1: run it.** In the grep-lite folder from 6.3:

```bash
cargo clippy -- -W clippy::pedantic
```

`-W` means "turn these into warnings". Expect a few dozen.

**Step 2: read one warning.** Each one names its lint and links to an explanation:

```text
warning: this argument is passed by value, but not consumed in the function body
  --> src/main.rs:12:17
   |
12 | fn print_line(line: String, n: usize) {
   |                     ^^^^^^ help: consider taking a reference instead: `&str`
   |
   = help: for further information visit https://rust-lang.github.io/rust-clippy/…#needless_pass_by_value
```

The last line is the lint's name (`needless_pass_by_value`). The linked page explains *why* it matters, and that's where the learning is.

**Step 3: fix it or allow it.** Most warnings come with a `help:` fix, and `cargo clippy --fix -- -W clippy::pedantic` applies the easy ones automatically (commit first, so you can see what changed). For a warning you disagree with, silence it where it occurs and say why:

```rust
#[allow(clippy::cast_possible_truncation)] // line counts never exceed u32 here
let n = count as u32;
```

Pedantic lints are opinions, and allowing one with a reason is a perfectly good outcome.

**Step 4: write down three.** Choose the three lints whose linked page taught you something new, and add them to the exercise's note in [Rust Exercises](page:nbnwy0q2zt).

<callout background="blue" icon="🏋️">

**Exercise 12.3 — Clippy pedantic cleanup**

Run `cargo clippy -- -W clippy::pedantic` on your largest exercise so far (grep-lite from 6.3 is a good one). Fix every warning or consciously `#[allow]` it with a comment saying why. Note the three lints that taught you something in [Rust Exercises](page:nbnwy0q2zt).

</callout>

## Benchmarking

```rust
// benches/sum.rs  (Cargo.toml: [[bench]] name = "sum", harness = false)
use criterion::{criterion_group, criterion_main, Criterion};
use std::hint::black_box;

fn bench(c: &mut Criterion) {
    let v: Vec<u64> = (0..10_000).collect();
    c.bench_function("sum 10k", |b| b.iter(|| black_box(&v).iter().sum::<u64>()));
}

criterion_group!(benches, bench);
criterion_main!(benches);
```

`black_box` stops the optimiser from computing the result at compile time and deleting your benchmark.

<toggle>

A CI pipeline worth copying

```yaml
# .github/workflows/ci.yml (excerpt)
- run: cargo fmt --check
- run: cargo clippy --all-targets -- -D warnings
- run: cargo test --all-features
- run: cargo doc --no-deps
  env: { RUSTDOCFLAGS: "-D warnings" }
```

Add `cargo +nightly miri test` for crates with `unsafe`, and `cargo deny check` for dependency policy.

</toggle>

## Checklist

- [ ] I write unit, integration and doc tests, and know what each can see
- [ ] I've had proptest find a bug I didn't think of
- [ ] My code is clean under `cargo fmt --check` and `cargo clippy -- -D warnings`
- [ ] I benchmark with `black_box` and criterion, not with `Instant` in a debug build

---

← [11 Async Rust](page:m5cbwfmofv) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [13 Macros](page:4s3fizqndu)
