# Rate Limiting

## 1️⃣ Simple Explanation (Beginner Friendly)

**Rate limiting** controls how fast operations can happen. Common patterns:
- **Fixed window**: N requests per time window
- **Token bucket**: Steady refill rate
- **Leaky bucket**: Constant output rate

```go
limiter := rate.NewLimiter(10, 1)  // 10 per second, burst of 1

limiter.Wait(ctx)  // Blocks until allowed
doWork()
```

---

## 2️⃣ Real-World Analogy

### 🚰 Water Faucet

Token bucket = faucet with steady flow. Can fill cup at set rate. Burst = opening faucet fully briefly.

---

## 3️⃣ Technical Working (Step-by-Step)

### Token Bucket Algorithm

```
┌─────────────────────────────────────────────┐
│              TOKEN BUCKET                   │
│                                             │
│    Tokens refill at rate R                  │
│         ▼                                   │
│    ┌─────────────┐                          │
│    │ ○ ○ ○ ○ ○   │ ← Max capacity B         │
│    │   BUCKET    │                          │
│    └──────┬──────┘                          │
│           │                                 │
│    Take 1 token per request                 │
│           ▼                                 │
│    [Request allowed]                        │
│                                             │
│    Empty bucket = wait or reject            │
└─────────────────────────────────────────────┘
```

### golang.org/x/time/rate

```go
// rate.Limiter(r, b)
// r = rate per second
// b = burst size (bucket capacity)

limiter := rate.NewLimiter(10, 3)
// 10 requests/second, burst of 3

limiter.Wait(ctx)    // Block until allowed
limiter.Allow()      // Returns bool, non-blocking
limiter.Reserve()    // Reserve future token
```

---

## 4️⃣ Where Used in Real Systems?

### API Rate Limiting

```go
func rateLimitMiddleware(rps float64, burst int) func(http.Handler) http.Handler {
    limiter := rate.NewLimiter(rate.Limit(rps), burst)
    
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            if !limiter.Allow() {
                http.Error(w, "Rate limit exceeded", 429)
                return
            }
            next.ServeHTTP(w, r)
        })
    }
}
```

---

## 5️⃣ Practical Examples

### Basic Rate Limiter

```go
package main

import (
    "context"
    "fmt"
    "time"
    
    "golang.org/x/time/rate"
)

func main() {
    limiter := rate.NewLimiter(2, 5)  // 2/sec, burst 5
    
    for i := 0; i < 10; i++ {
        limiter.Wait(context.Background())
        fmt.Printf("%d: %v\n", i, time.Now().Format("15:04:05.000"))
    }
}
```

### Per-Client Rate Limiting

```go
type ClientLimiter struct {
    clients map[string]*rate.Limiter
    mu      sync.Mutex
}

func (c *ClientLimiter) GetLimiter(clientID string) *rate.Limiter {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    if limiter, exists := c.clients[clientID]; exists {
        return limiter
    }
    
    limiter := rate.NewLimiter(10, 20)
    c.clients[clientID] = limiter
    return limiter
}
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Global limiter for all clients | Per-client limiters |
| Allow() in tight loop | Use Wait() for blocking |
| Ignore burst parameter | Burst allows temporary spike |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ rate.Limiter(rate, burst):                 │
│ • rate = requests per second               │
│ • burst = momentary spike allowed          │
│                                            │
│ Methods:                                   │
│ • Wait(ctx): block until allowed           │
│ • Allow(): non-blocking check              │
│ • Reserve(): reserve future token          │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What does burst parameter control?
2. Difference between Wait and Allow?
3. How to implement per-user limiting?
4. What's token bucket algorithm?
5. How does Reserve() work?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Maximum tokens that can accumulate. Allows temporary spike above rate.

**A2**: Wait blocks until token available. Allow returns immediately with bool.

**A3**: Map of client ID → *rate.Limiter. Create limiter on first request.

**A4**: Bucket fills with tokens at rate R. Each request takes 1 token. Empty = wait.

**A5**: Returns a Reservation for future use. Can calculate when token will be available.

</details>

---

[← Previous](./03-pipelines.md) | [Next →](./05-semaphore.md)
