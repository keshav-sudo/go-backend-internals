# Comparison with Node.js and Java Threading Model

## 1️⃣ Simple Explanation (Beginner Friendly)

Every backend language handles concurrent requests differently. Understanding these differences helps you choose the right tool and write better code.

| Language | Model | Key Characteristic |
|----------|-------|-------------------|
| **Go** | M:N threading (goroutines) | Many lightweight goroutines on few OS threads |
| **Node.js** | Single-threaded event loop | One thread, async callbacks |
| **Java** | 1:1 threading | One OS thread per concurrent task |

---

## 2️⃣ Real-World Analogy

### 🏢 Office Building Analogy

**Java Office**: Each employee (thread) has their own desk. 1000 employees = 1000 desks. Expensive!

**Node.js Office**: One super-fast receptionist (event loop) handles all calls. Efficient for calls, but can't do heavy lifting.

**Go Office**: Few desks (OS threads), but employees (goroutines) are virtual - they hot-desk. 1000 workers share 8 desks efficiently.

---

## 3️⃣ Technical Working (Step-by-Step)

### Threading Model Comparison

```
┌───────────────────────────────────────────────────────────────┐
│                         JAVA                                  │
├───────────────────────────────────────────────────────────────┤
│  Thread 1    Thread 2    Thread 3    Thread 4                │
│  [1MB stack] [1MB stack] [1MB stack] [1MB stack]             │
│      ↓           ↓           ↓           ↓                   │
│  [OS Thread] [OS Thread] [OS Thread] [OS Thread]             │
│                                                               │
│  1:1 mapping - expensive context switches                    │
└───────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────┐
│                       NODE.JS                                 │
├───────────────────────────────────────────────────────────────┤
│         Callbacks: cb1, cb2, cb3, cb4, cb5...                │
│                          ↓                                    │
│                  ┌───────────────┐                           │
│                  │  Event Loop   │ ← Single Thread           │
│                  └───────────────┘                           │
│                          ↓                                    │
│                  ┌───────────────┐                           │
│                  │    libuv      │ → Thread pool for I/O    │
│                  └───────────────┘                           │
└───────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────┐
│                          GO                                   │
├───────────────────────────────────────────────────────────────┤
│  G1 G2 G3 G4 G5 G6 G7 G8 G9... (thousands of goroutines)     │
│            ↓                                                  │
│      ┌────────────────────────┐                              │
│      │ P1    P2    P3    P4   │  ← Processors (GOMAXPROCS)   │
│      └────────────────────────┘                              │
│            ↓                                                  │
│      ┌────────────────────────┐                              │
│      │ M1    M2    M3    M4   │  ← OS Threads                │
│      └────────────────────────┘                              │
│                                                               │
│  M:N mapping - cheap context switches within runtime         │
└───────────────────────────────────────────────────────────────┘
```

### Memory Per Concurrent Task

| Language | Memory/Task | 10,000 Tasks |
|----------|-------------|--------------|
| Java | ~1 MB | ~10 GB |
| Go | ~2 KB | ~20 MB |
| Node.js | ~negligible | N/A (single-threaded) |

### Blocking Behavior

```go
// GO: Blocking one goroutine doesn't block others
go func() {
    time.Sleep(10 * time.Second)  // Blocks this goroutine only
}()
go func() {
    fmt.Println("I run immediately!")  // Runs on different goroutine
}()
```

```javascript
// NODE.JS: Blocking the event loop blocks EVERYTHING
setTimeout(() => console.log("delayed"), 1000);
while(true) {}  // BLOCKS ENTIRE SERVER!
```

```java
// JAVA: Each thread is independent
new Thread(() -> {
    Thread.sleep(10000);  // Blocks this thread only
}).start();
```

---

## 4️⃣ Where Used in Real Systems?

| Use Case | Best Choice | Why |
|----------|-------------|-----|
| Real-time APIs | Go | Low latency, high concurrency |
| I/O-heavy web apps | Node.js | Event loop efficient for I/O |
| Enterprise systems | Java | Mature ecosystem, JIT optimization |
| Microservices | Go | Small binaries, fast startup |
| CPU-heavy processing | Go/Java | True parallelism |

---

## 5️⃣ Practical Examples

### Go: Handling 10K Connections

```go
package main

import (
    "fmt"
    "net/http"
    "runtime"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Goroutines: %d", runtime.NumGoroutine())
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
    // Each request = 1 goroutine (~2KB)
    // 10K requests = ~20MB memory
}
```

### Node.js Equivalent

```javascript
const http = require('http');

http.createServer((req, res) => {
    res.end('Hello');
}).listen(8080);
// Single thread handles all requests
// CPU-heavy work blocks everything!
```

### Java Equivalent

```java
// Traditional: Thread per request
ExecutorService pool = Executors.newFixedThreadPool(200);
// 200 threads max, each ~1MB stack
// Beyond 200 = queueing
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Node.js is single-threaded = slow | It's efficient for I/O, just not CPU work |
| Go is always faster than Java | Java JIT can optimize hot paths over time |
| More Java threads = more performance | Thread overhead limits scalability |
| Goroutines are like async/await | Goroutines are preemptively scheduled |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────────────────────┐
│ GO:     M:N threading, lightweight goroutines, ~2KB each  │
│ NODE:   Single event loop, great I/O, blocks on CPU work  │
│ JAVA:   1:1 threading, ~1MB per thread, JIT optimized     │
│                                                            │
│ Choose Go for: Microservices, real-time, high concurrency │
│ Choose Node: I/O-heavy web apps, rapid prototyping        │
│ Choose Java: Enterprise, long-running optimized services  │
└────────────────────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why can Go handle more concurrent connections than Java with same RAM?
2. What happens in Node.js if you run CPU-intensive code in a request handler?
3. How does Go's scheduler differ from OS thread scheduling?
4. When would Java outperform Go?
5. Why is Go preferred for Kubernetes/Docker over Java?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Go goroutines use ~2KB stack vs Java's ~1MB. 10GB RAM = 5M goroutines vs 10K Java threads.

**A2**: Blocks the entire event loop! All other requests wait. Use worker threads for CPU work.

**A3**: Go scheduler runs in userspace with cooperative + preemptive scheduling. OS scheduling is more expensive (kernel transitions).

**A4**: Long-running services where JIT optimization kicks in. JVM optimizes hot paths over time.

**A5**: Small binaries (10MB vs 200MB+), instant startup (<100ms vs seconds), lower memory footprint.

</details>

---

[← Previous](./04-gomaxprocs.md) | [Next Module: Goroutines →](../02-goroutines/01-what-is-a-goroutine.md)
