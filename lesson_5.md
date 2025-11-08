# 🧩 LESSON 5 — Hoisting, Closures & Memory Persistence

🎯 Learning Goals

By the end of this lesson, you’ll be able to:

- Explain exactly how hoisting works for functions and variables
- Understand how closures let inner functions retain access to outer scope variables
- Visualize how closures are represented in memory (heap + environment links)
- Recognize closure-based patterns in real-world code (e.g., callbacks, async handlers)
- Avoid memory leaks and misuse of closures

## 1. The Final Word on Hoisting

### 🧩 Example

```js
console.log(x); // ?
console.log(add(2, 3)); // ?

var x = 10;
function add(a, b) {
  return a + b;
}
```

### 🧠 Step-by-Step

Creation Phase (Parsing):

- JS scans the code and builds an environment record:
  ```
  Memory:
    add → <function>
    x → undefined;
  ```
- Declarations are “hoisted” to the top of their scope.

Execution Phase:

- `console.log(x)` → `undefined`
- `console.log(add(2,3))` → 5
- Then `x = 10` assigns value

Key Rule Summary

| Type                   | Hoisted? | Initialized? | Example                  |
| ---------------------- | -------- | ------------ | ------------------------ |
| `var`                  | yes      | undefined    | `var x`                  |
| `let` / `const`        | yes      | TDZ          | `let y`                  |
| `function` declaration | yes      | with body    | `function f() {}`        |
| `function` expression  | no       | —            | `const f = function(){}` |

Mental Model:

```
During Creation Phase:
x → undefined
add → <function>
```

✅ Function declarations are fully hoisted
⚠️ Variables (var) hoisted with undefined
🚫 let/const in TDZ until declared

## 2. Closures — Functions That Remember

### 🧩 Example 1: Basic Closure

```js
function outer() {
  let counter = 0;

  function increment() {
    counter++;
    console.log(counter);
  }

  return increment;
}

const inc = outer();
inc(); // 1
inc(); // 2
```

### 🧠 Step-by-Step Breakdown

1. `outer()` called → new Execution Context created

   - `counter = 0`
   - `increment` function created
   - `increment.[[Environment]]` points to outer’s lexical environment
   - `outer()` returns the **function object** increment

2. Global memory now holds:

```
inc → <function increment>
```

3. Even though `outer()` has finished, `increment` still remembers `counter`.
   - This is a closure: increment closes over outer’s environment.

### 🧩 Visualization

```
Global LE:
  outer → <function>
  inc → <function increment>

outer LE (retained by closure):
  counter → 2
  increment → <function>
```

Each time `inc()` runs, it accesses and updates `counter` inside its preserved lexical environment.

## 3. Closure: Deep Internal Mechanics (V8 View)

When V8 creates a function, it attaches a hidden reference:

```
Function Object:
  [[Environment]] → pointer to parent Lexical Environment
```

When outer() finishes:

- Normally, outer()’s variables are removed from memory.
- But here, increment still points to outer’s lexical environment.
- V8 detects this and moves the retained environment to the heap, keeping it alive as long as any closure references it.

Memory Diagram

```
Stack (Call Stack):
[Global()]
[outer()] → returns increment()
↑
closure pointer
↓
Heap (Persistent Environment):
  counter → 2
```

So:

- Non-closed-over variables = live on stack (discarded after return)
- Closed-over variables = moved to heap (kept alive for closures)

## 4. Real-World Examples of Closures

### 🔹 Example A — Event Handler

```js
function setupButton() {
  let clicks = 0;
  const btn = document.querySelector("#btn");

  btn.addEventListener("click", function handleClick() {
    clicks++;
    console.log("Clicked:", clicks);
  });
}

setupButton();
```

Why this works:

- The callback (handleClick) is stored in memory by the browser’s event system.
- Even after setupButton() finishes, the closure persists.
- Each click still accesses the original clicks variable.

Memory:

```
Heap:
  handleClick.[[Environment]] → { clicks: 3, btn: <button> }
```

