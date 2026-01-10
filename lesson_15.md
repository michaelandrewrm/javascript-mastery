# 🧩 LESSON 15 — Event Delegation & The DOM Connection

How JavaScript Talks to the Browser and Manages Events Efficiently

So far we’ve been living in the world of the JS runtime and engine — now we step into how JavaScript interfaces with the browser’s environment, specifically the DOM and the event system.

This is where the JS execution model, the event loop, and the browser’s Web APIs all converge.

## 🎯 Learning Goals

By the end of this lesson, you’ll understand:

- How JavaScript connects to the DOM via browser APIs
- The event system: capturing, bubbling, and delegation
- How event callbacks are queued and executed
- Performance benefits of delegation patterns
- How to use events effectively in dynamic web apps

---

## 🧱 1️⃣ JavaScript’s Connection to the DOM

The DOM (Document Object Model) is a tree structure representing HTML elements. JS manipulates it through the browser’s Web APIs (exposed via the `window` and `document` objects).

Example:

```js
const button = document.querySelector("button");
button.addEventListener("click", () => console.log("Clicked!"));
```

Under the Hood:

- `document.querySelector` → DOM API call
- `addEventListener` → Registers callback in the EventTarget system
- When clicked → Browser creates an event object
- Event enters the capture → target → bubble phases
- If your listener matches → callback enqueued into the event loop

---

## 🧠 2️⃣ Event Phases: Capturing → Target → Bubbling

### 🧩 Example HTML

```html
<div id="outer">
  <button id="inner">Click me</button>
</div>
```

### 🧩 JavaScript

```js
document
  .getElementById("outer")
  .addEventListener("click", () => console.log("Outer"), true); // capture phase

document
  .getElementById("inner")
  .addEventListener("click", () => console.log("Inner")); // bubble phase
```

### Click Sequence

- Capture Phase (downward): Document → html → body → outer → inner → outer listener with capture flag fires
- Target Phase: Event hits inner
- Bubble Phase (upward): inner → outer → body → html → document → inner listener (default) fires during this phase

Output:

```
Outer
Inner
```

Visualization:

```
[Document]
↓ Capture
[HTML]
↓
[Body]
↓
[Outer]
↓
[Inner] ← Target → Bubble ↑
```

---

## 🧩 3️⃣ Event Delegation

Event Delegation is a technique where you attach one listener high up in the DOM to handle events for multiple child elements.

Example:

```js
document.querySelector("#list").addEventListener("click", (e) => {
  if (e.target.matches("li")) {
    console.log("Clicked item:", e.target.textContent);
  }
});
```

Even if you dynamically add `<li>` elements later, the single listener on `#list` still handles their events — thanks to event bubbling.

### ✅ Benefits of Delegation

- Performance: Fewer listeners in memory
- Dynamic content: Works for new elements (no re-binding)
- Cleaner code: Centralized event logic

### ⚙️ Under the Hood

When you click an `<li>`:

1. Browser dispatches click event at that node
2. Event bubbles up to ancestor elements
3. The `#list` listener catches it
4. `e.target` tells you where the click originated

---

## 🧩 4️⃣ Stopping Propagation

You can control how far events travel:

```js
e.stopPropagation(); // stops bubbling/capturing
e.stopImmediatePropagation(); // stops all further listeners
```

You can also prevent default browser actions:

```js
e.preventDefault(); // e.g., stop form submission
```

---

## 🧱 5️⃣ Integration with the Event Loop

Event callbacks are queued in the macrotask queue after the current stack clears.

Example:

```js
button.addEventListener("click", () => console.log("event"));
console.log("sync");
```

Output:

```
sync
event
```

Because the event callback is queued — the event loop ensures synchronous code finishes first.

Visual Timing Flow:

```
Call Stack: main()
↓
Registers listener
↓
Click → Browser API → Event Queue
↓
Call Stack empty → Event loop picks up event callback
↓
Executes handler
```

---

## ⚙️ 6️⃣ Event Delegation in Dynamic Apps

This pattern is critical for frameworks (React, Vue, etc.) where elements are created dynamically.

Example: Virtual DOM uses event delegation internally.

In React, all events are actually attached to a single listener on the root container — React simulates bubbling through its virtual DOM structure.

### ✅ Best Practices for Delegation

- Use `e.target.matches(selector)` or `e.target.closest(selector)`
- Attach at container level, not document (for scoping)
- Clean up event listeners when removing large elements
- Don’t rely on `e.path` (non-standard) — use `e.composedPath()`

⚡ Performance Tip: A single delegated listener is far cheaper than hundreds of per-element listeners — especially in large, dynamic lists.

---

## 📚 Terminology Glossary (Lesson 15)

| Term            | Meaning                                             |
| --------------- | --------------------------------------------------- |
| DOM             | Browser’s in-memory tree representation of HTML     |
| Event Bubbling  | Event traveling upward from target to ancestors     |
| Event Capturing | Event traveling downward before target              |
| Target Phase    | The moment the event reaches the target element     |
| Delegation      | Attaching a single listener to handle many children |
| Event Loop      | System that dequeues and executes callbacks         |
| Macrotask Queue | Queue for events, timers, I/O callbacks             |

---

## 🧩 Practice Tasks

