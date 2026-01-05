# 🧩 LESSON 11 — ES6 Classes & Inheritance

_(Syntactic Sugar Over the Prototype System)_

Welcome back! Now that you understand prototypes and the prototype chain, we’ll explore how ES6 classes simplify that mechanism while still being prototype-based under the hood. Classes look like traditional OOP, but they remain a cleaner syntax over JavaScript’s prototypes.

## 🎯 Learning Goals

- Map `class` to prototype-based behavior
- Use instance properties, static methods, and class fields
- Implement inheritance with `extends` and `super`
- Understand what happens behind the scenes when using `class`
- Visualize class hierarchies and prototype chains

## 🧱 1️⃣ ES6 Classes — Syntax over Prototypes

**Example**

```js
class Person {
  constructor(name) {
    this.name = name;
  }
  greet() {
    console.log(`Hello, ${this.name}`);
  }
}

const ada = new Person("Ada");
ada.greet(); // Hello, Ada
```

**Under the hood (pre-ES6 equivalent)**

```js
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function () {
  console.log(`Hello, ${this.name}`);
};

const ada = new Person("Ada");
ada.greet();
```

Both are functionally identical — `class` gives clearer syntax and implicit strict mode.

### 🧠 Behind the Hood: Creation Process

When JS encounters `class Person { ... }`, it:

- Creates a function object `Person` (a constructor)
- Assigns a `.prototype` object containing the methods from the class body
- Adds constructor references and `[[Prototype]]` linkages

Visualization:

```
ada
├─ name: "Ada"
└─ [[Prototype]] → Person.prototype
   ├─ greet()
   └─ [[Prototype]] → Object.prototype
```

## 🧩 2️⃣ Class Fields (Public & Private)

ES2022 introduced class fields for easier property initialization.

### Public fields

```js
class Counter {
  count = 0; // public field

  inc() {
    this.count++;
    console.log(this.count);
  }
}

const c = new Counter();
c.inc(); // 1
```

Equivalent to:

```js
function Counter() {
  this.count = 0;
}
Counter.prototype.inc = function () {
  this.count++;
};
```

### Private fields (`#`)

```js
class SecretBox {
  #secret = "xyz";

  reveal() {
    console.log(this.#secret);
  }
}

const box = new SecretBox();
box.reveal(); // xyz
console.log(box.#secret); // ❌ SyntaxError — not accessible
```

Private fields:

- Stored per instance
- Inaccessible outside the class
- Not properties on the prototype or `this` directly
- Avoid accidental overwrites

## ⚙️ 3️⃣ Static Methods and Properties

`static` members belong to the class itself, not instances.

```js
class MathUtil {
  static square(x) {
    return x * x;
  }
}

console.log(MathUtil.square(4)); // 16
```

**Under the hood**

```js
function MathUtil() {}
MathUtil.square = function (x) {
  return x * x;
};
```

Visualization:

```
MathUtil
├─ square() // static
└─ prototype → {}
```

Static members are accessed via the constructor, not instances:

```js
const m = new MathUtil();
console.log(m.square); // undefined
```

## 🧩 4️⃣ Inheritance with `extends`

Create a subclass that inherits from another class using `extends`.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    console.log(`${this.name} makes a sound`);
  }
}

class Dog extends Animal {
  speak() {
    console.log(`${this.name} barks`);
  }
}

const rex = new Dog("Rex");
rex.speak(); // Rex barks
```

### 🔍 Under the Hood: Prototype Linkage

- `Dog.prototype.[[Prototype]] → Animal.prototype`
- `Dog.[[Prototype]] → Animal`

Visualization:

```
rex
├─ name: "Rex"
└─ [[Prototype]] → Dog.prototype
   ├─ speak()
   └─ [[Prototype]] → Animal.prototype
      ├─ speak()
      └─ [[Prototype]] → Object.prototype
```

## 🧩 5️⃣ The `super` Keyword — Parent Calls

**Inside subclass constructors:** call `super()` before using `this`.

```js
class Animal {
  constructor(type) {
    this.type = type;
  }
}

class Dog extends Animal {
  constructor(name) {
    super("dog"); // calls Animal constructor
    this.name = name;
  }
}
```

**Inside methods:** `super.method()` calls the parent prototype’s method.

```js
class Parent {
  greet() {
    console.log("Hello from Parent");
  }
}

class Child extends Parent {
  greet() {
    super.greet();
    console.log("Hello from Child");
  }
}

