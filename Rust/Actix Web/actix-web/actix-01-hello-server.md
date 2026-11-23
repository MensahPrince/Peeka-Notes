---
id: gvpkmmg2hn
title: "Actix 01 Hello, Server"
sort: 8000
version: 1
---

<callout background="green" icon="👋">

**Your first web server in about 20 lines.** You'll create the `bank-api` project you'll grow through this whole course, run it, talk to it with `curl`, and understand what every line does.

</callout>

**Level:** 🟢 Basics · **Needs:** [02 Variables, Types and Control Flow](page:9ll61je1wh), a little of [11 Async Rust](page:m5cbwfmofv) helps but isn't required

## Step 0: tools

- **Rust 1.89 or newer.** Actix Web's minimum supported Rust version (MSRV) is 1.89. Run `rustup update`, then `rustc --version`. If you don't have Rust yet, install it with `rustup` ([official guide](https://doc.rust-lang.org/book/ch01-01-installation.html)).
- **`curl`** to send requests from a second terminal. (`curl --version` checks it's installed.)

You'll keep **two terminals** open throughout: one running the server, and one sending requests.

## Step 1: create the project

```bash
cargo new bank-api
cd bank-api
cargo add actix-web
```

`cargo add` writes this into `Cargo.toml` for you:

```toml
[dependencies]
actix-web = "4"
```

## Step 2: the smallest server

Replace `src/main.rs` with:

```rust
use actix_web::{get, App, HttpResponse, HttpServer, Responder};

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| App::new().service(hello))
        .bind(("127.0.0.1", 8080))?
        .run()
        .await
}
```

Run it in terminal 1. The first build downloads and compiles Actix, which takes a minute:

```bash
cargo run
```

It looks like nothing happens: the program is **waiting for requests**. In terminal 2:

```bash
curl localhost:8080/
# Hello world!
```

Or open `http://127.0.0.1:8080/` in a browser. Stop the server with **Ctrl+C**.

- [ ] I saw `Hello world!`

## Step 3: what each line does

| Code | Meaning |
| --- | --- |
| `#[get("/")]` | "call this function for `GET` requests to `/`" |
| `async fn hello()` | handlers are async, so one thread can juggle many waiting requests |
| `-> impl Responder` | returns *something that can become an HTTP response* |
| `HttpResponse::Ok().body(...)` | status 200 with this text as the body |
| `#[actix_web::main]` | starts Actix's async runtime, so `main` can be `async` |
| `HttpServer::new(\|\| App::new()...)` | an **app factory**: a closure Actix calls once per worker thread to build that worker's `App` |
| `.service(hello)` | registers the handler, **forgetting this is the #1 mistake** |
| `.bind(("127.0.0.1", 8080))?` | listen on this address and port, and `?` returns the error if the port is taken |
| `.run().await` | start serving, and keep going until Ctrl+C |

`std::io::Result<()>` is the return type because binding a port can fail with an I/O error.

## Step 4: more handlers

The handler below reads the request body as a `String`, and the next one is registered **without** a macro:

```rust
use actix_web::{get, post, web, App, HttpResponse, HttpServer, Responder};

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

#[post("/echo")]
async fn echo(req_body: String) -> impl Responder {
    HttpResponse::Ok().body(req_body)
}

async fn manual_hello() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(hello)
            .service(echo)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

```bash
curl -X POST localhost:8080/echo -d 'ping'   # ping
curl localhost:8080/hey                      # Hey there!
```

`-X POST` sets the method and `-d` sends a body. `echo` has a `String` parameter, so Actix fills it with the body. That's an **extractor**, covered properly in lesson 03.

## Break it on purpose

<toggle>

1. Remove `.service(echo)` and call `/echo`

`curl -i -X POST localhost:8080/echo -d ping` → `HTTP/1.1 404 Not Found`. It compiles fine, because the function exists, but nobody registered it. When a route 404s, **check `.service()` first**.

</toggle>

<toggle>

1. Start the server twice

Run `cargo run` in a second terminal while the first is still running, and you get `Error: Os { code: 98, kind: AddrInUse, message: "Address already in use" }`. Only one program can listen on a port. Stop the old one, or change `8080`.

</toggle>

<toggle>

1. Use `curl -i` to see the whole response

`-i` prints the status line and headers above the body:

```text
HTTP/1.1 200 OK
content-length: 12
content-type: text/plain; charset=utf-8

Hello world!
```

You'll use this all course long to check status codes.

</toggle>

<callout background="blue" icon="🏋️">

**Exercise 1.1: your own routes**

Add `GET /about` that returns your name, and `GET /time` that returns the current Unix time in seconds:

```rust
std::time::SystemTime::now()
    .duration_since(std::time::UNIX_EPOCH)
    .unwrap()
    .as_secs()
```

Register one with a macro and one with `.route()`. Check both with `curl -i`.

</callout>

## Checklist

- [ ] `cargo run` starts the server and `curl` gets a reply
- [ ] I can explain the app factory closure and `.service()`
- [ ] I know a missing `.service()` gives a 404, not a compile error
- [ ] I can send a POST with a body using `curl -X POST -d`

---

[Actix Web hub](page:4oh6zeq50g) · Next → [Actix 02 Routes and Handlers](page:xa3snk8iq3)
