# Syscalls and Scheduler Interaction

## 1️⃣ Simple Explanation (Beginner Friendly)

When a goroutine makes a **system call** (file read, network), it might block the OS thread. Go handles this specially:

1. **Non-blocking I/O**: Uses netpoller (no thread blocked)
2. **Blocking syscalls**: M blocks, P gets a new M

---

## 2️⃣ Real-World Analogy

### 📞 Call Center

- **Async call (netpoller)**: Put customer on hold, serve others
- **Sync call (blocking)**: Agent is stuck, send new agent to desk

---

## 3️⃣ Technical Working (Step-by-Step)

### Network I/O (Non-blocking via Netpoller)

```
G calls: conn.Read()
       │
       ▼
┌─────────────────────────────────────┐
│ 1. Register fd with netpoller       │
│ 2. G state → Gwaiting               │
│ 3. M runs another G (P keeps M!)    │
└─────────────────────────────────────┘

Data arrives:
┌─────────────────────────────────────┐
│ 1. epoll/kqueue signals ready       │
│ 2. G state → Grunnable              │
│ 3. G added to run queue             │
└─────────────────────────────────────┘
```

### Blocking Syscall (File I/O, CGO)

```
G calls: file.Read() (blocking syscall)
       │
       ▼
┌─────────────────────────────────────┐
│ 1. runtime.entersyscall()           │
│ 2. P detaches from M (P.m = nil)    │
│ 3. P finds/creates new M            │
│ 4. Old M + G block in syscall       │
└─────────────────────────────────────┘

                    ┌───────────┐
  Before:  M0 ──────│    P0     │
                    └───────────┘
                    
  During:  M0 (blocked)    P0 ─────── M1 (new)
             │                          │
             G (syscall)               other Gs
```

### Handoff Process

```
entersyscall():
┌────────────────────────────────────┐
│ M.p = nil     // Detach P          │
│ G.state = Gsyscall                 │
│ P.m = nil     // P is free         │
└────────────────────────────────────┘

sysmon (background thread):
┌────────────────────────────────────┐
│ Find idle Ps without Ms            │
│ Wake or create new M for P         │
│ New M picks up Gs from P           │
└────────────────────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### High-Performance Servers

```go
// HTTP server: mostly network I/O
// Netpoller handles efficiently, Ms not blocked

// File processing service: blocking I/O
// M handoff ensures other Gs continue
func processFiles(files []string) {
    for _, f := range files {
        data, _ := os.ReadFile(f)  // Blocking, but P handed off
        process(data)
    }
}
```

---

## 5️⃣ Practical Examples

### Observing Thread Growth

```go
package main

import (
    "os"
    "runtime"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    
    // Many blocking file reads = many Ms
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            os.ReadFile("/dev/urandom")  // Blocking syscall
        }()
    }
    
    wg.Wait()
    
    // Check thread count
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    println("Check thread count with pprof")
}
```

```bash
# Watch thread growth
GODEBUG=schedtrace=100 ./myapp 2>&1 | grep threads
# threads=X shows OS thread count
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| All I/O blocks threads | Network I/O uses netpoller |
| P always stays with M | P detaches on blocking syscalls |
| Thread count = GOMAXPROCS | Can grow much higher |
| CGO uses netpoller | CGO is blocking, causes handoff |

---

## 7️⃣ Quick Summary Box

```
┌────────────────────────────────────────────┐
│ Network I/O: netpoller, no thread blocked  │
│ File/CGO: M blocks, P gets new M (handoff) │
│                                            │
│ Key functions:                             │
│ • entersyscall(): P detaches from M        │
│ • exitsyscall(): G returns to scheduling   │
│ • sysmon: monitors and handles handoff     │
└────────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. Why doesn't network I/O block OS threads in Go?
2. What happens to P when M enters blocking syscall?
3. Can thread count exceed GOMAXPROCS?
4. What is sysmon's role in syscall handling?
5. Is CGO handled like network I/O?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Network I/O uses netpoller (epoll/kqueue). G parks, no thread blocked.

**A2**: P detaches and gets a new M. Original M stays blocked with G.

**A3**: Yes! Many blocking syscalls = many Ms. Check with schedtrace.

**A4**: Monitors for Ps without Ms, wakes/creates Ms for idle Ps.

**A5**: No. CGO is treated as blocking syscall. Causes handoff.

</details>

---

[← Previous](./04-scheduler-blocking-flow.md) | [Next Module: Channels →](../04-channels/01-why-channels-exist.md)
