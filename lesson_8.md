# 🧩 LESSON 8 — Timers, Tasks, and the Browser APIs

🎯 Learning Goals
By the end of this lesson, you’ll understand:

- How timers and browser APIs work behind the scenes
- How the event loop coordinates between the Call Stack, Web APIs, and Task Queues
- The timing nuances of setTimeout(…, 0) and why it’s never truly “0 ms”
- How **requestAnimationFrame** synchronizes with rendering
- How to visualize async execution order clearly and predictably

## 1. JavaScript & the Web APIs: Division of Labor

The JavaScript engine (V8) is just the executor — it doesn’t know about timers, the DOM, or fetch requests.
Those features are provided by the host environment (the browser).

Browser architecture:

```
┌────────────────────────────────────────────┐
│            Browser Runtime                 │
│                                            │
│  ┌───────────────┐   ┌────────────────┐    │
│  │  V8 Engine    │   │   Web APIs     │    │
│  │ (JS Execution)│   │ (DOM, Timers,  │    │
│  │               │   │  Fetch, etc.)  │    │
│  └───────────────┘   └────────────────┘    │
│          │                   │             │
│          ▼                   ▼             │
│     Call Stack           Async Queues      │
│          │                   │             │
│          └──────→ Event Loop ──────────────┘
```

Separation of responsibilities:

- V8 executes JS code (synchronous logic).
- Web APIs handle async tasks (timers, events, network, etc.).
- Event Loop coordinates — pushing callbacks back to JS once ready.

## 2. ⚙️ How setTimeout and setInterval Work

### 🧩 `setTimeout(callback, delay)`

Schedules a function to run after a minimum delay.
But — it does not guarantee exact timing!

Example:

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

console.log("C");
```

Output:

```
A
C
B
```

Step-by-Step:

1. `console.log("A")` runs immediately.
2. `setTimeout(...)` asks the Web Timer API to run the callback after delay ms.
3. Browser starts a timer in its internal thread (not JS).
4. After delay → callback added to macrotask queue.
5. JS finishes the current stack (`console.log("C")`).
6. Event loop sees stack empty → pulls callback from queue → runs it → prints “B”.

Even with `delay = 0`, there’s always at least one full tick before it executes.

### 🧩 `setInterval(callback, delay)`

Repeats the callback every `delay` ms — but again, the interval is approximate.

```js
let count = 0;
const id = setInterval(() => {
  console.log("Tick", ++count);
  if (count === 3) clearInterval(id);
}, 1000);
```

The timer is managed by the browser’s timer system, not by JS itself.
If the main thread is busy, intervals can lag or skip.

### 🧠 Why Timing Isn’t Precise

JS is single-threaded →
If the call stack is busy (e.g., heavy computation), timers can’t fire until the stack clears.

Example:

```js
setTimeout(() => console.log("Timer fired"), 100);
for (let i = 0; i < 1e9; i++) {} // heavy loop (~1s)
```

Expected delay = 100ms
Actual delay ≈ 1000+ ms, because the event loop is blocked.

## 3. 🕹️ requestAnimationFrame (rAF)

Purpose:
Schedules a callback just before the next browser repaint — ideal for smooth animations.

```js
function draw(timestamp) {
  console.log("Frame at:", timestamp);
  requestAnimationFrame(draw);
}
requestAnimationFrame(draw);
```

Characteristics:

- Called ~60 times per second (once per display refresh).
- Synchronized with browser rendering → smoother than setInterval.
- Pauses when the tab isn’t visible (saves CPU/battery).
- Runs before repaint but after all microtasks.

Timing order:

```
microtasks → requestAnimationFrame → repaint → macrotasks
```

## 4. 🧩 Example: Timers + Promises + rAF

```js
console.log("1");

setTimeout(() => console.log("2 (timer)"), 0);

Promise.resolve().then(() => console.log("3 (promise)"));

requestAnimationFrame(() => console.log("4 (rAF)"));

console.log("5");
```

Expected Output (Browser):

```
1
5
3 (promise)
4 (rAF)
2 (timer)
```

Explanation:

1. 1 and 5 → run immediately (synchronous)
2. Promise → microtask → runs after main script
3. rAF → runs before next repaint
4. setTimeout → macrotask → runs last

🧱 Visualization: Timeline of Events

```
Step 1: Run script
Call Stack: main()
Microtasks: [promise.then]
rAF queue: [frame callback]
Macrotasks: [timeout callback]

