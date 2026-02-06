# Context Best Practices

## 1️⃣ Simple Explanation (Beginner Friendly)

Follow these rules for context:

1. **First parameter**, named `ctx`
2. **Don't store** in structs
3. **Always defer cancel()**
4. **Check Done()** in long operations
5. **Use typed keys** for values

---

## 2️⃣ Real-World Analogy

### 📋 Standard Operating Procedures

Context rules = SOPs. Everyone follows the same pattern, making code predictable and maintainable.

---

## 3️⃣ Technical Working (Step-by-Step)

### DO's and DON'Ts

```go
// ✅ DO: First parameter named ctx
func fetch(ctx context.Context, url string) error

// ❌ DON'T: Any other position
func fetch(url string, ctx context.Context) error

// ✅ DO: Pass to functions
db.QueryContext(ctx, query)

// ❌ DON'T: Store in struct
type Service struct {
    ctx context.Context  // BAD!
}

// ✅ DO: Check in loops
for {
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
        doWork()
    }
}

// ❌ DON'T: Ignore ctx.Done()
for {
    doWork()  // Never checks cancellation!
}
```

### Context Flow

```
HTTP Handler
     │
     ├─► ctx := r.Context()
     │
     ├─► Database.QueryContext(ctx)
     │
     ├─► httpClient.Do(req.WithContext(ctx))
     │
     └─► worker.Process(ctx)
           │
           └─► All respect the same ctx!
```

---

## 4️⃣ Where Used in Real Systems?

### Complete Request Flow

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Add tracing
    ctx = context.WithValue(ctx, traceKey, traceID)
    
    // Add timeout for this specific operation
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    // All downstream respect ctx
    data, err := service.Fetch(ctx, id)
}
```

---

## 5️⃣ Practical Examples

### Complete Pattern

```go
package main

import (
    "context"
    "fmt"
    "time"
)

// First param, named ctx
func worker(ctx context.Context, job Job) error {
    for {
        select {
        case <-ctx.Done():
            // Clean up and exit
            return ctx.Err()
        default:
            // Do work chunk
            if done := process(job); done {
                return nil
            }
        }
    }
}

func main() {
    // Always get cancel, always defer
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    if err := worker(ctx, myJob); err != nil {
        fmt.Println("Error:", err)
    }
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| ctx second parameter | ctx first parameter |
| Context in struct field | Pass as parameter |
| Forget defer cancel() | Always defer cancel() |
| context.Background() everywhere | Use parent context |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Best Practices:                            │
│ 1. ctx is first param                      │
│ 2. Don't store ctx in structs              │
│ 3. Always defer cancel()                   │
│ 4. Check ctx.Done() in loops               │
│ 5. Use typed keys for values               │
│ 6. Pass parent ctx, not Background()       │
│ 7. Use TODO() only in WIP code             │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why first parameter convention?
2. Why not store context in struct?
3. When to use context.TODO()?
4. What if you forget cancel()?
5. Why typed keys for values?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Consistency across codebase, easy to spot ctx flow.

**A2**: Context is request-scoped, struct lifetime is different.

**A3**: Only in incomplete code where you'll add proper context later.

**A4**: Resource leak (timer goroutine stays until timeout).

**A5**: Prevent collision between packages using same string.

</details>

---

[← Previous](./03-context-value.md) | [Next →](./05-context-in-real-applications.md)
