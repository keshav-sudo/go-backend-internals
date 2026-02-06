# Timeouts and Graceful Shutdown

## 1️⃣ Simple Explanation (Beginner Friendly)

**Timeouts** prevent slow clients from holding resources forever.
**Graceful shutdown** finishes in-flight requests before stopping.

```go
srv := &http.Server{
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
}

// Graceful shutdown
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
srv.Shutdown(ctx)
```

---

## 2️⃣ Real-World Analogy

### 🏪 Store Closing

Timeouts = service time limit per customer.
Graceful = "We're closing, but we'll finish serving current customers."

---

## 3️⃣ Technical Working (Step-by-Step)

### Server Timeouts

```
┌────────────────────────────────────────────────────────┐
│                    CLIENT REQUEST                      │
└────────────────────────────────────────────────────────┘
        │              │              │
        │ ReadTimeout  │ WriteTimeout │
        │    ↓         │     ↓        │
        ├──────────────┼──────────────┤
        │  Read        │   Write      │
        │  Headers +   │   Response   │
        │  Body        │              │
        ├──────────────┴──────────────┤
        │       IdleTimeout           │
        │  (between requests)         │
        └─────────────────────────────┘
```

### Graceful Shutdown Flow

```
Signal received (SIGTERM/SIGINT)
         │
         ▼
┌─────────────────────────────────┐
│ 1. Stop accepting new conns     │
│ 2. Wait for active requests     │
│ 3. Close idle connections       │
│ 4. Timeout exceeded? Force close│
└─────────────────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Production Server Pattern

```go
srv := &http.Server{
    Addr:         ":8080",
    Handler:      handler,
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  60 * time.Second,
}

go func() {
    if err := srv.ListenAndServe(); err != http.ErrServerClosed {
        log.Fatal(err)
    }
}()

// Wait for shutdown signal
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
srv.Shutdown(ctx)
```

---

## 5️⃣ Practical Examples

### Complete Production Server

```go
package main

import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    srv := &http.Server{
        Addr:         ":8080",
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  60 * time.Second,
        Handler: http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            time.Sleep(2 * time.Second)  // Simulate work
            w.Write([]byte("OK"))
        }),
    }

    go func() {
        log.Println("Starting server on :8080")
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()

    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    log.Println("Shutting down...")

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        log.Fatal("Forced shutdown:", err)
    }
    log.Println("Server stopped gracefully")
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| No timeouts in production | Always set all timeout types |
| ListenAndServe blocks | Run in goroutine for shutdown |
| Close() for shutdown | Use Shutdown() for graceful |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Timeouts:                                  │
│ • ReadTimeout: reading request             │
│ • WriteTimeout: writing response           │
│ • IdleTimeout: between requests            │
│                                            │
│ Graceful Shutdown:                         │
│ • srv.Shutdown(ctx) - finish in-flight     │
│ • srv.Close() - immediate (not graceful)   │
│ • Listen for SIGTERM/SIGINT                │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What happens without WriteTimeout?
2. What signal does Kubernetes send?
3. Difference between Shutdown and Close?
4. What if client is slower than ReadTimeout?
5. When is IdleTimeout used?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Slow clients can hold connections forever (resource exhaustion).

**A2**: SIGTERM by default (15 second grace period).

**A3**: Shutdown() waits for active requests. Close() immediate termination.

**A4**: Connection closed, request fails. Client gets timeout error.

**A5**: For keep-alive connections between requests.

</details>

---

[← Previous](./04-connection-pooling.md) | [Next Module: Context →](../08-context/01-what-is-context.md)
