---
id: 8uyl4dfwco
title: "Actix 05 Errors and Status Codes"
sort: 4000
version: 1
---

<callout background="green" icon="🚦">

**Return `Result` from handlers and let Actix turn errors into proper HTTP responses.** You'll build a `BankError` enum, teach it which status code each error means, and use `?` exactly as you did in your Rust bank project.

</callout>

**Level:** 🟡 Intermediate · **Needs:** [Actix 04 App State](page:o0yhl6fhvd), [05 Collections, Strings and Errors](page:ubfingfdn1)

## Where we are

After lesson 04, `get_account` handles "not found" by hand:

```rust
match bank.accounts.get(&id) {
    Some(account) => HttpResponse::Ok().json(account),
    None => HttpResponse::NotFound().body("no such account"),
}
```

That's fine once, but deposit and withdraw each have several ways to fail, and writing a `match` for each one gets messy. In plain Rust you'd return `Result` and use `?`, and Actix lets you do exactly that.

## Status codes you'll use

| Code | Name | When |
| --- | --- | --- |
| `200` | OK | it worked, and here's the data |
| `201` | Created | a new thing was made (`POST /accounts`) |
| `400` | Bad Request | the client sent something invalid (age 12, amount 0) |
| `404` | Not Found | no such account |
| `409` | Conflict | valid request, but the current state forbids it (not enough money) |
| `500` | Internal Server Error | a bug on **your** side |

## Step 1: the error enum

This is the same idea as `ErrorTypes` in your Rust project:

```rust
#[derive(Debug)]
enum BankError {
    NotFound,
    InsufficientFunds,
    InvalidData(String),     // carries a message: "must be 18 or older"
}
```

## Step 2: implement `Display`

Actix needs to be able to turn the error into text, and `Display` is the trait that gives every type a `to_string()`:

```rust
use std::fmt;

impl fmt::Display for BankError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            BankError::NotFound => write!(f, "account not found"),
            BankError::InsufficientFunds => write!(f, "insufficient funds"),
            BankError::InvalidData(msg) => write!(f, "invalid data: {msg}"),
        }
    }
}
```

## Step 3: implement `ResponseError`

`ResponseError` is Actix's trait for "this error knows how to become an HTTP response". You give it two methods:

```rust
use actix_web::{http::StatusCode, HttpResponse, ResponseError};
use serde_json::json;

impl ResponseError for BankError {
    // which status code?
    fn status_code(&self) -> StatusCode {
        match self {
            BankError::NotFound => StatusCode::NOT_FOUND,
            BankError::InsufficientFunds => StatusCode::CONFLICT,
            BankError::InvalidData(_) => StatusCode::BAD_REQUEST,
        }
    }

    // what body?  {"error": "account not found"}
    fn error_response(&self) -> HttpResponse {
        HttpResponse::build(self.status_code()).json(json!({ "error": self.to_string() }))
    }
}
```

`json!` (from `serde_json`) builds JSON inline without a struct. Run `cargo add serde_json` if you don't have it yet.

<toggle>

Error: `BankError` doesn't implement `std::fmt::Display`

`ResponseError` requires `Debug` **and** `Display`. You skipped Step 2, or forgot `#[derive(Debug)]`.

</toggle>

## Step 4: handlers return `Result`

Now a handler can return `Result<HttpResponse, BankError>`. When it returns `Err(e)`, Actix calls your `error_response` for you.

```rust
#[get("/{id}")]
async fn get_account(
    state: web::Data<AppState>,
    path: web::Path<u32>,
) -> Result<HttpResponse, BankError> {
    let id = path.into_inner();
    let bank = state.bank.lock().unwrap();
    let account = bank.accounts.get(&id).ok_or(BankError::NotFound)?;
    Ok(HttpResponse::Ok().json(account))
}
```

The line doing the work, `bank.accounts.get(&id).ok_or(BankError::NotFound)?`, reads in three steps:

