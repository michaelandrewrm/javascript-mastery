# 🧩 Lesson 13 — Memory Management & The Garbage Collector

_(How JavaScript Allocates, Tracks, and Frees Memory)_

Welcome to one of the most under-the-hood lessons in this mastery path. Now that you can reason about scopes, closures, and objects — it’s time to understand what happens to all that data in memory.

We’ll go deep into how JavaScript allocates, tracks, and eventually cleans up memory, focusing on the V8 engine’s garbage collection system.

---

## 🎯 Learning Goals

By the end of this lesson, you’ll understand:

- How JavaScript uses stack vs heap memory
- How the mark-and-sweep algorithm works internally
- How circular references are handled safely
- How to avoid memory leaks and write GC-friendly code
- What you can and can’t control about memory in JS

---

## 🧱 1️⃣ Stack vs Heap Memory (Revisited)

JavaScript manages two main memory regions:

| Memory Area | Purpose                                              | Typical Contents                     |
| ----------- | ---------------------------------------------------- | ------------------------------------ |
| Stack       | Stores execution contexts and local primitive values | Numbers, booleans, references        |
| Heap        | Stores dynamically allocated objects                 | Arrays, objects, functions, closures |

### Example

```js
function demo() {
  let x = 10; // primitive → stack
  let obj = { y: 20 }; // reference → heap
}

demo();
```

Visualization:

```
Stack Frame: demo()
├─ x → 10
├─ obj → (ref #0xA1)

Heap:
#0xA1 → { y: 20 }
```

When demo() finishes:

- Stack frame is popped (x, obj references removed)
- GC checks heap → no references to { y: 20 } → eligible for collection

---

## ⚙️ 2️⃣ How JavaScript Allocates Memory

During execution:

- Variables declared → space reserved (stack or heap)
- Objects created → heap allocation
- Functions invoked → new execution context on stack
- When references are lost → GC can reclaim heap memory

You never call `malloc()` or `free()` manually — JS manages it automatically.

---

## 🔬 3️⃣ The Garbage Collector (GC)

The Garbage Collector automatically frees memory that’s no longer “reachable.”

### Reachability Definition

An object is considered reachable if it can be accessed from root references:

- The global object (`window` or `global`)
- The call stack (local variables)
- Any closures that still reference data

Everything else is considered garbage.

```
Root Objects
├─ window
│  ├─ document
│  └─ myApp → { user: { name: "Ada" } }
└─ call stack → current execution variables
```

If `myApp` is set to `null`, that subtree becomes unreachable → GC target.

---

## 🧩 4️⃣ Mark-and-Sweep Algorithm

This is the core algorithm used by most JS engines (including V8).

### Steps

- Mark Phase: Start from root objects and recursively mark all reachable references.
- Sweep Phase: Scan heap for unmarked objects → free their memory.
- Compact Phase (optional): Move live objects together to reduce fragmentation.

### Visual Representation

```
Heap:
┌──────────────┬──────────────┬──────────────┐
│ Object A (_) │ Object B (_) │ Object C ( ) │
└──────────────┴──────────────┴──────────────┘
↑ reachable         ↑ unreachable
```

- Mark Phase: marks reachable objects from roots
- Sweep Phase: deletes Object C
- Compact Phase: reorganizes memory blocks

✅ Objects A and B stay (reachable)
❌ Object C removed (unreachable)

### Example

```js
let a = { ref: null };
let b = { ref: a };
a.ref = b;

// Both reference each other (circular)
a = null;
b = null;
```

Question: Are they leaked?

Answer: No. Once both `a` and `b` are unreachable from any root, the GC detects that entire cycle as isolated → removes both.

---

## ⚙️ 5️⃣ Circular References (and Why They’re Safe)

In older environments (like early IE), reference-counting GC could leak cycles. But modern mark-and-sweep GC doesn’t rely on reference counts — it uses reachability graphs instead.

So even if A → B and B → A, as long as nothing references A or B from roots, they’re collectible.

---

## 🧠 6️⃣ How Closures Affect Memory

Closures retain variables from their lexical scope. This means data stays in memory as long as the closure exists.

```js
function makeCounter() {
  let count = 0;
  return () => ++count;
}

const counter = makeCounter();
counter(); // still has access to count
```

Visualization:

```
Heap:
counter.[[Environment]] → { count: 0 }

Stack:
counter → ref #0xB2
```

Even after `makeCounter()` returns, `count` remains reachable through the closure — so it cannot be garbage-collected until `counter` is discarded.

---

## 🔬 7️⃣ V8’s Multiple Garbage Collectors

V8 uses several specialized GC subsystems:

| Collector                   | Purpose                                | Description                            |
| --------------------------- | -------------------------------------- | -------------------------------------- |
| Scavenger (Minor GC)        | Handles new short-lived objects        | Copies “young generation” memory       |
| Mark-Compact (Major GC)     | Cleans old, long-lived objects         | Compacts heap to prevent fragmentation |
| Incremental / Concurrent GC | Runs in small slices to avoid blocking | Improves responsiveness                |
| Idle-Time GC                | Runs when CPU idle                     | Reduces overhead                       |

