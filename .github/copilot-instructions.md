# Go-Resty Copilot Instructions

## Project Overview

This is `go-resty` - a powerful HTTP, REST, and SSE client library for Go. Current development is on `v3` branch (v3.0.0-beta.4), targeting `resty.dev/v3`. This is a fork focused on specific enhancements.

**Core Architecture:** Request → Middleware Chain → Transport → Response → Middleware Chain

## Key Components

### Client-Request-Response Pattern
- **Client** (`client.go`): Reusable client with shared configuration (base URL, headers, timeouts, middlewares)
- **Request** (`request.go`): Per-request settings that override client defaults
- **Response** (`response.go`): Wraps `http.Response` with convenience methods

```go
// Standard usage pattern
client := resty.New()
resp, err := client.R().
    SetHeader("Accept", "application/json").
    SetResult(&MyStruct{}).
    Get("https://api.example.com/users")
```

### Middleware Architecture
Two-phase middleware system in `middleware.go`:
- **RequestMiddleware**: Executed before HTTP request (e.g., `PrepareRequestMiddleware`)
- **ResponseMiddleware**: Executed after response received (e.g., `AutoParseResponseMiddleware`)

Add custom middleware with `client.AddRequestMiddleware()` or `client.AddResponseMiddleware()`.

### Advanced Features

**Circuit Breaker** (`circuit_breaker.go`): Three-state machine (Closed → Open → Half-Open)
- Default: 3 failures → Open, 10s timeout, 1 success → Closed
- Policies evaluate response to determine failure/success

**Load Balancer** (`load_balancer.go`): Round-robin algorithm for multiple endpoints
- Implements `LoadBalancer` interface with `Next()`, `Feedback()`, `Close()`
- Use `NewRoundRobin(baseURLs...)` then `client.SetLoadBalancer(lb)`

**Retry Logic** (`retry.go`): Exponential backoff with jitter
- Default conditions: 429, 5xx (except 501), connection errors
- Customize via `SetRetryCount()`, `AddRetryCondition()`, `SetRetryStrategy()`
- Retry trace ID for debugging multiple attempts

**Server-Sent Events** (`sse.go`): Streaming event support
- Use `client.EventSource(url)` with `.OnOpen()`, `.OnMessage()`, `.OnError()` callbacks
- Auto-reconnection with configurable backoff

## Development Workflow

### Testing
```bash
# Run full test suite with race detection and coverage (CI standard)
go run gotest.tools/gotestsum@latest -f testname -- ./... -race -count=1 -coverprofile=coverage.txt -covermode=atomic -coverpkg=./... -shuffle=on

# Quick test run
go test ./... -v

# Benchmark tests
go test -bench=. -benchmem
```

**Test Coverage Requirements:** All changes must include tests with 100% patch coverage.

### Code Quality
```bash
# Format check (CI enforced)
diff -u <(echo -n) <(go fmt $(go list ./...))

# Format code
go fmt $(go list ./...)

# Clean dependencies
go mod tidy
```

### Build
```bash
go build ./...
```

## Code Conventions

### Error Handling
- Define package-level errors: `var ErrNotHttpTransportType = errors.New("resty: not a http.Transport type")`
- Prefix error messages with `"resty: "` for clarity

### Naming Patterns
- HTTP methods: `MethodGet`, `MethodPost`, etc. (constants in `client.go`)
- Header keys: Canonicalized with `http.CanonicalHeaderKey()` prefix `hdr` (e.g., `hdrUserAgentKey`, `hdrContentTypeKey`)
- Middleware function types: `RequestMiddleware`, `ResponseMiddleware`
- Hook function types: `ErrorHook`, `SuccessHook`, `CloseHook`

### Struct Tags
- Use `json` and `xml` tags for marshal/unmarshal
- Content-type detection: Check for `json` or `xml` tags in struct fields

### Debugging
- `SetDebug(true)` enables request/response logging
- `EnableTrace()` provides timing information via `TraceInfo`
- `GenerateCurlCommand()` creates equivalent cURL command for requests
- Custom debug formatters: `DebugLogFormatter` (default) or `DebugLogJSONFormatter`

## File Organization

**Core files:**
- `resty.go` - Client constructors (`New()`, `NewWithClient()`, etc.)
- `client.go` - Client struct and configuration methods
- `request.go` - Request building, execution (`Get()`, `Post()`, `Execute()`)
- `response.go` - Response parsing and helpers
- `middleware.go` - Built-in middleware implementations

**Feature files:**
- `retry.go`, `circuit_breaker.go`, `load_balancer.go` - Reliability patterns
- `sse.go` - Server-Sent Events
- `stream.go` - Streaming responses
- `multipart.go` - Multipart form handling
- `debug.go` - Debug logging infrastructure
- `trace.go` - Request tracing
- `curl.go` - cURL command generation

**Platform-specific:**
- `transport_dial.go` - Standard transport
- `transport_dial_wasm.go` - WebAssembly builds

## Testing Patterns

Tests use table-driven approach with helper functions:
- `assertEqual(t, expected, actual)` - Value assertions
- `assertError(t, err)` - Check err is nil
- `assertNil(t, value)` - Nil checks
- `dcnl()` - Create default client for tests (defined in test files)
- Mock servers: `createAuthServer(t)`, `createGenericServer(t)`, etc.

## Common Pitfalls

1. **Transport Compression**: Resty sets `DisableCompression: true` on `http.Transport` - use `AddContentDecoder()` for decompression
2. **Thread Safety**: Client uses `sync.RWMutex` - safe for concurrent use
3. **Context Cancellation**: Set context via `SetContext()` or per-request `SetContext()`
4. **Path Parameters**: Use `{paramName}` in URL, set via `SetPathParam("paramName", "value")`
5. **Content-Type Auto-detection**: Based on body type and struct tags - override with `SetHeader("Content-Type", "...")`

## Module Information
- Module: `github.com/rockcookies/go-resty`
- Minimum Go: `1.23`
- Main dependency: `golang.org/x/net`
- License: MIT