⚠️ Potential Leak:
If you remove the button but not its listener, that closure’s environment still holds references to the DOM element.
🧹 Always removeEventListener when done.

### 🔹 Example B — Async Callbacks

```js
function fetchData() {
  const url = "/api/data";

  setTimeout(() => {
    console.log("Fetching from:", url);
  }, 1000);
}

fetchData();
```

How this works:

- The arrow function forms a closure over url.
- When the timer callback runs later, it accesses url via its preserved environment.
- url is not destroyed when fetchData returns.

Call stack visualization:

```
Time 0: fetchData() pushed → runs → returns → popped
Heap: closure keeps url alive
Time 1000ms: callback pushed → logs → popped
```

### 🔹 Example C — Function Factory

```js
function makeMultiplier(multiplier) {
  return function (x) {
    return x * multiplier;
  };
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

Each returned function has its own closure environment:

```
double.[[Environment]] → { multiplier: 2 }
triple.[[Environment]] → { multiplier: 3 }
```

They’re distinct memory spaces.

## Closures, Garbage Collection & Memory Persistence

Closures retain only the variables they reference — not the entire parent scope.

- When no references remain to a closure, both the function and its environment are GC’d.
- Be mindful of long-lived closures holding big objects.

### ⚠️ Example Leak

```js
function start() {
  let bigArray = new Array(1000000).fill("*");
  document.getElementById("btn").onclick = function () {
    console.log("clicked");
  };
}
```

If the closure doesn’t use bigArray, V8 can release it.
But if you accidentally reference it inside the closure → the large array stays in memory until the listener is removed.

Fix:
Detach event handlers or set large unused variables to null once done.

## 6. Visual: Closure Retention

```
Function Call Stack:

[Global()]
  ↓
outer() -→ returns inner()
  ↓
inner() still references outer's environment

Memory:
Heap:
  [[Environment]]: { counter: 2 }

When inner() garbage collected → environment freed.
```

## 7. Terminology Glossary

| Term                         | Definition                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------ |
| **Hoisting**                 | The compile-time registration of declarations before code executes             |
| **Closure**                  | A function + its preserved lexical environment                                 |
| **Lexical Environment**      | Object holding variable bindings + outer link                                  |
| **Environment Record**       | Internal structure that stores bindings                                        |
| **[[Environment]]**          | Hidden property on function objects that links them to where they were defined |
| **TDZ (Temporal Dead Zone)** | Zone before initialization where variable access throws                        |
| **Memory Retention**         | Heap-stored data surviving after outer function returns                        |
| **Garbage Collection (GC)**  | Automatic removal of unreachable objects/environments                          |

## 8. Common Pitfalls & Best Practices

Pitfalls

1. Accidental shared state

   ```js
   for (var i = 0; i < 3; i++) {
     setTimeout(() => console.log(i), 100);
   }
   // → 3, 3, 3 (not 0,1,2)
   ```

   Reason: single shared `i` variable (var is function-scoped).

2. Memory leaks through closures
   Holding DOM nodes or large data unintentionally.

3. Misunderstanding hoisting
   Expecting let or const to behave like var.

Best Practices

- Prefer let / const — block-scoped and safer for closures
- Avoid referencing unnecessary outer variables
- Explicitly remove event listeners when cleaning up
- For loops with async callbacks → create new scope:
  ```js
  for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100); // prints 0,1,2
  }
  ```
- Keep closures small and purposeful

## 9. Practice Tasks

### Task 1 — Hoisting Order

Predict the output:

```js
sayHi();
console.log(name);

function sayHi() {
  console.log("Hi");
}
var name = "Ada";
```

Explain what’s hoisted and why.

### Task 2 — Closure Counter

Implement a closure that maintains a private counter with increment/decrement methods.

```js
const counter = createCounter();
counter.inc(); // 1
counter.inc(); // 2
counter.dec(); // 1
```

### Task 3 — Async Retention

Explain why this logs correctly even though fetchData has already returned:

```js
function fetchData() {
  const endpoint = "/api";
  setTimeout(() => console.log("Using:", endpoint), 1000);
}
fetchData();
```
