# 🧩 LESSON 20 — Putting It All Together: The JS Mastery Project

Design → Code → Trace → Optimize

We’ll build a small-but-rich system that exercises everything we’ve learned: modules, closures, prototypes/classes, the event loop, Promises/async–await, delegation, observers, and performance hygiene. Then we’ll trace execution (stack, queues, memory), and hunt bottlenecks & leaks.

---

## 🎯 Project: “Mini TaskBoard”

A single-page app that:

- Renders a list of tasks
- Supports add/complete/filter via event delegation
- Fetches “suggested tasks” from an API with retry + timeout
- Publishes state changes on an EventBus (Observer)
- Uses modules (ESM) and class-based models
- Includes one intentional perf bug and one intentional leak, then fixes

### Files

- eventBus.js — Observer
- store.js — Module (closure) with live state + selectors
- api.js — Fetch with retry + timeout (AbortController)
- models.js — Class-based Task
- view.js — DOM renderer using delegation
- index.js — Bootstrap & orchestration

---

## 📦 Code

### eventBus.js

```js
// Simple pub/sub (Observer)
export function createEventBus() {
  const subs = new Map(); // event → Set(callback)

  return Object.freeze({
    on(event, cb) {
      if (!subs.has(event)) subs.set(event, new Set());
      subs.get(event).add(cb);
      return () => subs.get(event)?.delete(cb); // unsubscribe fn
    },
    emit(event, payload) {
      // Microtask batching: schedule after current stack, before next macrotask
      queueMicrotask(() => subs.get(event)?.forEach((fn) => fn(payload)));
    },
  });
}
```

### store.js

```js
// Closure module: private state + public API (live bindings via ESM)
export function createStore(bus) {
  let tasks = []; // private
  let filter = "all"; // 'all' | 'open' | 'done'
  let idSeq = 1;

  const api = {
    add(title) {
      const t = { id: idSeq++, title, done: false, createdAt: Date.now() };
      tasks.push(t);
      bus.emit("tasks:changed", tasks);
      return t;
    },
    toggle(id) {
      const t = tasks.find((x) => x.id === id);
      if (t) {
        t.done = !t.done;
        bus.emit("tasks:changed", tasks);
      }
    },
    setFilter(next) {
      filter = next;
      bus.emit("filter:changed", filter);
    },
    getSnapshot() {
      return { tasks: [...tasks], filter };
    },
    getVisible() {
      return tasks.filter((t) =>
        filter === "all" ? true : filter === "done" ? t.done : !t.done
      );
    },
  };

  return Object.freeze(api);
}
```

### api.js

```js
// Fetch with retry + timeout; demonstrates AbortController + async/await patterns
async function fetchWithTimeout(url, { ms = 3000, signal } = {}) {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), ms);
  try {
    const res = await fetch(url, { signal: signal ?? controller.signal });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } finally {
    clearTimeout(id);
  }
}

export async function retry(fn, retries = 2, delayMs = 400) {
  let lastErr;
  for (let i = 0; i <= retries; i++) {
    try {
      return await fn();
    } catch (e) {
      lastErr = e;
      if (i < retries) await new Promise((r) => setTimeout(r, delayMs));
    }
  }
  throw lastErr;
}

export async function getSuggestedTasks() {
  return retry(() => fetchWithTimeout("/api/suggestions", { ms: 2500 }));
}
```

### models.js

```js
// Class + prototype-based instance methods (syntactic sugar)
export class Task {
  constructor({ id, title, done = false, createdAt = Date.now() }) {
    this.id = id;
    this.title = title;
    this.done = done;
    this.createdAt = createdAt;
  }
  toggle() {
    this.done = !this.done;
  }
  rename(next) {
    this.title = next;
  }
}
```

### view.js

