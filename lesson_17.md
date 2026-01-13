# 🧩 LESSON 17 — Debugging & Profiling

Mastering Chrome DevTools & Runtime Insight for High-Performance JavaScript

You’ve now explored how JavaScript works under the hood — from the event loop to the JIT compiler. Now it’s time to learn how to observe, debug, and measure that process in real time.

This lesson teaches how to see inside the runtime: track call stacks, inspect heap snapshots, follow async calls, and optimize code with DevTools profiling.

---

## 🎯 Learning Goals

By the end of this lesson, you’ll be able to:

- Use breakpoints and stepping tools effectively
- Inspect call stacks, closures, and execution contexts
- Profile CPU usage and async performance
- Capture and analyze memory leaks
- Understand how to trace async operations end-to-end

---

## 🧱 1️⃣ The DevTools Debugger: Your Runtime Microscope

Open Chrome DevTools → Sources tab → your script.

You can now:

- Set breakpoints
- Inspect local variables
- Step through function calls
- View call stack & scope chain

### 🧩 Example Code

```js
function outer() {
  const x = 10;
  function inner() {
    const y = 20;
    console.log(x + y);
  }
  inner();
}
outer();
```

### 🧭 Step-by-Step Debug Flow

1. Open the file in the Sources tab
2. Set a breakpoint inside `inner()`
3. Click “Resume script execution” ▶️
4. Observe the Call Stack panel:

```
Call Stack:
→ inner()
  outer()
  (anonymous) – main script
```

5. Check Scope Panel:

```
Local:   { y: 20 }
Closure: { x: 10 }
Global:  { ... }
```

This view mirrors the real execution context chain in memory — the same concept covered in Lesson 3 (Scopes).

---

## 🧩 2️⃣ Breakpoints Mastery

Breakpoints let you pause execution at any point to inspect program state.

### Types of Breakpoints

| Type                     | Usage                                                     |
| ------------------------ | --------------------------------------------------------- |
| Line breakpoint          | Click on a line number                                    |
| Conditional breakpoint   | Right-click → “Add conditional breakpoint” (e.g., `i > 10`) |
| DOM breakpoint           | Pause when element changes or is removed                  |
| XHR / Fetch breakpoint   | Pause when a network request is made                      |
| Event listener breakpoint| Pause on click, scroll, keypress, etc.                    |

### 🔍 Conditional Example

```js
for (let i = 0; i < 100; i++) {
  console.log(i);
}
// Right-click the line → "Add conditional breakpoint" → i === 50
// ✅ The debugger stops only when i hits 50.
```

---

## 🧠 3️⃣ Stepping Through Execution

While paused:

| Control            | Description                                      |
| ------------------ | ------------------------------------------------ |
| ▶️ Resume           | Continue until next breakpoint                    |
| ⏩ Step over (F10)  | Run current line, skip inside calls               |
| 🔽 Step into (F11)  | Enter called function                             |
| 🔼 Step out (Shift+F11) | Run rest of function, return to caller       |
| 🕓 Restart frame    | Rerun the current function (recreates local scope) |

Behind the Scenes: Every “step” shows how the call stack grows and shrinks in real time, so you can visually trace how functions are pushed/popped — exactly as studied in Lesson 4.

---

## ⚙️ 4️⃣ Async Debugging & Promise Tracing

Modern apps are async-heavy, so DevTools supports async call stack tracing.

### Example

```js
async function fetchData() {
  const res = await fetch("/api/user");
  const data = await res.json();
  console.log("Fetched:", data);
}
fetchData();
```

### How to Inspect

1. Open Sources → Call Stack Panel
2. Check “Async” box
3. When paused, DevTools reconstructs async frames:

```
Async Call Stack:
→ fetchData() await in main.js:3
  Promise.then()
  (anonymous) – event loop callback
```

This shows how execution jumped across microtasks — letting you trace code through Promise chains and awaits.

---

## 🧩 5️⃣ The Console Panel — Live Evaluation

While paused, the Console runs in the paused context.

Example:

```txt
> x
10
> y
20
```

You can modify variables or evaluate expressions in the scope of the breakpoint, which is powerful for live testing and debugging closures.

---

## 🧱 6️⃣ Performance Profiling — Measuring Execution Time

The Performance tab helps analyze CPU and event loop activity.

### Example Steps

1. Open DevTools → Performance tab
2. Click Record ⏺️
3. Perform an action (click, API call, animation)
4. Stop recording

You’ll see:

- Main Thread Activity (JS execution, painting, GC)
- Flame Chart — timeline of function calls
- Call Tree — percentage of time per function
- Bottom-Up View — hotspots by total CPU time

### 🔍 Reading a Flame Chart

