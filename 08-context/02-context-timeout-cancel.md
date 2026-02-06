# Context Timeout and Cancellation

## 1️⃣ Simple Explanation (Beginner Friendly)

| Function | Purpose |
|----------|---------|
| `WithCancel` | Cancel manually |
| `WithTimeout` | Cancel after duration |
| `WithDeadline` | Cancel at specific time |

```go
// Timeout: 5 seconds from now
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()

// Deadline: specific time
ctx, cancel := context.WithDeadline(ctx, time.Now().Add(5*time.Second))
defer cancel()
```

---

## 2️⃣ Real-World Analogy

### ⏰ Timer vs Alarm

**Timeout** = kitchen timer (5 minutes from now).
**Deadline** = alarm clock (stop at 3:00 PM).

---

## 3️⃣ Technical Working (Step-by-Step)

### Internal Structure

```
WithTimeout internally:
┌─────────────────────────────────────┐
│ WithDeadline(parent,                │
│              time.Now().Add(dur))   │
└─────────────────────────────────────┘

Cancellation flow:
┌─────────────────────────────────────┐
│ 1. Timer fires or cancel() called   │
│ 2. Done channel closed              │
│ 3. Err() returns appropriate error  │
│ 4. Children's Done channels closed  │
└─────────────────────────────────────┘
```

### Error Types

```go
ctx.Err() returns:
├── context.Canceled      // cancel() was called
├── context.DeadlineExceeded  // timeout/deadline hit
└── nil                   // not cancelled yet
```

---

## 4️⃣ Where Used in Real Systems?

### API Call with Timeout

```go
func callAPI(ctx context.Context) (*Response, error) {
    ctx, cancel := context.WithTimeout(ctx, 3*time.Second)
    defer cancel()
    
    req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
    return http.DefaultClient.Do(req)
}
```

---

## 5️⃣ Practical Examples

### Manual Cancellation

```go
ctx, cancel := context.WithCancel(context.Background())

go func() {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Worker stopped:", ctx.Err())
            return
        default:
            doWork()
        }
    }
}()

time.Sleep(2 * time.Second)
cancel()  // Stop the worker
```

### Nested Timeouts

```go
// Outer: 10 second budget
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

// Inner: 3 seconds per call (stricter)
callCtx, callCancel := context.WithTimeout(ctx, 3*time.Second)
defer callCancel()
result := apiCall(callCtx)
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Forget defer cancel() | Always defer cancel for cleanup |
| Inner timeout > outer | Inner must be <= outer |
| Check err before Done | Check ctx.Done() in select |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ WithCancel: manual control                 │
│ WithTimeout: duration from now             │
│ WithDeadline: absolute time                │
│                                            │
│ ctx.Done(): closed on cancel               │
│ ctx.Err(): Canceled or DeadlineExceeded    │
│                                            │
│ ALWAYS: defer cancel()                     │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Difference between Timeout and Deadline?
2. What to do if inner timeout > outer?
3. When is cancel() called automatically?
4. What's ctx.Err() before cancellation?
5. Can you extend a deadline?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Timeout = relative (5s from now). Deadline = absolute (at 3:00 PM).

**A2**: Inner timeout is effectively capped at outer. Set inner <= outer explicitly.

**A3**: On timeout/deadline. But always call cancel() anyway for cleanup.

**A4**: nil - not cancelled yet.

**A5**: No. Context is immutable. Create new context with longer deadline.

</details>

---

[← Previous](./01-what-is-context.md) | [Next →](./03-context-value.md)
