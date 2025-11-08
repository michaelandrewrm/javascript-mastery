# 🧩 LESSON 6 — The Event Loop Explained Visually

🎯 Learning Goals

By the end of this lesson, you’ll understand:

- Why JavaScript is single-threaded — and how that affects concurrency
- How the Call Stack, Callback Queue, and Microtask Queue work together
- The difference between macrotasks and microtasks
- How the Event Loop behaves in browsers vs Node.js
- Be able to predict the execution order of async operations confidently

## 1. 🧠 The Single-Threaded Model

### 🧩 What It Means

- JavaScript executes one thing at a time.
- Only one call stack — one thread of execution.
- This means no two pieces of JS code run simultaneously in the same thread.
- But then… how does JS handle asynchronous tasks like:
  - `setTimeout`
  - `fetch()`
  - `Promise`
  - `DOM events`
  - `I/O in Node.js`

🤔 The answer: JS doesn’t handle them alone — the runtime (Browser or Node.js) does.

### 🧱 The Runtime Pieces

```
┌──────────────────────────────────────┐
│         JavaScript Engine (V8)       │
│  ┌───────────────┐  ┌──────────────┐ │
│  │ Call Stack    │  │ Heap Memory  │ │
│  └───────────────┘  └──────────────┘ │
└──────────────────────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────┐
│        Web APIs / Node APIs            │
│ (Timers, DOM, HTTP, FS, process, etc.) │
└────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│ Task Queues:                         │
│   - Callback Queue (macrotasks)      │
│   - Microtask Queue (promises)       │
└──────────────────────────────────────┘
                    │
                    ▼
Event Loop — orchestrates all of it
```

## 2. ⚙️ The Call Stack + Queues in Action

Example:

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

### 🔍 Step-by-Step Breakdown

Step 1 — Call Stack Starts

- console.log("A") → runs immediately → prints “A”

Step 2 — Timer Scheduled

- setTimeout(..., 0)
  → Browser’s Web API Timer handles it (not V8)
  → Callback (“B”) placed in Callback Queue (macrotask) after delay

Step 3 — Promise Scheduled

- Promise.resolve().then(...)
  → Promise resolved immediately
  → `.then()` callback goes into Microtask Queue

Step 4 — console.log("D")

- Runs immediately → prints “D”

Step 5 — Call Stack Empty

- Event loop checks Microtask Queue first
  - Runs console.log("C") → prints “C”
- Then moves to Macrotask Queue
  - Runs console.log("B") → prints “B”

Final Output:

```
A
D
C
B
```

## 3. ⚖️ Microtasks vs Macrotasks

| Type           | Examples                                              | When They Run                                  | Priority |
| -------------- | ----------------------------------------------------- | ---------------------------------------------- | -------- |
| **Microtasks** | Promises, `queueMicrotask`, `process.nextTick` (Node) | After current stack, **before next macrotask** | High     |
| **Macrotasks** | `setTimeout`, `setInterval`, I/O, UI rendering        | One per loop tick, after all microtasks done   | Lower    |

Simplified Event Loop Cycle:

1. Execute global/script code
2. Execute all microtasks queued so far
3. Execute one macrotask (like a timer callback)
4. Repeat steps 2–3 forever

### 🧩 Example: Interleaving Tasks

```js
setTimeout(() => console.log("TIMER"), 0);
Promise.resolve().then(() => console.log("PROMISE 1"));
Promise.resolve().then(() => console.log("PROMISE 2"));
console.log("SYNC");
```

Order:

```
SYNC
PROMISE 1
PROMISE 2
TIMER

```

- “SYNC” → main stack
- Promises → microtasks → run before timers
- Timer → macrotask → runs last

## 4. 🧩 Visualization: The Event Loop in Motion

### Stage 1 — Script Starts

```
Call Stack:
    [main()]

Microtask Queue: []
Macrotask Queue: []
```

Stage 2 — Timers & Promises Added

```
Call Stack:
    [main() executing]

Microtask Queue: [Promise.then]
Macrotask Queue: [setTimeout callback]
```

Stage 3 — Script Ends

