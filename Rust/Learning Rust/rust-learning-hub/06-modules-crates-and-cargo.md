---
id: x4l9tlfa1i
title: "06 Modules, Crates and Cargo"
sort: 15000
version: 1
---

<callout background="green" icon="🗂️">

**Organising code that outgrows one file.** Modules control what's visible, crates are units of compilation, packages are what Cargo builds, and workspaces group packages.

</callout>

**Level:** 🟢 Basics · **Needs:** [05 Collections, Strings and Errors](page:ubfingfdn1)

## The vocabulary

| Term | Meaning |
| --- | --- |
| **Package** | a `Cargo.toml` and its source; what `cargo new` makes |
| **Crate** | one compilation unit: a library (`src/lib.rs`) or a binary (`src/main.rs`, `src/bin/*.rs`) |
| **Module** | a namespace inside a crate (`mod network;`) |
| **Path** | how you name an item: `crate::network::connect` |
| **Workspace** | several packages sharing one `Cargo.lock` and `target/` |

## Modules and visibility

```rust
// src/lib.rs
pub mod shapes;                 // loads src/shapes.rs (or src/shapes/mod.rs)

// src/shapes.rs
pub mod circle;                 // loads src/shapes/circle.rs

pub struct Square {
    pub side: f64,              // fields are private unless marked pub
    id: u32,
}

// src/shapes/circle.rs
pub fn area(r: f64) -> f64 { std::f64::consts::PI * r * r }
```

```text
src/
├── lib.rs
├── main.rs
├── shapes.rs
└── shapes/
    └── circle.rs
```

**Everything is private by default** (visible to its own module and children).

| Visibility | Visible to |
| --- | --- |
| *(none)* | this module and its children |
| `pub(super)` | the parent module |
| `pub(crate)` | the whole crate, not outside |
| `pub` | everyone who can reach the parent |

### Paths and `use`

```rust
use crate::shapes::circle;          // absolute, from the crate root
use super::helpers::clamp;          // relative: parent module
use std::collections::{HashMap, HashSet};
use std::io::{self, Write};         // io itself and io::Write
use std::fmt::Result as FmtResult;  // rename

pub use crate::shapes::Square;      // re-export: users can write my_crate::Square
```

Convention: bring **functions** in by their module (`circle::area(2.0)`), and **types** by name (`HashMap::new()`).

### Building up to 6.1

**The goal in plain words:** right now your config parser from 5.2 is all in `main.rs`. You'll move the real work into a **library** (code other programs could use) and leave `main.rs` as a thin program that just calls it. The code itself barely changes. What changes is *which file it lives in* and *who is allowed to see it*.

**Step 1: one package can hold a library and a program.** If `src/lib.rs` and `src/main.rs` both exist, Cargo builds two crates from one package. Both have the package's name, with `-` turned into `_`:

```text
config-parser/
├── Cargo.toml          name = "config-parser"
└── src/
    ├── lib.rs          the library: config_parser
    ├── main.rs         the program: uses config_parser::…
    ├── parser.rs       mod parser   (Config, parse, get_u32)
    └── error.rs        mod error    (ConfigError)
```

**Step 2: declare the modules in `lib.rs`.** `mod parser;` tells Rust "there is a file `parser.rs`, make it the module `parser`". Without that line the file is ignored:

```rust
// src/lib.rs
pub mod error;     // loads src/error.rs
pub mod parser;    // loads src/parser.rs

pub use error::ConfigError;     // optional: lets users write config_parser::ConfigError
pub use parser::Config;
```

**Step 3: reach across modules.** Inside `parser.rs`, the error type is in a sibling module, so name it from the crate root:

```rust
// src/parser.rs
use std::collections::HashMap;
use crate::error::ConfigError;      // crate:: = "from the top of this library"
```

**Step 4: choose what is visible.** Everything starts private. Mark what `main.rs` needs as `pub` (`Config`, `parse`, `get_u32`, `ConfigError`), and mark helpers that only the library uses as `pub(crate)`. They can be called from any module *inside* the library, but `main.rs` can't see them, because to the library `main.rs` is an outside user.

**Step 5: use the library from `main.rs`.**

```rust
// src/main.rs
use config_parser::Config;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let text = std::fs::read_to_string("app.conf")?;
    let config = Config::parse(&text)?;
    println!("port = {}", config.get_u32("port")?);
    Ok(())
}
```

**Step 6: see your public API.** `cargo doc --open` builds a website of the library's docs. Only `pub` items appear, so it shows exactly what an outside user can reach.

<callout background="blue" icon="🏋️">

**Exercise 6.1 — Split into lib + bin**

