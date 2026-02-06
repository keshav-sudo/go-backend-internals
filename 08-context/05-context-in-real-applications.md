# Context in Real Applications

## 1️⃣ Simple Explanation (Beginner Friendly)

In real applications, context flows through your entire request lifecycle:

```
HTTP Request → Middleware → Handler → Service → Database
                    ↓
              context flows through all layers
```

---

## 2️⃣ Real-World Analogy

### 🎟️ Event Wristband

Your wristband (context) is checked at entry, food stands, VIP areas - everywhere you go throughout the event.

---

## 3️⃣ Technical Working (Step-by-Step)

### Complete Request Flow

```
┌─────────────────────────────────────────────────────────┐
│                    HTTP Server                          │
│   r.Context() ← base context with connection deadline   │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────┐
│                  Trace Middleware                       │
│   WithValue(ctx, traceID, "abc-123")                    │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────┐
│                  Auth Middleware                        │
│   WithValue(ctx, userKey, user)                         │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────┐
│                    Handler                              │
│   WithTimeout(ctx, 5*time.Second)                       │
└───────────────────────────┬─────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
   ┌─────────┐        ┌─────────┐         ┌─────────┐
   │   DB    │        │  Cache  │         │   API   │
   │ Query   │        │  Get    │         │  Call   │
   └─────────┘        └─────────┘         └─────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Microservices Request

```go
// Gateway receives request
func Gateway(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Add tracing
    ctx = context.WithValue(ctx, traceIDKey, uuid.New().String())
    
    // Set overall timeout
    ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()
    
    // Call downstream services - all respect same ctx
    userResp, _ := userService.Get(ctx, userID)
    orderResp, _ := orderService.List(ctx, userID)
    
    // If ctx cancelled, all calls stop
}
```

---

## 5️⃣ Practical Examples

### Complete Application Pattern

```go
package main

import (
    "context"
    "database/sql"
    "net/http"
    "time"
)

type contextKey string

const (
    traceIDKey contextKey = "traceID"
    userKey    contextKey = "user"
)

// Middleware adds trace ID
func traceMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        traceID := r.Header.Get("X-Trace-ID")
        if traceID == "" {
            traceID = generateID()
        }
        ctx := context.WithValue(r.Context(), traceIDKey, traceID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// Handler uses context
func getUser(db *sql.DB) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx := r.Context()
        
        // Timeout for this operation
        ctx, cancel := context.WithTimeout(ctx, 3*time.Second)
        defer cancel()
        
        // DB respects context
        row := db.QueryRowContext(ctx, "SELECT * FROM users WHERE id = ?", id)
        
        // Check if cancelled during processing
        select {
        case <-ctx.Done():
            http.Error(w, ctx.Err().Error(), 504)
            return
        default:
            // Continue
        }
    }
}
```

### Graceful Shutdown with Context

```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    
    go func() {
        sig := make(chan os.Signal, 1)
        signal.Notify(sig, syscall.SIGINT)
        <-sig
        cancel()  // Stop all workers
    }()
    
    // Workers check ctx.Done()
    go worker(ctx, "worker-1")
    go worker(ctx, "worker-2")
    
    <-ctx.Done()
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| New background for each call | Pass parent context through |
| No timeout in HTTP handlers | Always set handler timeout |
| Ignore DB QueryContext | Use *Context variants |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Real Application Context Flow:             │
│                                            │
│ 1. Start: r.Context() in handler           │
│ 2. Enrich: middleware adds values          │
│ 3. Limit: handler adds timeout             │
│ 4. Propagate: pass to all downstream       │
│ 5. Respect: check Done() in long ops       │
│ 6. Exit: return ctx.Err() on cancel        │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Where does initial context come from in HTTP?
2. How to cancel all downstream when request cancelled?
3. Why use QueryContext vs Query?
4. How does graceful shutdown use context?
5. What happens when client disconnects?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: r.Context() - created by http.Server with connection deadline.

**A2**: Pass same ctx through entire call chain. All check ctx.Done().

**A3**: QueryContext respects cancellation. Query ignores it.

**A4**: Cancel root context → all goroutines see Done() → clean exit.

**A5**: r.Context() is cancelled. Handler should check and stop work.

</details>

---

[← Previous](./04-context-best-practices.md) | [Next Module: Concurrency Patterns →](../09-concurrency-patterns/01-worker-pools.md)
