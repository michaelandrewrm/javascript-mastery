# 🧩 LESSON 7 — Callbacks, Promises & Async/Await

🎯 Learning Goals
By the end of this lesson, you’ll understand:

- How callback-based async patterns work and why “callback hell” happens
- How Promises work internally (states, transitions, handlers)
- How async/`await` is built on top of Promises
- How V8 schedules async tasks using the microtask queue
- How errors propagate in async code

## 1. The Callback Pattern (and “Callback Hell”)

Before Promises existed, asynchronous code relied on callback functions.

### 🧩 Example

```js
getData(function (data) {
  processData(data, function (processed) {
    saveData(processed, function (result) {
      console.log("Done:", result);
    });
  });
});
```

Each async function receives another function to call when it’s done — leading to deeply nested code.

Problems:

- Hard to read and maintain (rightward drift)
- Difficult error handling (try/catch doesn’t work across callbacks)
- No clear sequencing or composition tools

This messy structure is called callback hell.

Mental image:

```
getData
└── processData
└── saveData
└── console.log
```

## 2. Promises — The Foundation of Modern Async

A Promise represents a value that may exist in the future.

### 🧩 Basic Example

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve(" Data loaded!"), 1000);
});

promise.then((result) => console.log(result));
```

### ⚙️ Internal Promise Mechanics

Every Promise has 3 states:

| State         | Description         | Transition             |
| ------------- | ------------------- | ---------------------- |
| **pending**   | initial state       | → fulfilled / rejected |
| **fulfilled** | operation succeeded | final                  |
| **rejected**  | operation failed    | final                  |

Each Promise also holds:

```
value → resolved result (if fulfilled)
reason → error (if rejected)
handlers → list of .then/.catch/.finally callbacks
```

🧠 Behind the Hood (Microtasks)

- When you call `.then()` on a Promise, the callback doesn’t run immediately.
- Instead, it’s queued in the Microtask Queue.
- Once the current call stack is empty, the event loop processes all microtasks before moving to the next macrotask.

Example:

```js
console.log("A");

Promise.resolve().then(() => console.log("B"));
console.log("C");
```

Order:

```
A
C
B
```

Why?

- Promise.resolve().then(...) schedules “B” in the microtask queue.
- The microtask queue runs after the current stack finishes (after “C”).

## 3. Promise Chaining

You can chain Promises to flatten asynchronous logic:

```js
fetch("https://api.example.com/data")
  .then((response) => response.json())
  .then((json) => console.log(json))
  .catch((err) => console.error("Error:", err))
  .finally(() => console.log("Done"));
```

- `.then()` returns a new Promise → allows chaining
- Each `.then()` callback runs in a microtask
- .catch() handles rejections in the chain
- .finally() always runs at the end

## 4. Async/Await — The Elegant Syntax

async/`await` is syntactic sugar built on Promises.
It allows you to write asynchronous code as if it were synchronous.

🧩 Example

```js
async function getUserData() {
  try {
    const res = `await` fetch("/api/user");
    const data = `await` res.json();
    console.log("User:", data);
  } catch (err) {
    console.error("Error:", err);
  }
}

getUserData();
```

🧠 What’s Happening Internally

1. Declaring async function makes it return a Promise.
2. When execution hits `await`, the function:
   - Pauses execution.
   - Returns control to the event loop.
   - Schedules the rest of the function as a microtask once the awaited Promise resolves.

Equivalent Promise form:

```js
function getUserData() {
  return fetch("/api/user")
    .then((res) => res.json())
    .then((data) => console.log("User:", data))
    .catch((err) => console.error("Error:", err));
}
```

So `await` simply unwraps a Promise and resumes the function later.

🔍 Visual Timeline

```
Call Stack:                 Microtask Queue:
────────────────────────────────────────────
getUserData() →
fetch()                     → Web API
await                       → suspends function
                            ↓
                            Promise resolved
                            ↓
                            enqueue continuation in Microtask Queue
```

Event Loop Order:

1. JS runs synchronous code (top of async fn)
2. Encounters `await` → pauses function
3. Event loop handles other tasks
4. When awaited Promise settles → microtask enqueued
5. Resumes async function with result

## 5. Error Propagation in Async Code

### 🔹 Callback Version

```js
doSomething((err, data) => {
  if (err) return console.error(err);
  // ...
});
```

→ Errors must be manually passed and handled.

### 🔹 Promise Version

```js
fetch("/bad-url")
  .then((res) => res.json())
  .catch((err) => console.error("Caught:", err));
