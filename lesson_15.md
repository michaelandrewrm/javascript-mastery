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