```js
// DOM rendering + event delegation; one intentional perf smell, then a fixed version
export function mountView({ bus, store, root }) {
  root.innerHTML = `
    <div class="toolbar">
      <input id="newTitle" placeholder="Add a task..." />
      <button id="addBtn">Add</button>
      <select id="filter">
        <option value="all">All</option>
        <option value="open">Open</option>
        <option value="done">Done</option>
      </select>
      <button id="suggestBtn">Suggest</button>
      <span id="status"></span>
    </div>
    <ul id="list" class="list"></ul>
  `;

  const els = {
    input: root.querySelector("#newTitle"),
    add: root.querySelector("#addBtn"),
    filter: root.querySelector("#filter"),
    list: root.querySelector("#list"),
    status: root.querySelector("#status"),
    suggest: root.querySelector("#suggestBtn"),
  };

  // Event delegation for list
  els.list.addEventListener("click", (e) => {
    const li = e.target.closest("li[data-id]");
    if (!li) return;
    if (e.target.matches("button.toggle")) store.toggle(Number(li.dataset.id));
  });

  els.add.addEventListener("click", () => {
    const title = els.input.value.trim();
    if (title) {
      store.add(title);
      els.input.value = "";
    }
  });

  els.filter.addEventListener("change", () =>
    store.setFilter(els.filter.value)
  );
  els.suggest.addEventListener("click", () => bus.emit("ui:suggest"));

  // Render (INTENTIONAL PERF ISSUE: naive full re-render causes layout thrash)
  function renderSlow() {
    // ⚠️ SLOW: innerHTML in a loop + sync layout reads
    els.list.innerHTML = "";
    const items = store.getVisible();
    for (const t of items) {
      // forced sync read
      // eslint-disable-next-line no-unused-vars
      const _w = els.list.offsetWidth; // forces layout each iteration
      const li = document.createElement("li");
      li.dataset.id = t.id;
      li.className = t.done ? "done" : "";
      li.innerHTML = `
        <span>${t.title}</span>
        <button class="toggle">${t.done ? "Undo" : "Done"}</button>`;
      els.list.appendChild(li);
    }
    els.status.textContent = `${items.length} visible`;
  }

  // Fixed render using DocumentFragment + single layout read
  function renderFast() {
    const items = store.getVisible();
    const frag = document.createDocumentFragment();
    for (const t of items) {
      const li = document.createElement("li");
      li.dataset.id = t.id;
      li.className = t.done ? "done" : "";
      li.innerHTML = `<span>${t.title}</span>
                      <button class="toggle">${
                        t.done ? "Undo" : "Done"
                      }</button>`;
      frag.appendChild(li);
    }
    els.list.replaceChildren(frag);
    els.status.textContent = `${items.length} visible`;
  }

  // Subscribe renders
  const off1 = bus.on("tasks:changed", renderFast); // use fast render
  const off2 = bus.on("filter:changed", renderFast);

  // Return cleanup (important to avoid leaks)
  return () => {
    off1();
    off2();
    els.list.replaceChildren();
  };
}
```

### index.js

```js
import { createEventBus } from "./eventBus.js";
import { createStore } from "./store.js";
import { getSuggestedTasks } from "./api.js";
import { Task } from "./models.js";
import { mountView } from "./view.js";

const bus = createEventBus();
const store = createStore(bus);

// Seed with a few tasks (class + prototype)
store.add(new Task({ id: 1, title: "Read docs" }).title);
store.add(new Task({ id: 2, title: "Write code" }).title);

const cleanup = mountView({ bus, store, root: document.getElementById("app") });

// Intentional leak (we'll fix later): a global interval capturing store snapshot
// ⚠️ BAD: never cleared, retains closures and memory
let leakInterval = setInterval(() => {
  const snap = store.getSnapshot(); // captures arrays repeatedly
  // console.log("heartbeat", snap.tasks.length);
}, 1000);

// Suggestion flow
bus.on("ui:suggest", async () => {
  try {
    const suggestions = await getSuggestedTasks();
    for (const s of suggestions) store.add(s.title);
  } catch (e) {
    console.error("Suggest failed:", e);
  }
});

// Graceful teardown example (if needed)
// window.addEventListener("beforeunload", () => { cleanup(); clearInterval(leakInterval); });
```

---

## 🧭 Execution Trace (What the Engine Does)

### 1) Load & Parse (Creation)

- Browser discovers `type="module"` → module graph parsed & linked
- Hoists import/export bindings; creates module scopes
- Builds AST → Ignition compiles bytecode

### 2) Bootstrap (Execution)

- `createEventBus()` returns an object with closures over `subs`
- `createStore(bus)` creates private `tasks`, `filter`, `idSeq`
- `mountView(...)` paints UI; hooks listeners (delegation)
- `store.add(...)` emits `tasks:changed` → microtask enqueues render

### 3) Event Loop + Queues

