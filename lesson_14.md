# 🧩 LESSON 14 — JIT Compilation & Performance

_(How V8 Turns JavaScript into Machine-Speed Code)_

Welcome to one of the most fascinating parts of JavaScript internals — where your code becomes real machine instructions.

In this lesson, we’ll uncover how modern JS engines (like Google’s V8) turn JavaScript from a dynamic, interpreted language into near-native performance code — using JIT compilation, hidden classes, and runtime optimizations.

---

## 🎯 Learning Goals

By the end of this lesson, you’ll understand:

- The full execution pipeline from parsing → interpretation → optimization
- The role of Ignition (interpreter) and TurboFan (optimizer) in V8
- How hidden classes and inline caches make property access fast
- Why de-optimizations occur (and how to avoid them)
- How to measure and tune performance in real-world apps

---

## ⚙️ 1️⃣ From JavaScript Source → Machine Code

JavaScript is a JIT-compiled language (Just-In-Time). Unlike static languages (C, Rust), which compile before running, JS compiles while it’s running, optimizing hot code paths on the fly.

Flow (V8 Engine):

1. Source Code
2. Parser → AST → Ignition (Interpreter) → Bytecode
3. Profiler detects “hot” code
4. TurboFan (JIT Compiler) → Optimized Machine Code
5. Runtime executes native instructions

```
Source Code
↓
Parser → AST → Ignition (Interpreter) → Bytecode
↓
Profiler detects “hot” code
↓
TurboFan (JIT Compiler) → Optimized Machine Code
↓
Runtime executes native instructions
```

---

## 🧱 2️⃣ Ignition — The Interpreter

Ignition is V8’s lightweight interpreter.

- Converts parsed JS (AST) into compact bytecode.
- Executes bytecode line-by-line (like a virtual CPU).
- Collects type feedback to help the optimizer later.

Example:

```js
function add(a, b) {
  return a + b;
}

add(2, 3);
add(5, 10);
```

Ignition:

- Parses and compiles to bytecode.
- Records that both `a` and `b` are usually numbers.
- Feeds this info to TurboFan to produce specialized code.

Bytecode example (simplified):

```
Bytecode for add(a, b):
LdaNamedProperty a
Add b
Return
```

Think of it as V8’s own machine-independent instruction set.

---

## 🚀 3️⃣ TurboFan — The JIT Optimizer

TurboFan takes frequently executed ("hot") bytecode functions and optimizes them into native machine code.

Process:

1. Detects a “hot��� function (called many times).
2. Uses the type feedback Ignition collected.
3. Generates specialized, fast machine code for that pattern.

✅ Result: Repeated arithmetic or property lookups can approach compiled C++ performance.

Example: Type specialization

```js
function multiply(a, b) {
  return a * b;
}

// If V8 sees a and b are always numbers →
// TurboFan compiles machine code that assumes numeric operands.

// Later:
multiply("2", 3); // type mismatch → de-optimization
```

---

## 🧠 4️⃣ De-Optimization: When Things Go Wrong

When assumptions break, optimized code must be discarded and replaced with slower interpreted code.

Example:

```js
function area(shape) {
  return shape.w * shape.h;
}

area({ w: 2, h: 3 });        // optimized for consistent object shape
area({ w: 2, h: 3, x: 1 });   // hidden class changed → de-opt!
```

TurboFan optimized based on the original object structure (“hidden class”). When the shape changed, it had to bail out and re-interpret.

De-optimization triggers:

| Cause                    | Description                                      |
|-------------------------|--------------------------------------------------|
| Type changes            | Using strings where numbers were expected       |
| Shape changes           | Adding/removing object properties dynamically   |
| Polymorphism            | Same function called with different object types|
| Inlining mismatch       | Different function versions can’t be inlined    |
| OSR failures            | Hot loop de-opts mid-execution                  |

---

## 🧱 5️⃣ Hidden Classes — The Secret Behind Fast Objects

Even though JavaScript objects are dynamic, V8 treats them like C-style structs internally — by creating hidden classes that describe their layout in memory.

Example:

```js
function Point(x, y) {
  this.x = x;
  this.y = y;
}
const p1 = new Point(1, 2);
const p2 = new Point(3, 4);
```

Both objects share the same hidden class:

```
HiddenClass(Point#1)
├─ x @ offset 0
└─ y @ offset 1
```

✅ Property access becomes fast because offsets are known.

De-optimizing hidden classes (order matters):

```js
const a = {};
a.x = 1;
a.y = 2;

const b = {};
b.y = 2;
b.x = 1;
```

V8 must create two different hidden classes — so `a.x` and `b.x` have different lookup paths → slower.

Best practice: Always initialize object properties in the same order to help V8 reuse hidden classes.

---

## ⚙️ 6️⃣ Inline Caches (ICs) — Remembering Property Locations

Each time V8 looks up a property (e.g., `obj.prop`), it caches how that lookup was resolved.

Example:

```js
function getX(obj) {
  return obj.x;
}

getX({ x: 10 });
getX({ x: 20 });
```

After first call:

- V8 records where `x` lives (offset in hidden class).
- Future calls skip lookup → direct access.

If you later call `getX({ y: 10 })`, the IC becomes megamorphic (supports multiple shapes) → slower again.