```
[Main Thread]
│
├─ script.js: initializeApp()
│  ├─ renderUI()
│  └─ fetchData() → Promise callback
│
└─ event handlers, GC, painting, etc.
```

The width of each bar = execution time. The depth shows nested function calls.

Wide functions are bottlenecks — candidates for optimization.

---

## 🧩 7️⃣ Memory Profiling & Leak Detection

1. Open Memory tab → Take Heap Snapshot.
2. Take a baseline snapshot.
3. Interact with your app.
4. Take another snapshot.
5. Compare — if object counts keep growing → leak.

### Example: Detached DOM Leak

```js
let div = document.createElement("div");
document.body.append(div);
div.addEventListener("click", () => console.log("hi"));
div.remove(); // ❌ still referenced by event listener
```

✅ Fix:

```js
function handler() { console.log("hi"); }
div.addEventListener("click", handler);
// ... later
div.removeEventListener("click", handler);
div = null;
```

### Memory Snapshot Views

| View         | Description                             |
| ------------ | --------------------------------------- |
| Summary      | Objects by constructor name             |
| Comparison   | Growth since previous snapshot          |
| Containment  | Retaining paths (who’s holding what)    |
| Statistics   | Memory usage overview                   |
| Allocation Timeline | See live memory allocations over time |

Use Record Allocation Timeline to see live memory usage while interacting with your app. Look for steadily increasing graphs → potential leaks.

---

## 🔍 8️⃣ Network & Performance Metrics

In Network tab, monitor:

- Request waterfalls
- Response sizes
- Timing breakdown (DNS, connect, TTFB, content download)

For advanced cases: Use Lighthouse → Audits performance, accessibility, and best practices.

---

## 🧠 9️⃣ Performance Profiling in Node.js

Node provides command-line equivalents to Chrome profiling.

### CPU Profiling

```bash
node --inspect --inspect-brk app.js
```

- Open chrome://inspect → start profiling.

Or use:

```bash
node --prof app.js
```

Then view results:

```bash
node --prof-process isolate*.log
```

### Memory Profiling

```bash
node --inspect app.js
```

- Open Memory tab in Chrome DevTools.

---

## ⚙️ 🔟 Real-World Debug Pattern

Example: Intermittent async bug

```js
async function load() {
  const res = await fetch("/api/items");
  const data = await res.json();
  render(data);
}
```

Symptom:

- UI freezes occasionally.

Steps:

1. Add Performance recording → see `recalculateStyle` taking long
2. Inspect `render()` → discovered heavy DOM reflows
3. Fix: batch DOM updates or use `DocumentFragment`

✅ Debugger shows logic
✅ Profiler reveals bottleneck

Together → full performance insight.

---

## 📚 Glossary

| Term              | Meaning                                             |
| ----------------- | --------------------------------------------------- |
| Breakpoint        | Point in code where execution pauses                |
| Call Stack        | Active function frames (top = current)              |
| Scope Chain       | Local → closure → global variable lookup            |
| Async Stack       | Extended call stack across awaited promises         |
| Flame Chart       | Visualization of function durations                 |
| Heap Snapshot     | Memory graph of objects and references              |
| Detached DOM Node | Element still in memory after removal               |
| Hot Path          | Performance-critical function frequently executed   |

---

## ⚠️ Common Pitfalls

| Mistake                            | Fix                                         |
| ---------------------------------- | ------------------------------------------- |
| Forgetting to disable breakpoints  | Use “Deactivate breakpoints” toggle         |
| Interpreting async stack wrong     | Check “Async” option in Call Stack panel    |
| Misreading GC spikes               | Use multiple samples; GC is normal          |
| Debugging minified code            | Use source maps from your bundler           |
| Blocking UI with `debugger` keyword | Remove after development                   |

---

## ✅ Best Practices

- Use source maps for production debugging
- Combine Performance + Memory profiling for full diagnosis
- Always test async code with Async Stack Traces enabled
- Profile hot paths after feature completion, not during active changes
- Record small, repeatable sessions — not your entire app boot

---

## 🧩 Practice Tasks

### Task 1 — Inspect Scope Chain

Run this in DevTools and pause inside `inner()`:

```js
function outer() {
  const a = 5;
  function inner() {
    const b = 10;
    console.log(a + b);
  }
  inner();
}
outer();
```

Identify Local, Closure, and Global scopes.

### Task 2 — Profile CPU

Record a Performance trace on this:

```js
for (let i = 0; i < 1e6; i++) {}
```

Find it in the flame chart and measure its duration.

### Task 3 — Detect Leak

Repeatedly add and remove elements with event listeners — take heap snapshots and find retained nodes.
