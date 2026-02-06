# Stack Growth and Shrinking

## 1️⃣ Simple Explanation (Beginner Friendly)

Unlike OS threads with fixed-size stacks, goroutine stacks are **dynamic**:

- **Start small**: ~2KB initial allocation
- **Grow on demand**: When function calls need more space
- **Shrink when possible**: After deep call chains return

This is why Go can support millions of goroutines efficiently!

---

## 2️⃣ Real-World Analogy

### 🎒 Expandable Backpack

**Fixed thread stack** = Large suitcase. Must carry full size even if nearly empty.

**Goroutine stack** = Expandable backpack. Starts small, expands when you add stuff, compresses when emptied.

---

## 3️⃣ Technical Working (Step-by-Step)

### Stack Growth Mechanism

```
Initial: 2KB stack
┌────────┐
│ func A │ ← Current frame
└────────┘

After calling B → C → D (needs more space):
┌────────────────┐
│ func A         │
├────────────────┤
│ func B         │
├────────────────┤
│ func C         │  ← Stack overflow check triggers
├────────────────┤
│ func D         │  ← GROW: Allocate 2x stack
└────────────────┘
        ↓
New 4KB stack (copy all data):
┌────────────────────────────────┐
│ func A (copied)                │
├────────────────────────────────┤
│ func B (copied)                │
├────────────────────────────────┤
│ func C (copied)                │
├────────────────────────────────┤
│ func D                         │
├────────────────────────────────┤
│ (room to grow)                 │
└────────────────────────────────┘
```

### Stack Check (Preamble)

Every function has a stack check:
```go
func example() {
    // Compiler inserts: if SP < stackguard { runtime.morestack() }
    // ... function body
}
```

### Contiguous Stack (Go 1.3+)

```
OLD: Segmented stacks (problematic)
┌────┐    ┌────┐    ┌────┐
│Seg1│───►│Seg2│───►│Seg3│  Linked list = pointer overhead
└────┘    └────┘    └────┘

NEW: Contiguous stacks (current)
┌──────────────────────────────┐
│  Single contiguous block     │  Copy on growth = simpler, faster
└──────────────────────────────┘
```

---

## 4️⃣ Where Used in Real Systems?

### Recursive Algorithms

```go
func fibonacci(n int) int {
    if n <= 1 { return n }
    return fibonacci(n-1) + fibonacci(n-2)
}

// Deep recursion: stack grows automatically
// No StackOverflowError like in Java!
result := fibonacci(10000)  // Works (but slow)
```

---

## 5️⃣ Practical Examples

### Observing Stack Growth

```go
package main

import (
    "fmt"
    "runtime"
)

func recurse(depth int) {
    var stack [1024]byte  // Force stack usage
    _ = stack
    
    if depth > 0 {
        recurse(depth - 1)
    }
}

func main() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    before := m.StackInuse
    
    go func() {
        recurse(1000)  // Deep recursion
    }()
    
    runtime.Gosched()  // Let goroutine run
    runtime.ReadMemStats(&m)
    
    fmt.Printf("Stack grew: %d KB\n", (m.StackInuse-before)/1024)
}
```

### Checking Stack Size

```bash
# See escape analysis + stack info
go build -gcflags="-m" main.go 2>&1
```

---

## 6️⃣ Common Mistakes & Interview Traps

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| Stack never moves | Stack is copied to new location on growth |
| Pointers to stack are safe | Pointers are updated during stack copy |
| Deep recursion crashes Go | Stacks grow up to 1GB (then panic) |
| Stack growth is free | Copying costs O(n) for n bytes |

---

## 7️⃣ Quick Summary Box

```
┌──────────────────────────────────────────┐
│ • Initial size: 2KB                      │
│ • Max size: ~1GB                         │
│ • Growth: 2x current size (copy all)     │
│ • Shrink: During GC if oversized         │
│ • Contiguous memory (not segmented)      │
│ • All pointers updated on move           │
└──────────────────────────────────────────┘
```

---

## 8️⃣ Quiz Questions

1. What triggers stack growth?
2. Why did Go switch from segmented to contiguous stacks?
3. What happens to pointers when stack moves?
4. What's the max stack size for a goroutine?
5. When does stack shrinking happen?

---

## 9️⃣ Answer Key

<details>
<summary>Click to reveal answers</summary>

**A1**: Function preamble checks SP < stackguard. If true, calls `runtime.morestack()`.

**A2**: Segmented stacks had "hot split" problem - functions at segment boundary caused repeated alloc/free. Contiguous stacks avoid this.

**A3**: Runtime updates all pointers during copy. This is safe because Go knows all pointer locations.

**A4**: ~1GB (configurable with `runtime/debug.SetMaxStack`). Beyond this: panic.

**A5**: During garbage collection, if stack is <25% used, it may be shrunk.

</details>

---

[← Previous](./02-goroutines-are-lightweight.md) | [Next →](./04-goroutine-lifecycle.md)
