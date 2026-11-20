---
id: c6iofyqc4i
title: "FileServer"
cover: "_assets/c26874a02e34f6cfb9543221dfeb9d1e.jpg"
cover_position: 26
sort: 1000
version: 1
---

# fileserver — Documentation

A Go file server that stores file contents on a pluggable storage backend and keeps file metadata in MySQL. The project is under active development: the storage and metadata layers are implemented, while the server entry point (`main.go`) and the gRPC API (`proto/file.proto`) are still stubs.

- **Module:** `github.com/MensahPrince/fileserver`
- **Go version:** 1.26.3

---

## Project layout

```
.
├── main.go                      # Entry point (currently empty: `package main`)
├── go.mod
├── .env.example                 # Environment template (currently empty)
├── proto/
│   └── file.proto               # gRPC service definition (stub)
├── types/
│   └── types.go                 # Shared domain types (FileMeta)
└── internal/
    ├── config/config.go         # Loads and validates config from env vars
    ├── storage/
    │   ├── storage.go           # Storage interface + DiskStorage implementation
    │   └── storage_test.go      # Path-traversal tests for resolvePath
    ├── metadata/
    │   ├── metadata.go          # Metadata interface
    │   └── mysql.go             # MySQL implementation of Metadata
    └── utils/utils.go           # Path helper (Traversal)
```

## Architecture

The server separates **file bytes** from **file metadata**:

| Concern | Interface | Implementation | Backing store |
| --- | --- | --- | --- |
| Contents | `storage.Storage` | `DiskStorage` | Local filesystem |
| Metadata | `metadata.Metadata` | `MySQLMetadata` | MySQL `metadata` table |

Both are interfaces, so alternative backends (e.g. S3, which `STORAGE_TYPE` already anticipates) can be added without changing callers. Files are addressed by an opaque `id` (e.g. a UUID) shared between the two layers.

---

## Configuration

Configuration is read from environment variables by `config.Load()` (`internal/config/config.go`).

| Variable | Required | Default | Validation |
| --- | --- | --- | --- |
| `GRPC_PORT` | No | `:50051` | Integer 1–65535; a leading `:` is optional |
| `HTTP_PORT` | No | `:8800` | Integer 1–65535; a leading `:` is optional |
| `DB_DSN` | **Yes** | — | Must be non-empty |
| `STORAGE_ROOT` | No | `./data` | None (prints a notice when defaulting) |
| `STORAGE_TYPE` | **Yes** | — | Must be exactly `disk` or `s3` |

`Load()` returns an error if any value fails validation.

Example `.env`:

```env
GRPC_PORT=:50051
HTTP_PORT=:8800
DB_DSN=user:password@tcp(localhost:3306)/fileserver?parseTime=true
STORAGE_ROOT=./data
STORAGE_TYPE=disk
```

> `parseTime=true` is needed with the `go-sql-driver/mysql` driver so `created_at` scans into `time.Time`.

---

## Types

### `types.FileMeta`

```go
type FileMeta struct {
    ID        string
    Name      string
    Hash      string    // hex-encoded SHA-256 of the contents
    Owner     string
    CreatedAt time.Time
    Size      int64     // bytes
}
```

---

## Storage layer (`internal/storage`)

### Interface

```go
type Storage interface {
    Save(ctx context.Context, id string, r io.Reader) (int64, error)
    Open(ctx context.Context, id string) (io.ReadCloser, error)
    Delete(ctx context.Context, id string) error
}
```

### `DiskStorage`

Created with `storage.NewDiskStorage(root string)`. Each file is stored at `<root>/<id>`.

| Method | Behavior | Errors |
| --- | --- | --- |
| `Save` | Creates (or truncates) the file and streams `r` into it. Returns bytes written. | `"path escaped"`, create errors, `failed to save: %w` |
| `Open` | Opens the file for reading. **The caller must `Close()` the returned reader.** | `"path escaped"`, `ErrNotFound`, `failed to open: %w` |
| `Delete` | Removes the file. | `"path escaped"`, `ErrNotFound`, `failed to delete: %w` |

