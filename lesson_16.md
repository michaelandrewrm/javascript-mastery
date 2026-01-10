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
