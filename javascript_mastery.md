🧠 JavaScript Mastery Curriculum: From Fundamentals to Engine Internals

📘 Level 1: Core Foundations — How JavaScript Really Works
Goal: Build mental models of how JS runs code — memory, execution context, scope, and the call stack.

---

[Lesson 1: The JavaScript Execution Model](lesson_1.md)

- What happens from writing a script -> to it running.
- Compilation vs Interpretation.
- The role of the V8 engine.
- Phases: Parsing -> AST creation -> Compilation -> Execution.
- Code Demo: Simple console program; trace how it’s executed.
- Visual: Show call stack and memory.

---

[Lesson 2: Variables, Data Types, and Memory](lesson_2.md)

- var, let, const — how they differ under the hood.
- Primitive vs Reference types.
- How JS stores values in stack vs heap.
- Hoisting and the Temporal Dead Zone (TDZ).
- Garbage collection and reference counting basics.

---

[Lesson 3: Functions and Scope](lesson_3.md)

- Function declarations vs expressions.
- Function execution context.
- Lexical scope and environment chains.
- How scope resolution happens (LE -> VE -> outer).
- Visualization of nested scopes.

---

[Lesson 4: The Call Stack and Execution Contexts](lesson_4.md)

- Step-by-step execution context creation.
- Stack frames in V8.
- Global, function, and eval execution contexts.
- Call stack overflow demonstration.
- Visualization: Stack growth & pop mechanics.

---

[Lesson 5: Hoisting, Closures & Memory Persistence](lesson_5.md)

- Deep dive into hoisting rules.
- Inner functions retaining access to outer variables.
- How closures are implemented in memory.
- Real-world examples (event handlers, async tasks).

---

📗 Level 2: Advanced Core — The Event Loop and Asynchronous JS
Goal: Master how JS handles concurrency, async code, and browser APIs.

---

[Lesson 6: The Event Loop Explained Visually](lesson_6.md)

- Single-threaded model and why it matters.
- The call stack + callback queue + microtask queue.
- Macrotasks vs microtasks.
- Event loop in browsers vs Node.js.

---

Lesson 7: Callbacks, Promises & Async/Await

- Callback patterns and “callback hell.”
- Promises as syntactic sugar + internal states.
- Async/await — how it rewrites Promise chains.
- Behind the hood: microtask queue management.
- Error propagation in async code.

---

Lesson 8: Timers, Tasks, and the Browser APIs

- How setTimeout, setInterval, and requestAnimationFrame integrate with the event loop.
- The role of the Web APIs.
- Example visualization of async execution order.

---

📙 Level 3: Deep Internals — Objects, Prototypes, and Classes
Goal: Fully understand how JavaScript handles object-oriented behavior and inheritance.

---

Lesson 9: Objects and Property Descriptors

- Internal structure of JS objects.
- Property attributes (configurable, enumerable, writable).
- Object cloning and shallow vs deep copies.
- Visual: object in memory and reference behavior.

---

Lesson 10: Prototypes and the Prototype Chain

- [[Prototype]] linkage and lookup mechanism.
- Object.create, **proto**, prototype.
- How JS finds methods up the chain.
- Visualization: object chain traversal.

---

Lesson 11: ES6 Classes and Inheritance

- Syntactic sugar over prototypes.
- Class fields, static methods, inheritance model.
- Super calls and constructor behavior.
- Under-the-hood equivalent of class syntax.

---

📒 Level 4: Modern JS & Advanced Mechanisms
Goal: Learn how modern JS features and internals optimize runtime performance.

---

Lesson 12: Modules and the JS Runtime

- ES Modules vs CommonJS.
- Import/export behavior and module scope.
- How bundlers and runtime loaders handle modules.

---

Lesson 13: Memory Management and the Garbage Collector

- Stack vs heap memory revisited.
- Mark-and-sweep algorithm overview.
- How circular references are handled.
- Performance tuning via GC awareness.

---

Lesson 14: JIT Compilation & Performance

- How V8 optimizes code via Ignition & TurboFan.
- Hidden classes and inline caches.
- Why de-optimizations occur.
- Performance measurement tools and coding tips.

---

📔 Level 5: Expert Topics & Real-World Mastery
Goal: Apply internal knowledge to real-world challenges, debugging, and performance optimization.

---

Lesson 15: Event Delegation & The DOM Connection

- How JS interfaces with the DOM and Event Loop.
- Bubbling, capturing, and delegation patterns.
- Lesson 16: Asynchronous Patterns in the Wild
- Combining async/await with event streams.
- Practical API request handling patterns.
- Error management strategies.

---

Lesson 17: Debugging & Profiling

- Using DevTools to inspect call stacks and memory.
- Breakpoints, performance profiling, async tracing.

---

Lesson 18: Deep Dive into Engine Internals

- Detailed architecture of V8 Engine.
- Parsing, bytecode, and optimization passes.
- JIT compilation in action.
- How engines differ (V8, SpiderMonkey, JavaScriptCore).

---

Lesson 19: Design Patterns in JavaScript

- Singleton, Observer, Factory, Module patterns.
- Understanding patterns in JS context (prototypes & closures).

---

Lesson 20: Putting It All Together — The JS Mastery Project

- Apply everything to build a small but complex JS system.
- Trace its execution model manually.
- Identify performance bottlenecks and memory leaks.

---

🎓 Bonus Add-ons

- Mini quizzes at the end of each level.
- Hands-on coding exercises.
- Deep-dive visual maps (execution context tree, memory map, prototype chain).
- Real-world code analysis (reading open-source JS and explaining the internals).
