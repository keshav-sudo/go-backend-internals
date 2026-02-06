# How Go Executes Code

## 1️⃣ Simple Explanation (Beginner Friendly)

When you run a Go program, here's what happens step by step:

1. **You type `go run main.go`** (or execute a compiled binary)
2. **The runtime initializes** - Sets up scheduler, memory allocator, garbage collector
3. **The main goroutine starts** - Runs your `main()` function
4. **Your code executes** - With the runtime managing everything behind the scenes
5. **Program terminates** - When `main()` returns (or `os.Exit()` is called)

The key insight: **Go code doesn't run directly on the CPU like C**. Instead, it runs within the Go runtime environment, which provides services like automatic memory management and goroutine scheduling.

---

## 2️⃣ Real-World Analogy

### 🎭 The Theater Production Analogy

**Your Go Program = A Theater Play**

```
┌─────────────────────────────────────────────────────────────────┐
│                    THEATER (Go Runtime)                         │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  STAGE (CPUs)                                           │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │   │
│  │  │ Core 0  │ │ Core 1  │ │ Core 2  │ │ Core 3  │       │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Director (Scheduler) - Decides which actors perform when      │
│  Stage Manager (Runtime) - Ensures everything runs smoothly    │
│  Cleanup Crew (GC) - Clears props no longer needed             │
│                                                                 │
│  Actors (Goroutines) - Wait backstage until called on stage    │
└─────────────────────────────────────────────────────────────────┘
```

- **Actors (Goroutines)**: Many actors can wait backstage (runqueue), but only a few can be on stage (CPU) at once
- **Director (Scheduler)**: Decides who performs, manages timeouts, ensures no actor hogs the stage
- **Cleanup Crew (GC)**: Works during scene transitions to remove unused props (memory)

---

## 3️⃣ Technical Working (Step-by-Step)

### Program Startup Sequence

```
┌─────────────────────────────────────────────────────────────────┐
│                    GO PROGRAM STARTUP                           │
└─────────────────────────────────────────────────────────────────┘

1. OS loads binary into memory
   │
   ▼
2. Entry point: runtime·rt0_go (assembly)
   │
   ├── Initialize CPU-specific settings
   ├── Set up TLS (Thread Local Storage)
   ├── Create initial stack
   │
   ▼
3. runtime·schedinit()
   │
   ├── Initialize memory allocator (mallocinit)
   ├── Initialize garbage collector (gcinit)
   ├── Initialize scheduler
   ├── Set GOMAXPROCS
   ├── Create P structures (one per logical CPU)
   │
   ▼
4. runtime·newproc() - Creates main goroutine (G)
   │
   ▼
5. runtime·mstart() - Starts the first M (thread)
   │
   ├── Acquires a P
   ├── Runs the scheduler loop
   │
   ▼
6. main·main() - YOUR CODE STARTS HERE
   │
   ├── Runs init() functions (in dependency order)
   ├── Runs main() function
   │
   ▼
7. Program terminates when main() returns
```

### The Scheduler Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCHEDULER LOOP                               │
│                  (runs on each M thread)                        │
└─────────────────────────────────────────────────────────────────┘

            ┌─────────────────────┐
            │   schedule()        │
            └──────────┬──────────┘
                       │
                       ▼
            ┌─────────────────────┐
            │   findRunnable()    │ ◄─────────────────────┐
            │   (find a G to run) │                       │
            └──────────┬──────────┘                       │
                       │                                  │
                       ▼                                  │
            ┌─────────────────────┐                       │
            │   execute(G)        │                       │
            │   (run goroutine)   │                       │
            └──────────┬──────────┘                       │
                       │                                  │
                       ▼                                  │
            ┌─────────────────────┐     Goroutine         │
            │   G finishes or     │     continues         │
            │   yields?           │─────NO───────────────►│
            └──────────┬──────────┘                       │
                       │ YES                              │
                       ▼                                  │
            ┌─────────────────────┐                       │
            │   goexit() or       │                       │
            │   put G back in     │───────────────────────┘
            │   runqueue          │
            └─────────────────────┘
```

### Memory Layout of a Go Program

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROCESS MEMORY LAYOUT                        │
└─────────────────────────────────────────────────────────────────┘

High Address
┌─────────────────────────────────────────────────────────────────┐
│                         STACK                                   │
│  (main goroutine stack, starts small ~2KB, grows as needed)     │
├─────────────────────────────────────────────────────────────────┤
│                           ↓                                     │
│                      Stack grows down                           │
│                                                                 │
│                      Heap grows up                              │
│                           ↑                                     │
├─────────────────────────────────────────────────────────────────┤
│                         HEAP                                    │
│  (dynamically allocated memory, managed by GC)                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  mspan (8KB pages)                                       │   │
│  │  ├── Small objects (≤32KB, size classes)                │   │
│  │  └── Large objects (>32KB, allocated directly)          │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                      BSS (uninitialized data)                   │
├─────────────────────────────────────────────────────────────────┤
│                      DATA (initialized globals)                 │
├─────────────────────────────────────────────────────────────────┤
│                      TEXT (code)                                │
│  (your compiled code + runtime code)                            │
└─────────────────────────────────────────────────────────────────┘
Low Address
```