### Task 1

Predict the output:

```html
<div id="parent">
  <button id="child">Click</button>
</div>

<script>
  parent.addEventListener("click", () => console.log("Parent"));
  child.addEventListener("click", () => console.log("Child"));
</script>
```

### Task 2

Implement a delegated handler that logs clicks only on elements with `data-action`.

---

# 🧩 LESSON 16 — Asynchronous Patterns in the Wild

Combining async/await, Promises, and Event Streams in Real Apps

Now we merge everything: the event loop, Promises, and async code — into real-world async patterns for API requests, UI updates, and streams.

## 🎯 Learning Goals

- Combine events + async/await in real-world scenarios
- Manage multiple asynchronous sources (timers, fetch, user input)
- Handle errors gracefully across async flows
- Learn patterns for sequential, parallel, and race async work

---

## ⚙️ 1️⃣ Async + Events in Action

Example: API request triggered by user click.

```js
document.querySelector("button").addEventListener("click", async () => {
  try {
    const res = await fetch("/api/user");
    const data = await res.json();
    console.log("User:", data);
  } catch (err) {
    console.error("Request failed:", err);
  }
});
```

Flow:

- Event fires → callback enqueued.
- Inside callback → async function runs → fetch starts.
- Fetch promise resolves → continuation enqueued in microtask queue.
- Data logged → all non-blocking.

---

## 🧩 2️⃣ Sequential vs Parallel Async Flows

Sequential

```js
async function loadSequential() {
  const user = await fetchUser();
  const posts = await fetchPosts(user.id);
  return { user, posts };
}
```

Waits for each step to complete → ensures dependency order.

Parallel

```js
async function loadParallel() {
  const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
  return { user, posts };
}
```

Executes both concurrently → faster if independent.

---

## ⚡ 3️⃣ Advanced Patterns — Racing and Throttling

### Promise.race

Run multiple tasks but take the first result.

```js
const first = await Promise.race([fetch("/api/primary"), fetch("/api/backup")]);
```

### Throttling Events

Control how often an event handler runs.

```js
let last = 0;
window.addEventListener("scroll", () => {
  const now = Date.now();
  if (now - last > 100) {
    console.log("Scroll position:", window.scrollY);
    last = now;
  }
});
```

### Debouncing

Wait until activity stops.

```js
let timeout;
input.addEventListener("input", () => {
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    console.log("Search:", input.value);
  }, 500);
});
```

---

## 🧠 4️⃣ Combining Async & Event Streams

Using EventTarget + async iterators:

```js
async function* listen(el, event) {
  while (true) {
    yield await new Promise((resolve) =>
      el.addEventListener(event, resolve, { once: true })
    );
  }
}

for await (const e of listen(button, "click")) {
  console.log("Clicked:", e);
}
```

Each event waits asynchronously → elegant event streams!

---

## 🧩 5️⃣ Error Management Strategies

| Pattern           | Example                         | Behavior                  |
| ----------------- | ------------------------------- | ------------------------- |
| try/catch         | `try { await fn() } catch(e){}` | Synchronous async error   |
| Promise.catch     | `fn().catch(handle)`            | Chain-level catch         |
| Global Handler    | `window.onunhandledrejection`   | Catch uncaught rejections |
| Graceful fallback | Retry, timeout, default value   | Keeps app stable          |

Example — Retry Wrapper

```js
async function retry(fn, retries = 3) {
  while (retries--) {
    try {
      return await fn();
    } catch (e) {
      if (!retries) throw e;
      console.warn("Retrying...");
    }
  }
}
```

---

## 🧱 6️⃣ Real-World Async Flow Example

```js
async function fetchWithTimeout(url, ms) {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), ms);

  try {
    const res = await fetch(url, { signal: controller.signal });
    return await res.json();
  } finally {
    clearTimeout(id);
  }
}
```

��� Uses AbortController, Promise, and timeout for robust requests.

---

## 📚 Glossary (Lesson 16)

| Term            | Meaning                                           |
| --------------- | ------------------------------------------------- |
| `Promise.all`   | Waits for all promises to resolve                 |
| `Promise.race`  | Resolves/rejects with first completed             |
| Throttle        | Executes callback at fixed intervals              |
| Debounce        | Executes callback after quiet delay               |
| AbortController | Cancels pending async operations                  |
| Async Iterator  | Iteration protocol supporting await between steps |
| Event Stream    | Series of asynchronous events over time           |

---

## ⚠️ Common Pitfalls

- ❌ Forgetting to handle rejections — Always `.catch()` or wrap in `try/catch`.
- ❌ Over-listening — Too many event listeners cause leaks — use delegation.
- ❌ Async inside loops without await

Bad:

```js
for (const item of items) fetchItem(item); // ❌ not awaited
```

Good:

```js
for (const item of items) await fetchItem(item);
// or:
await Promise.all(items.map(fetchItem));
```

---

## 🧩 Practice Tasks

### Task 1 — Event Delegation + Async

Build a list where clicking a user triggers an async fetch of their posts.

### Task 2 — Retry Logic

Wrap a flaky API in your own retry mechanism using async/await.

### Task 3 — Parallel Requests

Load 3 APIs at once with `Promise.all`, handle one failure gracefully.