```

Any error inside `.then()` (sync or async) rejects the chain automatically.

### 🔹 Async/Await Version

```js
async function run() {
  try {
    const res = await fetch("/bad-url");
    const data = await res.json();
  } catch (e) {
    console.error("Caught:", e);
  }
}
```

`try/catch` works seamlessly inside async functions — it’s just syntactic sugar for `.catch()`.

### ⚙️ Under the Hood (Error Flow)

1. await unwraps a Promise.
2. If the Promise rejects:
   - The async function’s returned Promise also rejects.
   - If wrapped in `try/catch`, the error is caught there.
   - If not caught → unhandled rejection warning (Node) or console error (browser).

## 6. Real-World Examples

### 🧩 Sequential Async Work

```js
async function fetchPosts() {
  const user = await fetchUser();
  const posts = await fetchUserPosts(user.id);
  return posts;
}
```

- Each line waits for the previous to complete.
- Still runs asynchronously — doesn’t block the event loop.

### 🧩 Parallel Async Work

```js
async function fetchAll() {
  const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
  return { user, posts };
}
```

Both Promises run concurrently — faster than awaiting one after another.

### 🧩 Error Propagation Example

```js
async function main() {
  try {
    await Promise.reject("fail");
    console.log("never runs");
  } catch (e) {
    console.error("Caught:", e);
  }
}
main();
```

Behind the Hood:

- Promise rejects → async function halts
- Error propagates to `catch` block (same as `.catch()`)

## 7. Microtasks and Async/Await Scheduling

Example:

```js
async function demo() {
  console.log("1");
  await Promise.resolve();
  console.log("2");
}

demo();
console.log("3");
```

Output:

1
3
2

Why:

1. "1" runs before `await`
2. await `Promise.resolve()` → schedules continuation as a microtask
3. "3" runs after main script (stack is empty)
4. Microtask "2" executes next

🧱 Visual Model

```
Time →
───────────────────────────────
| Script Start |
↓
[Call Stack] : demo()
    - logs "1"
    - pauses at await
↓
[Microtask Queue] : resume demo()
↓
[Call Stack empty]
↓
run "3"
↓
run microtask → logs "2"
```

📚 Terminology Glossary

| Term                               | Meaning                                                            |
| ---------------------------------- | ------------------------------------------------------------------ |
| **Callback**                       | Function passed as argument to run later                           |
| **Promise**                        | Object representing a future value                                 |
| **Pending / Fulfilled / Rejected** | Promise lifecycle states                                           |
| **Microtask Queue**                | Queue for high-priority async jobs (Promises, async/await resumes) |
| **Macrotask Queue**                | Queue for lower-priority async jobs (timers, I/O)                  |
| **Async Function**                 | Function returning a Promise implicitly                            |
| **Await**                          | Pauses async function until Promise resolves/rejects               |
| **Unhandled Rejection**            | Error from a Promise that has no `.catch()` or `try/catch`         |

## 8. Common Pitfalls & Best Practices

Pitfalls

1. Forgetting await

   ```js
   const data = fetch("/api"); // returns Promise, not data!
   ```

2. Mixing callbacks + promises — creates race conditions
3. Blocking code in async functions — still blocks event loop
4. Ignoring rejected Promises — leads to silent failures

Best Practices

- Always handle rejections: `.catch()` or `try/catch`
- Use `Promise.all()` for parallel async work
- Chain Promises instead of nesting
- Use `async/await` for clarity
- Keep async functions small and focused
- In Node, listen for `process.on('unhandledRejection', handler)` to debug missing catches

## 9. Practice Tasks

### Task 1 — Predict Output

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

(async () => {
  console.log("D");
  await null;
  console.log("E");
})();

console.log("F");
```

🧠 Predict the output order and explain which steps involve microtasks/macrotasks.

### Task 2 — Convert to Async/Await

Refactor:

```js
getUser()
  .then((user) => getPosts(user.id))
  .then((posts) => showPosts(posts))
  .catch((err) => console.error(err));
```

into an async function.

### Task 3 — Error Propagation

Explain why this throws an unhandled rejection:

```js
async function run() {
  await Promise.reject("boom!");
}
run();
```

How can you fix it?
