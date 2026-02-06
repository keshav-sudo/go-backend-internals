# Connection Pooling

## 1️⃣ Simple Explanation (Beginner Friendly)

**Connection pooling** reuses connections instead of creating new ones. Creating TCP connections is expensive!

```go
// Default HTTP client pools connections
client := &http.Client{}
client.Get(url)  // Connection reused for future requests
```

---

## 2️⃣ Real-World Analogy

### 🚕 Taxi vs Personal Car

Without pooling = calling new taxi each time.
With pooling = owning a car you reuse.

---

## 3️⃣ Technical Working (Step-by-Step)

### Client-Side Pooling

```
┌────────────────────────────────────────────┐
│          http.Transport                    │
│  ┌──────────────────────────────────────┐ │
│  │  Connection Pool                      │ │
│  │  ┌────────────────────────────────┐   │ │
│  │  │ host:port → [conn, conn, conn] │   │ │
│  │  │ host:port → [conn, conn]       │   │ │
│  │  └────────────────────────────────┘   │ │
│  └──────────────────────────────────────┘ │
└────────────────────────────────────────────┘

Request arrives:
1. Check pool for idle connection to host
2. Found? → Reuse
3. Not found? → Create new (if under limit)
4. At limit? → Wait for available connection
```

### Key Config

```go
transport := &http.Transport{
    MaxIdleConns:        100,              // Total idle
    MaxIdleConnsPerHost: 10,               // Per host
    MaxConnsPerHost:     20,               // Max per host
    IdleConnTimeout:     90 * time.Second, // Cleanup idle
}
```

---

## 4️⃣ Where Used in Real Systems?

### Database Connection Pools

```go
db, _ := sql.Open("postgres", connStr)
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(10)
db.SetConnMaxLifetime(5 * time.Minute)
```

---

## 5️⃣ Practical Examples

### HTTP Client Pool

```go
var client = &http.Client{
    Transport: &http.Transport{
        MaxIdleConnsPerHost: 100,
        IdleConnTimeout:     90 * time.Second,
    },
    Timeout: 10 * time.Second,
}

func fetch(url string) {
    resp, _ := client.Get(url)  // Pooled!
    defer resp.Body.Close()
    io.Copy(io.Discard, resp.Body)  // IMPORTANT: drain body
}
```

### Common Bug: Not Draining Body

```go
// BUG: Connection not reused!
resp, _ := client.Get(url)
// forget to read body
resp.Body.Close()

// FIX: Always drain body
resp, _ := client.Get(url)
io.Copy(io.Discard, resp.Body)
resp.Body.Close()
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Close body = connection reused | Must drain body first |
| New client per request | Reuse single client |
| Default limits are enough | Tune for your usage |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Connection pooling avoids TCP overhead     │
│                                            │
│ http.Client: automatic pooling             │
│ sql.DB: built-in pool                      │
│                                            │
│ Key settings:                              │
│ • MaxIdleConnsPerHost                      │
│ • MaxConnsPerHost                          │
│ • IdleConnTimeout                          │
│                                            │
│ MUST drain response body for reuse!        │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why is connection creation expensive?
2. What must you do with response body?
3. What's MaxIdleConnsPerHost default?
4. How does sql.DB handle pooling?
5. When are connections removed from pool?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: TCP handshake (3-way), TLS handshake (if HTTPS), socket creation overhead.

**A2**: Read it fully (drain) before closing, or connection can't be reused.

**A3**: 2 (very low! increase for high-volume services).

**A4**: sql.DB IS a connection pool. SetMaxOpenConns/SetMaxIdleConns.

**A5**: IdleConnTimeout, MaxConnLifetime, or when server closes them.

</details>

---

[← Previous](./03-netpoller.md) | [Next →](./05-timeouts-and-graceful-shutdown.md)