### How a Goroutine Executes

```
┌─────────────────────────────────────────────────────────────────┐
│               GOROUTINE EXECUTION FLOW                          │
└─────────────────────────────────────────────────────────────────┘

go myFunction()
      │
      ▼
┌────────────────┐
│  runtime.newproc()
│  • Allocate G struct
│  • Allocate 2KB stack
│  • Set up stack frame
│  • Add to runqueue
└───────┬────────┘
        │
        ▼
┌────────────────┐
│  Scheduler picks G
│  • M.execute(G)
│  • Switch to G's stack
│  • Jump to function
└───────┬────────┘
        │
        ▼
┌────────────────┐
│  Function runs
│  • Uses G's stack
│  • May allocate heap
│  • May spawn more Gs
└───────┬────────┘
        │
        ▼ (Function returns)
┌────────────────┐
│  runtime.goexit
│  • Mark G as dead
│  • Put G in free list
│  • Re-enter scheduler
└────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Understanding Startup Latency

```
┌─────────────────────────────────────────────────────────────────┐
│              STARTUP TIME COMPARISON                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Java (Spring Boot)     ████████████████████████████  ~3-5 sec  │
│  Node.js                ████████████  ~500ms                    │
│  Python (Django)        ██████████████  ~800ms                  │
│  Go                     ██  ~10-50ms                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Why This Matters:**
- **Kubernetes Pod Scaling**: Pods spin up instantly
- **Serverless Functions**: Cold start under 100ms
- **CLI Tools**: Feel instant to users
- **Health Checks**: Pass quickly after restart

### Init Functions in Production

```go
// Order of execution:
// 1. Imported packages' init() (in dependency order)
// 2. Current package's init()
// 3. main()

package main

import (
    "database/sql"
    "log"
    _ "github.com/lib/pq"  // init() registers postgres driver
)

var db *sql.DB

func init() {
    // This runs BEFORE main()
    // Good for: loading config, initializing connections
    var err error
    db, err = sql.Open("postgres", "...")
    if err != nil {
        log.Fatal(err)  // Application won't start if DB fails
    }
}

func main() {
    // db is already initialized here
    defer db.Close()
    // ... start server
}
```

---

## 5️⃣ Practical Examples

### Example 1: Observing Program Startup

```go
package main

import (
    "fmt"
    "os"
    "runtime"
    "time"
)

func init() {
    fmt.Println("1. init() called")
}

func main() {
    fmt.Println("2. main() started")
    
    // Process info
    fmt.Printf("PID: %d\n", os.Getpid())
    fmt.Printf("PPID: %d\n", os.Getppid())
    fmt.Printf("NumCPU: %d\n", runtime.NumCPU())
    fmt.Printf("GOMAXPROCS: %d\n", runtime.GOMAXPROCS(0))
    fmt.Printf("NumGoroutine: %d\n", runtime.NumGoroutine())
    
    // Start time (approximately)
    start := time.Now()
    
    // Create some goroutines
    for i := 0; i < 5; i++ {
        go func(id int) {
            fmt.Printf("Goroutine %d running\n", id)
            time.Sleep(10 * time.Millisecond)
        }(i)
    }
    
    time.Sleep(50 * time.Millisecond)
    fmt.Printf("Goroutines now: %d\n", runtime.NumGoroutine())
    fmt.Printf("Elapsed: %v\n", time.Since(start))
    
    fmt.Println("3. main() ending")
}
```

### Example 2: Watching Scheduler Decisions

```go
package main

import (
    "fmt"
    "runtime"
    "sync"
)

func main() {
    // Set to 1 to see sequential execution
    runtime.GOMAXPROCS(1)
    
    var wg sync.WaitGroup
    
    for i := 0; i < 4; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for j := 0; j < 3; j++ {
                fmt.Printf("G%d: iteration %d\n", id, j)
                // Gosched yields to scheduler
                runtime.Gosched()
            }
        }(i)
    }
    
    wg.Wait()
}
```

**Output with GOMAXPROCS=1:**
```
G0: iteration 0
G1: iteration 0
G2: iteration 0
G3: iteration 0
G0: iteration 1
...
```

### Example 3: Memory Allocation Tracing

```bash
# Build with escape analysis output
go build -gcflags="-m -m" main.go 2>&1 | head -30

# Output shows what escapes to heap vs stays on stack:
# ./main.go:15:6: x escapes to heap
# ./main.go:16:6: y does not escape
```

### Linux Commands for Execution Analysis

```bash
# Time the startup
time ./myapp --help

# See system calls during startup
strace -c ./myapp 2>&1 | tail -20

# Detailed trace (first 0.1 seconds)
timeout 0.1 strace -tt ./myapp 2>&1 | head -100

# Check CPU usage during execution
perf stat ./myapp

# Monitor in real-time
./myapp &
PID=$!
watch -n 0.1 "ps -p $PID -o pid,ppid,pcpu,pmem,vsz,rss,state,time"
```

