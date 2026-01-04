# 🧩 LESSON 10 — Prototypes & The Prototype Chain

🎯 Learning Goals
By the end of this lesson, you’ll understand:

- What [[Prototype]] is and how objects are linked
- The difference between `__proto__`, `prototype`, and `Object.create()`
- How property and method lookups traverse the chain
- How classes and constructor functions use prototypes internally
- How to visualize the object chain traversal process

## 1. 🧠 Every Object Has a Hidden Link: [[Prototype]]

Each object in JavaScript has an internal, hidden reference called [[Prototype]], which points to another object — its parent prototype.

When you try to access a property that doesn’t exist on the object, JS automatically looks up the chain until it finds it (or reaches the end).

🧩 Example

```js
const user = { name: "Ada" };
console.log(user.toString);
```

- user doesn’t have its own toString property.
- The engine looks up user.[[Prototype]] → Object.prototype
- Finds toString there → returns it.

```
user → [[Prototype]] → Object.prototype → [[Prototype]] → null
```

🧩 Visualization

```
Object Lookup Chain:

user
    ├─ name: "Ada"
    └─ [[Prototype]] → Object.prototype
                            ├─ toString()
                            ├─ hasOwnProperty()
                            └─ [[Prototype]] → null
```

If lookup reaches null, the engine stops → property is undefined.

## 2. ⚙️ Object.create() — Explicit Prototype Creation

You can manually set an object’s prototype using `Object.create()`.

```js
const base = { kind: "base" };

const derived = Object.create(base);
derived.name = "Derived";

console.log(derived.kind); // "base"
console.log(Object.getPrototypeOf(derived) === base); // true
```

derived doesn’t have kind, so it looks up to base.

Visualization:

```
derived
    ├─ name: "Derived"
    └─ [[Prototype]] → base
                            ├─ kind: "base"
                            └─ [[Prototype]] → Object.prototype
```

## 3. 🧩 The `__proto__` Property

`__proto__` is a public accessor for the internal [[Prototype]].

```js
const a = {};
console.log(a.`__proto__` === Object.prototype); // true
```

✅ It’s mostly for debugging — avoid using it in production.
🧩 Instead, use:

```js
Object.getPrototypeOf(a);
Object.setPrototypeOf(a, newProto);
```

## 4. 🧱 Constructor Functions and .prototype

Before class syntax existed, JavaScript used constructor functions for creating linked objects.

Example:

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function () {
  console.log("Hi, I’m " + this.name);
};

const p1 = new Person("Ada");
p1.sayHi(); // "Hi, I’m Ada"
```

Under the Hood:

1. When you call `new Person()`:

- A new empty object is created.
- Its `[[Prototype]]` is set to `Person.prototype`.
- The function Person executes, setting properties on `this`.

2. p1 inherits from `Person.prototype`.

Visualization:

```
p1
    ├─ name: "Ada"
    └─ [[Prototype]] → Person.prototype
                            ├─ sayHi()
                            └─ [[Prototype]] → Object.prototype
```

Important Distinctions

| Concept         | Description                                                                       |
| --------------- | --------------------------------------------------------------------------------- |
| `__proto__`     | Reference on **instance** to its prototype                                        |
| `.prototype`    | Property on **constructor function** that becomes the prototype for new instances |
| `[[Prototype]]` | Internal hidden link used during property lookup                                  |

### 🧩 Example: Constructor Chain

```js
function Animal(kind) {
  this.kind = kind;
}
Animal.prototype.speak = function () {
  console.log(`${this.kind} makes a noise.`);
};

function Dog(name) {
  Animal.call(this, "dog");
  this.name = name;
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.speak = function () {
  console.log(`${this.name} barks!`);
};

const rex = new Dog("Rex");
rex.speak(); // Rex barks!
```

### 🧠 Prototype Chain for rex

```
rex
  ├─ name: "Rex"
  ├─ kind: "dog"
  └─ [[Prototype]] → Dog.prototype
                          ├─ speak()  // overridden
                          └─ [[Prototype]] → Animal.prototype
                                                  ├─ speak()
                                                  └─ [[Prototype]] → Object.prototype
```

## 5. 🧩 Property Lookup Mechanism

When you access a property:

1. Engine checks the object’s own properties.
2. If not found → follow the [[Prototype]] link.
3. Repeat until:
   - Found the property (returns value), or
   - Reached null (returns undefined).

Example:

```js
console.log(rex.toString);
```

1. rex → not found
2. Dog.prototype → not found
3. Animal.prototype → not found
4. Object.prototype → ✅ found
5. Returns Object.prototype.toString

### 🧠 Visualization of Lookup

```
[Start] rex
    ↓
[Dog.prototype]
    ↓
[Animal.prototype]
    ↓
[Object.prototype]
    ↓
[null]
```
