# 🧩 LESSON 19 — Design Patterns in JavaScript

Harnessing Closures, Prototypes, and Modules for Elegant Architecture

Welcome to the architecture layer of your JavaScript mastery. Now that you understand how the engine runs your code, we’ll focus on how to structure that code — using time-tested design patterns adapted to JS’s unique features: 👉 closures, prototypes, and the module system.

---

## 🎯 Learning Goals

By the end of this lesson, you’ll be able to:

- Implement classical patterns — Singleton, Observer, Factory, and Module — in idiomatic JavaScript
- Understand how closures, prototypes, and modules shape JS-style design
- Recognize when and why to apply each pattern
- Avoid anti-patterns that fight the JS runtime model

---

## 🧠 1️⃣ Why Patterns Matter in JS

Design patterns are blueprints for solving common architectural problems. In JavaScript, they must respect:

- Dynamic typing
- First-class functions
- Closures (for private state)
- Prototypes (for inheritance & object sharing)
- Modules (for encapsulation & dependency control)

JS doesn’t mimic classical OOP — it embraces flexibility.

---

## 🧱 2️⃣ Singleton Pattern

“One Instance to Rule Them All.” A Singleton ensures that only one instance of a class or object exists globally.

### Example (ES6 Class)

```js
class Database {
  constructor() {
    if (Database.instance) return Database.instance;
    this.connection = "Connected to DB";
    Database.instance = this;
  }
}

const db1 = new Database();
const db2 = new Database();

console.log(db1 === db2); // true
```

Behavior: Both `db1` and `db2` reference the same instance.

🔍 Under the Hood: When `new Database()` runs, the constructor checks if `Database.instance` exists; if so, it returns the existing object. This exploits shared references via the constructor’s static property.

### ✅ Closure-based Singleton (Module Form)

```js
const Settings = (function() {
  let instance;
  function create() {
    return { theme: "dark", version: "1.0" };
  }
  return {
    getInstance() {
      if (!instance) instance = create();
      return instance;
    }
  };
})();

const s1 = Settings.getInstance();
const s2 = Settings.getInstance();
console.log(s1 === s2); // true
```

Mechanism: a closure preserves the private `instance` variable, guaranteeing single instantiation.

---

## 🧩 3️⃣ Observer Pattern

“Publish / Subscribe — Reactive by Nature.” The Observer pattern enables one-to-many communication between objects: when a subject changes, observers are notified.

### Example (Custom Event Emitter)

```js
class EventEmitter {
  constructor() {
    this.events = {};
  }
  on(event, fn) {
    (this.events[event] = this.events[event] || []).push(fn);
  }
  off(event, fn) {
    this.events[event] = (this.events[event] || []).filter(f => f !== fn);
  }
  emit(event, data) {
    (this.events[event] || []).forEach(fn => fn(data));
    // Optionally async: queue microtasks or macrotasks before invoking
    // queueMicrotask(() => (this.events[event] || []).forEach(fn => fn(data)));
  }
}

const emitter = new EventEmitter();
const log = data => console.log("Received:", data);

emitter.on("message", log);
emitter.emit("message", "Hello observers!");
emitter.off("message", log);
```

🔍 Under the Hood:

- `events` is a map of arrays keyed by event name
- `on()` registers listeners; `emit()` iterates them
- Works asynchronously if `emit()` wraps calls in timers or microtasks

Real-World Use:

- DOM events (`addEventListener`)
- Node.js’s `EventEmitter`
- React’s `useEffect` subscriptions
- RxJS / Signals / Streams systems

---

## 🧱 4️⃣ Factory Pattern

“Encapsulate Object Creation Logic.” The Factory pattern abstracts how objects are created — especially useful when the exact type isn’t known ahead of time.

### Example

```js
class Dog {
  speak() { console.log("Woof!"); }
}
class Cat {
  speak() { console.log("Meow!"); }
}

function AnimalFactory(type) {
  if (type === "dog") return new Dog();
  if (type === "cat") return new Cat();
  throw new Error("Unknown type");
}

const pet = AnimalFactory("dog");
pet.speak(); // Woof!
```

🔍 Why Use It:

- Simplifies construction logic
- Centralizes configuration or dependency injection
- Useful for testing, plugin systems, or switching implementations