The `root` directory must already exist; `Save` does not create it.

### Sentinel errors

- `storage.ErrNotFound` — the file does not exist. Check with `errors.Is(err, storage.ErrNotFound)`.

### Path-traversal protection

Every method resolves the `id` through `resolvePath`, which rejects any `id` that is not a local path (`filepath.IsLocal`) **before** joining it with `root`. This blocks:

- parent traversal (`../../etc/passwd`, `../main.go`)
- absolute paths (`/etc/passwd`)
- empty ids

Validating the id on its own (rather than the joined path) is deliberate: checking the joined result would reject every id when `root` is absolute, and would let `../x` slip through a relative root because `Join` cleans `data/../x` to `x`.

---

## Metadata layer (`internal/metadata`)

### Interface

```go
type Metadata interface {
    Save(ctx context.Context, meta types.FileMeta) error
    Get(ctx context.Context, id string) (types.FileMeta, error)
    Delete(ctx context.Context, id string) error
    List(ctx context.Context) ([]types.FileMeta, error)
}
```

### `MySQLMetadata`

Created with `metadata.NewMySQLMetadata(db *sql.DB)`. The caller owns opening the `*sql.DB` and importing a MySQL driver.

| Method | SQL | Errors |
| --- | --- | --- |
| `Save` | `INSERT INTO metadata (...)` | `metadata: save failed: %w` |
| `Get` | `SELECT ... WHERE id = ?` | `ErrNotFound` if no row; `metadata: get failed: %w` |
| `Delete` | `DELETE ... WHERE id = ?` | `ErrNotFound` if 0 rows affected; `metadata: delete failed: %w` |
| `List` | `SELECT ...` (all rows, no pagination) | Raw driver errors |

`List` returns a `nil` slice when the table is empty.

### Sentinel errors

- `metadata.ErrNotFound` — no metadata row for that id.

### Database schema

The queries expect a table shaped like this:

```sql
CREATE TABLE metadata (
    id         VARCHAR(64)  NOT NULL PRIMARY KEY,
    name       VARCHAR(255) NOT NULL,
    hash       CHAR(64)     NOT NULL,   -- hex SHA-256
    created_at DATETIME     NOT NULL,
    size       BIGINT       NOT NULL,
    owner      VARCHAR(255) NOT NULL
);
```

---

## Utilities (`internal/utils`)

`utils.Traversal(path string) bool` cleans `path` and returns `true` if it is a local (safe) path. It currently prints debug output to stdout. `DiskStorage` does not use it; it validates paths with `resolvePath` instead.

---

## Usage example

```go
store := storage.NewDiskStorage(cfg.StorageRoot)
meta  := metadata.NewMySQLMetadata(db)

id := uuid.NewString()
n, err := store.Save(ctx, id, uploadReader)
if err != nil { /* handle */ }

err = meta.Save(ctx, types.FileMeta{
    ID: id, Name: "report.pdf", Hash: sha256Hex,
    Owner: "alice", CreatedAt: time.Now(), Size: n,
})

rc, err := store.Open(ctx, id)
if errors.Is(err, storage.ErrNotFound) { /* 404 */ }
defer rc.Close()
```

---

## Testing

```bash
go test ./...
```

`internal/storage/storage_test.go` runs `resolvePath` against both a relative (`./data`) and an absolute (`/var/lib/fileserver`) root, checking that a valid UUID is accepted and that traversal, absolute, and empty ids are rejected.

---

## Status and known gaps

- `main.go` declares only `package main`; there is no server wiring or `main()` yet.
- `proto/file.proto` declares only the package; no messages or services are defined. Its `go_package` (`dotasks/gen/task`) and package (`task`) don't match this module yet.
- `.env.example` and `README.md` are empty.
- No MySQL driver is listed in `go.mod`, and there is no schema migration.
- `STORAGE_TYPE=s3` passes validation, but there is no S3 implementation.
- `types.MySQLMetadata` in `types/types.go` duplicates `metadata.MySQLMetadata` and is unused.
- Storing bytes and metadata are separate operations with no transaction across them, so a failure between the two can leave an orphaned file or row.
