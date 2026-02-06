# Goroutine-Per-Connection Model

## 1️⃣ Simple Explanation (Beginner Friendly)

Go spawns a **goroutine for each connection**, not each request. This is simple yet scales well because goroutines are cheap.

```
Connection 1 → Goroutine 1 (handles multiple requests)
Connection 2 → Goroutine 2
Connection 3 → Goroutine 3
...
Connection N → Goroutine N
```

---

## 2️⃣ Real-World Analogy

### 📞 Phone Support

Each phone line (connection) gets one agent (goroutine). The agent handles everything on that line until the call ends.

---

## 3️⃣ Technical Working (Step-by-Step)

### Connection Flow

```
http.Server.Serve():
┌────────────────────────────────────┐
│ for {                              │
│     conn := listener.Accept()      │
│     go c.serve()  // One G per conn│
│ }                                  │
└────────────────────────────────────┘

conn.serve():
┌────────────────────────────────────┐
│ for {                              │
│     req := readRequest()           │
│     handler.ServeHTTP(w, req)      │
│     if !keepAlive { break }        │
│ }                                  │
└────────────────────────────────────┘
```

### Resource Math

```
10,000 connections = 10,000 goroutines
Each goroutine ≈ 2KB-8KB stack

10,000 × 4KB = 40MB (manageable!)

Compare to Java threads:
10,000 × 1MB = 10GB (impossible!)
```

---

## 4️⃣ Where Used in Real Systems?

### High-Connection Servers

```go
srv := &http.Server{
    Addr: ":8080",
    Handler: handler,
    // Limit connections
    MaxHeaderBytes: 1 << 20,
}

// Can handle 100K+ concurrent connections
```

---

## 5️⃣ Practical Examples

### Counting Active Connections

```go
var activeConns int64

func connTracker(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        atomic.AddInt64(&activeConns, 1)
        defer atomic.AddInt64(&activeConns, -1)
        next.ServeHTTP(w, r)
    })
}

func main() {
    handler := connTracker(myHandler)
    http.ListenAndServe(":8080", handler)
}
```

### Connection Limiting

```go
sem := make(chan struct{}, 1000)  // Max 1000 concurrent

func handler(w http.ResponseWriter, r *http.Request) {
    select {
    case sem <- struct{}{}:
        defer func() { <-sem }()
        process(w, r)
    default:
        http.Error(w, "Too many requests", 503)
    }
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Goroutine per request | Per connection (in HTTP/1.1) |
| Unlimited is always OK | Can exhaust memory with many conns |
| All connections are equal | Keep-alive vs short-lived differs |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ • One goroutine per connection             │
│ • Handles multiple requests (keep-alive)   │
│ • Scales to 100K+ connections              │
│                                            │
│ Memory: connections × ~4KB                 │
│ Limit via semaphore if needed              │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why goroutine per connection, not per request?
2. How much memory do 100K connections use?
3. How do you limit concurrent connections?
4. What happens with keep-alive connections?
5. How does HTTP/2 differ?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Keep-alive allows multiple requests per connection. Creating/destroying goroutines per request would be wasteful.

**A2**: ~100K × 4KB = 400MB (rough estimate).

**A3**: Semaphore pattern: buffered channel as counter.

**A4**: Same goroutine handles all requests on that connection.

**A5**: HTTP/2 multiplexes streams, may need more goroutines per connection.

</details>

---

[← Previous](./01-net-http-internals.md) | [Next →](./03-netpoller.md)
