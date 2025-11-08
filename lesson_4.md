# 🧩 LESSON 4 — The Call Stack & Execution Contexts

🎯 Learning Goals

- Build a precise mental model of execution context creation
- See how V8 represents stack frames
- Distinguish global, function, and eval execution contexts
- Understand call stack overflow (why it happens and how to avoid it)
- Visualize stack growth and teardown for real code

## Concept Overview

### Execution Context (EC)

A container V8 creates whenever code starts executing (global script/module, function call, or `eval`). It holds:

- Lexical Environment (LE): bindings and a link to the outer environment
- Variable Environment (VE): historically distinct from LE; in practice, both manage bindings
- ThisBinding: the this value for that context
- Code type: Script, Module, or Eval
- Realm/Scope info: (hosted structures for intrinsics, not usually visible)

### Call Stack

A LIFO stack of stack frames (one frame per active execution context).
Top frame = currently running function.

## Step-by-Step: Execution Context Creation

We’ll use a small program and trace it:

```js
// file: lesson4.js
"use strict";

const a = 2;

function square(x) {
  const y = x * x;
  return y;
}

function hypotenuse(p, q) {
  const s1 = square(p);
  const s2 = square(q);
  return Math.sqrt(s1 + s2);
}

console.log("h:", hypotenuse(3, 4));
```

Creation (compile) phase:

- Parse -> AST
- Hoist declarations into the Global EC:
  - square -> <function>
  - hypotenuse -> <function>
  - `a` binding exists (TDZ) then initialized on list line

Execution (run) phase:

1. Enter Global EC -> push Global frame
   - Initialize `a = 2`
2. Evaluate `console.log("h:", hypotenuse(3, 4))`
   - Need to call hypotenuse(3,4) -> create Function EC for hypotenuse -> push frame
3. Inside `hypotenuse`

   - Bind params: `p=3`, `q=4`
   - `const s1 = square(p)` → create Function EC for square → push
     - Bind `x=3`, compute `y=9`, return 9 → pop square
   - `const s2 = square(q)` → create another square EC → push
     - Bind `x=4`, compute `y=16`, return 16 → pop square
   - `return Math.sqrt(9 + 16)` → returns 5 → pop hypotenuse

4. Back in Global EC: evaluate `console.log("h:", 5)` and print

Visual: Call stack over time

```
(1) start
[top]
│ Global()
└──────────

(2) call hypotenuse
[top]
│ hypotenuse()
│ Global()
└──────────

(3) hypotenuse → call square(p)
[top]
│ square(x=3)
│ hypotenuse()
│ Global()
└──────────

(4) return from square
[top]
│ hypotenuse()
│ Global()
└──────────

(5) hypotenuse → call square(q)
[top]
│ square(x=4)
│ hypotenuse()
│ Global()
└──────────

(6) return → finish hypotenuse
[top]
│ Global()
└──────────
```

Visual: Memory snapshots (simplified)

```
Global LE:
  a → 2
  square → <function>
  hypotenuse → <function>

hypotenuse LE (during call):
  p → 3
  q → 4
  s1 → 9
  s2 → 16

square LE (during call):
  x → 3 | 4
  y → 9 | 16
```

## Stack Frames in V8 (Engine View)

V8 represents each active call as a stack frame containing (conceptually):

- Return address (where to continue in caller)
- Frame pointer / stack pointer bookkeeping
- Arguments and receiver (the this value)
- Local variables & temporaries (registers/spill slots)
- Metadata (which function, which bytecode offset / optimized code info)

Two common kinds of frames:

- Interpreter frames (Ignition): executing bytecode
- Optimized frames (TurboFan): executing machine code (may be deoptimized back)

You can often see frames via stack traces:

```js
function a() {
  b();
}
function b() {
  c();
}
function c() {
  console.log(new Error("trace").stack);
}
a();
```

## Global, Function, and Eval Execution Contexts

- Global EC
  - Created once when the script/modules starts
  - Hosts global binding (`globalThis` / `window` / Node’s `global`)
  - `this` binding:
    - Module: undefined by spec
    - Script (sloppy mode): global object
    - Script (strict): undefined
- Function EC
  - Created on every call
  - Binds parameters, local variables, function declarations
  - Sets this depending on call form:
    - Plain call in strict mode → `undefined`
    - Method call → object before the dot
    - `call/apply/bind` → explicit
    - Arrow functions capture `this` lexically (no own `this`)
- Eval EC
  - `eval("code")` executes within the current execution context’s environment (direct eval)
  - Indirect eval (e.g., (0, eval)("code")) runs in global scope
  - Usually best avoided for security/perf reasons

Mini-demo:

```js
"use strict";
const outer = 1;

function run() {
  const inner = 2;
  eval("console.log('eval sees:', outer, inner)"); // direct eval → sees both
}

run();

(0, eval)("console.log('indirect eval sees only global');"); // global eval
```

