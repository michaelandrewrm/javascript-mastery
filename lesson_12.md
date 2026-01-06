# 🧩 LESSON 12 — Modules and the JavaScript Runtime

_(How Modularization and Runtime Loading Work Under the Hood)_

Welcome back!
You've now mastered execution contexts, prototypes, and classes.
This lesson takes us into how modern JavaScript organizes and executes code across multiple files — through modules.

Modules form the foundation for building scalable apps, both in the browser (ES Modules) and in Node.js (CommonJS).
We'll explore how imports, exports, and module scopes work, and how the runtime actually loads and links them together.

## 🎯 Learning Goals

By the end of this lesson, you'll understand:

- The difference between ES Modules (ESM) and CommonJS (CJS)
- How the JS runtime parses and executes modules
- How import/export works (and when it's synchronous vs asynchronous)
- How bundlers (like Webpack, Rollup, Vite) resolve dependencies
- The internal module graph and execution order

## 🧱 1️⃣ The Motivation for Modules

Before modules, JS used:

- Global variables (easy conflicts 😬)
- IIFEs (Immediately Invoked Function Expressions)
- Script tag ordering dependencies

```html
<script src="a.js"></script>
<script src="b.js"></script>
<!-- b.js depends on a.js being loaded first -->
```

This was fragile.
Modules solved that with scoped imports/exports and declarative dependencies.

## 📦 2️⃣ ES Modules (ESM) Overview

### Example: math.js

```javascript
export const PI = 3.14159;

export function area(r) {
  return PI * r * r;
}
```

### Example: app.js

```javascript
import { PI, area } from "./math.js";

console.log(area(2)); // 12.56636
```

### 🧠 Key Properties

| Feature          | Description                                  |
| ---------------- | -------------------------------------------- |
| Static structure | Imports/exports are known at parse time      |
| Scope isolation  | Each module has its own top-level scope      |
| Single instance  | Module code runs once, cached afterward      |
| Async loading    | Browser loads ES modules asynchronously      |
| Strict mode      | Always executed in strict mode automatically |

### 🔬 Under the Hood (Execution Flow)

**Parse phase**

- The engine scans all import / export statements.
- Builds a module dependency graph.

**Linking phase**

- Each module's exports are bound to importers (live bindings).

**Evaluation phase**

- Modules are executed in dependency order (topological sort).

**Visualization:**

```
app.js ─┬─> math.js
        └─> utils.js
```

- math.js executes first (its exports must exist)
- Then app.js runs with those imports already bound

## 🧩 3️⃣ CommonJS (CJS) — The Node.js Module System

Before ES Modules, Node.js used CommonJS syntax:

### Example: math.js

```javascript
const PI = 3.14159;

function area(r) {
  return PI * r * r;
}

module.exports = { PI, area };
```

### Example: app.js

```javascript
const { PI, area } = require("./math");

console.log(area(2));
```

### ⚙️ Key Differences: CommonJS vs ES Modules

| Feature     | CommonJS (CJS)                             | ES Modules (ESM)                      |
| ----------- | ------------------------------------------ | ------------------------------------- |
| Syntax      | require() / module.exports                 | import / export                       |
| Load time   | Runtime (synchronous)                      | Parse time (asynchronous in browsers) |
| Scope       | Function wrapper per file                  | Module scope (top-level)              |
| Exports     | Copied values (snapshot)                   | Live bindings                         |
| Execution   | Executes immediately when require() called | Executes once, after linking          |
| Environment | Node.js (legacy)                           | Browser + Node (modern)               |

### 🧠 CJS Execution Internals

When Node executes:

```javascript
const math = require("./math");
```

1. Checks the module cache.
2. If not cached:
   - Reads the file from disk.
   - Wraps it in a function:

```javascript
(function (exports, require, module, __filename, __dirname) {
  // module code here
});
```

3. Executes that function, populating module.exports.
4. Caches the result.

**Module Cache:**

```
┌──────────────┐
│ ./math.js    │ → exports object
└──────────────┘
```

### 🧩 Example: Live vs Snapshot

**CommonJS:**

```javascript
// counter.js
let count = 0;
exports.inc = () => ++count;
exports.value = count;

// app.js
const c = require("./counter");
c.inc();
console.log(c.value); // 0 ❌ snapshot, not live
```

**ES Module:**

```javascript
// counter.mjs
let count = 0;
export function inc() {
  count++;
}
export { count };

// app.mjs
import { inc, count } from "./counter.mjs";
inc();
console.log(count); // 1 ✅ live binding
```

## 🌐 4️⃣ Browser Module Loading

Modern browsers support ESM natively.

```html
<script type="module" src="app.js"></script>
```

- Loads asynchronously
- Each `<script type="module">` runs in module scope
- Uses deferred execution (like defer scripts)
- Relative paths required (./module.js)

### 🧠 Browser Execution Flow

1. Parse HTML
2. Encounter `<script type="module" src="app.js">`
3. Fetch app.js
4. Parse app.js → detect imports
5. Fetch dependencies recursively
6. Link all exports/imports
7. Execute in correct dependency order

All modules run once, then cached in the module registry.

## 🧩 5️⃣ How Bundlers Handle Modules

In production, bundlers (like Webpack, Rollup, Vite) process modules before runtime:

- Parse all imports/exports → build dependency graph.
- Resolve file paths (./math.js, node_modules/uuid, etc.)
- Bundle all into one or few optimized files.
- Transform ESM/CJS syntax for compatibility.
- Minify and perform tree-shaking (remove unused exports).

**Visualization:**

```
src/
├─ math.js
├─ utils.js
└─ app.js

↓ (Bundler)

dist/
└─ bundle.js
```

### 🔧 Tree-Shaking Example

```javascript
// math.js
export function square(x) {
  return x * x;
}
export function cube(x) {
  return x * x * x;
}

// app.js
import { square } from "./math.js";
console.log(square(3));
```

A bundler will remove cube() entirely if it's unused —
because imports are statically analyzable.

## 🧠 6️⃣ Module Scope and Top-Level Behavior

Modules are always in their own lexical scope — no global pollution.

```javascript
// a.js
var x = 10;

// b.js
console.log(typeof x); // ❌ ReferenceError (not global)
```

Each module has its own module scope and top-level await support:

```javascript
// data.js
export const data = await fetch("/api/data").then((r) => r.json());
```

✅ await works at top-level in ESM — because modules are asynchronous by design.

## ⚙️ 7️⃣ Mixed Environments (Node ESM + CJS)

Node supports both systems:

- `.cjs` → CommonJS
- `.mjs` → ES Modules
- `"type": "module"` in package.json makes .js be ESM

**Interop:**

```javascript
// Import CommonJS into ESM
import pkg from "./legacy.cjs";

// Require ESM into CJS (async)
(async () => {
  const { default: mod } = await import("./modern.mjs");
})();
```

## 🧩 8️⃣ The Module Graph

Each JS app forms a directed acyclic graph (DAG) of dependencies:

```
app.js
├─ utils.js
│  └─ helpers.js
└─ math.js
```

**Execution order (depth-first):**

```
helpers.js → utils.js → math.js → app.js
```

- Each module is evaluated once and cached.
- Subsequent imports reuse the existing instance.

## 🔬 9️⃣ Behind the Hood (Engine View)

JS engines build a module record for each file:

- Exports table
- Imports table
- Instantiation state (unlinked, linking, linked)
- Linking resolves references → sets up live bindings
- Evaluation runs code in dependency order
- Export bindings remain live and mutable

**V8 Internal Steps:**

1. Parse source into AST
2. Build module record
3. Resolve imports
4. Link bindings
5. Execute

## 📚 10️⃣ Terminology Glossary

| Term                    | Meaning                                             |
| ----------------------- | --------------------------------------------------- |
| Module                  | Independent JS file with its own scope              |
| Import / Export         | Declarations for sharing code                       |
| Module Graph            | Network of module dependencies                      |
| Live Binding            | Imported variable reflects source updates           |
| Tree Shaking            | Removal of unused exports                           |
| Module Scope            | Lexical environment unique to each module           |
| Top-level await         | Await allowed at module root                        |
| CommonJS                | Legacy synchronous Node module system               |
| ESM (ECMAScript Module) | Modern async module standard                        |
| Bundler                 | Tool that resolves, optimizes, and packages modules |
| Module Cache            | Registry of executed module instances               |

## ⚠️ 11️⃣ Common Pitfalls & Best Practices

### ❌ Pitfalls

**Circular dependencies**

```javascript
// a.js imports b.js, which imports a.js → can cause undefined exports
```

**Dynamic import paths**

```javascript
import x from variable; // ❌ must be static
```

**Mixing CJS and ESM without care** — can cause async import delays.

### ✅ Best Practices

- Prefer ESM (import/export) for all new code
- Keep exports pure and stateless where possible
- Avoid circular dependencies — refactor shared logic
- Use named exports over default for clarity
- Use bundler tree-shaking for performance
- Prefer structuredClone() or modular helpers over global utilities

## 🧩 12️�� Practice Tasks

### Task 1 — Import Order

Given:

```javascript
// math.js
console.log("math loaded");
export const sum = (a, b) => a + b;

// app.js
import { sum } from "./math.js";
console.log(sum(2, 3));
```

Explain the sequence of logs during execution.

### Task 2 — Live Bindings

```javascript
// counter.js
export let count = 0;
export function inc() {
  count++;
}

// main.js
import { count, inc } from "./counter.js";
inc();
console.log(count); // ?
```

Predict and explain why.

### Task 3 — CommonJS Cache

```javascript
// a.js
console.log("A loaded");
module.exports = 42;

// b.js
require("./a");
require("./a");
```

How many times does "A loaded" print? Why?
