---
id: 07vn6lyy82
title: "gRPC in Go"
cover: "_assets/ff2d490d8075304b26a74738d90bcd2a.jpg"
cover_position: 14
sort: 1000
version: 1
---

Notes, reference and a hands-on tutorial for building gRPC services in Go. The tutorial puts a [GoFiber](page:phgfxrea1z10a) REST API in front of the gRPC service (4.5): REST for external clients, gRPC between internal services, the shape [Persephone](page:615lx823qx) is built on.

---

## 1\. What is gRPC?

**gRPC** is a high-performance, open-source **Remote Procedure Call (RPC)** framework originally built at Google. Instead of thinking in terms of URLs and HTTP verbs (like REST), you call a method on a remote server **as if it were a local function**:

```go
note, err := client.GetNote(ctx, &notesv1.GetNoteRequest{Id: "1"})
```

Under the hood that call is serialized, sent over the network, executed on the server, and the result is sent back.

### Key ingredients

| Piece | What it does |
| --- | --- |
| **Protocol Buffers (protobuf)** | The <b><i>Interface Definition Language</i> </b>(IDL) **and** the binary serialization format. You describe services and messages in a `.proto` file. |
| **Code generation** | `protoc` + Go plugins turn the `.proto` file into Go structs, a client stub and a server interface. |
| **HTTP/2** | The transport. Gives multiplexing (many calls on one TCP connection), header compression, and full-duplex streams. |
| **Runtime library** | `google.golang.org/grpc` handles connections, load balancing, deadlines, cancellation, metadata, interceptors, etc. |

### Why use it?

- **Fast & compact** – binary protobuf is much smaller and quicker to parse than JSON.
- **Strongly typed contract** – the `.proto` file is the single source of truth; client and server can't drift apart silently.
- **Polyglot** – generate clients/servers for Go, Python, Java, TS, Rust, C#, … from the same `.proto`.
- **Streaming built in** – server, client and bidirectional streaming are first-class.
- **Deadlines & cancellation** – propagate across service boundaries automatically via `context.Context`.

### When *not* to use it

