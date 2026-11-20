---
id: y5rcdv9xv3
title: "Fiber"
sort: 1000
version: 1
source: https://gofiber.io/
clipped: 2026-10-28 09:11
tags: [docs]
---

<heading level=2>

An Express-inspired web framework\
written in Go.

</heading>

Fiber is a Go web framework built on top of Fasthttp, the **fastest** HTTP engine for Go. It's designed to **ease** development with **performance** in mind.

```
package main
import (
    "log"
    "github.com/gofiber/fiber/v3"
)
func main() {
    app := fiber.New()
    app.Get("/", func (c fiber.Ctx) error {
        return c.SendString("Hello, World!")
    })
    log.Fatal(app.Listen(":3000"))
}
```

→

http://localhost:3000

```
Hello, World!
```

[Latest release: **v3.6.1** (Oct 14, 2026)](https://github.com/gofiber/fiber/releases/tag/v3.6.1)

## Fiber in Action[​](#fiber-in-action)

How Fiber applications are built: pick a topic, read the code.

Routes are declared Express-style with a method, a path, and a handler. Dynamic segments like `:name` are captured and read through `c.Params`, and wildcards like `*` match everything below a prefix. [Routing guide →](https://docs.gofiber.io/guide/routing/)

```
app.Get("/", func(c fiber.Ctx) error {
    return c.SendString("Hello, World!")
})
app.Get("/user/:name", func(c fiber.Ctx) error {
    return c.SendString("Hello, " + c.Params("name"))
})
app.Get("/files/*", func(c fiber.Ctx) error {
    return c.SendString("Path: " + c.Params("*"))
})
```

### Extreme Performance

Since Fiber is built on top of Fasthttp, your apps will enjoy unmatching performance! Don't believe us? Here's a benchmark that proves how Fiber shines compared to other frameworks:

![Benchmark graph](_assets/f5eda463baee77083099dd9addf4598a.png)

The framework

## Built into the Core[​](#built-into-the-core)

Everyday capabilities that ship with the framework itself. No extra packages, nothing else to install.

## Ready to ship something fast?[​](#ready-to-ship)

Start with one file, add middleware as you grow, and deploy a single binary.

Beyond the core

## The Official Ecosystem[​](#official-ecosystem)

Complete catalogs of official building blocks: middleware, storage drivers, template engines, and contrib integrations, all maintained by the Fiber team.

### [Core Middleware](https://docs.gofiber.io/category/-middleware)

[32 middleware](https://docs.gofiber.io/category/-middleware)

[The deepest catalog in the box: authentication, caching, compression, rate limiting, security headers, sessions, and more, each one app.Use away.](https://docs.gofiber.io/category/-middleware)

[AdaptorBasicAuthCacheCompressCORSCSRFEarlyDataEncrypt CookieEnvVarETagExpVarFaviconHealth CheckHelmetHost AuthorizationIdempotencyKeyAuthLimiterLoggerPaginatePprofProxyRecoverRedirectRequestIDResponseTimeRewriteSessionSkipSSEStaticTimeout](https://docs.gofiber.io/category/-middleware)

[Explore middleware](https://docs.gofiber.io/category/-middleware)

### [Storage Drivers](https://docs.gofiber.io/storage/)

[34 drivers](https://docs.gofiber.io/storage/)

[One unified interface for every major database and key-value store. Plug them into sessions, caching, or rate limiting without changing your code.](https://docs.gofiber.io/storage/)

[AerospikeArangoDBAzure BlobBadgerBboltCassandraClickhouseCloudflare KVCoherenceCouchbaseDynamoDBEtcdFirestoreLevelDBMemcacheMemoryMinioMockStorageMongoDBMSSQLMySQLNatsNeo4jPebblePostgresRedisRistrettoRueidisS3ScyllaDbSQLite3SurrealDBTest HelpersValkey](https://docs.gofiber.io/storage/)

[Browse storage drivers](https://docs.gofiber.io/storage/)

### [Template Engines](https://docs.gofiber.io/template/)

[9 engines](https://docs.gofiber.io/template/)

[Server-side rendering with the syntax you already know. One official package, one interface, your choice of engine.](https://docs.gofiber.io/template/)

[AceAmberDjangoHandlebarsHTMLJetMustachePugSlim](https://docs.gofiber.io/template/)

[Pick your engine](https://docs.gofiber.io/template/)

### [Contrib Packages](https://docs.gofiber.io/contrib/)

[24 packages](https://docs.gofiber.io/contrib/)

[Officially maintained integrations with the wider ecosystem: tracing, logging, authentication, API documentation, and real-time communication.](https://docs.gofiber.io/contrib/)

[CasbinCircuit BreakerCorazaFgprofHCaptchaI18nJWTLoadShedMonitorNew RelicOPAOTelPasetoPrometheusSentrySocket.ioSPNEGO Kerberos Authentication Middleware for FiberSwagger UISwaggoTestcontainersUptimeWebsocketZapZerolog](https://docs.gofiber.io/contrib/)

[Discover contrib](https://docs.gofiber.io/contrib/)

Fiber is supported by these organisations. Want to join them? [Become a sponsor on GitHub](https://github.com/sponsors/gofiber).

**Tool Sponsors** - supporting Fiber with free IDE licenses and AI credits.

## Help & Support[​](#help-support)

Join our community on **Discord**: ask questions, share, help others.

[📖 Go Fiber by Examples: Working with middlewares and boilerplates](https://dev.to/koddr/go-fiber-by-examples-working-with-middlewares-and-boilerplates-3p0m)

[Vic Shóstak • september 13, 2021 • dev.to](https://dev.to/koddr/go-fiber-by-examples-working-with-middlewares-and-boilerplates-3p0m)

[📖 Go Fiber by Examples: Testing the application](https://dev.to/koddr/go-fiber-by-examples-testing-the-application-1ldf)

[Vic Shóstak • august 30, 2021 • dev.to](https://dev.to/koddr/go-fiber-by-examples-testing-the-application-1ldf)

[📖 Go Fiber by Examples: Delving into built-in functions](https://dev.to/koddr/go-fiber-by-examples-delving-into-built-in-functions-1p3k)

[Vic Shóstak • august 24, 2021 • dev.to](https://dev.to/koddr/go-fiber-by-examples-delving-into-built-in-functions-1p3k)

[📖 Go Fiber by Examples: How can the Fiber Web Framework be useful?](https://dev.to/koddr/go-fiber-by-examples-how-can-the-fiber-web-framework-be-useful-487a)

[Vic Shóstak • august 16, 2021 • dev.to](https://dev.to/koddr/go-fiber-by-examples-how-can-the-fiber-web-framework-be-useful-487a)

[📖 Build a RESTful API on Go: Fiber, PostgreSQL, JWT and Swagger docs in isolated Docker containers](https://dev.to/koddr/build-a-restful-api-on-go-fiber-postgresql-jwt-and-swagger-docs-in-isolated-docker-containers-475j)

[Vic Shóstak • march 22, 2021 • dev.to](https://dev.to/koddr/build-a-restful-api-on-go-fiber-postgresql-jwt-and-swagger-docs-in-isolated-docker-containers-475j)

[Getting started with Fiber ⚡](https://dev.to/fenny/getting-started-with-fiber-36b6)

[Fenny 🔥 • june 10, 2020 • dev.to](https://dev.to/fenny/getting-started-with-fiber-36b6)

[Building an Express-style API in Go with Fiber](https://blog.logrocket.com/express-style-api-go-fiber/)

[Alexander Nnakwue • june 10, 2020 • logrocket.com](https://blog.logrocket.com/express-style-api-go-fiber/)

[Fiber v1.9.6 🔥 How to improve performance by 817% and stay fast, flexible and friendly?](https://dev.to/koddr/fiber-v1-9-5-how-to-improve-performance-by-817-and-stay-fast-flexible-and-friendly-2dp6)

[Vic Shóstak • May 12, 2020 • dev.to](https://dev.to/koddr/fiber-v1-9-5-how-to-improve-performance-by-817-and-stay-fast-flexible-and-friendly-2dp6)

[🌎 Create a travel list app with Go, Fiber, Angular, MongoDB and Google Cloud Secret Manager](https://blog.yongweilun.me/create-a-travel-list-app-with-go-fiber-angular-mongodb-and-google-cloud-secret-manager-ck9fgxy0p061pcss1xt1ubu8t)

[Wei Lun • April 25, 2020 • yongweilun.me](https://blog.yongweilun.me/create-a-travel-list-app-with-go-fiber-angular-mongodb-and-google-cloud-secret-manager-ck9fgxy0p061pcss1xt1ubu8t)

[Building a Basic REST API in Go using Fiber](https://tutorialedge.net/golang/basic-rest-api-go-fiber/)

[Elliot Forbes • April 23, 2020 • tutorialedge.net](https://tutorialedge.net/golang/basic-rest-api-go-fiber/)

[Creating Fast APIs In Go Using Fiber](https://dev.to/jozsefsallai/creating-fast-apis-in-go-using-fiber-59m9)

[József Sallai • April 7, 2020 • dev.to](https://dev.to/jozsefsallai/creating-fast-apis-in-go-using-fiber-59m9)

[Is switching from Express to Fiber worth it? 🤔](https://dev.to/koddr/are-sure-what-your-lovely-web-framework-running-so-fast-2jl1)

[Vic Shóstak • April 1, 2020 • dev.to](https://dev.to/koddr/are-sure-what-your-lovely-web-framework-running-so-fast-2jl1)

[🚀 Fiber v1.8. What's new, updated and re-thinked?](https://dev.to/koddr/fiber-v1-8-what-s-new-updated-and-re-thinked-339h)

[Vic Shóstak • March 3, 2020 • dev.to](https://dev.to/koddr/fiber-v1-8-what-s-new-updated-and-re-thinked-339h)

[Fiber released v1.7! 🎉 What's new and is it still fast, flexible and friendly?](https://dev.to/koddr/fiber-v2-is-out-now-what-s-new-and-is-he-still-fast-flexible-and-friendly-3ipf)

[Vic Shóstak • February 21, 2020 • dev.to](https://dev.to/koddr/fiber-v2-is-out-now-what-s-new-and-is-he-still-fast-flexible-and-friendly-3ipf)

[Welcome to Fiber — an Express.js styled web framework written in Go with ❤️](https://dev.to/koddr/welcome-to-fiber-an-express-js-styled-fastest-web-framework-written-with-on-golang-497)

[Vic Shóstak • February 3, 2020 • dev.to](https://dev.to/koddr/welcome-to-fiber-an-express-js-styled-fastest-web-framework-written-with-on-golang-497)