- User actions (clicks) → macrotask events
- `bus.emit` handlers → microtasks (run before timers)
- `getSuggestedTasks`:
  - `fetch` offloads network to Web API
  - Promise resolution → microtask resumption
- Intentional interval leak → macrotask every 1s

Order sample on “Add”:

```
[Call Stack] click handler → store.add → bus.emit
[Microtasks] renderFast
[Macrotasks] next events (timers, UI events)
```

---

## 🧠 Memory & Stack Visuals

Call Stack (adding a task):

```
[top]
│ renderFast() ← microtask (after emit)
│ store.add()
│ click handler
│ Global (module)
└──────────────────────────
```

Heap (selected):

```
Heap:
EventBus.subs → Map { "tasks:changed" → Set[fns], "filter:changed" → Set[fns] }
Store (closure) → { tasks: Array<TaskLite>, filter: "all", idSeq: 3 }
DOM nodes → <ul#list> children
INTERVAL (leak) → closure capturing store.getSnapshot (retains arrays)
```

---

## 🛑 Bottlenecks & Leaks (and Fixes)

1. Layout Thrash in `renderSlow` (Fixed)

- Bug: Repeated `offsetWidth` reads inside loop force sync layout each iteration
- Fix: `renderFast` uses DocumentFragment + no forced layout reads
- Why it matters: Forced synchronous layouts block the main thread, freeze UI

2. Interval Memory Leak in `index.js`

- Bug: `setInterval` keeps running forever and captures closures → retains snapshots
- Fix: Clear interval on teardown or after inactivity; or avoid capturing large objects

```js
// FIX
window.addEventListener("beforeunload", () => {
  cleanup();
  clearInterval(leakInterval);
});
```

- Better: redesign heartbeat with `WeakRef`/`WeakMap` or use a single global status flag.

3. Fan-out Rendering

- Many emits can cause multiple renders per tick.
- Mitigation: Debounce/batch via microtask:

```js
// In view.js (optional optimization)
let scheduled = false;
function scheduleRender() {
  if (scheduled) return;
  scheduled = true;
  queueMicrotask(() => {
    scheduled = false;
    renderFast();
  });
}

bus.on("tasks:changed", scheduleRender);
bus.on("filter:changed", scheduleRender);
```

---

## 🔎 How to Measure (DevTools Plan)

Performance Panel:

- Record adding 100 tasks via suggestion
- Look for long frames > 16ms
- Validate that `renderFast` minimized Layout/Style recalcs

Memory Panel:

- Take snapshots before/after 30s
- Without clearing interval → retained closures (compare)
- After fix → stable retained size

Async Stack Traces:

- Trigger “Suggest”
- Observe `await fetch` → microtask resume path

---

## ✅ Checklist: JIT-Friendly Coding Present

- Stable object shapes for tasks (`{id,title,done,createdAt}` in consistent order)
- Avoid polymorphism at hot callsites (store API is monomorphic)
- No dynamic prototype swapping
- Short functions → better inlining

---

## 📚 Glossary (new items)

| Term               | Meaning                                                      |
| ------------------ | ------------------------------------------------------------ |
| Layout Thrash      | Alternating reads/writes to layout causing repeated reflows  |
| DocumentFragment   | Off-DOM container to batch insertions cheaply                |
| Microtask Batching | Queueing work via `queueMicrotask` to coalesce updates       |
| Teardown           | Cleanup phase to release listeners/timers and DOM references |

---

## 🧩 Practice Extensions

- Concurrency Guard: Prevent multiple concurrent “Suggest” calls (use a flag or AbortController to cancel the previous).
- Virtualized List: Render only visible tasks for 10,000 items (IntersectionObserver or manual virtualization).
- Persistence: Add a `storage.js` module with `localStorage` sync; debounced saves to avoid thrash.
- Error Sentinel: Display an inline `<div class="error">` with retry on API failure; auto-hide after success.

---

## 🧠 Manual Reasoning Exercise (Your Turn)

Given this sequence:

1. User clicks Suggest
2. API returns 5 tasks
3. User immediately clicks Done on 3 tasks

Explain the exact order of:

- Call stack pushes/pops
- Microtasks vs macrotasks
- Which renders occur and why they don’t interleave incorrectly

Write your trace using the stack/queue blocks we’ve used in earlier lessons.