Take your config parser from 5.2. Move the parsing into `src/lib.rs` with modules `parser` and `error`, keep only argument handling and printing in `src/main.rs` (which uses the library as `your_crate_name::...`). Make the error type `pub`, but keep helper functions `pub(crate)`. Run `cargo doc --open` and see what your public API looks like.

</callout>

<toggle>

Why split lib and bin?

Integration tests (`tests/`) and other crates can only use a library. Keeping `main.rs` thin means almost all your logic is testable. This is the standard layout for CLI tools.

</toggle>

## Cargo.toml

```toml
[package]
name = "grep-lite"
version = "0.1.0"
edition = "2024"

[dependencies]
clap = { version = "4", features = ["derive"] }   # "4" means >=4.0.0, <5.0.0
regex = "1.11"

[dev-dependencies]          # only for tests, examples, benchmarks
tempfile = "3"

[features]
default = ["color"]
color = []

[profile.release]
lto = true
```

- **SemVer:** `"1.11"` accepts any `1.x` from 1.11 up. Cargo picks the newest compatible version and records it in `Cargo.lock`.
- **Commit `Cargo.lock`** for binaries and apps (reproducible builds). For libraries it's optional.
- `cargo add regex`, `cargo remove`, `cargo update`, `cargo tree` (see the dependency graph).
- Find crates on [crates.io](https://crates.io) and read docs on [docs.rs](https://docs.rs). [blessed.rs](https://blessed.rs) lists the de-facto standard crates.

## Workspaces

```toml
# Cargo.toml at the root
[workspace]
members = ["app-core", "cli"]
resolver = "3"

[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
```

```toml
# cli/Cargo.toml
[dependencies]
app-core = { path = "../app-core" }   # not "core": that name is taken by Rust's core library
serde = { workspace = true }
```

### Building up to 6.2

**The goal in plain words:** one folder, two packages. A library `temp-core` holds the temperature maths from 2.1, and a program `temp-cli` uses it so you can type `temp-cli 100 C` and get `212.0 °F`. A **workspace** lets you build and test both with one command, sharing one `target/` folder.

**Step 1: make the folders.**

```bash
mkdir temps && cd temps
cargo new --lib temp-core     # a library: src/lib.rs
cargo new temp-cli            # a program: src/main.rs
```

**Step 2: the root `Cargo.toml` lists the members.** Create `temps/Cargo.toml` by hand. It has no `[package]`, only `[workspace]`:

```toml
[workspace]
members = ["temp-core", "temp-cli"]
resolver = "3"
```

**Step 3: the library, with a test.** Put `c_to_f` and `f_to_c` in `temp-core/src/lib.rs`, marked `pub`. A test is a function marked `#[test]`, and `assert_eq!` fails it if the two sides differ:

```rust
pub fn c_to_f(c: f64) -> f64 { c * 9.0 / 5.0 + 32.0 }

#[cfg(test)]
mod tests {
    use super::*;              // bring in everything from the file above
    #[test]
    fn boiling() { assert_eq!(c_to_f(100.0), 212.0); }
}
```

**Step 4: the program depends on the library by path.** In `temp-cli/Cargo.toml`:

```toml
[dependencies]
temp-core = { path = "../temp-core" }
```

and in code it's `temp_core::c_to_f(…)` (`-` becomes `_`).

**Step 5: read the command-line arguments.** `std::env::args()` gives the words typed after the program name, and the first one is the program itself:

```rust
let args: Vec<String> = std::env::args().collect();   // ["temp-cli", "100", "C"]
let value: f64 = args[1].parse().expect("first argument must be a number");
match args[2].as_str() {
    "C" => println!("{:.1} °F", temp_core::c_to_f(value)),
    "F" => println!("{:.1} °C", temp_core::f_to_c(value)),
    other => eprintln!("unknown unit {other}"),
}
```

**Step 6: share a dependency.** Choose any crate both could use (for example `anyhow` for the CLI's errors). Write its version once under `[workspace.dependencies]` in the root, then `anyhow = { workspace = true }` in the member that uses it.

**Step 7: run it from the root.** `cargo test --workspace` runs every member's tests. `cargo run -p temp-cli -- 100 C` runs one member, and everything after `--` goes to *your* program, not to Cargo.

<callout background="blue" icon="🏋️">

**Exercise 6.2 — Cargo workspace**

Create a workspace with a library `temp-core` (your temperature conversions from 2.1, with tests) and a binary `temp-cli` that depends on it by path. Share one dependency via `[workspace.dependencies]`. Run `cargo test --workspace` and `cargo run -p temp-cli -- 100 C`.

</callout>

### Building up to 6.3

**The goal in plain words:** write a small version of `grep`, the Unix tool that prints the lines of a file that contain a pattern. `grep-lite error log.txt` prints every line of `log.txt` containing "error". Flags change the behaviour, and the program reports through its **exit code** whether it found anything, so shell scripts can use it.

How it should behave:

```bash
grep-lite -n fn src/main.rs      # matching lines, with line numbers: "3: fn main() {"
grep-lite -c TODO a.rs b.rs      # just how many lines matched
grep-lite -iv debug < app.log    # no files given: read stdin; -i ignore case, -v lines that DON'T match
echo $?                          # 0 = matched something, 1 = nothing matched, 2 = error
```

**Step 1: add the two crates.** `cargo add clap --features derive` and `cargo add regex`.

**Step 2: let clap read the arguments.** You describe the arguments as a struct, and `#[derive(Parser)]` writes the parsing code, including `--help`. A plain field is a required argument, a `Vec` collects the rest, and a `bool` with `#[arg(short = 'x')]` is a flag. The Starter toggle below has the full struct. Try it with only two fields first:

```rust
use clap::Parser;

#[derive(Parser)]
struct Args {
    pattern: String,
    #[arg(short = 'n')]
    line_numbers: bool,
}

fn main() {
    let args = Args::parse();       // try: cargo run -- -n hello   and   cargo run -- --help
    println!("pattern={} n={}", args.pattern, args.line_numbers);
}
```

**Step 3: match lines with a regex.** A `Regex` is a compiled pattern, and `is_match` tests a line:

```rust
let re = regex::Regex::new(r"err(or)?").unwrap();
re.is_match("an error here");    // true
re.is_match("all fine");         // false
```

`RegexBuilder` (in the Starter) is the same thing with options, such as `.case_insensitive(true)` for `-i`.

**Step 4: read lines from a file or from stdin, with the same code.** Both can be turned into something that implements `BufRead`, the trait with `.lines()`. Put either one in a `Box<dyn BufRead>` and the rest of the code doesn't care which it is:

```rust
use std::fs::File;
use std::io::{self, BufRead, BufReader};

let reader: Box<dyn BufRead> = match path {
    Some(p) => Box::new(BufReader::new(File::open(p)?)),
    None => Box::new(io::stdin().lock()),
};
for (i, line) in reader.lines().enumerate() {
    let line = line?;
    // i + 1 is the line number for -n
}
```

**Step 5: apply the flags.** For each line, `let hit = re.is_match(&line) != args.invert;`. Comparing to `invert` with `!=` flips the answer when `-v` is set. Then print it (with `i + 1` if `-n`), or only count it if `-c`.

**Step 6: exit codes.** Keep `let mut any = false;` and set it on every hit. End with `std::process::exit(if any { 0 } else { 1 })`. For errors (bad regex, missing file), print to stderr with `eprintln!` and `exit(2)`.

<callout background="blue" icon="🏋️">

**Exercise 6.3 — grep-lite CLI with clap**

Build `grep-lite PATTERN [FILES...]` with clap's derive API: flags `-i` (ignore case), `-n` (line numbers), `-c` (count only), `-v` (invert). Read stdin when no files are given. Use the `regex` crate. Exit code 0 if anything matched, 1 if not, 2 on error, like real grep.

</callout>

<toggle>

Starter

```rust
use clap::Parser;
use std::path::PathBuf;

#[derive(Parser)]
#[command(version, about = "A tiny grep")]
struct Args {
    pattern: String,
    files: Vec<PathBuf>,
    #[arg(short = 'i')]
    ignore_case: bool,
    #[arg(short = 'n')]
    line_numbers: bool,
    #[arg(short = 'c')]
    count: bool,
    #[arg(short = 'v')]
    invert: bool,
}

fn main() {
    let args = Args::parse();
    let re = regex::RegexBuilder::new(&args.pattern)
        .case_insensitive(args.ignore_case)
        .build()
        .unwrap_or_else(|e| { eprintln!("{e}"); std::process::exit(2) });
    // read each file (or stdin) with BufRead::lines() …
}
```

For stdin vs file, take a `Box<dyn BufRead>`: `Box::new(io::stdin().lock())` or `Box::new(BufReader::new(File::open(p)?))`.

</toggle>

## Checklist

- [ ] I can lay out a crate across files and folders
- [ ] I know the visibility levels and use `pub use` for a clean API
- [ ] I can add dependencies, features and dev-dependencies
- [ ] I can set up a workspace

---

← [05 Collections, Strings and Errors](page:ubfingfdn1) · [Rust Learning Hub](page:7jlv8afgwm) · Next → [07 Generics, Traits and Lifetimes](page:8s3iybt7d1)
