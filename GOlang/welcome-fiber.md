---
id: 3pk38363zv
title: "👋 Welcome | Fiber"
sort: 2000
version: 1
source: https://docs.gofiber.io/
clipped: 2026-10-28 09:12
tags: [docs]
---

Welcome to Fiber's online API documentation, complete with examples to help you start building web applications right away!

**Fiber** is an [Express](https://github.com/expressjs/express)-inspired **web framework** built on top of [Fasthttp](https://github.com/valyala/fasthttp), the **fastest** HTTP engine for [Go](https://go.dev/doc/). It is designed to facilitate rapid development with **zero memory allocations** and a strong focus on **performance**. Fiber also ships batteries included: built-in middleware, officially maintained integrations, storage drivers, and template engines cover most production needs (see [Explore the Ecosystem](#explore-the-ecosystem) below).

These docs cover **Fiber v3**.

tip

Coming from Fiber v2? See [What's New in v3](https://docs.gofiber.io/whats_new) for the migration guide and the CLI migration tool.

## Installation[​](#installation)

First, [download](https://go.dev/dl/) and install Go. Version `1.26` or higher is required.

Install Fiber using the [`go get`](https://pkg.go.dev/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them) command:

```
go get github.com/gofiber/fiber/v3
```

## Hello, World[​](#hello-world)

Create a file named `server.go` with the simplest **Fiber** application you can write:

```
package main
import (
    "log"
    "github.com/gofiber/fiber/v3"
)
func main() {
    app := fiber.New()
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })
    log.Fatal(app.Listen(":3000"))
}
```

Run it:

```
go run server.go
```

Browse to `http://localhost:3000` and you should see `Hello, World!` displayed on the page.

Three calls carry the whole program:

- `fiber.New()` creates the app, the central object that holds routes, middleware, and configuration
- `app.Get("/", ...)` registers a handler: a function that receives the request context `c fiber.Ctx` and returns an `error`
- `app.Listen(":3000")` starts the server

## Build Your First App, Step by Step[​](#build-your-first-app-step-by-step)

The walkthrough below grows the Hello, World program into a small app with route parameters, static files, middleware, a JSON endpoint, and error handling. Step through it and fire the example requests to see exactly how the app responds at every stage:

1. Hello, World
1. Add a route parameter
1. Serve static files
1. Write your own middleware
1. Return JSON
1. Handle errors

Step 1 of 6: Hello, World

fiber.New() creates the app, app.Get registers a handler for GET requests to /, and app.Listen starts the server on port 3000. Every handler receives a fiber.Ctx and returns an error. Requests that match no route get Fiber's default 404 response.

main.go

```
package main
import (
    "log"
    "github.com/gofiber/fiber/v3"
)
func main() {
    app := fiber.New()
    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })
    log.Fatal(app.Listen(":3000"))
}
```

Try a request:

```
$ curl -i http://localhost:3000/
HTTP/1.1 200 OK
 text/plain; charset=utf-8

Hello, World!
```

Responses are simulated and show only the headers each step is about.

## What Can You Build?[​](#what-can-you-build)

Fiber covers the everyday shapes of web backends out of the box. Pick a use case and read the code; every tab is just complete enough to recognize your project in it:

Route parameters with `constraints`, request binding, and `c.JSON` keep classic REST services short. [Routing guide →](https://docs.gofiber.io/guide/routing/)

```
app.Get("/api/users/:id<int>", func(c fiber.Ctx) error {
    return c.JSON(fiber.Map{"id": c.Params("id"), "name": "Alice"})
})
```

Want complete, runnable projects instead? The [Recipes collection](https://docs.gofiber.io/recipes/) has working examples for Docker, GORM, JWT auth, clean architecture, and much more.

## Basic Routing[​](#basic-routing)

Routing determines how an application responds to a client request at a particular endpoint, a combination of path and HTTP request method (`GET`, `PUT`, `POST`, etc.).

Route definitions follow the structure below:

```
// Function signature
func (app *App) Get(path string, handler any, handlers ...any) Router
```

- `app` is an instance of **Fiber**
- `Get` is an [HTTP request method](https://docs.gofiber.io/api/app#route-handlers); `Post`, `Put`, `Delete`, and the other methods work the same way
- `path` is a virtual path on the server
- `handler` is a function executed when the route is matched; the canonical form is `func(fiber.Ctx) error`, and a route can register multiple handlers

For an interactive breakdown of every part of a route definition, see the [anatomy of a route](https://docs.gofiber.io/guide/routing#anatomy-of-a-route).

A simple route and a route with a parameter:

```
// Respond with "Hello, World!" on root path "/"
app.Get("/", func(c fiber.Ctx) error {
    return c.SendString("Hello, World!")
})
```

```
// GET http://localhost:3000/hello%20world
app.Get("/:value", func(c fiber.Ctx) error {
    return c.SendString("value: " + c.Params("value"))
    // => Response: "value: hello world"
})
```

See the [routing guide](https://docs.gofiber.io/guide/routing) for optional parameters, wildcards, constraints, route groups, and the full list of supported handler types. Or skip ahead and fire requests at your own route table in the interactive [Route Matcher](https://docs.gofiber.io/extra/route-matcher).

## Static Files[​](#static-files)

To serve static files such as **images**, **CSS**, and **JavaScript** files, register the [static middleware](https://docs.gofiber.io/middleware/static):

```
import "github.com/gofiber/fiber/v3/middleware/static"
app.Use("/", static.New("./public"))
```

Files in the `./public` directory are now reachable in the browser, for example at `http://localhost:3000/css/style.css`.

## Using Middleware[​](#using-middleware)

Middleware runs before or after your handlers and takes care of cross-cutting concerns. Registering one is a single `app.Use` call; here is the [logger](https://docs.gofiber.io/middleware/logger) middleware printing every request:

```
import "github.com/gofiber/fiber/v3/middleware/logger"
app.Use(logger.New())
```

Middleware for logging, CORS, rate limiting, sessions, compression, panic [recovery](https://docs.gofiber.io/middleware/recover), and much more ships with Fiber itself; the ecosystem section below shows where to find it all.

## Zero Allocation[​](#zero-allocation)

caution

Fiber is optimized for **high performance**, so values returned from **fiber.Ctx** are **not** immutable by default and **will** be reused across requests. Use context values only within the handler, and do not keep any references after the handler returns.

If you need to persist a context value beyond the handler, make a copy of its **underlying buffer** using the [copy](https://pkg.go.dev/builtin/#copy) builtin:

```
func handler(c fiber.Ctx) error {
    // Variable is only valid within this handler
    result := c.Params("foo")
    // Make a copy
    buffer := make([]byte, len(result))
    copy(buffer, result)
    resultCopy := string(buffer)
    // Variable is now valid indefinitely
    // ...
}
```

Alternatively, you can enable the `Immutable` setting. This makes all values returned from the context immutable, allowing you to persist them anywhere, at the cost of some performance:

```
app := fiber.New(fiber.Config{
    Immutable: true,
})
```

For details, see [Immutable](https://docs.gofiber.io/api/fiber#immutable) in the configuration reference and the [GetString and GetBytes](https://docs.gofiber.io/api/app#getstring) helpers.

## Explore the Ecosystem[​](#explore-the-ecosystem)

Fiber is more than the core module. When your application grows, these officially maintained building blocks are one import away:

- **Built-in middleware**: 30+ middleware for logging, CORS, security headers, caching, rate limiting, and more live in the core module; browse the [middleware overview](https://docs.gofiber.io/category/-middleware).
- [**Contrib packages**](https://docs.gofiber.io/contrib/): officially maintained integrations with external dependencies, such as JWT, WebSocket, OpenTelemetry, Casbin, and structured logging adapters.
- [**Storage drivers**](https://docs.gofiber.io/storage/): a growing list of backends (Redis, Postgres, MongoDB, S3, and more) behind one interface, ready to plug into the session, limiter, cache, and idempotency middleware.
- [**Template engines**](https://docs.gofiber.io/template/): server-side rendering through the Views interface, with engines like html, django, handlebars, and pug.
- [**HTTP client**](https://docs.gofiber.io/client/rest): a built-in client, also built on Fasthttp, for calling other services with the same performance philosophy.
- [**Recipes**](https://docs.gofiber.io/recipes/): runnable example projects (Docker, GORM, JWT auth, clean architecture, and more) to copy a working starting point from.

For a visual map of how these repositories plug into each other, see the [Ecosystem overview](https://docs.gofiber.io/ecosystem).

## Next Steps[​](#next-steps)

Work through these in order and you will have touched everything a typical production service needs:

1. [Routing](https://docs.gofiber.io/guide/routing): parameters, wildcards, and constraints; try them live in the [Route Matcher](https://docs.gofiber.io/extra/route-matcher)
1. [Grouping](https://docs.gofiber.io/guide/grouping) and the [middleware catalog](https://docs.gofiber.io/category/-middleware): structure the app and its cross-cutting concerns
1. [Error handling](https://docs.gofiber.io/guide/error-handling): central error handlers and status codes
1. [Request binding](https://docs.gofiber.io/api/bind) and [validation](https://docs.gofiber.io/guide/validation): map request data onto structs safely
1. [Minimal CRUD and health check](https://docs.gofiber.io/guide/crud-and-health): a runnable starter with probes, for learning and CI smoke tests
1. [Templates](https://docs.gofiber.io/guide/templates): render views with your favorite template engine
1. [HTTP client](https://docs.gofiber.io/client/rest): call other services with the same performance philosophy
1. [Performance](https://docs.gofiber.io/guide/faster-fiber): custom JSON encoders and the tricks behind the benchmarks
1. [Testing](https://docs.gofiber.io/api/app#test): test handlers without a running server using `app.Test`

The [configuration reference](https://docs.gofiber.io/api/fiber) lists every option accepted by `fiber.New`, and the [learning resources](https://docs.gofiber.io/extra/learning-resources) page collects tutorials and hands-on challenges.

## Community and Help[​](#community-and-help)

Stuck or have questions? Join the [Discord](https://gofiber.io/discord) server or check the [FAQ](https://docs.gofiber.io/extra/faq). Fiber is developed in the open on [GitHub](https://github.com/gofiber/fiber); issues, discussions, and contributions are welcome.