## Call Stack Overflow Demonstration

What it is:
The stack is finite. If recursive calls never bottom out, frames pile up until the engine throws RangeError: Maximum call stack size exceeded.

Demo (safe to run, small depth):

```js
function recurse(n) {
  if (n === 0) return "done";
  return recurse(n - 1);
}
console.log(recurse(10)); // "done"
```

Overflow (don’t run with huge depth; this is illustrative):

```js
function boom() {
  boom();
}
boom(); // RangeError: Maximum call stack size exceeded
```

Safer pattern (trampolining / iteration):

```js
// Replace recursion with a loop when possible:
function sumTo(n) {
  let s = 0;
  for (let i = 1; i <= n; i++) s += i;
  return s;
}
```

In JS, proper tail calls (PTC) are specified but not enabled in V8 in typical environments, so deep recursion can overflow. Prefer loops or chunked async recursion (e.g., setTimeout or microtask breaks) for very deep operations.

## Visualization: Stack Growth & Pop Mechanics

Let’s watch a nested call evolve:

```js
function f1() {
  f2();
}
function f2() {
  f3();
}
function f3() {
  /* base work */
}

f1();
```

Growth (push):

```
Call stack after f1() call:
[top]
│ f1
│ Global
└───────

After f1 → f2:
[top]
│ f2
│ f1
│ Global
└───────

After f2 → f3:
[top]
│ f3
│ f2
│ f1
│ Global
└───────
```

Pop:

```
f3 returns → pop f3
f2 returns → pop f2
f1 returns → pop f1
Resume Global
```

Key rules:

- Only one top frame runs at any moment (single-threaded JS)
- A function must return for its frame to pop
- Async callbacks don’t “pause” a frame; they run later in new frames when dequeued by the event loop

## Step-Through Demo with Comments

```js
"use strict";

function A(m) {
  // Declared in Global EC
  const doubled = m * 2; // A's LE: { m, doubled }
  return B(doubled); // Call → push B frame
}

function B(n) {
  // Declared in Global EC
  const squared = n * n; // B's LE: { n, squared }
  return squared;
}

const result = A(5); // Push A frame
console.log(result); // 100
```

Resolution & frames:

- A(5) → push A: binds m=5, compute doubled=10
- B(10) → push B: binds n=10, compute squared=100, return → pop B
- Return to A, return 100 → pop A
- Global prints 100

## Behind the Hood (Engine View)

- Parser builds AST; functions become function objects with a hidden [[Environment]] link to the outer environment
- Ignition executes bytecode; each call → interpreter frame
- TurboFan may optimize hot calls → optimized frames
- Deopt can unwind optimized frames to interpreter frames if type assumptions break
- GC walks from roots (globals, current stack frames, live closures) to determine reachability; locals in active frames are roots

## Terminology Glossary

- Execution Context (EC): Runtime box holding LE, VE, and ThisBinding
- Stack Frame: The concrete activation record for one EC on the call stack
- Global EC: Root context for a script/module
- Function EC: Context created per function call
- Eval EC: Context for eval code (direct/indirect rules)
- Lexical Environment (LE): Scope record + outer link used for identifier lookup
- ThisBinding: The this value chosen by call-site semantics
- Deoptimization (Deopt): Falling back from optimized code to interpreter
- RangeError (Stack Overflow): Error when the call stack exceeds its limit

## Common Pitfalls & Best Practices

Pitfalls

1. Deep recursion without base cases -> stack overflow
2. Assuming async “pauses” a frame -> it doesn’t; callbacks run later in new frames
3. Relying on tail-call optimization -> not available in V8’s typical configs
4. Confusing `this` with lexical scope -> `this` is from the call site; scope is lexical

Best Practices

- Prefer loops for very deep iterations; or break work into chunks (timers/microtasks)
- Keep functions small; easier to optimize and reason about
- Use strict mode (modules are strict by default)
- Avoid `eval` (security/perf)
- For debugging, capture stacks early (`new Error().stack`)

## Practice Tasks

### Task 1 — Draw the Stack

Predict the stack sequence for:

```js
function g(x) {
  return h(x - 1) + 1;
}
function h(y) {
  return y > 0 ? g(y) : 0;
}
console.log(g(2));
```

### Task 2 — Prevent Overflow

Refactor a recursive factorial into an iterative version that handles n=100000 without overflowing (no BigInt required—just structure).

### Task 3 — Eval Context

What does this print and why?

```js
"use strict";
let x = 1;
function run() {
  let x = 2;
  eval("console.log(x)");
}
run();
(0, eval)("try { console.log(x); } catch(e) { console.log(e.name); }");
```

Explain using direct vs indirect eval

## Quick Cheatsheet (Mental Model)

```
Write code → Parse (AST) → Create Global EC → Execute
Function call → Create Function EC → Push frame
Return/throw → Pop frame → Resume caller
Async callback → scheduled → later: new Function EC pushed

```