new Child().greet();
// Hello from Parent
// Hello from Child
```

### 🧠 Behind the Hood — What `extends` Does

When you write `class Dog extends Animal {}`, the engine effectively does:

```js
function Dog() {
  return Reflect.construct(Animal, arguments, Dog);
}
Object.setPrototypeOf(Dog.prototype, Animal.prototype);
Object.setPrototypeOf(Dog, Animal);
```

This sets up both prototype and constructor-level inheritance.

## 🧱 6️⃣ Class Hierarchy Visualization

Example with `extends` and static methods:

```js
class A {
  static hello() {
    console.log("Hello from A");
  }
}
class B extends A {
  static hello() {
    super.hello();
    console.log("Hello from B");
  }
}

B.hello();
```

Prototype and constructor linkage:

- `B.[[Prototype]] → A`
- `B.prototype.[[Prototype]] → A.prototype`
  `super.hello()` inside `B.hello()` looks up `hello` on `A` (the parent constructor).

## 🧩 7️⃣ Common Class Patterns

**Factory + class combo**

```js
class User {
  constructor(name) {
    this.name = name;
  }
  static fromJSON(json) {
    const data = JSON.parse(json);
    return new User(data.name);
  }
}

const u = User.fromJSON('{"name": "Ada"}');
console.log(u.name); // Ada
```

**Abstract base (by convention)**

```js
class Shape {
  area() {
    throw new Error("Abstract method must be implemented!");
  }
}
```

## 🔬 8️⃣ Engine Internals: How V8 Implements Classes

- Each class becomes a function constructor internally
- Instance methods live on `ClassName.prototype`
- Static methods live on the constructor (`ClassName` itself)
- Prototype chain uses hidden `[[Prototype]]` links:
  - Instances: `instance → Class.prototype`
  - Constructors: `Child → Parent`
- Simplified model:
  - `instance.[[Prototype]] === Class.prototype`
  - `Class.[[Prototype]] === ParentClass`

## 📚 9️⃣ Terminology Glossary

| Term                  | Meaning                                          |
| --------------------- | ------------------------------------------------ |
| Class                 | Syntactic sugar for constructor + prototype      |
| Constructor           | Special method called on `new`                   |
| Prototype Chain       | Inheritance path for instance methods            |
| Static Method         | Belongs to class, not instances                  |
| Class Field           | Property initialized per instance                |
| Private Field         | Instance-scoped variable inaccessible externally |
| `super()`             | Calls parent constructor or method               |
| `extends`             | Sets up inheritance between classes              |
| `Reflect.construct()` | Low-level operation for subclass construction    |

## ⚠️ 1️⃣0️⃣ Common Pitfalls & Best Practices

**Pitfalls**

- Forgetting `super()` in subclass constructors leads to `ReferenceError`

```js
class A {}
class B extends A {
  constructor() {
    this.x = 1; // ❌ ReferenceError
  }
}
```

- Accessing instance props in static context

```js
class X {
  static f() {
    console.log(this.nameProp); // undefined
  }
}
```

- Using arrow functions as methods (no own `this`); prefer regular methods on the prototype

**Best practices**

- Always call `super()` before using `this` in subclasses
- Use static methods for utility helpers
- Avoid deep inheritance chains (favor composition)
- Prefer private fields (`#x`) for encapsulation
- Remember classes are prototype-based underneath

## 🧩 1️⃣1️⃣ Practice Tasks

**Task 1 — Inheritance chain**

```js
class Animal {
  speak() {
    console.log("generic sound");
  }
}
class Dog extends Animal {
  speak() {
    console.log("bark");
  }
}

const d = new Dog();
d.speak();
```

Question: What does `Dog.prototype.[[Prototype]]` point to?

**Task 2 — Static vs instance**

```js
class MathOps {
  static add(a, b) {
    return a + b;
  }
  multiply(a, b) {
    return a * b;
  }
}
console.log(MathOps.add(2, 3)); // ?
console.log(new MathOps().multiply(2, 3)); // ?
```

**Task 3 — `super` constructor flow**

```js
class A {
  constructor(x) {
    this.x = x;
  }
}
class B extends A {
  constructor(x, y) {
    super(x);
    this.y = y;
  }
}
const obj = new B(1, 2);
console.log(obj);
```

Question: Describe step-by-step what happens when `new B(1, 2)` executes.

## 🧭 Coming Next

**Lesson 12: The `this` Keyword & Binding Mechanics**

- How `this` is bound dynamically at call time
- The four binding rules (default, implicit, explicit, `new`)
- How `.call()`, `.apply()`, and `.bind()` manipulate `this`
- Arrow functions and lexical `this`