Step 2: Stack empty
→ Run all microtasks
→ Run rAF before paint
→ Run one macrotask (timeout)
```

## 5. 🌍 Browser vs Node.js Timer APIs

While both use V8, they rely on different host APIs.

| Feature         | Browser                    | Node.js                         |
| --------------- | -------------------------- | ------------------------------- |
| Timers          | Web Timer APIs             | libuv timers                    |
| rAF             | (Browser only)             | (Not in Node)                   |
| Timer precision | Throttled in inactive tabs | Throttled in background threads |
| Minimum delay   | 4ms after 5 nested timers  | ≈1ms typical                    |

## 6. 🔬 Behind the Hood — Timer Internals

When you call setTimeout(fn, 1000):

- The Web Timer module starts a real system timer in a separate thread.
- When 1000ms passes, the browser adds your callback to the macrotask queue.
- The event loop picks it up when the JS stack is empty.

### 🧩 Summary Pseudocode (mental model):

```
setTimeout(fn, delay):
    Start timer (Web API)
    After delay → push fn into Task Queue
    Event Loop → runs fn when stack empty
```

### 🧠 requestAnimationFrame vs setTimeout(…, 16)

| Method                  | Purpose         | Frame Sync | Paused when Tab Hidden | Ideal Use              |
| ----------------------- | --------------- | ---------- | ---------------------- | ---------------------- |
| `setTimeout`            | Generic delay   | No         | No                     | Background async tasks |
| `setInterval`           | Repeating delay | No         | No                     | Periodic updates       |
| `requestAnimationFrame` | Animation       | Yes        | Yes                    | Smooth visuals         |

## 8. ⚠️ Common Pitfalls

1. Assuming setTimeout(…, 0) is instant
   It’s not — it runs after the current stack and all microtasks.

2. Long-running code blocks timers
   JS can’t check the queue until the stack clears.

3. setInterval drift
   If the callback takes longer than the delay,
   intervals can pile up — prefer setTimeout recursion for accurate intervals.

```js
function preciseInterval() {
  setTimeout(() => {
    // do work
    preciseInterval();
  }, 1000);
}
preciseInterval();
```

4. Using timers for animation
   → causes jitter and frame drops; use `requestAnimationFrame()` instead.

### 📚 Terminology Glossary

| Term                 | Description                                                           |
| -------------------- | --------------------------------------------------------------------- |
| **Web APIs**         | Browser-provided async interfaces (timers, DOM, fetch, etc.)          |
| **Macrotask Queue**  | Queue of callbacks from timers, I/O, events                           |
| **Microtask Queue**  | Queue for Promises and async/await continuations                      |
| **Timer Throttling** | Browser optimization that delays timers when tab inactive             |
| **rAF**              | `requestAnimationFrame`, syncs callbacks with browser paint           |
| **Task Tick**        | One full event loop cycle                                             |
| **Callback Delay**   | Time between scheduling and actual execution (affected by stack load) |

### 🧩 🔬 Practice Tasks

### Task 1 — Predict the Output

```js
console.log("start");

setTimeout(() => console.log("timeout"), 0);

Promise.resolve().then(() => console.log("promise"));

requestAnimationFrame(() => console.log("rAF"));

console.log("end");
```

🧠 Explain why the order is start → end → promise → rAF → timeout.

### Task 2 — requestAnimationFrame Timing

Create a smooth 60 fps animation loop:

```js
let last = 0;
function step(timestamp) {
  const delta = timestamp - last;
  last = timestamp;
  console.log("Frame:", delta.toFixed(2), "ms");
  requestAnimationFrame(step);
}
requestAnimationFrame(step);
```

Observe how `delta` hovers around ~16ms (≈60fps).

### Task 3 — Timer Drift Challenge

Write two loops:

```js
setInterval(() => console.log("interval"), 1000);
```

vs

```js
(function loop() {
  setTimeout(() => {
    console.log("recursive timeout");
    loop();
  }, 1000);
})();
```

Compare timing accuracy after 10 seconds — which one drifts more?
