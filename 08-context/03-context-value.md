# Context Values

## 1️⃣ Simple Explanation (Beginner Friendly)

`context.WithValue` attaches key-value data to context for passing request-scoped information.

```go
// Store value
ctx := context.WithValue(parentCtx, "requestID", "abc-123")

// Retrieve value
reqID := ctx.Value("requestID").(string)
```

---

## 2️⃣ Real-World Analogy

### 🏷️ Luggage Tag

Context value = tag on your luggage. It travels with your bag (request) and anyone handling it can read the tag.

---

## 3️⃣ Technical Working (Step-by-Step)

### Value Lookup

```
Context chain:
ctx3 (key3=val3)
  └── ctx2 (key2=val2)
        └── ctx1 (key1=val1)
              └── Background()

ctx3.Value("key2"):
1. Check ctx3 → not found
2. Check ctx2 → found! return val2

ctx3.Value("missing"):
1. Check ctx3 → not found
2. Check ctx2 → not found
3. Check ctx1 → not found
4. Check Background() → return nil
```

### Best Practice: Custom Key Type

```go
// Define private key type
type contextKey string

const (
    userIDKey contextKey = "userID"
    traceIDKey contextKey = "traceID"
)

// Use typed key (prevents collisions)
ctx := context.WithValue(ctx, userIDKey, "user-123")
userID := ctx.Value(userIDKey).(string)
```

---

## 4️⃣ Where Used in Real Systems?

### Request Tracing

```go
func middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        traceID := r.Header.Get("X-Trace-ID")
        if traceID == "" {
            traceID = generateID()
        }
        ctx := context.WithValue(r.Context(), traceIDKey, traceID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

---

## 5️⃣ Practical Examples

### Common Usage

```go
package main

import (
    "context"
    "fmt"
)

type ctxKey string

const userKey ctxKey = "user"

func main() {
    ctx := context.Background()
    ctx = context.WithValue(ctx, userKey, "john_doe")
    
    processRequest(ctx)
}

func processRequest(ctx context.Context) {
    user, ok := ctx.Value(userKey).(string)
    if !ok {
        fmt.Println("No user in context")
        return
    }
    fmt.Println("Processing for:", user)
}
```

### What NOT to Pass

```go
// DON'T: Pass function arguments via context
ctx = context.WithValue(ctx, "db", database)  // Bad!
processData(ctx, db)  // Good! Pass db directly

// DO: Pass request-scoped metadata
ctx = context.WithValue(ctx, requestIDKey, reqID)  // Good!
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| String keys | Private typed keys |
| Store db, logger | Only request-scoped data |
| Rely on Value in business logic | Optional metadata only |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ WithValue: attach key-value to context     │
│                                            │
│ Best practices:                            │
│ • Private key types (prevent collision)    │
│ • Request-scoped only (traceID, userID)    │
│ • Not for function parameters              │
│ • Type assert when retrieving              │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why use custom key types?
2. What should NOT be stored in context?
3. What happens if key not found?
4. How does value lookup work?
5. Is context.Value thread-safe?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Prevent key collisions between packages using same string.

**A2**: Database, loggers, config - pass as explicit parameters instead.

**A3**: Returns nil (not panic).

**A4**: Walks up the context chain until found or reaches Background.

**A5**: Yes, context is immutable and safe for concurrent use.

</details>

---

[← Previous](./02-context-timeout-cancel.md) | [Next →](./04-context-best-practices.md)