Visualization:

```
Call: obj.x
├─ First time → lookup path stored
├─ Second time → fast IC access
└─ Different shape → IC fallback, slower
```

---

## 🧩 7️⃣ Inline Functions and Hot Code

V8 can inline small functions into their callers for speed:

```js
function add(a, b) { return a + b; }
function calc(x) { return add(x, 10); }

// After optimization, roughly becomes:
// calc(x) {
//   return x + 10; // add() inlined
// }
```

Inlining avoids call overhead but can be reverted if argument types vary.

---

## 🔬 8️⃣ V8 Optimization Pipeline Summary

| Stage     | Engine Component | Description                                  |
|-----------|-------------------|----------------------------------------------|
| Parse     | Parser            | Converts JS → AST                            |
| Interpret | Ignition          | Runs bytecode and collects feedback          |
| Optimize  | TurboFan          | Compiles hot code to native machine code     |
| De-Opt    | Runtime           | Falls back when assumptions break            |

Visualization:

```
Source Code
↓
Parser → AST
↓
Ignition → Bytecode
↓ (Profile hot code)
TurboFan → Optimized Machine Code
↑ (Deopt when invalid)
```

---

## ⚙️ 9️⃣ Performance Measurement & Tools

Chrome DevTools:

- Performance Panel → visualize execution & GC pauses
- Memory Tab → heap snapshots, allocation timeline
- Sources → Blackbox Scripts to focus on your code

Node.js / V8 flags:

- `--trace-opt` / `--trace-deopt` (debug JIT behavior)
- `--prof` for V8 profiler output
- Tools: Chrome DevTools (connected via `node --inspect`)

Example:

```bash
node --trace-opt --trace-deopt app.js
```

You’ll see logs like:

```
[optimizing : multiply]
[deoptimizing: multiply - type mismatch]
```

---

## 🧠 1️⃣0️⃣ Writing JIT-Friendly JavaScript

Best practices:

| Area           | Tip                                      | Why                                   |
|----------------|-------------------------------------------|---------------------------------------|
| Object Shapes  | Initialize properties in same order       | Enables hidden class reuse            |
| Types          | Keep consistent types per variable        | Avoids type confusion de-opt          |
| Hot Functions  | Keep them monomorphic (same shapes)       | Enables stable ICs                    |
| Arrays         | Avoid “holes” (sparse arrays)             | Keeps them in packed mode             |
| Loops          | Prefer `for` loops in critical paths      | Less callback overhead                |
| Functions      | Don’t redefine functions dynamically      | Prevents inlining                     |

Micro-optimizations (modern engines):

- `const`/`let` over `var` (more predictable scoping)
- Avoid large `try/catch` inside hot code (can force de-opt)
- Use typed arrays (`Float64Array`, etc.) for numeric-heavy loops
- Use `Map`/`Set` instead of objects for large dynamic key sets

---

## 📚 1️⃣1️⃣ Terminology Glossary

| Term                   | Meaning                                         |
|------------------------|-------------------------------------------------|
| JIT (Just-In-Time)     | Runtime compilation into native code            |
| Interpreter (Ignition) | Executes bytecode, collects type info           |
| Optimizer (TurboFan)   | Generates specialized machine code              |
| Hidden Class           | Internal shape definition for object layout     |
| Inline Cache (IC)      | Cached property lookup for speed                |
| Hot Function           | Frequently called function worth optimizing     |
| De-Optimization        | Reverting to slower path when assumptions fail  |
| Type Feedback          | Runtime info about variable types               |
| Inlining               | Embedding a small function’s code into caller   |
| Megamorphic Call Site  | Callsite with too many shapes (unoptimizable)   |

---

## ⚠️ 1️⃣2️⃣ Common Pitfalls

Pitfalls:

- Changing object shapes dynamically

  ```js
  const obj = {};
  obj.newProp = 42; // slow if added late
  ```

- Mixing types in hot code

  ```js
  function sum(a, b) { return a + b; }
  sum(1, 2);
  sum("a", "b"); // causes de-opt
  ```

- Overusing polymorphism (too many shapes → megamorphic callsites)
- Frequent `try/catch` in tight loops → disables optimization

---

## ⚡ 1️⃣3️⃣ Example: Performance Improvement

Slow:

```js
function addTo(obj, key, val) {
  obj[key] = val; // each call may use different keys → megamorphic site
}
```

Better:

```js
function addTo(obj) {
  obj.x = 1;
  obj.y = 2;
}
```

Consistent property names → stable hidden class → faster property access.

---

## 🧩 1️⃣4️⃣ Practice Tasks

Task 1 — Predict De-Opt

```js
function add(a, b) { return a + b; }
add(2, 3);
add("2", 3);
```

Explain why the second call de-optimizes the function.

Task 2 — Hidden Class Efficiency

```js
function createPoint() {
  const p = {};
  p.x = 0;
  p.y = 0;
  return p;
}
```

Explain why this pattern is fast, and how reordering `p.y` before `p.x` might affect performance.

Task 3 — Measure Optimization

Run:

```bash
node --trace-opt --trace-deopt test.js
```

What output would indicate successful optimization vs de-optimization?