### Example 4: Execution Tracing

```go
package main

import (
    "os"
    "runtime/trace"
)

func main() {
    // Create trace file
    f, _ := os.Create("trace.out")
    defer f.Close()
    
    // Start tracing
    trace.Start(f)
    defer trace.Stop()
    
    // Your program here
    ch := make(chan int)
    go func() { ch <- 42 }()
    <-ch
}
```

```bash
# Run and analyze
go run main.go
go tool trace trace.out
# Opens browser with interactive visualization
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong Understanding | ✅ Correct Understanding |
|------------------------|-------------------------|
| `main()` starts first | `init()` functions run before `main()` |
| Goroutines continue after main exits | When `main()` returns, all goroutines are killed |
| Go programs run directly on CPU | Go programs run within the runtime environment |
| GOMAXPROCS defaults to 1 | GOMAXPROCS defaults to number of CPUs (since Go 1.5) |
| Scheduler runs in a separate thread | Scheduler code runs on M threads between goroutine switches |
| Each goroutine has its own OS thread | Many goroutines share fewer OS threads (M:N threading) |
| init() is called only once per program | init() is called once per package, multiple packages = multiple inits |

---

## 7️⃣ Quick Summary Box

```
┌─────────────────────────────────────────────────────────────────┐
│                    📝 KEY TAKEAWAYS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STARTUP SEQUENCE:                                              │
│  1. OS loads binary                                             │
│  2. Runtime initializes (scheduler, memory, GC)                 │
│  3. Main goroutine created                                      │
│  4. init() functions run (in dependency order)                  │
│  5. main() executes                                             │
│  6. Program ends when main() returns                            │
│                                                                 │
│  SCHEDULER LOOP:                                                │
│  schedule() → findRunnable() → execute() → repeat               │
│                                                                 │
│  KEY POINTS:                                                    │
│  • Go runtime is always running alongside your code             │
│  • Goroutines are scheduled cooperatively with preemption       │
│  • Memory layout: text, data, heap (up), stack (down)           │
│  • Program termination kills all goroutines immediately         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

### Q1: Conceptual
What happens to running goroutines when the `main()` function returns?

### Q2: Order of Execution
Given packages A imports B, and B imports C, what is the order of init() execution?

### Q3: Debugging
Your program exits immediately after spawning a goroutine. The goroutine never seems to run. Why?

### Q4: System Design
Why is Go's fast startup time particularly important for Kubernetes deployments?

### Q5: Technical
What is the role of `runtime.Gosched()` and when would you use it?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

### A1: Goroutines on main() Return
When `main()` returns:
- The program terminates immediately
- All running goroutines are killed
- No cleanup or deferred functions in other goroutines execute

**Solution**: Use `sync.WaitGroup`, channels, or `select{}` to wait for goroutines:
```go
func main() {
    var wg sync.WaitGroup
    wg.Add(1)
    go func() {
        defer wg.Done()
        // work
    }()
    wg.Wait() // Wait for goroutine to finish
}
```

### A2: Init Order
Order: **C's init() → B's init() → A's init() → main()**

Init functions run in reverse dependency order (dependencies first).

If multiple init() in same package, they run in source file order (alphabetically by filename), then top-to-bottom within each file.

### A3: Immediate Exit
The goroutine is created but `main()` returns before the scheduler can run it:
```go
func main() {
    go doWork() // Goroutine created, scheduled to run
    // main() returns immediately, killing everything
}
```

**Fix**: Add synchronization:
```go
func main() {
    done := make(chan bool)
    go func() {
        doWork()
        done <- true
    }()
    <-done // Wait for goroutine
}
```

### A4: Fast Startup for Kubernetes
- **Horizontal Pod Autoscaling**: New pods can handle traffic in milliseconds
- **Rolling Deployments**: Less downtime during updates
- **Pod Restarts**: Failed pods recover quickly
- **Liveness Probes**: Pass quickly after restart
- **Serverless/Knative**: Near-instant cold starts

JVM-based apps may take 30+ seconds, causing probe failures and cascading restarts.

### A5: runtime.Gosched()
`runtime.Gosched()` voluntarily yields the processor, allowing other goroutines to run.

**Use cases**:
1. Long-running CPU-bound loops without function calls
2. Testing scheduler behavior
3. Fairness in single-GOMAXPROCS scenarios

```go
for {
    // CPU-heavy work
    runtime.Gosched() // Give others a chance
}
```

Modern Go (1.14+) has asynchronous preemption, making `Gosched()` less necessary but still useful for explicit cooperation.

</details>

---

## Navigation

[← Previous: Go Compiler vs Runtime](./02-go-compiler-vs-runtime.md) | [Next: GOMAXPROCS →](./04-gomaxprocs.md)
