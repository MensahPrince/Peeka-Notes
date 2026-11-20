---
id: phgfxrea1z10a
title: "GoFiber"
cover: "_assets/f0156048e8664424adb085df213d38a4.jpg"
cover_position: 50
sort: 1000
version: 1
---

##

## **Anatomy of a route**

A route ties together an HTTP method, a path, and one or more handlers. Hover or click any colored part to jump to the section that explains it:

```go
app.Get("/users/:id", func(c fiber.Ctx) error {
    return c.SendString("GET /users")
})
```

`Get` is the [routing method](https://docs.gofiber.io/guide/routing/#route-handlers), `"/users/:id"` is the [route path](https://docs.gofiber.io/guide/routing/#paths) (the resource, in REST terms) with `:id` a [route parameter](https://docs.gofiber.io/guide/routing/#parameters), and `func(c fiber.Ctx) error` is the [handler](https://docs.gofiber.io/guide/routing/#handler-types) (or [middleware](https://docs.gofiber.io/guide/routing/#middleware)) run when the route matches.

Want to see matching happen? Fire requests at your own route table in the interactive [Route Matcher](https://docs.gofiber.io/extra/route-matcher).

## **Route Handlers**

Registers a route bound to a specific [HTTP method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods). The canonical handler is `func(fiber.Ctx) error`; Fiber also accepts `func(fiber.Ctx)` and runs it as if it returned `nil`.

Signatures

```go
// HTTP methods
func (app *App) Get(path string, handler any, handlers ...any) Router
func (app *App) Head(path string, handler any, handlers ...any) Router
func (app *App) Post(path string, handler any, handlers ...any) Router
func (app *App) Put(path string, handler any, handlers ...any) Router
func (app *App) Delete(path string, handler any, handlers ...any) Router
func (app *App) Connect(path string, handler any, handlers ...any) Router
func (app *App) Options(path string, handler any, handlers ...any) Router
func (app *App) Trace(path string, handler any, handlers ...any) Router
func (app *App) Patch(path string, handler any, handlers ...any) Router
func (app *App) Query(path string, handler any, handlers ...any) Router

// Add registers the same handlers on multiple methods at once.
// The handlers run in order, starting with `handler` and then the variadic `handlers`.
func (app *App) Add(methods []string, path string, handler any, handlers ...any) Router

// All registers the route on every HTTP method at the EXACT path
// (unlike Use, which is prefix-matched).
func (app *App) All(path string, handler any, handlers ...any) Router
```

Examples

```go
// Simple GET handler
app.Get("/api/list", func(c fiber.Ctx) error {
    return c.SendString("I'm a GET request!")
})

// Simple POST handler
app.Post("/api/register", func(c fiber.Ctx) error {
    return c.SendString("I'm a POST request!")
})
```

Here is a complete, runnable app for context:

A minimal Fiber app

```go
package main

import "github.com/gofiber/fiber/v3"

func main() {
    app := fiber.New()

    app.Get("/", func(c fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })

    app.Listen(":3000")
}
```

In the shorter examples throughout this guide, `app` is the `*fiber.App` returned by `fiber.New()`, and `handler`/`middleware` stand in for any `func(c fiber.Ctx) error`. Snippets that call `fmt.Println` or `fmt.Fprintf` also need `import "fmt"`.

Beyond the native `func(fiber.Ctx)` forms, Fiber also adapts Express-style, `net/http`, and `fasthttp` handlers. See [Handler types](https://docs.gofiber.io/guide/routing/#handler-types) at the end of this guide for the full list of supported shapes.

## **Get vs Use vs All**

`Get` (and the other method helpers like `Post` and `Put`) match a **single HTTP method** at an **exact path**. `All` matches an **exact path** across **every** HTTP method. `Use` registers **middleware** that matches by **prefix** and runs in **declaration order**, calling [`c.Next()`](https://docs.gofiber.io/api/ctx#next) to continue the chain.

- **Get (one method)**
- **All (every method)**
- **Use (prefix middleware)**
- **Ordered chain**
- **Multiple handlers in one call**

```go
app.Get("/users", func(c fiber.Ctx) error {
    return c.SendString("GET /users")
})

// GET    /users      -> "GET /users"
// POST   /users      -> 405 Method Not Allowed
// GET    /users/42   -> 404 Not Found  (exact match only)
```

| **Helper** | **Methods matched** | **Path matching** | **Typical use** |
| --- | --- | --- | --- |
| `Get`/`Post`/… | one | exact | a specific endpoint |
| `All` | every method | exact | one path, any verb |
| `Use` | every method | prefix (slash boundary); all paths if none given | middleware, mounting sub-apps |

A path that exists only for a different method returns **405 Method Not Allowed**; a path that matches no route at all (including one rejected by a [constraint](https://docs.gofiber.io/guide/routing/#constraints)) returns **404 Not Found**.

## **Paths**

A route path paired with an HTTP method defines an endpoint. It can be a plain **string** or a **pattern**.

```go
// This route path will match requests to the root route, "/":
app.Get("/", func(c fiber.Ctx) error {
    return c.SendString("root")
})

// This route path will match requests to "/about":
app.Get("/about", func(c fiber.Ctx) error {
    return c.SendString("about")
})

// This route path will match requests to "/random.txt":
app.Get("/random.txt", func(c fiber.Ctx) error {
    return c.SendString("random.txt")
})
```

The order in which you declare routes matters: like Express.js, routes are matched in registration order (first match wins), so declare more specific paths before those that contain parameters. Note that method helpers such as `Get` match the exact path only.

info

Place routes with variable parameters after fixed paths to avoid unintended matches.

## **Parameters**

Route parameters are dynamic segments in a path, either named or unnamed, used to capture values from the URL. Retrieve them with the [Params](https://docs.gofiber.io/api/ctx#params) function using the parameter name or, for unnamed parameters, the wildcard (`*`) or plus (`+`) symbol with an index.

The characters `:`, `+`, and `*` introduce parameters. Append `?` to a named segment to make it optional. `+` is a greedy, required wildcard (it must match at least one character); `*` is a greedy, optional wildcard (it can match nothing).

- **Named, optional, greedy**
- **Literal separators**
- **Escaped characters**
- **Multiple params per segment**

```go
// Named parameters
app.Get("/user/:name/books/:title", func(c fiber.Ctx) error {
    fmt.Fprintf(c, "%s\n", c.Params("name"))
    fmt.Fprintf(c, "%s\n", c.Params("title"))
    return nil
})

// Plus - greedy, required (matches at least one character)
app.Get("/user/+", func(c fiber.Ctx) error {
    return c.SendString(c.Params("+"))
})

// Optional named parameter
app.Get("/user/:name?", func(c fiber.Ctx) error {
    return c.SendString(c.Params("name"))
})

// Wildcard - greedy, optional (may match nothing)
app.Get("/user/*", func(c fiber.Ctx) error {
    return c.SendString(c.Params("*"))
})
```

When a route has several wildcard (`*`) or plus (`+`) segments, retrieve them positionally with a 1-based index matching the symbol: `c.Params("*1")` and `c.Params("*2")` for wildcards, `c.Params("+1")` and `c.Params("+2")` for plus segments. A single wildcard or plus is just `c.Params("*")` or `c.Params("+")`.

Fiber's routing is inspired by Express but intentionally omits regex route patterns due to their performance cost. To validate a parameter against a regular expression, use the [`regex()` constraint](https://docs.gofiber.io/guide/routing/#constraints) described below.

### **Constraints**

Route constraints execute when a match has occurred to the incoming URL and the URL path is tokenized into route values by parameters. The feature was introduced in `v2.37.0` and inspired by [.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing?view=aspnetcore-6.0#route-constraints).

caution

Constraints are matching rules, not input validation: if a value fails a constraint, the route simply does not match and Fiber returns **404 Not Found**.

| **Constraint** | **Example** | **Example matches** |
| --- | --- | --- |
| int | `:id<int>` | 123456789, -123456789 |
| bool | `:active<bool>` | true,false |
| guid | `:id<guid>` | CD2C1638-1638-72D5-1638-DEADBEEF1638 |
| float | `:weight<float>` | 1\.234, -1001.01e8, 3.14 |
| minLen(value) | `:username<minLen(4)>` | Test (must be at least 4 characters) |
| maxLen(value) | `:filename<maxLen(8)>` | MyFile (must be no more than 8 characters) |
| len(length) | `:filename<len(12)>` | somefile.txt (exactly 12 characters) |
| min(value) | `:age<min(18)>` | 19 (Integer value must be at least 18) |
| max(value) | `:age<max(120)>` | 91 (Integer value must be no more than 120) |
| range(min,max) | `:age<range(18,120)>` | 91 (Integer value must be at least 18 but no more than 120) |
| alpha | `:name<alpha>` | Rick (String must consist of one or more alphabetical characters, a-z and case-insensitive) |
| datetime | `:dob<datetime(2006\\-01\\-02)>` | 2005-11-01 |
| regex(expression) | `:date<regex(\d{4}-\d{2}-\d{2})>` | 2022-08-27 (Must match regular expression) |

#### **Examples**

- **Single Constraint**
- **Multiple Constraints**
- **Regex Constraint**

```go
app.Get("/:test<min(5)>", func(c fiber.Ctx) error {
    return c.SendString(c.Params("test"))
})

// curl -X GET http://localhost:3000/12
// 12

// curl -X GET http://localhost:3000/1
// Not Found
```

caution

When using the datetime constraint, prefix routing characters (`*`, `+`, `?`, `:`, `/`, `<`, `>`, `;`, `(`, `)`) with `\\` to avoid misparsing.

#### **Optional Parameter Example**

You can impose constraints on optional parameters as well.

```go
app.Get("/:test<int>?", func(c fiber.Ctx) error {
  return c.SendString(c.Params("test"))
})
// curl -X GET http://localhost:3000/42
// 42
// curl -X GET http://localhost:3000/
//
// curl -X GET http://localhost:3000/7.0
// Not Found
```

#### **Custom Constraint**

Custom constraints can be added to Fiber using the `app.RegisterCustomConstraint` method. Your constraints have to be compatible with the `CustomConstraint` interface.

caution

Attention, custom constraints can now override built-in constraints. If a custom constraint has the same name as a built-in constraint, the custom constraint will be used instead. This allows for more flexibility in defining route parameter constraints.

Add external constraints when you need stricter rules, such as verifying that a parameter is a valid ULID.

```go
// CustomConstraint is an interface for custom constraints
type CustomConstraint interface {
    // Name returns the name of the constraint.
    // This name is used in the constraint matching.
    Name() string

    // Execute executes the constraint.
    // It returns true if the constraint is matched and right.
    // param is the parameter value to check.
    // args are the constraint arguments.
    Execute(param string, args ...string) bool
}
```

You can check the example below:

```go
type UlidConstraint struct {
    fiber.CustomConstraint
}

func (*UlidConstraint) Name() string {
    return "ulid"
}

func (*UlidConstraint) Execute(param string, args ...string) bool {
    _, err := ulid.Parse(param)
    return err == nil
}

func main() {
    app := fiber.New()
    app.RegisterCustomConstraint(&UlidConstraint{})

    app.Get("/login/:id<ulid>", func(c fiber.Ctx) error {
        return c.SendString("...")
    })

    app.Listen(":3000")

    // /login/01HK7H9ZE5BFMK348CPYP14S0Z -> 200
    // /login/12345 -> 404
}
```

#### **ConstraintHandler Interface**

In addition to the `CustomConstraint` interface, Fiber v3 provides a more powerful `ConstraintHandler` interface that supports precomputation at route registration time. All built-in constraints implement this interface.

```go
// ConstraintHandler is the interface that all constraints must implement.
type ConstraintHandler interface {
    // Name returns the constraint identifier used in route patterns.
    Name() string

    // Execute validates a request parameter value against the constraint.
    // data contains the pre-typed constraint data produced by Analyze().
    Execute(param string, data []any) bool
}
```

Optionally, a constraint can implement `ConstraintAnalyzer` to preprocess data at registration time, avoiding repeated parsing on every request:

```go
// ConstraintAnalyzer is an optional interface for registration-time precomputation.
type ConstraintAnalyzer interface {
    // Analyze preprocesses constraint data at route registration time.
    // Returns pre-typed values that will be stored in Constraint.Data.
    Analyze(args []string) ([]any, error)
}
```

note

Existing `CustomConstraint` implementations continue to work unchanged. They are automatically wrapped to satisfy `ConstraintHandler`. Custom constraints that also implement `ConstraintAnalyzer` will have their `Analyze` method called at registration time.

Try it live

Test route patterns, registration order, and constraints against real requests in the interactive [<u>Route Matcher</u>](https://docs.gofiber.io/extra/route-matcher) tool.

## **Middleware**

Functions that are designed to make changes to the request or response are called **middleware functions**. [`c.Next()`](https://docs.gofiber.io/api/ctx#next) passes control to the next handler in the matched chain (middleware or route handler); if a handler returns without calling it, the remaining handlers are skipped.

Example of a middleware function

```go
app.Use(func(c fiber.Ctx) error {
    // Set a custom header on all responses:
    c.Set("X-Custom-Header", "Hello, World")

    // Go to next middleware:
    return c.Next()
})

app.Get("/", func(c fiber.Ctx) error {
    return c.SendString("Hello, World!")
})
```

Once a route matches, its handlers run as one chain in registration order; each handler decides with `c.Next()` whether the rest of the chain runs. Step through the chain yourself, including what happens when a middleware short-circuits instead of calling `c.Next()`:

GET /admin with header "Authorization: Bearer ..."

```go
app.Use(func(c fiber.Ctx) error { // logger
    start := time.Now()
    err := c.Next() // hand over to auth
    log.Printf("%s %s took %v", c.Method(), c.Path(), time.Since(start))
    return err
})

app.Use(func(c fiber.Ctx) error { // auth
    if c.Get("Authorization") == "" {
        return fiber.ErrUnauthorized // stop here, no c.Next()
    }
    return c.Next() // hand over to the handler
})

app.Get("/admin", func(c fiber.Ctx) error { // handler
    return c.SendString("Admin dashboard")
})
```

**client**

**logger**

**auth**

**GET /admin**

request pathresponse path

The request arrives with a valid Authorization header and enters the chain.

1 / 7

See [Get vs Use vs All](https://docs.gofiber.io/guide/routing/#get-vs-use-vs-all) for how `Use` prefix matching differs from exact route matching, and how multiple handlers run in order.

### **Use**

`Use` mounts middleware on a **prefix** (or **mount**) path: it runs for every request whose path begins with that prefix, on any HTTP method. Prefixes require either an exact match or a slash boundary, so `/john` matches `/john` and `/john/doe` but not `/johnnnnn`. Parameter tokens like `:name`, `:name?`, `*`, and `+` are still expanded before the boundary check runs. Called without a path, `Use` matches every request.

Signature

```go
func (app *App) Use(args ...any) Router

// Fiber inspects args to support these common usage patterns:
// - app.Use(handler, handlers ...any)
// - app.Use(path string, handler, handlers ...any)
// - app.Use(paths []string, handler, handlers ...any)
// - app.Use(path string, subApp *App)
```

Each handler argument can independently be a Fiber handler (with or without an `error` return), an Express-style callback, a `net/http` handler, or any other supported shape including fasthttp callbacks that return errors.

Examples

```go
// Match any request
app.Use(func(c fiber.Ctx) error {
    return c.Next()
})

// Match request starting with /api
app.Use("/api", func(c fiber.Ctx) error {
    return c.Next()
})

// Match requests starting with /api or /home (multiple-prefix support)
app.Use([]string{"/api", "/home"}, func(c fiber.Ctx) error {
    return c.Next()
})

// Attach multiple handlers (they run in order; each must call c.Next() to continue)
app.Use("/api", func(c fiber.Ctx) error {
    c.Set("X-Custom-Header", "value")
    return c.Next()
}, func(c fiber.Ctx) error {
    return c.Next()
})

// Mount a sub-app
app.Use("/api", api)
```

### **Adding or removing routes at runtime**

caution

Defining all routes before the app starts is strongly recommended. You can still change them at runtime with [<u>`RebuildTree`</u>](https://docs.gofiber.io/api/app#rebuildtree), [<u>`RemoveRoute`</u>](https://docs.gofiber.io/api/app#removeroute), [<u>`RemoveRouteByName`</u>](https://docs.gofiber.io/api/app#removeroutebyname), and [<u>`RemoveRouteFunc`</u>](https://docs.gofiber.io/api/app#removeroutefunc), but these operations are not thread-safe and are performance-intensive, so use them sparingly and only in development.

## **Grouping**

If you have many endpoints, you can organize your routes using `Group`.

```go
func main() {
    app := fiber.New()

    api := app.Group("/api", middleware) // /api

    v1 := api.Group("/v1", middleware)   // /api/v1
    v1.Get("/list", handler)             // /api/v1/list
    v1.Get("/user", handler)             // /api/v1/user

    v2 := api.Group("/v2", middleware)   // /api/v2
    v2.Get("/list", handler)             // /api/v2/list
    v2.Get("/user", handler)             // /api/v2/user

    log.Fatal(app.Listen(":3000"))
}
```

More information about this in our [Grouping Guide](https://docs.gofiber.io/guide/grouping).

### **Route**

[`Route`](https://docs.gofiber.io/api/app#route) is shorthand for [`Group`](https://docs.gofiber.io/guide/routing/#grouping): it scopes a set of routes under a common prefix declared inside a single callback, with an optional name prefix.

```go
app.Route("/api/v1", func(r fiber.Router) {
    r.Get("/users", handler).Name("users")   // /api/v1/users  (name: v1.users)
    r.Post("/users", handler).Name("create") // /api/v1/users  (name: v1.create)
}, "v1.")
```

### **RouteChain**

When several HTTP methods share the **same path**, [`RouteChain`](https://docs.gofiber.io/api/app#routechain) lets you declare the path once and chain the verb handlers. An `All` in the chain runs before the verb handlers on that path, acting as route-specific middleware.

```go
app.RouteChain("/events").
    All(func(c fiber.Ctx) error { return c.Next() }). // route-local middleware
    Get(func(c fiber.Ctx) error { return c.SendString("GET /events") }).
    Post(func(c fiber.Ctx) error { return c.SendString("POST /events") })
```

note

Within a chain, `All` registers prefix-matched middleware (like [<u>`Use`</u>](https://docs.gofiber.io/guide/routing/#use)), not the exact-path `App.All`, so it also runs for sub-paths of the chain path.

Pick the helper that fits: a single endpoint uses `Get`/`Post`/…; a fixed set of methods on one path uses [`Add`](https://docs.gofiber.io/guide/routing/#route-handlers); one path with many methods (fluently) uses `RouteChain`; many paths under a shared prefix use [`Group`](https://docs.gofiber.io/guide/routing/#grouping) or `Route`.

## **Automatic HEAD routes**

Fiber automatically registers a `HEAD` route for every `GET` route you add. The generated handler chain mirrors the `GET` chain, so `HEAD` requests reuse middleware, status codes, and headers while the response body is suppressed.

GET handlers automatically expose HEAD

```go
app := fiber.New()

app.Get("/users/:id", func(c fiber.Ctx) error {
    c.Set("X-User", c.Params("id"))
    return c.SendStatus(fiber.StatusOK)
})

// HEAD /users/:id now returns the same headers and status without a body.
```

You can still register dedicated `HEAD` handlers, even with auto-registration enabled, and Fiber replaces the generated route so your implementation wins:

Override the generated HEAD handler

```go
app.Head("/users/:id", func(c fiber.Ctx) error {
    return c.SendStatus(fiber.StatusNoContent)
})
```

To opt out globally, start the app with `DisableHeadAutoRegister`:

Disable automatic HEAD registration

```go
handler := func(c fiber.Ctx) error {
    c.Set("X-User", c.Params("id"))
    return c.SendStatus(fiber.StatusOK)
}

app := fiber.New(fiber.Config{DisableHeadAutoRegister: true})
app.Get("/users/:id", handler) // HEAD /users/:id now returns 405 unless you add it manually.
```

Auto-generated `HEAD` routes participate in every router scope, including `Group` hierarchies, mounted sub-apps, parameterized and wildcard paths, and static file helpers. They also appear in route listings such as `app.Stack()` so tooling sees both the `GET` and `HEAD` entries.

## **Handler types**

Fiber's adapter converts a variety of handler shapes into native `func(fiber.Ctx) error` callbacks. The 17 supported shapes are grouped below; any other signature is rejected when the route is registered. This lets you mix Fiber-style handlers with Express-style callbacks and even reuse `net/http` or `fasthttp` functions.

### **Fiber-native handlers (cases 1-2)**

- **Case 1.** `fiber.Handler` - the canonical `func(fiber.Ctx) error` form.
- **Case 2.** `func(fiber.Ctx)` - Fiber runs the function and treats it as if it returned `nil`.

### **Express-style request handlers (cases 3-12)**

- **Case 3.** `func(fiber.Req, fiber.Res) error`
- **Case 4.** `func(fiber.Req, fiber.Res)`
- **Case 5.** `func(fiber.Req, fiber.Res, func() error) error`
- **Case 6.** `func(fiber.Req, fiber.Res, func() error)`
- **Case 7.** `func(fiber.Req, fiber.Res, func()) error`
- **Case 8.** `func(fiber.Req, fiber.Res, func())`
- **Case 9.** `func(fiber.Req, fiber.Res, func(error))`
- **Case 10.** `func(fiber.Req, fiber.Res, func(error)) error`
- **Case 11.** `func(fiber.Req, fiber.Res, func(error) error)`
- **Case 12.** `func(fiber.Req, fiber.Res, func(error) error) error`

The adapter injects a `next` callback when your signature accepts one. Fiber propagates downstream errors from `c.Next()` back through the wrapper, so returning those errors remains optional. If you never call the injected `next` function, the handler chain stops, matching Express semantics.

When you accept `next` callbacks that take an `error`, calling `next(nil)` continues the chain and passing a non-nil error short-circuits with that error. If the handler itself returns an error, Fiber prioritizes that value over any recorded `next` error.

Fiber has no Express-style four-argument error handler (`func(err, req, res, next)`); a non-nil error propagates to the app's central `ErrorHandler` instead.

### **net/http handlers (cases 13-15)**

- **Case 13.** `http.HandlerFunc`
- **Case 14.** `http.Handler`
- **Case 15.** `func(http.ResponseWriter, *http.Request)`

Compatibility overhead

Fiber adapts these handlers through `fasthttpadaptor`. They do not receive `fiber.Ctx`, cannot call `c.Next()`, and therefore always terminate the handler chain. The compatibility layer also adds more overhead than running a native Fiber handler, so prefer the other forms when possible.

### **fasthttp handlers (cases 16-17)**

- **Case 16.** `fasthttp.RequestHandler`
- **Case 17.** `func(*fasthttp.RequestCtx) error`

fasthttp handlers run with full access to the underlying `fasthttp.RequestCtx`. They are expected to manage the response directly. Fiber will propagate any error returned by the `func(*fasthttp.RequestCtx) error` variant but otherwise does not inspect the context state.

Examples

```go
// Reuse an existing net/http handler without manual adaptation
httpHandler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusNoContent)
})

app.Get("/foo", httpHandler)

// Align with Express-style handlers using fiber.Req and fiber.Res helpers (works
// for middleware and routes alike)
app.Use(func(req fiber.Req, res fiber.Res, next func() error) error {
    if req.IP() == "192.168.1.254" {
        return res.SendStatus(fiber.StatusForbidden)
    }
    return next()
})

app.Get("/express", func(req fiber.Req, res fiber.Res) error {
    return res.SendString("Hello from Express-style handlers!")
})

// Mount a fasthttp.RequestHandler directly (case 16)
app.Get("/bar", func(ctx *fasthttp.RequestCtx) {
    ctx.SetStatusCode(fiber.StatusAccepted)
})

// ...or the error-returning variant (case 17)
app.Get("/baz", func(ctx *fasthttp.RequestCtx) error {
    ctx.SetStatusCode(fiber.StatusAccepted)
    return nil
})
```

## **Fiber in front of gRPC services**

The pattern of [Persephone](page:615lx823qx): Fiber is the **REST API external clients talk to**, and its handlers call **internal services over gRPC**. The full runnable version is in [gRPC in Go](page:07vn6lyy82) (4.5, The Fiber REST API); this section is the Fiber side of it.

```
client ──HTTP/JSON──► Fiber (routes, middleware, ErrorHandler) ──gRPC──► video / thumbnail / metadata services
```

| **Piece** | **Fiber API** | **Why** |
| --- | --- | --- |
| Shared gRPC client | a field on a struct whose methods are the handlers | one `grpc.ClientConn` for the app, never one per request |
| Per-call deadline | `context.WithTimeout(c.Context(), 5*time.Second)` | a hung service can't hang the HTTP request |
| Auth to the services | `c.Get(fiber.HeaderAuthorization)` → `metadata.AppendToOutgoingContext` | the services' interceptors check the token |
| Request body | `c.Bind().Body(&in)` | JSON into a struct, then into the protobuf request |
| Response body | `protojson.Marshal(msg)` + `c.Send(b)` | proto field names and Timestamps as the proto defines them |
| Errors | `fiber.Config{ErrorHandler: …}` | handlers `return err`; gRPC codes become HTTP statuses in one place |
| Live progress | `c.SendStreamWriter(func(w *bufio.Writer) { … })` | relay a gRPC server stream as Server-Sent Events |

### **Handlers that call gRPC**

```go
type api struct {
    videos videov1.VideoServiceClient // made once in main, from one grpc.ClientConn
}

func (a *api) getVideo(c fiber.Ctx) error {
    ctx, cancel := context.WithTimeout(c.Context(), 5*time.Second)
    defer cancel()

    v, err := a.videos.GetVideo(ctx, &videov1.GetVideoRequest{Id: c.Params("id")})
    if err != nil {
        return err // the ErrorHandler maps codes.NotFound → 404, …
    }
    b, err := protojson.Marshal(v)
    if err != nil {
        return err
    }
    c.Set(fiber.HeaderContentType, fiber.MIMEApplicationJSON)
    return c.Send(b)
}

app := fiber.New(fiber.Config{ErrorHandler: errorHandler})
app.Get("/videos/:id", a.getVideo)
```

### **One ErrorHandler for gRPC statuses**

```go
func errorHandler(c fiber.Ctx, err error) error {
    var fe *fiber.Error
    if errors.As(err, &fe) { // fiber.NewError, 404s from routing, …
        return c.Status(fe.Code).JSON(fiber.Map{"error": fe.Message})
    }
    if st, ok := status.FromError(err); ok { // an error from a gRPC call
        return c.Status(httpStatus(st.Code())).JSON(fiber.Map{"error": st.Message(), "code": st.Code().String()})
    }
    return c.Status(fiber.StatusInternalServerError).JSON(fiber.Map{"error": "internal error"})
}
```

`httpStatus` and the whole code → status table are in [gRPC in Go](page:07vn6lyy82) (4.5): `NotFound` → 404, `InvalidArgument` → 400, `Unauthenticated` → 401, `Unavailable` → 503, `DeadlineExceeded` → 504.

### **Passing auth on as metadata**

Middleware can do it once for a whole group, keeping the context in `c.Locals`:

```go
v1 := app.Group("/api/v1", func(c fiber.Ctx) error {
    if c.Get(fiber.HeaderAuthorization) == "" {
        return fiber.ErrUnauthorized // stop here, no c.Next()
    }
    ctx := metadata.AppendToOutgoingContext(c.Context(), "authorization", c.Get(fiber.HeaderAuthorization))
    c.Locals("rpc", ctx)
    return c.Next()
})
```

Handlers then start from `c.Locals("rpc").(context.Context)` and add their deadline.

### **Streaming progress (Server-Sent Events)**

Persephone's `GET /videos/:id/progress` relays the `WatchProcessing` server stream:

```go
app.Get("/videos/:id/progress", func(c fiber.Ctx) error {
    ctx, cancel := context.WithTimeout(c.Context(), 30*time.Minute) // outlives the handler
    stream, err := a.videos.WatchProcessing(ctx, &videov1.WatchRequest{Id: c.Params("id")})
    if err != nil {
        cancel()
        return err
    }
    c.Set(fiber.HeaderContentType, "text/event-stream")
    c.Set(fiber.HeaderCacheControl, "no-cache")
    return c.SendStreamWriter(func(w *bufio.Writer) {
        defer cancel()
        for {
            update, err := stream.Recv()
            if err != nil { // io.EOF: processing finished
                return
            }
            b, _ := protojson.Marshal(update)
            fmt.Fprintf(w, "data: %s\n\n", b)
            if w.Flush() != nil { // the client disconnected
                return
            }
        }
    })
})
```

caution

The writer runs **after** the handler returns: don't use `c` inside it, and give the gRPC call a context that lives as long as the stream (cancelled by the writer when it ends), not one cancelled by `defer` in the handler.