- Browser clients talking directly to your API (browsers can't speak raw gRPC; you need **gRPC-Web** or **Connect**, or a REST gateway).
- Public APIs where humans want to `curl` and read JSON.
- Very simple CRUD where REST/JSON is "good enough" and tooling familiarity matters more.

---

## 2\. How gRPC works

### 2\.1 The flow of a single call

```
 ┌──────────── Client process ────────────┐            ┌──────────── Server process ────────────┐
 │                                         │            │                                         │
 │  your code                              │            │                     your handler code   │
 │    │  client.GetNote(ctx, req)          │            │   func (s *server) GetNote(...) ▲       │
 │    ▼                                    │            │                                 │       │
 │  generated client stub                  │            │   generated server "glue"       │       │
 │    │  proto.Marshal(req) → bytes        │            │   proto.Unmarshal(bytes) → req  │       │
 │    ▼                                    │   HTTP/2   │                                 │       │
 │  grpc-go runtime  ───────────────────────────────────────────►  grpc-go runtime ────────┘       │
 │    POST /notes.v1.NoteService/GetNote   │            │                                         │
 │    ◄─────────────────────────────────────────────────────────  response bytes + status code    │
 └─────────────────────────────────────────┘            └─────────────────────────────────────────┘
```

1. You call a method on the **generated client stub**.
1. The stub **marshals** the request message to protobuf bytes.
1. grpc-go sends an **HTTP/2 POST** to the path `/<package>.<Service>/<Method>` with `content-type: application/grpc`. Each message is framed as: 1-byte compression flag + 4-byte length + payload.
1. The server's runtime **routes** by path to the registered service, **unmarshals** the bytes, and calls *your* handler.
1. Your handler returns a response (or an error). The response is marshalled and sent back, followed by **trailers** containing `grpc-status` and `grpc-message`.
1. The client stub unmarshals the response and returns it to your code (or returns a `status` error).

### 2\.2 The four kinds of RPC

| Type | `.proto` signature | Use case |
| --- | --- | --- |
| **Unary** | `rpc Get(Req) returns (Res);` | Classic request → response. |
| **Server streaming** | `rpc List(Req) returns (stream Res);` | Server sends many results (feeds, large lists, progress). |
| **Client streaming** | `rpc Upload(stream Req) returns (Res);` | Client sends many items, server replies once (uploads, batching). |
| **Bidirectional streaming** | `rpc Chat(stream Req) returns (stream Res);` | Both sides send independently (chat, live sync). |

All four share one HTTP/2 stream per call — that's why streaming is cheap in gRPC.

### 2\.3 Protocol Buffers in 60 seconds

```proto
syntax = "proto3";

message Note {
  string id    = 1;   // ← field NUMBER, not a default value
  string title = 2;
  repeated string tags = 3;   // list
}
```

- The **field numbers** (`= 1`, `= 2`) identify fields on the wire. **Never change or reuse** a number once deployed — that's what keeps old and new clients compatible.
- Removing a field? Mark it `reserved 4;` so nobody reuses it.
- Proto3 scalars have zero-value defaults (`""`, `0`, `false`). Use `optional` if you need to detect "not set".
- Common types: `string`, `int32/int64`, `uint32/uint64`, `bool`, `double`, `bytes`, `enum`, nested `message`, `repeated`, `map<string, string>`.
- Well-known types: `google.protobuf.Timestamp`, `Duration`, `Empty`, `Struct`, wrappers (`StringValue`…).

### 2\.4 Status codes (errors)

gRPC doesn't use HTTP status codes for app errors; it uses its own codes from `google.golang.org/grpc/codes`:

| Code | Meaning |
| --- | --- |
| `OK` | Success |
| `InvalidArgument` | Bad input from client |
| `NotFound` | Resource doesn't exist |
| `AlreadyExists` | Conflict on create |
| `PermissionDenied` / `Unauthenticated` | AuthZ / AuthN failure |
| `DeadlineExceeded` | Took longer than the client allowed |
| `Unavailable` | Server down / transient — usually safe to retry |
| `Internal` | Server bug |
| `Unimplemented` | Method not implemented |

Return them with `status.Error(codes.NotFound, "note not found")` and read them on the client with `status.FromError(err)` or `status.Code(err)`.

### 2\.5 Deadlines, cancellation & metadata

- **Deadlines**: the client sets one with `context.WithTimeout`. It travels to the server as the `grpc-timeout` header; the server's `ctx` is cancelled when it expires. **Always set a deadline on the client.**
- **Cancellation**: if the client cancels/disconnects, the server's `ctx.Done()` fires — check it in long work and streams.
- **Metadata**: key/value headers (like HTTP headers) for auth tokens, request IDs, etc. Use `google.golang.org/grpc/metadata`.

### 2\.6 Interceptors (middleware)

Interceptors wrap every call — perfect for logging, auth, metrics, panics → errors, tracing. There are unary and stream variants, on both client and server.

---

## 3\. Adding gRPC to a Go project

### 3\.1 Install the tools (one-time)

**1. Protocol buffer compiler `protoc`**

```bash
# Fedora
sudo dnf install protobuf-compiler protobuf-devel   # -devel ships google/protobuf/*.proto (Timestamp etc.)

# Debian/Ubuntu
sudo apt install -y protobuf-compiler

# macOS
brew install protobuf

protoc --version   # libprotoc 3.x / 2x.x
```

**2. Go code generator plugins**

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest        # messages  → *.pb.go
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest       # services  → *_grpc.pb.go
```

**3. Make sure the plugins are on your `PATH`** (`protoc` looks for binaries named `protoc-gen-*`):

```bash
export PATH="$PATH:$(go env GOPATH)/bin"   # add to ~/.bashrc to make permanent
```

### 3\.2 Add the Go libraries to your module

```bash
go get google.golang.org/grpc
go get google.golang.org/protobuf
```

### 3\.3 The general recipe

1. Write a `.proto` file describing your service and messages.
1. Run `protoc` with `--go_out` and `--go-grpc_out` to generate code.
1. **Server**: implement the generated `XxxServer` interface, register it with `grpc.NewServer()`, call `Serve(listener)`.
1. **Client**: `grpc.NewClient(addr, ...)` → `NewXxxClient(conn)` → call methods.

---

## 4\. Tutorial: a Notes service with a Fiber API

You'll build two Go programs in one module and run them side by side:

```
 curl / browser ──HTTP + JSON──► Fiber API  (api/main.go, :3000)
                                     │
                                     │ gRPC (HTTP/2)
                                     ▼
                                 Notes service  (main.go, :50051)
```

- **The Notes service** (`main.go`) is a gRPC server with `CreateNote`, `GetNote` (which can fail with `NotFound`) and `ListNotes` (server streaming).
- **The Fiber API** (`api/main.go`) is what clients use: plain HTTP and JSON. Each request turns into a gRPC call to the service.

That's the shape of [Persephone](page:615lx823qx): **REST for external clients, gRPC between internal services**. Every command below is run from the `notes-grpc/` folder.

### 4\.1 Set up the project

Do 3.1 first (protoc and the two plugins). Then:

```bash
mkdir -p notes-grpc/proto/notes/v1 notes-grpc/gen notes-grpc/api
cd notes-grpc
go mod init example.com/notes-grpc
go get google.golang.org/grpc google.golang.org/protobuf github.com/gofiber/fiber/v3
```

By the end of the tutorial the folder looks like this:

```
notes-grpc/
├── go.mod, go.sum
├── proto/notes/v1/notes.proto   ← 4.2  the contract (you write it)
├── gen/notes/v1/                ← 4.3  generated by protoc (never edit)
│   ├── notes.pb.go
│   └── notes_grpc.pb.go
├── main.go                      ← 4.4  the gRPC service          → go run .
└── api/main.go                  ← 4.5  the Fiber REST API        → go run ./api
```

Each `main.go` is its own program (`package main`): `go run .` runs the one in `notes-grpc/`, `go run ./api` the one in `api/`. The module name `example.com/notes-grpc` matters: it's how both programs import the generated code, as `example.com/notes-grpc/gen/notes/v1`.

### 4\.2 Define the service — `proto/notes/v1/notes.proto`

```proto
syntax = "proto3";

package notes.v1;

import "google/protobuf/timestamp.proto";

// Where the generated Go code lives, and its package name (after the ';').
option go_package = "example.com/notes-grpc/gen/notes/v1;notesv1";

service NoteService {
  rpc CreateNote(CreateNoteRequest) returns (Note);
  rpc GetNote(GetNoteRequest) returns (Note);
  rpc ListNotes(ListNotesRequest) returns (stream Note);   // server streaming
}

message Note {
  string id = 1;
  string title = 2;
  string body = 3;
  google.protobuf.Timestamp created_at = 4;
}

message CreateNoteRequest {
  string title = 1;
  string body = 2;
}

message GetNoteRequest {
  string id = 1;
}

message ListNotesRequest {}
```

> **Tip:** put a version (`v1`) in the package name. When you need breaking changes, create `notes.v2` side-by-side instead of breaking clients.

### 4\.3 Generate the Go code

```bash
protoc -I proto \
  --go_out=gen      --go_opt=paths=source_relative \
  --go-grpc_out=gen --go-grpc_opt=paths=source_relative \
  proto/notes/v1/notes.proto
```

- `-I proto` – where to look for imports; output paths are relative to it.
- `paths=source_relative` – put output next to the matching path (`gen/notes/v1/`).

You now have:

- **`notes.pb.go`** – Go structs: `Note`, `CreateNoteRequest`, … with getters like `GetTitle()`.
- **`notes_grpc.pb.go`** – `NoteServiceClient` (stub), `NoteServiceServer` (interface you implement), `UnimplementedNoteServiceServer`, `RegisterNoteServiceServer`, `NewNoteServiceClient`.

The generated server interface looks like:

```go
type NoteServiceServer interface {
	CreateNote(context.Context, *CreateNoteRequest) (*Note, error)
	GetNote(context.Context, *GetNoteRequest) (*Note, error)
	ListNotes(*ListNotesRequest, grpc.ServerStreamingServer[Note]) error
	mustEmbedUnimplementedNoteServiceServer()
}
```

> Re-run `protoc` every time you edit the `.proto`. Many projects put this in a `Makefile` or a `//go:generate` comment.

### 4\.4 The gRPC service — `main.go`

This file goes at the top of `notes-grpc/`, next to `go.mod`. It keeps notes in memory and listens on port **50051**.

```go
package main

import (
	"context"
	"log"
	"net"
	"strconv"
	"sync"
	"time"

	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/reflection"
	"google.golang.org/grpc/status"
	"google.golang.org/protobuf/types/known/timestamppb"

	notesv1 "example.com/notes-grpc/gen/notes/v1"
)

// server implements notesv1.NoteServiceServer.
type server struct {
	// Embedding this makes the server forward-compatible: if new RPCs are
	// added to the .proto, they return codes.Unimplemented instead of
	// breaking the build.
	notesv1.UnimplementedNoteServiceServer

	mu     sync.Mutex
	notes  map[string]*notesv1.Note
	order  []string
	nextID int
}

func newServer() *server {
	return &server{notes: make(map[string]*notesv1.Note)}
}

// Unary RPC.
func (s *server) CreateNote(ctx context.Context, req *notesv1.CreateNoteRequest) (*notesv1.Note, error) {
	if req.GetTitle() == "" {
		return nil, status.Error(codes.InvalidArgument, "title is required")
	}

	s.mu.Lock()
	defer s.mu.Unlock()

	s.nextID++
	id := strconv.Itoa(s.nextID)
	note := &notesv1.Note{
		Id:        id,
		Title:     req.GetTitle(),
		Body:      req.GetBody(),
		CreatedAt: timestamppb.Now(),
	}
	s.notes[id] = note
	s.order = append(s.order, id)
	return note, nil
}

// Unary RPC returning a gRPC status error.
func (s *server) GetNote(ctx context.Context, req *notesv1.GetNoteRequest) (*notesv1.Note, error) {
	s.mu.Lock()
	defer s.mu.Unlock()

	note, ok := s.notes[req.GetId()]
	if !ok {
		return nil, status.Errorf(codes.NotFound, "note %q not found", req.GetId())
	}
	return note, nil
}

// Server-streaming RPC: call stream.Send once per item, return nil when done.
func (s *server) ListNotes(req *notesv1.ListNotesRequest, stream grpc.ServerStreamingServer[notesv1.Note]) error {
	s.mu.Lock()
	snapshot := make([]*notesv1.Note, 0, len(s.order))
	for _, id := range s.order {
		snapshot = append(snapshot, s.notes[id])
	}
	s.mu.Unlock()

	for _, note := range snapshot {
		// Stop early if the client cancelled or the deadline passed.
		if err := stream.Context().Err(); err != nil {
			return status.FromContextError(err).Err()
		}
		if err := stream.Send(note); err != nil {
			return err
		}
	}
	return nil
}

// A unary server interceptor (middleware) that logs every call.
func loggingInterceptor(
	ctx context.Context,
	req any,
	info *grpc.UnaryServerInfo,
	handler grpc.UnaryHandler,
) (any, error) {
	start := time.Now()
	resp, err := handler(ctx, req)
	log.Printf("%s took=%s code=%s", info.FullMethod, time.Since(start), status.Code(err))
	return resp, err
}

func main() {
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("listen: %v", err)
	}

	grpcServer := grpc.NewServer(
		grpc.UnaryInterceptor(loggingInterceptor),
	)
	notesv1.RegisterNoteServiceServer(grpcServer, newServer())

	// Reflection lets tools like grpcurl discover your services. Optional.
	reflection.Register(grpcServer)

	log.Printf("gRPC server listening on %s", lis.Addr())
	if err := grpcServer.Serve(lis); err != nil {
		log.Fatalf("serve: %v", err)
	}
}
```

Check it starts:

```bash
go run .
# gRPC server listening on [::]:50051
```

Leave it running and open a second terminal for the next step.

### 4\.5 The Fiber REST API — `api/main.go`

A [GoFiber](page:phgfxrea1z10a) app on port **3000**. Its routes:

| HTTP request | gRPC call it makes | Answers with |
| --- | --- | --- |
| `POST /api/v1/notes` with `{"title": "…", "body": "…"}` | `CreateNote` | 201 and the new note |
| `GET /api/v1/notes/:id` | `GetNote` | 200 and the note, or 404 |
| `GET /api/v1/notes` | `ListNotes` (stream) | 200 and every note in one JSON array |
| `GET /api/v1/notes/stream` | `ListNotes` (stream) | each note as it arrives, as Server-Sent Events |

What each part of the file does:

- **`main`** opens **one** gRPC connection to `localhost:50051`, makes the Fiber app and registers the routes.
- **`rpcContext`** gives each gRPC call a deadline and passes the caller's `Authorization` header on to the service.
- **`createNote`, `getNote`, `listNotes`, `streamNotes`** are the four handlers, one per route.
- **`sendProto`** writes a protobuf message as JSON.
- **`errorHandler` and `httpStatus`** turn errors into JSON with the right HTTP status, e.g. gRPC `NotFound` → 404.

If you skipped 4.1's `go get`, add Fiber now:

```bash
go get github.com/gofiber/fiber/v3
```

```go
package main

import (
	"bufio"
	"context"
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"log"
	"time"

	"github.com/gofiber/fiber/v3"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/credentials/insecure"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
	"google.golang.org/protobuf/encoding/protojson"
	"google.golang.org/protobuf/proto"

	notesv1 "example.com/notes-grpc/gen/notes/v1"
)

// api holds ONE gRPC client for the whole app: a grpc.ClientConn is safe for concurrent
// use and multiplexes every request over a single HTTP/2 connection. Never dial per request.
type api struct {
	notes notesv1.NoteServiceClient
}

func main() {
	conn, err := grpc.NewClient("localhost:50051",
		grpc.WithTransportCredentials(insecure.NewCredentials()), // local dev only
	)
	if err != nil {
		log.Fatalf("grpc client: %v", err)
	}
	defer conn.Close()

	a := &api{notes: notesv1.NewNoteServiceClient(conn)}

	// One ErrorHandler turns every error (Fiber's or gRPC's) into a JSON response.
	app := fiber.New(fiber.Config{ErrorHandler: errorHandler})

	v1 := app.Group("/api/v1")
	v1.Post("/notes", a.createNote)
	v1.Get("/notes", a.listNotes)
	v1.Get("/notes/stream", a.streamNotes) // declared before /notes/:id: first match wins
	v1.Get("/notes/:id", a.getNote)

	log.Fatal(app.Listen(":3000"))
}

// rpcContext gives every gRPC call a deadline and passes the caller's Authorization
// header on as gRPC metadata, so the services' auth interceptors (5.2) see it.
func rpcContext(c fiber.Ctx, timeout time.Duration) (context.Context, context.CancelFunc) {
	ctx, cancel := context.WithTimeout(c.Context(), timeout)
	if auth := c.Get(fiber.HeaderAuthorization); auth != "" {
		ctx = metadata.AppendToOutgoingContext(ctx, "authorization", auth)
	}
	return ctx, cancel
}

type createNoteBody struct {
	Title string `json:"title"`
	Body  string `json:"body"`
}

// POST /api/v1/notes  →  NoteService.CreateNote (unary)
func (a *api) createNote(c fiber.Ctx) error {
	var in createNoteBody
	if err := c.Bind().Body(&in); err != nil {
		return fiber.NewError(fiber.StatusBadRequest, "invalid JSON body")
	}
	ctx, cancel := rpcContext(c, 5*time.Second)
	defer cancel()

	note, err := a.notes.CreateNote(ctx, &notesv1.CreateNoteRequest{Title: in.Title, Body: in.Body})
	if err != nil {
		return err // e.g. InvalidArgument → 400, done by errorHandler
	}
	return sendProto(c.Status(fiber.StatusCreated), note)
}

// GET /api/v1/notes/:id  →  NoteService.GetNote (unary, may be NotFound → 404)
func (a *api) getNote(c fiber.Ctx) error {
	ctx, cancel := rpcContext(c, 5*time.Second)
	defer cancel()

	note, err := a.notes.GetNote(ctx, &notesv1.GetNoteRequest{Id: c.Params("id")})
	if err != nil {
		return err
	}
	return sendProto(c, note)
}

// GET /api/v1/notes  →  NoteService.ListNotes (server streaming), collected into one JSON array.
func (a *api) listNotes(c fiber.Ctx) error {
	ctx, cancel := rpcContext(c, 5*time.Second)
	defer cancel()

	stream, err := a.notes.ListNotes(ctx, &notesv1.ListNotesRequest{})
	if err != nil {
		return err
	}
	out := []json.RawMessage{}
	for {
		note, err := stream.Recv()
		if errors.Is(err, io.EOF) {
			break
		}
		if err != nil {
			return err
		}
		b, err := protojson.Marshal(note)
		if err != nil {
			return err
		}
		out = append(out, b)
	}
	return c.JSON(out)
}

// GET /api/v1/notes/stream  →  the same stream, passed on as Server-Sent Events as it arrives.
// This is how Persephone's GET /videos/:id/progress can relay WatchProcessing updates.
func (a *api) streamNotes(c fiber.Ctx) error {
	// The writer below runs after this handler returns, so the context must outlive it:
	// a long timeout of its own, cancelled when the stream ends.
	ctx, cancel := rpcContext(c, 10*time.Minute)
	stream, err := a.notes.ListNotes(ctx, &notesv1.ListNotesRequest{})
	if err != nil {
		cancel()
		return err
	}
	c.Set(fiber.HeaderContentType, "text/event-stream")
	c.Set(fiber.HeaderCacheControl, "no-cache")
	return c.SendStreamWriter(func(w *bufio.Writer) {
		defer cancel()
		for {
			note, err := stream.Recv()
			if err != nil { // io.EOF when the service is done
				return
			}
			b, _ := protojson.Marshal(note)
			fmt.Fprintf(w, "data: %s\n\n", b)
			if err := w.Flush(); err != nil { // the client went away
				return
			}
		}
	})
}

// sendProto writes a protobuf message as JSON with protojson (not encoding/json):
// camelCase field names, Timestamps as RFC 3339 strings, enums by name.
func sendProto(c fiber.Ctx, m proto.Message) error {
	b, err := protojson.Marshal(m)
	if err != nil {
		return err
	}
	c.Set(fiber.HeaderContentType, fiber.MIMEApplicationJSON)
	return c.Send(b)
}

// errorHandler: Fiber errors keep their code; gRPC status errors get the HTTP status that means the same.
func errorHandler(c fiber.Ctx, err error) error {
	var fe *fiber.Error
	if errors.As(err, &fe) {
		return c.Status(fe.Code).JSON(fiber.Map{"error": fe.Message})
	}
	if st, ok := status.FromError(err); ok {
		return c.Status(httpStatus(st.Code())).JSON(fiber.Map{"error": st.Message(), "code": st.Code().String()})
	}
	log.Printf("unexpected error: %v", err)
	return c.Status(fiber.StatusInternalServerError).JSON(fiber.Map{"error": "internal error"})
}

func httpStatus(code codes.Code) int {
	switch code {
	case codes.OK:
		return fiber.StatusOK
	case codes.InvalidArgument, codes.FailedPrecondition, codes.OutOfRange:
		return fiber.StatusBadRequest
	case codes.Unauthenticated:
		return fiber.StatusUnauthorized
	case codes.PermissionDenied:
		return fiber.StatusForbidden
	case codes.NotFound:
		return fiber.StatusNotFound
	case codes.AlreadyExists, codes.Aborted:
		return fiber.StatusConflict
	case codes.ResourceExhausted:
		return fiber.StatusTooManyRequests
	case codes.Unimplemented:
		return fiber.StatusNotImplemented
	case codes.Unavailable:
		return fiber.StatusServiceUnavailable
	case codes.DeadlineExceeded:
		return fiber.StatusGatewayTimeout
	default: // Internal, Unknown, DataLoss, Canceled
		return fiber.StatusInternalServerError
	}
}
```

#### gRPC status → HTTP status

The same mapping grpc-gateway uses, so clients get the HTTP code they expect:

| gRPC code | HTTP |
| --- | --- |
| `InvalidArgument`, `FailedPrecondition`, `OutOfRange` | 400 Bad Request |
| `Unauthenticated` | 401 Unauthorized |
| `PermissionDenied` | 403 Forbidden |
| `NotFound` | 404 Not Found |
| `AlreadyExists`, `Aborted` | 409 Conflict |
| `ResourceExhausted` | 429 Too Many Requests |
| `Unimplemented` | 501 Not Implemented |
| `Unavailable` | 503 Service Unavailable |
| `DeadlineExceeded` | 504 Gateway Timeout |
| `Internal`, `Unknown`, … | 500 Internal Server Error |

Why it's written this way:


- **One `grpc.ClientConn`**, made in `main` and shared by every handler. Dialing per request throws away HTTP/2 multiplexing.
- **Every call gets a deadline** from `rpcContext`. Without it a hung service hangs the HTTP request too.
- **Auth travels as metadata**: the API doesn't check the token itself; it forwards `Authorization`, and the services' interceptors do (Persephone's Phase 4).
- **Handlers just `return err`**: the central `ErrorHandler` turns gRPC statuses into HTTP ones. More on Fiber's side in [GoFiber](page:phgfxrea1z10a) (Fiber in front of gRPC services).
- **Streams have two REST shapes**: collect into a JSON array (`/notes`), or relay live as Server-Sent Events (`/notes/stream`) — Persephone's progress endpoint is the second.

### 4\.6 Run it and call it

Three terminals, all in `notes-grpc/`:

```bash
# terminal 1: the gRPC service
go run .

# terminal 2: the Fiber API
go run ./api
```

```bash
# terminal 3: plain HTTP + JSON, no gRPC tools needed
curl -s -X POST localhost:3000/api/v1/notes -H 'Content-Type: application/json' -d '{"title":"From Fiber","body":"hi"}'
curl -s localhost:3000/api/v1/notes/1
curl -s localhost:3000/api/v1/notes/999
curl -s -X POST localhost:3000/api/v1/notes -H 'Content-Type: application/json' -d '{"title":""}'
curl -s localhost:3000/api/v1/notes
curl -N localhost:3000/api/v1/notes/stream
```

What you get back, in the same order (add `-w ' %{http_code}'` to curl to see the status):

```
{"id":"1","title":"From Fiber","body":"hi","createdAt":"2026-10-19T22:12:07.788Z"}   201
{"id":"1","title":"From Fiber","body":"hi","createdAt":"2026-10-19T22:12:07.788Z"}   200
{"code":"NotFound","error":"note \"999\" not found"}                                404
{"code":"InvalidArgument","error":"title is required"}                               400
[{"id":"1","title":"From Fiber","body":"hi","createdAt":"…"}]                          200
data: {"id":"1","title":"From Fiber","body":"hi","createdAt":"…"}
```

Meanwhile terminal 1 logs every unary call it served:

```
/notes.v1.NoteService/CreateNote took=6.194µs code=OK
/notes.v1.NoteService/GetNote took=1.284µs code=OK
/notes.v1.NoteService/GetNote took=3.639µs code=NotFound
/notes.v1.NoteService/CreateNote took=1.193µs code=InvalidArgument
```

> `ListNotes` isn't logged: `grpc.UnaryInterceptor` only wraps unary calls. Add a `grpc.StreamInterceptor` for streams.

**If something fails:**

- `protoc-gen-go: program not found` in 4.3 → the plugins aren't on your `PATH`: run `export PATH="$PATH:$(go env GOPATH)/bin"` (3.1, step 3) and generate again.
- `connection refused` from curl → the Fiber API (terminal 2) isn't running.
- `{"code":"Unavailable",…}` with 503 → the API is up but the gRPC service (terminal 1) isn't.
- `cannot find package "example.com/notes-grpc/gen/notes/v1"` → run 4.3 again, from `notes-grpc/`.
- `address already in use` → something else is on port 3000 or 50051; stop it, or change the port in `app.Listen(":3000")` / `net.Listen("tcp", ":50051")`.

### 4\.7 Optional: a Go gRPC client — `client/main.go`

The Fiber API is already a gRPC client. This one calls the service straight from Go, with no HTTP in between, which is useful for seeing the raw gRPC calls. Make the folder with `mkdir client` and put this in `client/main.go`:

```go
package main

import (
	"context"
	"errors"
	"io"
	"log"
	"time"

	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/credentials/insecure"
	"google.golang.org/grpc/status"

	notesv1 "example.com/notes-grpc/gen/notes/v1"
)

func main() {
	// NewClient doesn't connect immediately; it connects lazily on the first RPC.
	conn, err := grpc.NewClient("localhost:50051",
		grpc.WithTransportCredentials(insecure.NewCredentials()), // no TLS — local dev only!
	)
	if err != nil {
		log.Fatalf("new client: %v", err)
	}
	defer conn.Close()

	client := notesv1.NewNoteServiceClient(conn)

	// Always give RPCs a deadline.
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// --- Unary: CreateNote ---
	for _, title := range []string{"Learn protobuf", "Learn gRPC", "Ship it"} {
		n, err := client.CreateNote(ctx, &notesv1.CreateNoteRequest{Title: title, Body: "todo"})
		if err != nil {
			log.Fatalf("CreateNote: %v", err)
		}
		log.Printf("created note id=%s title=%q", n.GetId(), n.GetTitle())
	}

	// --- Unary: GetNote, handling a NotFound error ---
	_, err = client.GetNote(ctx, &notesv1.GetNoteRequest{Id: "999"})
	if st, ok := status.FromError(err); ok && st.Code() == codes.NotFound {
		log.Printf("expected error: code=%s msg=%s", st.Code(), st.Message())
	}

	// --- Validation error ---
	_, err = client.CreateNote(ctx, &notesv1.CreateNoteRequest{Title: ""})
	log.Printf("empty title → code=%s", status.Code(err))

	// --- Server streaming: ListNotes ---
	stream, err := client.ListNotes(ctx, &notesv1.ListNotesRequest{})
	if err != nil {
		log.Fatalf("ListNotes: %v", err)
	}
	for {
		note, err := stream.Recv()
		if errors.Is(err, io.EOF) {
			break // server finished sending
		}
		if err != nil {
			log.Fatalf("recv: %v", err)
		}
		log.Printf("streamed note %s: %s (created %s)",
			note.GetId(), note.GetTitle(), note.GetCreatedAt().AsTime().Format(time.Kitchen))
	}
}
```

Run it with the service still running in terminal 1:

```bash
go run ./client
```

```
created note id=1 title="Learn protobuf"
created note id=2 title="Learn gRPC"
created note id=3 title="Ship it"
expected error: code=NotFound msg=note "999" not found
empty title → code=InvalidArgument
streamed note 1: Learn protobuf (created 3:04PM)
…
```

(If you already created notes through the API, the ids carry on from there: the service keeps notes in memory until you stop it.)

### 4\.8 Optional: call the service with `grpcurl`

`grpcurl` is curl for gRPC. It checks the service on its own, with no API and no Go code:

```bash
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest

grpcurl -plaintext localhost:50051 list
grpcurl -plaintext localhost:50051 describe notes.v1.NoteService
grpcurl -plaintext -d '{"title":"from grpcurl","body":"hi"}' \
  localhost:50051 notes.v1.NoteService/CreateNote
grpcurl -plaintext -d '{}' localhost:50051 notes.v1.NoteService/ListNotes
```

This works because the service registers **reflection** (`reflection.Register` in `main.go`).

---

## 5\. Beyond the basics

### 5\.1 Client streaming & bidirectional streaming

```proto
rpc ImportNotes(stream CreateNoteRequest) returns (ImportSummary);  // client streaming
rpc Sync(stream SyncEvent) returns (stream SyncEvent);              // bidirectional
```

Server side (client streaming):

```go
func (s *server) ImportNotes(stream grpc.ClientStreamingServer[notesv1.CreateNoteRequest, notesv1.ImportSummary]) error {
	count := 0
	for {
		req, err := stream.Recv()
		if errors.Is(err, io.EOF) {
			return stream.SendAndClose(&notesv1.ImportSummary{Count: int32(count)})
		}
		if err != nil {
			return err
		}
		// ... save req ...
		count++
	}
}
```

Server side (bidi) – read and write in any order, often with a goroutine:

```go
func (s *server) Sync(stream grpc.BidiStreamingServer[notesv1.SyncEvent, notesv1.SyncEvent]) error {
	for {
		ev, err := stream.Recv()
		if errors.Is(err, io.EOF) {
			return nil
		}
		if err != nil {
			return err
		}
		if err := stream.Send(ev); err != nil { // echo back
			return err
		}
	}
}
```

Client side: call `Send()` repeatedly, then `CloseAndRecv()` (client streaming) or `CloseSend()` + `Recv()` loop (bidi).

### 5\.2 Metadata (headers) — e.g. auth tokens

```go
// client: attach
ctx = metadata.AppendToOutgoingContext(ctx, "authorization", "Bearer "+token)

// server: read
md, _ := metadata.FromIncomingContext(ctx)
vals := md.Get("authorization") // keys are lower-cased
```

Usually you'd check the token in an **interceptor** rather than every handler.

### 5\.3 TLS

```go
// server
creds, err := credentials.NewServerTLSFromFile("server.crt", "server.key")
grpcServer := grpc.NewServer(grpc.Creds(creds))

// client
creds, err := credentials.NewClientTLSFromFile("ca.crt", "")
conn, err := grpc.NewClient("api.example.com:443", grpc.WithTransportCredentials(creds))
```

Use `insecure.NewCredentials()` **only** for local dev.

### 5\.4 Graceful shutdown

```go
go func() {
	sig := make(chan os.Signal, 1)
	signal.Notify(sig, os.Interrupt, syscall.SIGTERM)
	<-sig
	grpcServer.GracefulStop() // stop accepting, finish in-flight RPCs
}()
```

### 5\.5 Health checks

```go
import (
	"google.golang.org/grpc/health"
	healthpb "google.golang.org/grpc/health/grpc_health_v1"
)

hs := health.NewServer()
healthpb.RegisterHealthServer(grpcServer, hs)
hs.SetServingStatus("notes.v1.NoteService", healthpb.HealthCheckResponse_SERVING)
```

Kubernetes supports gRPC health probes natively.

### 5\.6 Testing without a network: `bufconn`

```go
import "google.golang.org/grpc/test/bufconn"

lis := bufconn.Listen(1 << 20)
s := grpc.NewServer()
notesv1.RegisterNoteServiceServer(s, newServer())
go s.Serve(lis)

conn, _ := grpc.NewClient("passthrough:///bufnet",
	grpc.WithContextDialer(func(ctx context.Context, _ string) (net.Conn, error) {
		return lis.DialContext(ctx)
	}),
	grpc.WithTransportCredentials(insecure.NewCredentials()),
)
client := notesv1.NewNoteServiceClient(conn)
```

Fast, in-memory, real gRPC stack — great for unit tests.

### 5\.7 Alternative tooling: `buf`

[`buf`](https://buf.build) replaces raw `protoc` commands with config files, and adds linting + breaking-change detection.

```yaml
# buf.gen.yaml
version: v2
plugins:
  - local: protoc-gen-go
    out: gen
    opt: paths=source_relative
  - local: protoc-gen-go-grpc
    out: gen
    opt: paths=source_relative
```

```bash
buf generate proto
buf lint
buf breaking --against '.git#branch=main'
```

### 5\.8 Exposing gRPC to browsers / REST

- **grpc-gateway** – generates a REST/JSON reverse proxy from annotations in your `.proto`.
- **Connect (connectrpc.com)** – Go library that speaks gRPC, gRPC-Web **and** a simple HTTP/JSON protocol from one handler.
- **gRPC-Web + Envoy** – proxy browser gRPC-Web calls to a normal gRPC server.
- **A Fiber API written by hand** (4.5) – full control over routes, auth, errors and how streams look over HTTP; what [Persephone](page:615lx823qx) does, with [GoFiber](page:phgfxrea1z10a).

---

## 6\. Cheat sheet

| Task | Code |
| --- | --- |
| Generate code | `protoc -I proto --go_out=gen --go_opt=paths=source_relative --go-grpc_out=gen --go-grpc_opt=paths=source_relative proto/.../x.proto` |
| Start server | `s := grpc.NewServer(); pb.RegisterXServer(s, impl); s.Serve(lis)` |
| Create client | `conn, _ := grpc.NewClient(addr, grpc.WithTransportCredentials(...)); c := pb.NewXClient(conn)` |
| Deadline | `ctx, cancel := context.WithTimeout(ctx, 5*time.Second); defer cancel()` |
| Return error | `return nil, status.Error(codes.NotFound, "…")` |
| Inspect error | `st, _ := status.FromError(err); st.Code(); st.Message()` |
| Send metadata | `metadata.AppendToOutgoingContext(ctx, "k", "v")` |
| Read metadata | `md, _ := metadata.FromIncomingContext(ctx)` |
| Middleware | `grpc.NewServer(grpc.UnaryInterceptor(f), grpc.StreamInterceptor(g))` |
| Reflection | `reflection.Register(s)` |
| Graceful stop | `s.GracefulStop()` |
| Call gRPC from Fiber | `ctx, cancel := context.WithTimeout(c.Context(), 5*time.Second); defer cancel(); client.X(ctx, req)` |
| gRPC error → HTTP | `status.FromError(err)` in Fiber's `ErrorHandler`, code → status table in 4.5 |
| Stream to the browser | `c.SendStreamWriter(...)` writing `data: …\n\n` (Server-Sent Events) |

## 7\. Common gotchas

- **`protoc-gen-go: program not found`** → `$(go env GOPATH)/bin` isn't on your `PATH`.
- **`google/protobuf/timestamp.proto: File not found`** → install the well-known protos (`protobuf-devel` on Fedora) or add their directory with another `-I`.
- **Forgot to embed `UnimplementedXServer`** → compile error about `mustEmbedUnimplemented...`. Embed it **by value**.
- **No deadline on client calls** → a hung server hangs your client forever.
- **Changing field numbers** → silently corrupts data between old/new versions. Add new fields with new numbers; `reserved` removed ones.
- **`grpc.Dial` / `grpc.DialContext`** are deprecated → use `grpc.NewClient` (note: it defaults to the `dns` resolver and connects lazily).
- **Older generated code** uses named stream types like `NoteService_ListNotesServer` instead of `grpc.ServerStreamingServer[Note]` — both work; it depends on your `protoc-gen-go-grpc` version.
- **Messages are pointers** – always pass `*pb.Message`, and never copy a message struct by value (`go vet` will warn).

## 8\. References

- gRPC Go quick start: https://grpc.io/docs/languages/go/quickstart/
- gRPC Go basics tutorial: https://grpc.io/docs/languages/go/basics/
- Protobuf language guide (proto3): https://protobuf.dev/programming-guides/proto3/
- grpc-go repo & examples: https://github.com/grpc/grpc-go/tree/master/examples
- Fiber docs: https://docs.gofiber.io/ (and these notes: [GoFiber](page:phgfxrea1z10a))
- Status codes: https://grpc.io/docs/guides/status-codes/
