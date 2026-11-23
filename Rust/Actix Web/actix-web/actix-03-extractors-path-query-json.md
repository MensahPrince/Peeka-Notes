---
id: caj66ugnge
title: "Actix 03 Extractors: Path, Query, Json"
sort: 6000
version: 1
---

<callout background="green" icon="📥">

**Get data out of the request.** In Actix you don't parse requests yourself. You list what you want as **function parameters**, and Actix fills them in before your handler runs, or rejects the request with a 400/404 if it can't.

</callout>

**Level:** 🟢 Basics · **Needs:** [Actix 02 Routes and Handlers](page:xa3snk8iq3), [04 Structs, Enums and Pattern Matching](page:fn56do8fw7)

## The big idea

```rust
async fn handler(path: web::Path<u32>, query: web::Query<Filter>, body: web::Json<NewAccount>) -> …
//               ^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^^^
//               from /accounts/7      from ?min_balance=50         from the JSON body
```

Each parameter type is an **extractor**. Actix looks at the type and knows where to get the data.

| Extractor | Reads from | Example request | If it fails |
| --- | --- | --- | --- |
| `String` | the raw body | `-d 'hello'` | none |
| `web::Path<T>` | `{…}` parts of the URL | `/accounts/7` | **404** |
| `web::Query<T>` | `?key=value` after the path | `/search?q=rust` | **400** |
| `web::Json<T>` | a JSON body | `-d '{"name":"Ama"}'` | **400** |
| `web::Data<T>` | the app's shared state | (lesson 04) | **500** |

## Step 1: add serde

`Query` and `Json` turn text into your structs with **serde**, Rust's standard serialisation library:

```bash
cargo add serde --features derive
cargo add serde_json
```

- `#[derive(Deserialize)]` means "this struct can be **built from** JSON or a query string" (incoming).
- `#[derive(Serialize)]` means "this struct can be **turned into** JSON" (outgoing).

## Step 2: `Path`, values in the URL

One value:

```rust
#[get("/{id}")]
async fn get_account(path: web::Path<u32>) -> impl Responder {
    let id = path.into_inner();          // Path<u32> → u32
    format!("you asked for account {id}")
}
```

Several values come as a **tuple**, in the order they appear in the URL:

```rust
#[get("/users/{name}/posts/{post_id}")]
async fn user_post(path: web::Path<(String, u32)>) -> impl Responder {
    let (name, post_id) = path.into_inner();
    format!("post {post_id} by {name}")
}
```

```bash
curl localhost:8080/users/ama/posts/7      # post 7 by ama
curl -i localhost:8080/users/ama/posts/abc # 404  can not parse "abc" to a u32
```

Actix checks the **type** for you: `abc` isn't a `u32`, so your handler never runs.

## Step 3: `Query`, the `?key=value` part

Describe the query as a struct. `Option` fields are optional, and anything else is required:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct Search {
    q: String,           // required
    page: Option<u32>,   // optional
}

#[get("/search")]
async fn search(query: web::Query<Search>) -> impl Responder {
    format!("searching {} on page {}", query.q, query.page.unwrap_or(1))
}
```

```bash
curl 'localhost:8080/search?q=rust'          # searching rust on page 1
curl 'localhost:8080/search?q=rust&page=3'   # searching rust on page 3
curl -i localhost:8080/search                # 400  Query deserialize error: missing field `q`
```

Put URLs with `?` or `&` in quotes, or the shell treats those characters specially.

<toggle>

Why can I write `query.q` instead of `query.into_inner().q`?

`web::Query<T>`, `web::Json<T>` and `web::Path<T>` all implement `Deref` to `T`, so `.field` reaches through the wrapper, the same way `info.age` worked through `&User` in your Rust project. Use `.into_inner()` when you need to **own** the value, for example to move a `String` out of it.

</toggle>

## Step 4: `Json`, the request body

This is the important one for an API. Your bank's "new account" request:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct NewAccount {
    name: String,
    age: u16,
}

#[post("")]
async fn create_account(body: web::Json<NewAccount>) -> impl Responder {
    HttpResponse::Ok().json(serde_json::json!({
        "message": format!("would create an account for {}, age {}", body.name, body.age)
    }))
}
```

`HttpResponse::Ok().json(…)` turns any `Serialize` value into a JSON response and sets the `content-type` header. `serde_json::json!` builds a JSON value inline.

```bash
curl -X POST localhost:8080/accounts \
  -H 'content-type: application/json' \
  -d '{"name":"Ama","age":25}'
# {"message":"would create an account for Ama, age 25"}
```

### Break it on purpose

<columns>

<column>

**Wrong field name**

```bash
-d '{"nam":"Ama","age":25}'
```

`400 Json deserialize error: missing field 'name'`

</column>

<column>

**Wrong type**

```bash
-d '{"name":"Ama","age":"old"}'
```

`400 … invalid type: string "old", expected u16`

</column>

<column>

**No content-type header**

```bash
curl -X POST … -d '{"name":"Ama","age":25}'
```

`400 Content type error`. `Json` insists on `application/json`.

</column>

</columns>

In every case your handler **never runs**. Bad input is rejected before your code sees it, so inside the handler `body.age` is always a real `u16`.

## Step 5: wire them into the bank

Replace the stubs from lesson 02:

```rust
App::new().service(
    web::scope("/accounts")
        .service(list_accounts)      // GET  /accounts       (still a stub)
        .service(create_account)     // POST /accounts       Json<NewAccount>
        .service(get_account),       // GET  /accounts/{id}  Path<u32>
)
```

Nothing is **stored** yet: create an account, ask for it, and the server has already forgotten it. Lesson 04 fixes that.

<callout background="blue" icon="🏋️">

**Exercise 3.1: deposit request**

Add `POST /accounts/{id}/deposit` that takes **both** `web::Path<u32>` and `web::Json<Amount>`, where `Amount { amount: u64 }`, and replies `"would deposit 100 into account 7"`. Then try a negative amount (`-d '{"amount":-5}'`) and explain the error you get.

</callout>

<toggle>

Answer: the negative amount

`400 Json deserialize error: invalid value: integer '-5', expected u64`. A `u64` can't be negative, so serde rejects it. Choosing the right type removes a whole class of checks.

</toggle>

## Checklist

- [ ] I can say where `Path`, `Query` and `Json` each read from
- [ ] I used a tuple `Path` for two URL values
- [ ] I made a query field optional with `Option`
- [ ] I've seen the 400 for a bad JSON body and the 400 for a missing content-type

---

[Actix Web hub](page:4oh6zeq50g) · Next → [Actix 04 App State](page:o0yhl6fhvd)