### ⚙️ Factory via Closures

```js
function createUser(type) {
  const base = { active: true };
  if (type === "admin") return { ...base, role: "admin" };
  return { ...base, role: "user" };
}
```

No classes needed — closures + object literals achieve the same goal.

---

## 🧩 5️⃣ Module Pattern

“Encapsulation via Closures.” The Module pattern provides private scope and a public interface — a precursor to modern ES Modules.

### Classic IIFE Module

```js
const CounterModule = (function() {
  let count = 0; // private

  function increment() { count++; }
  function get() { return count; }

  return { increment, get }; // public API
})();

CounterModule.increment();
console.log(CounterModule.get()); // 1
```

🔍 Why It Works: The IIFE (Immediately Invoked Function Expression) creates a lexical closure retaining access to `count`. Only the returned object can access it — no external mutation possible.

### ✅ Modern Equivalent (ES Modules)

counter.js

```js
let count = 0;
export function increment() { count++; }
export function get() { return count; }
```

main.js

```js
import { increment, get } from './counter.js';
increment();
console.log(get()); // 1
```

ESM does the same thing natively — encapsulated state + explicit exports.

---

## 🧠 6️⃣ How JS Makes Patterns Unique

Because functions are first-class and support closures, many classic OOP patterns simplify dramatically:

| Classical Pattern | JS Equivalent                 |
| ----------------- | ---------------------------- |
| Singleton         | Closure + Module              |
| Factory           | Function returning object     |
| Observer          | EventEmitter or callback list |
| Decorator         | Higher-order function         |
| Strategy          | Function injection            |
| Command           | Function wrapper              |

JavaScript lets you express patterns functionally — without rigid class hierarchies.

---

## 🧩 7️⃣ Example: Combining Patterns

Observer + Factory + Module

```js
const ChatApp = (() => {
  const users = [];
  const events = new EventTarget();

  function createUser(name) {
    const user = { name };
    users.push(user);
    events.dispatchEvent(new CustomEvent("user:join", { detail: user }));
    return user;
  }

  events.addEventListener("user:join", e => console.log("Joined:", e.detail.name));
  return { createUser };
})();

ChatApp.createUser("Ada");
```

Concept Stack:

- Module: encapsulates app state
- Factory: creates user objects
- Observer: notifies listeners

---

## ⚙️ 8️⃣ Patterns & Performance

Closures, modules, and observers have runtime implications:

| Pattern   | Runtime Cost | Notes                                      |
| --------- | ------------ | ------------------------------------------ |
| Singleton | Minimal      | Static reference, no leak risk             |
| Observer  | Moderate     | Avoid too many listeners                   |
| Factory   | Variable     | Depends on object creation rate; optimize constructors |
| Module    | Minimal      | Closures persist → avoid large captures    |

Be mindful that closures retain memory; clear references when modules or listeners are no longer needed.

---

## 📚 9️⃣ Terminology Glossary

| Term         | Meaning                                 |
| ------------ | --------------------------------------- |
| Closure      | Function + lexical environment snapshot |
| Prototype    | Shared object for inheritance           |
| Encapsulation| Restricting direct access to state      |
| Factory      | Encapsulated object creator             |
| Observer     | Publish/subscribe communication model   |
| Singleton    | Ensures one instance globally           |
| Module       | Self-contained logical unit of code     |
| IIFE         | Immediately Invoked Function Expression |

---

## ⚠️ 1️⃣0️⃣ Common Pitfalls

| Mistake                             | Fix                                               |
| ----------------------------------- | ------------------------------------------------- |
| Overusing Singletons                | Use dependency injection or factories instead     |
| Memory leaks via Observers          | Remove listeners (`off`/`removeEventListener`)    |
| Bloated Modules                     | Split by responsibility; favor ES Modules        |
| Factory returning inconsistent shapes | Keep property order consistent (hidden classes) |

---

## 🧩 1️⃣1️⃣ Practice Tasks

- Task 1 — Singleton Check: Implement a singleton logger that counts how many times it logs messages.
- Task 2 — Observer Implementation: Build a mini “PubSub” system with `.subscribe`, `.unsubscribe`, and `.publish`.
- Task 3 — Module Privacy: Create a counter module that keeps its count private but exposes `increment()` and `getCount()`.
