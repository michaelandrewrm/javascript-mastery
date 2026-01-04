# Lesson 1: The JavaScript Execution Model

**Learning Goals:**

By the end of this lesson, you’ll understand:

1. What happens when you write and run JavaScript, from source code to execution.
2. The difference between compilation and interpretation in JavaScript.
3. How the V8 engine (used in Chrome and Node.js) runs your code.
4. The complete lifecycle: Parsing → AST → Compilation → Execution.
5. How the call stack and memory behave step-by-step.

## 1. From Script to Running Code

When you write:

```js
console.log("Hello, World!");
```

...it feels like JS "just runs."
But under the hood, the engine performs a complex pipeline before printing that message.

Let's walk through it:

```
Your Source Code → Lexing → Parsing → AST → Compilation → Execution
```

## 2. Compilation vs Interpretation

### The Classic Distinction

| Phase  | Compiler               | Interpreter           |
| ------ | ---------------------- | --------------------- |
| When   | Before execution       | During execution      |
| Output | Machine code (binary)  | Immediate execution   |
| Speed  | Faster (pre-optimized) | Slower (line-by-line) |

### JavaScript’s Reality

Javascript is both compiled and interpreted.
Modern engines like V8 use Just-In-Time (JIT) compilation strategy:

1. Parsing: Source → AST
2. Interpreter (Ignition): Converts AST → Bytecode, runs it immediately
3. Profiler: Detects "hot" code (frequently run functions)
4. Optimizing compiler (TurboFan): Converts hot bytecode → Machine Code
5. De-optimizer: Reverts optimized code if assumptions break so JS begins interpreted, but "heats up" into compiled native code.

## 3. The Role of the V8 Engine

V8 = JavaScript Engine used in Chrome & Node.js

### Main Components:

```
Parser → Converts code to AST
Ignition → Interprets to bytecode
TurboFan → Optimizes to machine
Garbage Collector (Orinoco)
Heap & Stack Memory
```

Workflow:

1. Parser → Reads source, builds AST
2. Ignition → Translate AST → Bytecode
3. TurboFan → Optimizes hot bytecode → Native machine code
4. Runtime → Executes with memory management, Garbage Collector (GC), and event loop integration

## 4. Code Demo

```js
// file: example.js
function greet(name) {
  const message = "Hello, " + name + "!";
  return message;
}

const user = "Ada";
const output = greet(user);

console.log(output);
```

### Step-by-Step Breakdown

#### Phase 1: Parsing

- Engine scans the code, token by token (function, const, return, etc.)
- Builds an AST representing structure:

  ```
  Program
  ├─ FunctionDeclaration (greet)
  │   ├─ Identifier(name)
  │   └─ BlockStatement
  │       ├─ VariableDeclaration (message)
  │       └─ ReturnStatement (message)
  ├─ VariableDeclaration (user)
  ├─ VariableDeclaration (output)
  └─ ExpressionStatement (console.log)
  ```

#### Phase 2: Compilation

- Ignition compiles AST → Bytecode instructions like:

  ```
  LoadConstant "hello, "
  Add name
  Add "!"
  Return message
  ```

- This bytecode runs on the V8 interpreter

### Phase 3: Execution

- The engine creates the Global Execution Context.
- It allocates memory for greet, user, and output.
- Then runs top-to-bottom.

#### Execution timeline:

| Step | Action                                       | Stack                | Memory                                             |
| ---- | -------------------------------------------- | -------------------- | -------------------------------------------------- |
| 1.   | Parse & hoist `greet`                        | `Global()`           | `greet → func`, `user → uninit`, `output → uninit` |
| 2.   | Assign `user = "Ada"`                        | `Global()`           | `user → "Ada"`                                     |
| 3.   | Call `greet(user)`                           | `Global() → greet()` | `name → "Ada"`, `message → "Hello, Ada!"`          |
| 4.   | Return `"Hello, Ada!"`                       | `Global()`           | `output → "Hello, Ada!"`                           |
| 5.   | `console.log(output)` → prints "Hello, Ada!" | `Global()`           | -                                                  |

## 5. Visual Interpretation

### Call Stack

```
Call stack
[top]
│
│ console.log()
│ greet()
│ Global()
└────────────────────
```

Each function call pushes a frame; each return pops it.

### Memory Snapshot

```
Memory
greet → <function>
user → "Ada"
output → "Hello, Ada!"
message → (temporary, inside greet)
```

## 6. Behind the Hood (V8 Internals)

- Scanner / Parser
  - Breaks code into tokens and syntax tree (AST).
  - Detects syntax errors before execution.
- Ignition (Interpreter)
  - Converts AST → bytecode.
  - Executes bytecode line by line.
  - Collects runtime information (types, shapes).
- TurboFan (JIT Compiler)
  - Identifies "hot" functions (like `greet` if called often).
  - Compiles them into machine code for your CPU.
  - Uses inline caching and hidden classes for performance.
- Garbage Collector (Orinoco)
  - Frees memory of variables no longer reachable.

## 7. Key Terms

| Term                        | Meaning                                                        |
| --------------------------- | -------------------------------------------------------------- |
| AST                         | Abstract Syntax Tree - internal structured form of your code   |
| Execution Context           | Environment holding variables, functions references and `this` |
| Call Stack                  | Stack structure storing active execution context               |
| Heap                        | Memory area where objects and functions live                   |
| Interpreter (Ignition)      | Executes bytecode line by line                                 |
| JIT Compiler (TurboFan)     | Converts hot bytecode to machine code                          |
| Garbage Collector (Orinoco) | Automatic memory cleanup                                       |
| Inline Cache (IC)           | Optimization caching property access patterns                  |

## 8. Common Pitfalls

- Misconception: _"JavaScript ins't compiled."_
  - It is, but just in time - compilation happens dynamically before execution.
- Assuming "line-by-line" execution:
  - JS engines preprocess and compile functions before running.
- Believing code runs instantly:
  - Parsing, compiling, and optimizing all happen under the hood - modern engines are just incredibly fast.

## 9. Practice

Predict the order of operations here:

```js
sayHi();

function sayHi() {
  console.log("Hi!");
}
```

**Think:**

- During parsing, what happens to `sayHi`?
- During execution, what's in memory?
- What's on the call stack when it runs?

**During parsing:**

- The function `sayHi` is hoisted
- its entire definition is stored in memory.

**At runtime:**

- Global Execution Context created
- `sayHi` already defined
- `sayHi()` is called → pushes `sayHi()` to stack → runs logs "Hi!" → pops.

**Output:**

```
Hi!
```

## 🧭 Summary Mental Model

```
Source Code
    ↓
[ Parser ] → AST
    ↓
[ Ignition ] → Bytecode → Run
    ↓
[ TurboFan ] → Optimized Machine Code
    ↓
[ Call Stack + Heap ] → Execution
```

> JavaScript isn't "interpreted line by line", it's parsed, compiled, executed, and optimized in a dynamic, multi-phase pipeline.