```
Call Stack: []
Microtask Queue: [Promise.then]
Macrotask Queue: [setTimeout callback]
```

Stage 4 — Event Loop Tick

```
→ Run all Microtasks:
    Promise.then() → executes → "C"

→ Then run one Macrotask:
    setTimeout callback() → executes → "B"
```

## 5. 🧠 Event Loop in Browsers vs Node.js

Both use V8 but have different task queue architectures.

### 🌐 Browser Event Loop

Simplified Phases:

1. Run script (global)
2. Process all microtasks
3. Render updates (UI paint)
4. Process one macrotask
5. Repeat

Browser microtask sources:

- Promises
- queueMicrotask()
- MutationObservers

Browser macrotask sources:

- Timers
- DOM events
- Network callbacks

### ⚙️ Node.js Event Loop

Node’s event loop (powered by libuv) has phases:

| Phase                    | What Happens                                           |
| ------------------------ | ------------------------------------------------------ |
| **1. timers**            | Executes `setTimeout`, `setInterval` callbacks         |
| **2. pending callbacks** | I/O callbacks deferred from previous cycle             |
| **3. idle/prepare**      | Internal use                                           |
| **4. poll**              | Retrieves new I/O events, executes I/O callbacks       |
| **5. check**             | Executes `setImmediate()` callbacks                    |
| **6. close callbacks**   | E.g., `socket.on('close')`                             |
| **Microtasks**           | Processed **after each phase**, not just once per tick |
| **`process.nextTick()`** | Runs **before** other microtasks                       |

Example:

```js
setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("promise"));
```

Possible order:

nextTick
promise
timeout
immediate

- nextTick runs before promise microtasks
- Timers and immediates depend on the phase order (may swap if delay=0)

## 6. 🧩 Why the Event Loop Matters

- Keeps the UI responsive (no blocking)
- Enables asynchronous concurrency despite one thread
- Explains why Promises resolve before timers
- Crucial for performance tuning & avoiding race conditions

### ⚠️ Common Pitfalls

1. Blocking the main thread

   ```js
   while (true) {} // freezes everything
   ```

   No event loop progress → browser locks up.

2. Assuming setTimeout(0) runs “immediately”
   It waits for the current stack + all microtasks first.

3. Forgetting microtask priority
   Promise-heavy code can “starve” the UI if you never yield back to the macrotask phase.

### ✅ Best Practices

- Never block the main thread (split heavy work using setTimeout, requestIdleCallback, or Web Workers)
- Know that Promises > Timers in scheduling
- Use queueMicrotask for small follow-up tasks that must run before next timer
- In Node, use `setImmediate` instead of `setTimeout(fn, 0)` for I/O-safe scheduling

### 📚 Terminology Glossary

| Term                                 | Meaning                                                                         |
| ------------------------------------ | ------------------------------------------------------------------------------- |
| **Call Stack**                       | Stack of active execution contexts                                              |
| **Web APIs / Node APIs**             | Host-provided background systems (timers, network, fs, etc.)                    |
| **Callback Queue (Macrotask Queue)** | Holds callbacks like timers and I/O ready to run                                |
| **Microtask Queue**                  | Holds high-priority callbacks (Promises, queueMicrotask)                        |
| **Event Loop**                       | Continuously checks if the stack is empty and moves queued tasks into execution |
| **Tick (Node)**                      | One full iteration of Node’s event loop cycle                                   |
| **Render Phase (Browser)**           | UI update between microtask and macrotask cycles                                |

## 7. 🧩 Practice Tasks

### Task 1 — Predict Output

```js
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));

console.log("4");
```

🧠 What’s the order and why?

### Task 2 — Microtasks in Loops

```js
for (let i = 0; i < 3; i++) {
  Promise.resolve().then(() => console.log("micro", i));
  setTimeout(() => console.log("macro", i), 0);
}
```

🧩 Predict the full output order.

### Task 3 — Browser vs Node

Try running:

```js
setImmediate(() => console.log("immediate"));
setTimeout(() => console.log("timeout"), 0);
```

in both browser and Node — explain any difference in ordering.
