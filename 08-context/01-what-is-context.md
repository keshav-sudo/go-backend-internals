# What is Context

## 1️⃣ Simple Explanation (Beginner Friendly)

`context.Context` carries:
1. **Cancellation signals** - Tell goroutines to stop
2. **Deadlines** - Automatic timeout
3. **Values** - Request-scoped data (like request ID)

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

result := doWork(ctx)  // Work respects the timeout
```

---

## 2️⃣ Real-World Analogy

### 🎫 VIP Wristband

Context = wristband with your info, access level, and expiry time. Security (functions) check it at every door.

---

## 3️⃣ Technical Working (Step-by-Step)

### Context Interface

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key any) any
}
```

### Context Tree

```
context.Background()
        │
        ├── WithCancel() 
        │       └── Child can be cancelled
        │
        ├── WithTimeout()
        │       └── Cancels after duration
        │
        ├── WithDeadline()
        │       └── Cancels at specific time
        │
        └── WithValue()
                └── Carries request data
```

### Cancellation Propagation

```
Parent cancelled → All children cancelled
                        │
┌───────────────────────┼────────────────────────┐
│                       │                        │
▼                       ▼                        ▼
Child 1              Child 2                 Child 3
(cancelled)          (cancelled)             (cancelled)
```

---

## 4️⃣ Where Used in Real Systems?

### HTTP Request Handler

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()  // Already has cancellation
    
    result, err := fetchData(ctx)  // Pass context
    if err == context.Canceled {
        return  // Client disconnected
    }
}
```

---

## 5️⃣ Practical Examples

### Timeout Pattern

```go
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

select {
case result := <-doWork(ctx):
    fmt.Println(result)
case <-ctx.Done():
    fmt.Println("Timeout:", ctx.Err())
}
```

### Cancellation Pattern

```go
ctx, cancel := context.WithCancel(context.Background())

go worker(ctx)

time.Sleep(time.Second)
cancel()  // Stop the worker
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Store context in struct | Pass as first parameter |
| Ignore ctx.Done() | Always check in loops |
| Use context.TODO() in prod | Use proper parent context |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Context carries:                           │
│ • Cancellation (Done() channel)            │
│ • Deadline/Timeout                         │
│ • Values (request-scoped)                  │
│                                            │
│ Rules:                                     │
│ • First parameter, named ctx               │
│ • Always defer cancel()                    │
│ • Check ctx.Done() in loops                │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What does ctx.Done() return?
2. When is cancel() called automatically?
3. Should context be stored in struct?
4. What's context.Background() for?
5. Does child cancellation affect parent?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: A channel that closes when context is cancelled.

**A2**: When timeout/deadline is reached. Manual cancel() still needed for cleanup.

**A3**: No! Pass as first function parameter.

**A4**: Root context for main, init, tests. Starting point for chain.

**A5**: No. Only parent → child propagation.

</details>

---

[← Previous Module](../07-networking/05-timeouts-and-graceful-shutdown.md) | [Next →](./02-context-timeout-cancel.md)
