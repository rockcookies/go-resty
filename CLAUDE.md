# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Testing
```bash
# Run all tests with race detection and coverage
go run gotest.tools/gotestsum@latest -f testname -- ./... -race -count=1 -coverprofile=coverage.txt -covermode=atomic -coverpkg=./... -shuffle=on

# Run tests for specific package
go test ./... -v

# Run benchmark tests
go test -bench=. -benchmem
```

### Code Quality
```bash
# Format code (check for formatting issues)
go fmt $(go list ./...)

# Verify no formatting differences
diff -u <(echo -n) <(go fmt $(go list ./...))
```

### Build
```bash
# Build the module
go build ./...

# Run mod tidy to clean dependencies
go mod tidy
```

## Repository Architecture

### Core Package Structure
This is the `go-resty` HTTP client library with the following key architectural components:

**Main Files:**
- `resty.go` - Entry point with client constructors (New, NewWithClient, etc.)
- `client.go` - Core Client struct and HTTP client functionality
- `request.go` - Request building and execution logic
- `response.go` - Response handling and processing

**Key Features:**
- `middleware.go` - HTTP middleware chain implementation
- `retry.go` - Automatic retry logic with configurable strategies
- `redirect.go` - HTTP redirect handling
- `digest.go` - Digest authentication support
- `sse.go` - Server-Sent Events (SSE) client
- `circuit_breaker.go` - Circuit breaker pattern implementation
- `load_balancer.go` - Load balancing for multiple endpoints

**Platform-Specific:**
- `transport_dial.go` - Standard Go transport dialer
- `transport_dial_wasm.go` - WebAssembly-specific transport

### Version Information
- Current development: `v3` branch (v3.0.0-beta.4)
- Legacy support: `v2` branch
- Go vanity URL: `resty.dev/v3`
- Minimum Go version: `1.23`

### Testing Strategy
- Comprehensive test coverage with race condition detection
- Benchmark tests for performance validation
- Platform-specific test variations (standard vs WASM)
- Test shuffling enabled for reliability

### Module Information
- Module name: `github.com/rockcookies/go-resty`
- Primary dependency: `golang.org/x/net v0.43.0`
- MIT License

## Important Notes

### Development Workflow
- All changes must include test coverage
- Use `gotestsum` for comprehensive testing with coverage reporting
- Format code with `go fmt` before commits
- Tests run on multiple Go versions in CI (stable, 1.23.x)

### Branch Strategy
- `v3` - Current development branch (beta)
- `v2` - Legacy stable branch
- `main` - Primary integration target

### Code Patterns
- Consistent error handling patterns across all components
- Middleware chain architecture for extensibility
- Context-based request cancellation and timeouts
- Interface-based design for testability and mocking

### Client Configuration
The Client struct contains many configurable fields that can be moved to middleware for better separation of concerns:
- **Query Parameters**: `queryParams` field + `SetQueryParam()` methods
- **Form Data**: `formData` field + `SetFormData()` methods
- **Headers**: `header` field + `SetHeader()` methods
- **Authentication**: `authToken`, `authScheme`, `credentials` fields + auth methods
- **Method Payloads**: `allowMethodGetPayload`, `allowMethodDeletePayload` fields
- **Query Escaping**: `unescapeQueryParams` field
- **Custom Auth Keys**: `headerAuthorizationKey` field

### Middleware System
The library uses a powerful middleware architecture:
```go
// Request middleware chain
type RequestMiddleware func(*Client, *Request) error

// Response middleware chain
type ResponseMiddleware func(*Client, *Response) error

// Default middlewares
- PrepareRequestMiddleware: Request preparation and URL parsing
- AutoParseResponseMiddleware: Automatic response body parsing
- SaveToFileResponseMiddleware: Response saving to files
```

### Key High-Level Features
- **Authentication**: Basic Auth, Digest Auth (RFC 7616), Bearer Tokens, custom auth schemes
- **Retry Logic**: Exponential backoff with jitter, custom retry conditions, configurable strategies
- **Circuit Breaker**: Failure threshold management with three states (closed, open, half-open)
- **Load Balancing**: Round-robin algorithm for multiple endpoints
- **Server-Sent Events**: Real-time event stream processing
- **Streaming**: Large response handling with memory-efficient processing
- **Debug Tools**: cURL command generation, detailed request/response logging