1. **`.get(&id)`** gives `Option<&Account>`: `Some(account)` or `None`.
1. **`.ok_or(BankError::NotFound)`** turns it into a `Result`: `Ok(account)` or `Err(NotFound)`.
1. **`?`** means: if it's `Err`, return it from the handler right now; if it's `Ok`, unwrap it.

It's the same trick as `find_user` in your Rust bank project.

## Step 5: deposit and withdraw

```rust
#[derive(Deserialize)]
struct Amount {
    amount: u64,
}

#[post("/{id}/deposit")]
async fn deposit(
    state: web::Data<AppState>,
    path: web::Path<u32>,
    body: web::Json<Amount>,
) -> Result<HttpResponse, BankError> {
    if body.amount == 0 {
        return Err(BankError::InvalidData("amount must be above 0".into()));
    }
    let mut bank = state.bank.lock().unwrap();
    let account = bank.accounts.get_mut(&path.into_inner()).ok_or(BankError::NotFound)?;
    account.balance += body.amount;
    Ok(HttpResponse::Ok().json(account.clone()))
}

#[post("/{id}/withdraw")]
async fn withdraw(
    state: web::Data<AppState>,
    path: web::Path<u32>,
    body: web::Json<Amount>,
) -> Result<HttpResponse, BankError> {
    let mut bank = state.bank.lock().unwrap();
    let account = bank.accounts.get_mut(&path.into_inner()).ok_or(BankError::NotFound)?;
    if account.balance < body.amount {
        return Err(BankError::InsufficientFunds);
    }
    account.balance -= body.amount;
    Ok(HttpResponse::Ok().json(account.clone()))
}
```

Register both with `.service(deposit).service(withdraw)` inside your `/accounts` scope. Also change `create_account` to return `Err(BankError::InvalidData("must be 18 or older".into()))` for under-18s, and `HttpResponse::Created()` (201) on success.

<toggle>

Why `get_mut` and `account.clone()`?

`get_mut` gives `&mut Account`, so you can change the balance in place inside the map, the same as `&mut self` in `deposit` in your Rust project. `.json(...)` needs a value it can serialize, and cloning it is the simplest way while the lock is still held.

</toggle>

## Step 6: try every path

```bash
curl -X POST localhost:8080/accounts -H 'content-type: application/json' -d '{"name":"Ama","age":25}'
curl -X POST localhost:8080/accounts/1/deposit  -H 'content-type: application/json' -d '{"amount":100}'
# {"id":1,"name":"Ama","balance":100}

curl -i -X POST localhost:8080/accounts/1/withdraw -H 'content-type: application/json' -d '{"amount":999}'
# HTTP/1.1 409 Conflict
# {"error":"insufficient funds"}

curl -i localhost:8080/accounts/42
# HTTP/1.1 404 Not Found
# {"error":"account not found"}

curl -i -X POST localhost:8080/accounts/1/deposit -H 'content-type: application/json' -d '{"amount":0}'
# HTTP/1.1 400 Bad Request
# {"error":"invalid data: amount must be above 0"}
```

`-i` prints the status line and headers, so you can see the code.

<callout background="blue" icon="🏋️">

**Exercise 5.1: overflow**

`u64` has a maximum. Deposit `18446744073709551615` and then `1`. In debug builds, `+=` **panics** and the client gets an empty reply. Use `account.balance.checked_add(body.amount)`, which returns `None` on overflow, with `.ok_or(...)?` to return a `400` instead.

</callout>

## Checklist

- [ ] I made an error enum with `Debug`, `Display` and `ResponseError`
- [ ] My handlers return `Result<HttpResponse, BankError>`
- [ ] I can explain `.ok_or(…)?` in three steps
- [ ] I saw 201, 400, 404 and 409 with `curl -i`

---

[Actix Web hub](page:4oh6zeq50g) · Next → [Actix 06 Project Layout with Modules](page:2o447fwgl4)
