# 🧩 LESSON 3 — Functions and Scope

🎯 Learning Goals
By the end of this lesson, you’ll understand:

- The difference between function declarations and function expressions
- How the engine creates a Function Execution Context (FEC)
- What Lexical Scope and Scope Chains really are
- How scope resolution works step-by-step
- How to visualize nested environments and the call stack

## 🧠 1. Functions in JavaScript

Functions are first-class objects - they can be:

- Stored in variables,
- passed as arguments,
- returned from other functions.

But how you declare them affects when and hpw they exist in memory.

### 🧩 Function Declaration

```js
function greet() {
  console.log("Hello!");
}

greet(); // Works even before the declaration
```

Why?

- Function declarations are hoisted - both their name and body are stored during the creation phase.

Memory before execution:

```
greet -> <function>
```

### 🧩 Function Expression

```js
sayHi(); // ❌ TypeError: sayHi is not a function

var sayHi = function () {
  console.log("Hi!");
};
```

Why?

- During hoisting:
  - `var sayHi` is created -> `undefined`
  - The function itself isn't assigned until runtime

Memory before execution:

```
sayHi → undefined
```

So at the first call, sayHi isn’t a function yet.

### 🧩 Arrow Function Expression

```js
const add = (x, y) => x + y;
```

- Arrow functions are also expressions
- Not hoisted
- No own `this`, `arguments`, or `prototype`
- Defined only when the engine reaches that line

## ⚙️ 2. The Function Execution Context (FEC)

Whenever a function is invoked, the JS engine creates a new execution context.
Each context has:

1. Variable Environment (VE) - holds local variables & function declarations
2. Lexical Environment (LE) - tracks scope & outer references
3. `this` binding - context-specific reference

### 🧩 Example

```js
function outer() {
  let a = 10;

  function inner() {
    let b = 20;
    console.log(a + b);
  }

  inner();
}

outer();
```

### 🧮 Step-by-Step Breakdown

Phase 1: Creation

- Global Execution Context (GEC) created
  ```
  Memory:
  outer → <function>
  ```

Phase 2: Execution

1. `outer()` is called
   -> Pushes a Function Execution Context (FEC) for `outer` onto the stack
   ```
   Call Stack:
   [outer()]
   [Global()]
   ```
2. `inner()` is called
   -> Pushes another FEC for `inner`
   ```
   Call Stack:
   [inner()]
   [outer()]
   [Global()]
   ```
3. `console.log(a + b)`
   - Engine looks for `a`:
     - Not in `inner`'s VE -> go to outer lexical scope -> found in `outer`.
   - Adds 10 + 20 -> prints 30
4. `inner()` returns, pops off stack
5. `outer()` returns, pops off stack

### 🧠 Visualization

Call Stack:

```
[top]
│ console.log()
│ inner()
│ outer()
│ Global()
|_______________
```

Memory:

```
Global Memory:
  outer → <function>

outer Memory:
  a → 10
  inner → <function>

inner Memory:
  b → 20
```

## 🧭 3. Lexical Scope and Environment Chains

### 🔹 Lexical Scope

"Lexical" means "defined by position in source code".
Where you write your function determines what it can access.

Each function remembers the environment in which it was created - not where it was called.

### 🧩 Example

```js
let x = 5;

function parent() {
  let y = 10;

  function child() {
    console.log(x + y);
  }

  return child;
}

const fn = parent();
fn(); // prints 15
```

Why?

- When child was created, it closed over (x, y).
- Even after parent finished, child still remembers its lexical scope chain.

### ⚙️ Scope Chain Resolution

When JS looks up a variable:

1. It checks the current execution context's environment.
2. If not found, moves to the user lexical environment.
3. Continues upward until reaching global scope.
4. If not found -> ReferenceError.

### 🧩 Example with Resolution Steps

```js
const globalVar = "global";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";
    console.log(innerVar, outerVar, globalVar);
  }

  inner();
}

outer();
```

Resolution Path for globalVar:

```
inner LE → outer LE → global LE → found!
```

## 🧱 4. Visualizing Nested Scopes

Let’s visualize the Lexical Environment Chain as nested boxes:

```
Global LE
|
|- outer LE
|   |
|   |- inner LE
```

When the engine executes `inner()`:

```
inner LE: { innerVar }
↑
outer LE: { outerVar }
↑
global LE: { globalVar }
```

Each function has a [[Environment]] reference to its parent scope — stored internally by the engine.

## 🔍 5. Behind the Hood (Engine View)

Inside V8:

- Parser builds AST → creates function objects
- Each function object stores a hidden link: [[Environment]]
- On execution:
  - The engine creates a Lexical Environment Record
  - It binds variables and stores the reference to its outer environment
- Scope resolution uses environment chain traversal
- When a variable is no longer reachable → eligible for GC

## 📚 6. Terminology Glossary

| Term                          | Meaning                                                             |
| ----------------------------- | ------------------------------------------------------------------- |
| **Function Declaration**      | Fully hoisted; available before execution                           |
| **Function Expression**       | Created during runtime; not hoisted                                 |
| **Execution Context (EC)**    | Container for variables, scope, and `this`                          |
| **Variable Environment (VE)** | Holds variables defined inside the context                          |
| **Lexical Environment (LE)**  | Structure that connects variables to outer scopes                   |
| **Scope Chain**               | Series of linked environments used for identifier lookup            |
| **[[Environment]]**           | Hidden internal slot that connects a function to its defining scope |
| **Global EC**                 | The root context that exists when the program starts                |
| **Function EC**               | Created each time a function is called                              |

## ⚠️ 7. Common Pitfalls & Best Practices

Pitfalls

1. Confusing definition vs. call site
   -> Functions close over where they’re defined, not where they’re called.
2. Using var inside functions
   -> var ignores block scope; can lead to unexpected sharing of variables.
3. Accidentally shadowing variables
   -> Redeclaring with let or var inside a nested scope hides outer variables.
4. Memory leaks via closures
   -> Functions retaining large objects from outer scopes can prevent garbage collection.

### Best Practices

✅ Use function declarations for reusable named functions
✅ Use function expressions or arrow functions for inline callbacks
✅ Prefer block scope (let/const) inside functions
✅ Avoid deep nesting when possible — consider early returns
✅ Understand closure behavior to manage memory intentionally

## 🧩 8. Practice Tasks

### 🧠 Task 1 — Predict Output

```js
function a() {
  let x = 10;
  function b() {
    console.log(x);
  }
  return b;
}

let fn = a();
fn();
```

Question:
When fn() runs, a() has already finished.
How does b() still access x?

### 🧠 Task 2 — Variable Lookup

```js
let v = 1;

function one() {
  let v = 2;
  function two() {
    console.log(v);
  }
  two();
}

one();
```

Explain why the output is what it is.
Draw the scope chain manually.

### 🧠 Task 3 — Declarations vs Expressions

```js
greet(); // ?

function greet() {
  console.log("Hello!");
}

sayHi(); // ?
var sayHi = function () {
  console.log("Hi!");
};
```

Predict the errors (if any) and explain using the creation vs execution phase.
