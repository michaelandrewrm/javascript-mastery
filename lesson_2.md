# 🧩 LESSON 2 — Variables, Data Types, and Memory

🎯 Learning Goals
By the end of this lesson, you’ll understand:

1. How var, let, and const actually work in memory
2. How JavaScript stores primitive vs reference types (stack vs heap)
3. What hoisting really means for each declaration type
4. What the Temporal Dead Zone (TDZ) is and why it exists
5. How garbage collection works conceptually (reference counting & reachability)

## 🧠 1. Variable Declaration in JavaScript

JavaScript provides three keywords variables:

```js
var name = "Ada";
let age = 30;
const PI = 3.14;
```

At first glance, they look similar - but under the hood, they behave very differently.

| Keyword | Scope          | Hoisted? | Initialized? | Mutable? | TDZ? |
| ------- | -------------- | -------- | ------------ | -------- | ---- |
| `var`   | Function scope | Yes      | `undefined`  | Yes      | No   |
| `let`   | Block scope    | Yes      | No (TDZ)     | Yes      | Yes  |
| `const` | Block scope    | Yes      | No (TDZ)     | No       | Yes  |

Example:

```js
console.log(a);
console.log(b);
console.log(c);

var a = 1;
let b = 2;
const c = 3;
```

Output:

```js
undefined
ReferenceError: Cannot access 'b' before initialization
```

## ⚙️ 2. Step-by-Step Breakdown (Creation → Execution)

### 🏗️ Phase 1: Creation (Compile Time)

When the JS engine parses this script:

1. It creates Global Execution Context (GEC).
2. It scans for declarations and allocates memory for each variable.

```
Memory Allocation (Before Execution)
a → undefined          // var is hoisted & initialized
b → <uninitialized>    // let exists but not initialized
b → <uninitialized>    // const exists but not initialized
```

At this stage:

- `a` is in **Variable Environment** (VE) with `undefined`
- `b` and `c` are in **Lexical Environment** (LE) but in the **Temporal Dead Zone**

The **TDZ** is the period between the start of scope creation and the line of declaration.

### 🔥 Phase 2: Execution (Run Time)

The engine executes line-by-line:

1. `console.log(a)` → reads `a` = `undefined`
2. `console.log(b)` → tries to read before initialization **ReferenceError**
3. Once it reaches `let b = 2`, `b` is initialized.
4. Then `const c = 3` initializes `c`.

## 🧩 3. Visual Representation

### 📦 Memory Model

```
Before Execution
Global Memory:
    a → undefined
    b → TDZ
    c → TDZ

During Execution
    a = 1
    b = 2
    c = 3
```

### 🧠 Call Stack

```
Call Stack:
[top]
|   console.log()
|   Global()
|________________
```

## ⚛️ 4. Primitive vs Reference Types

### 🔹 Primitive Types (stored in stack)

- `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, and `bigint`.

Stored by value - meaning the variable directly contains the data.

```js
let x = 10;
let y = x; // a copy of the value

y = 20;

console.log(x); // 10
console.log(y); // 20
```

Both `x` and `y` point to **separate copies** in memory.

```
Memory (stack):
x → 10
y → 20
```

### 🔸 Reference Types (stored in heap)

- `object`, `array`, and `function`.

Stored by reference - variables hold a pointer to a memory location on the heap.

```js
let user1 = { name: "Ada" };
let user2 = user1; // copy of the reference (pointer)

user2.name = "Grace";

console.log(user1.name); // "Grace"
```

Both `user1` and `user2` point to the same heap object.

```
Stack:
user1 → 0x100 (pointer)
user2 → 0x100 (pointer)

Heap:
0x100 → { name: "Grace" }
```

## 🧬 5. Hoisting Deep Dive

Hoisting = moving declarations to the top of their scope **during compilation**.

What actually happens:

- Variables and functions are registered in memory before code runs.
- Functions are hoisted with their definitions.
- `var` is hoisted but initialized with `undefined`.
- `let`/`const` are hoisted but not initialized → remain in TDZ.

### 🧩 Example

```js
sayHi(); // works