### Two-Generation Model

V8 divides heap memory:

| Space                 | Description         | Example Objects               |
| --------------------- | ------------------- | ----------------------------- |
| New Space (Young Gen) | Short-lived         | Local variables, temp objects |
| Old Space (Old Gen)   | Promoted long-lived | Closures, cached data         |

Objects surviving several minor GCs are promoted to the old generation.

Visualization:

```
Heap:
┌──────────────────────────┐
│ New Space (young)  │ → Minor GC runs frequently
├──────────────────────────┤
│ Old Space (long-lived) │ → Major GC runs occasionally
└──────────────────────────┘
```

---

## ⚙️ 8️��� GC Pause & Performance Implications

Garbage collection can cause pause times (brief halts in JS execution) because it must stop the world to safely update memory references.

Modern engines minimize this via:

- Incremental marking
- Concurrent sweeping
- Idle-time GC

Example of GC pause (conceptual):

```
[--------- Execution ---------][GC][-------- Resume --------]
```

You can inspect GC activity in Chrome DevTools → Performance → Memory.

---

## 🧩 9️⃣ Common Memory Leaks

Even with GC, you can accidentally retain references that prevent cleanup.

1. Global Variables

```js
window.leak = {};
```

Never collected while the window lives.

2. Forgotten Timers / Event Listeners

```js
function start() {
  const bigObj = { data: new Array(1e6).fill("*") };
  setInterval(() => console.log(bigObj.data.length), 1000);
}
```

`setInterval` holds `bigObj` forever → leak.

✅ Fix:

```js
const id = setInterval(...);
clearInterval(id);
```

3. Detached DOM Nodes

```js
const div = document.createElement("div");
document.body.append(div);
div.remove();
// ❌ if JS variable still references it, memory stays
```

✅ Fix: set `div = null` when done.

4. Accumulating Closures

```js
const handlers = [];
function register() {
  const large = new Array(1e6).fill("*");
  handlers.push(() => console.log(large[0]));
}
```

`large` stays in memory because closures in `handlers` keep it reachable.

---

## ⚙️ 1️⃣0️⃣ Performance Tuning Tips

✅ Do

- Break unnecessary references (`obj = null`)
- Unregister event listeners on removal
- Use `WeakMap` / `WeakSet` for temporary associations

```js
const cache = new WeakMap();
```

Objects in `WeakMap`s don’t prevent GC.

- Avoid large global variables or caches without eviction.

❌ Don’t

- Manually “force” GC — it’s not exposed to JS for a reason.
- Rely on memory size or counts for correctness.
- Keep growing arrays or sets without cleanup.

---

## 🧠 1️⃣1️⃣ Weak References

`WeakMap` / `WeakSet` allow objects to be garbage-collected if no other strong references exist.

```js
let user = { name: "Ada" };
const weak = new WeakMap();
weak.set(user, "meta");

user = null; // user eligible for GC now
```

✅ GC removes entry automatically when `user` is unreachable.
🚫 Cannot iterate or inspect `WeakMap`s — they’re ephemeral.

---

## 📚 1️⃣2️⃣ Terminology Glossary

| Term                   | Meaning                                           |
| ---------------------- | ------------------------------------------------- |
| Heap                   | Dynamic memory for objects and closures           |
| Stack                  | Fast memory for execution frames and primitives   |
| GC (Garbage Collector) | Automatic memory manager                          |
| Mark-and-Sweep         | Core algorithm for detecting unreachable objects  |
| Reachability           | Whether an object can be accessed from roots      |
| Minor GC / Major GC    | Scavenger vs Mark-Compact collectors              |
| Promotion              | Moving objects from new → old generation          |
| Memory Leak            | Retained objects that should’ve been freed        |
| WeakMap / WeakSet      | Structures that don’t prevent GC of keys          |
| Compaction             | Reorganizing live objects to remove fragmentation |

---

## ⚠️ 1️⃣3️⃣ Common Pitfalls & Best Practices

❌ Pitfalls

- Holding onto DOM nodes after removal
- Never clearing intervals or listeners
- Large closures capturing unnecessary state
- Overusing caches (especially global or static ones)

✅ Best Practices

- Use `WeakMap` for ephemeral data
- Nullify large objects after use
- Profile memory usage in DevTools regularly
- Avoid unnecessary object retention
- Trust the GC — focus on avoiding leaks, not manual freeing

---

## 🧩 1️⃣4️⃣ Practice Tasks

### Task 1 — Predict GC Behavior

```js
function make() {
  const data = { big: new Array(1e6).fill("*") };
  return () => console.log(data.big.length);
}

const f = make();
f();
```

When can `data` be garbage-collected?

### Task 2 — Leak Detection

What’s wrong with this code?

```js
let cache = {};
function memoize(key, val) {
  cache[key] = val;
}
```

How can you make it GC-safe? _(Hint: `WeakMap`)_

### Task 3 — Reachability Challenge

```js
let obj1 = { name: "A" };
let obj2 = { name: "B" };
obj1.ref = obj2;
obj2.ref = obj1;
obj1 = null;
obj2 = null;
```

Does this leak? Why or why not?

---

End of Lesson 13.
