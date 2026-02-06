# net/http Internals

## 1️⃣ Simple Explanation (Beginner Friendly)

Go's `net/http` package provides a production-ready HTTP server in the standard library. Each request gets its own goroutine!

```go
http.HandleFunc("/", handler)
http.ListenAndServe(":8080", nil)
```

---

## 2️⃣ Real-World Analogy

### 🏨 Hotel Reception

Server = hotel lobby. Listeners = receptionists. Each guest (request) gets a personal assistant (goroutine) to handle their needs.

---

## 3️⃣ Technical Working (Step-by-Step)

### Server Architecture

```
http.ListenAndServe(":8080", nil)
          │
          ▼
┌─────────────────┐
│  net.Listener   │  Accept TCP connections
└────────┬────────┘
         │
         ▼ (for each connection)
┌─────────────────┐
│  go c.serve()   │  New goroutine per connection
└────────┬────────┘
         │
         ▼ (for each request on connection)
┌─────────────────┐
│  Handler.Serve  │  Call registered handler
│  Request/Write  │
└─────────────────┘
```

### Key Types

```go
// Server manages connections
type Server struct {
    Addr    string
    Handler Handler  // nil = DefaultServeMux
}

// Handler interface
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

// The mux
type ServeMux struct {
    m map[string]muxEntry
}
```

---

## 4️⃣ Where Used in Real Systems?

### Default Server Problems

```go
// PROBLEMATIC: No timeouts
http.ListenAndServe(":8080", nil)

// PRODUCTION: Configure timeouts
srv := &http.Server{
    Addr:         ":8080",
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  60 * time.Second,
}
srv.ListenAndServe()
```

---

## 5️⃣ Practical Examples

### Basic Server

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World!")
    })
    
    http.ListenAndServe(":8080", nil)
}
```

### Custom Handler

```go
type apiHandler struct {
    db Database
}

func (h *apiHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Access h.db...
    w.Write([]byte("OK"))
}

func main() {
    handler := &apiHandler{db: connectDB()}
    http.ListenAndServe(":8080", handler)
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| DefaultServeMux is fine | Has no timeouts, less secure |
| One goroutine per request | One per connection (HTTP/1.1) |
| Handler must be function | Must implement Handler interface |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • Goroutine per connection                 │
│ • Handler interface for custom handling    │
│ • ServeMux for pattern routing             │
│                                            │
│ Production must-haves:                     │
│ • Timeouts (Read, Write, Idle)             │
│ • Graceful shutdown                        │
│ • Custom error handling                    │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. How many goroutines per connection?
2. What interface must handlers implement?
3. Why are default timeouts dangerous?
4. What's the default handler if nil?
5. How does HTTP/2 change the model?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: One goroutine per connection (not per request in HTTP/1.1).

**A2**: `Handler` interface with `ServeHTTP(ResponseWriter, *Request)`.

**A3**: No timeouts = slowloris attacks, resource exhaustion.

**A4**: DefaultServeMux (global mux).

**A5**: HTTP/2 multiplexes requests over single connection, may use more goroutines.

</details>

---

[← Previous Module](../06-memory-management/05-memory-leaks.md) | [Next →](./02-goroutine-per-connection.md)