function sayHi() {
  console.log("Hi!");
}

console.log(x); // undefined
var x = 5;

console.log(y); // ReferenceError
let y = 10;
```

Memory before execution:

```
sayHi → <function>
x → undefined
y → TDZ
```

## 🧮 6. The Temporal Dead Zone (TDZ)

TDZ is the "no-access" period between scope creation and the actual line where a variable is declared.

```js
{
  // TDZ starts here
  console.log(value); // ReferenceError
  let value = 10; // TDZ ends here
  console.log(value); // 10
}
```

This mechanism prevents accidental use of variables before they're safely initialized - making you code more predictable.

## 🧹 7. Garbage Collection Basics

JavaScript uses **automatic memory management** - you never manually free memory.
The engine periodically reclaims memory that's no longer reachable.

### 🔄 Concept: Reachability

- Reachable values: values accessible from the root (global scope, call stack, closures).
- Unreachable values: detached objects that can't be reached anymore.

```js
let user = { name: "Ada" };
user = null; // original object now unreachable → elegible for GC
```

### 🧮 Reference Counting (simplified mental model)

V8 doesn't use pure reference counting, but the idea helps:

```
object { name: "Ada" }
→ reference count = 1 (user)
user = null
→ reference count = 0 → Garbage Collector frees memory
```

### 🧠 In V8 (modern engines):

- Young Generation (New Space): short-lived objects
- Old Generation (Old Space): long'live ones promoted over time.
- Mark-and-sweep algorithm: marks reachable objects, sweeps the rest.
- Generational GC: collects short-lived and long-lived objects differently for efficiency.

## 📚 8. Terminology Glossary

| Term                         | Definition                                                             |
| ---------------------------- | ---------------------------------------------------------------------- |
| **Stack**                    | Memory structure for primitives and call contexts (LIFO).              |
| **Heap**                     | Large memory area for objects and functions.                           |
| **Hoisting**                 | Declarations are registered before execution.                          |
| **TDZ (Temporal Dead Zone)** | Time between variable binding creation and initialization.             |
| **Reference Type**           | Variable points to object in heap memory.                              |
| **Primitive Type**           | Variable holds a direct immutable value.                               |
| **Reachability**             | Determines if data is still accessible and thus not garbage collected. |
| **Mark-and-Sweep**           | GC algorithm marking live objects, sweeping dead ones.                 |

## ⚠️ 9. Common Pitfalls & Best Practices

Pitfalls:

1. Assuming `var` and `let` behave the same
   → `var` is function-scoped; `let`/`const` are block-scoped.
2. Accessing `let` before declaration
   → TDZ ReferenceError surprises many beginners.
3. Confusing reference copies with value copies
   → Objects are referenced; primitives are copied.
4. Memory leaks from closures or DOM references
   → Retaining references prevents garbage collection.

### Best Practices:

- Use `const` by default - prevents accidental reassignment
- Use `let` when you must reassign
- Avoid `var` - it's function-scoped and hoisted with `undefined`
- Break object references when done: `obj = null`
- Initialize variables before using them

## 🧩 10. Practice Tasks

### 🧠 Task 1 — Predict the Output

```js
console.log(a);
var a = 10;
console.log(b);
let b = 20;
```

Explain line-by-line what’s happening in memory.

### 🧠 Task 2 — Stack vs Heap

Predict what happens:

```js
let n1 = 5;
let n2 = n1;
n2 = 10;

let obj1 = { val: 1 };
let obj2 = obj1;
obj2.val = 2;

console.log(n1, n2);
console.log(obj1.val, obj2.val);
```

Why are the results different?

### 🧠 Task 3 — Garbage Collection

```js
let car = { model: "Tesla" };
let garage = { car };
car = null;
garage = null;
```

Which objects remain in memory? Why?
