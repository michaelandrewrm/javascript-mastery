# 🧩 LESSON 9 — Objects and Property Descriptors

🎯 Learning Goals
By the end of this lesson, you’ll understand:

- The internal layout of JavaScript objects in memory
- The role of property descriptors — and how to use them
- The difference between data and accessor properties
- How to clone objects safely (shallow vs deep copies)
- How reference behavior works when objects are assigned or copied

## 1. Objects: The Core Data Structure of JavaScript

Everything (almost) in JavaScript — arrays, functions, classes — is built on objects.

```js
const user = {
  name: "Ada",
  age: 30,
};
```

🧠 Conceptually:

Each object is a key-value store, but with extra hidden metadata — descriptors, internal links, and shape info.

In memory (conceptually):

```
user (object)
    ├─ name → "Ada"
    ├─ age → 30
    └─ [[Prototype]] → Object.prototype
```

Each property has its own descriptor, which defines:

- Value (actual data)
- Writable (can be changed?)
- Enumerable (shows up in loops?)
- Configurable (can be deleted or redefined?)

## 2. Property Descriptors Explained

Let’s inspect a property’s descriptor:

```js
const user = { name: "Ada" };

console.log(Object.getOwnPropertyDescriptor(user, "name"));
```

🧩 Output:

```js
{
    value: "Ada",
    writable: true,
    enumerable: true,
    configurable: true
}
```

So every normal property actually hides three flags controlling behavior.

You can define your own descriptor manually:

```js
const user = {};

Object.defineProperty(user, "id", {
  value: 101,
  writable: false,
  enumerable: false,
  configurable: false,
});
```

Now:

```js
user.id = 200; // ignored (not writable)
delete user.id; // fails silently (not configurable)
console.log(user.id); // 101
```

### 🧩 Data vs Accessor Properties

A property can either store a value or compute one dynamically via getters/setters.

Data Property

```js
const obj = { x: 10 };
```

Descriptor:

```js
{
    value: 10,
    writable: true,
    enumerable: true,
    configurable: true
}
```

Accessor Property

```js
const circle = {
    radius: 2,
    get area() {
        return Math.PI \* this.radius \*\* 2;
    }
};

console.log(circle.area); // computes dynamically
```

Descriptor:

```js
{
    get: ƒ,
    set: undefined,
    enumerable: true,
    configurable: true
}
```

❗ Accessor properties don’t have a value or writable flag — they use get and set functions instead.

## 3. Enumerability & Iteration

Enumerable = visible in enumeration loops.

```js
const user = {};
Object.defineProperty(user, "secret", {
  value: "hidden",
  enumerable: false,
});

for (let key in user) console.log(key); // nothing
console.log(Object.keys(user)); // []
console.log(Object.getOwnPropertyNames(user)); // ["secret"]
```

## 4. Configurability — the “Lock” Flag

`configurable: false` makes a property immutable to deletion or descriptor change.

```js
const car = {};
Object.defineProperty(car, "model", {
  value: "Tesla",
  configurable: false,
});

delete car.model; // cannot delete
Object.defineProperty(car, "model", { value: "Ford" }); // throws TypeError
```

Once configurable is false, the property can never be redefined.

## 5. Writable — Controlling Mutability

If writable is false, the property value can’t change:

```js
const obj = {};
Object.defineProperty(obj, "pi", {
  value: 3.14159,
  writable: false,
});

obj.pi = 4; // ignored in non-strict mode, TypeError in strict
console.log(obj.pi); // 3.14159
```

## 6. Object Cloning & Reference Behavior

### 🧩 Reference Assignment

Objects are reference types — assignments copy the reference, not the data.

```js
const a = { value: 10 };
const b = a;

b.value = 99;

console.log(a.value); // 99 same object
```

Memory Visualization:

```
Stack:
a → 0x100
b → 0x100

Heap:
0x100 → { value: 99 }
```

### 🧩 Shallow Copy

```js
const obj = { name: "Ada", skills: { lang: "JS" } };
const clone = Object.assign({}, obj);

clone.name = "Grace";
clone.skills.lang = "Python";

console.log(obj.skills.lang); // "Python" ❗ shared inner object
```

Why?

- Object.assign() only copies top-level properties.
- Nested objects are still references.

### 🧩 Deep Copy

To truly copy nested structures:

Option 1 — JSON Trick (simple, but limited)

```js
const deepClone = JSON.parse(JSON.stringify(obj));
```

⚠️ Loses functions, undefined, and special types (Date, Map, etc.).

Option 2 — Structured Clone (modern browsers)

```js
const clone = structuredClone(obj);
```

✅ Handles nested objects, Dates, Maps, Sets, TypedArrays, etc.

Option 3 — Manual recursive clone (for learning)

```js
function deepClone(obj) {
  if (typeof obj !== "object" || obj === null) return obj;
  const copy = Array.isArray(obj) ? [] : {};
  for (let key in obj) {
    copy[key] = deepClone(obj[key]);
  }
  return copy;
}
```

## 7. Visualizing an Object in Memory

Let’s model:

```js
const user = {
  name: "Ada",
  contact: { email: "ada@lab.com" },
};

const copy = user;
```

Memory Representation:

```
Stack:
    user → 0xA1
    copy → 0xA1

Heap:
    0xA1 → {
        name: "Ada",
        contact: → 0xB2
    }

    0xB2 → { email: "ada@lab.com" }
```

Both user and copy point to the same heap object.

## 8. Hidden Classes and Shapes (V8 Optimization Insight)

V8 uses hidden classes (also called shapes) for performance.

When you add properties consistently, objects share the same internal structure, making property access fast (JIT optimizations).

Example:

```js
function User(name, age) {
  this.name = name;
  this.age = age;
}
```

All User objects with identical property layout share one hidden class.
If you later add a property inconsistently, V8 must change shapes, which slows things down.

**Best Practice:**
Always define object properties in the same order and shape for predictable optimization.

## 9. Object Freezing and Sealing

| Method                          | Effect                                                |
| ------------------------------- | ----------------------------------------------------- |
| `Object.preventExtensions(obj)` | No new properties can be added                        |
| `Object.seal(obj)`              | No new or deleted properties, existing still writable |
| `Object.freeze(obj)`            | Fully immutable (no adds, deletes, or writes)         |

Example:

```js
const config = Object.freeze({ version: 1.0 });
config.version = 2; // ignored
delete config.version; // ignored
console.log(config.version); // 1
```

## 10. 📚 Terminology Glossary

| Term                  | Meaning                                                           |
| --------------------- | ----------------------------------------------------------------- |
| **Object**            | A collection of key-value pairs (properties) with hidden metadata |
| **Descriptor**        | Internal record defining property behavior                        |
| **Writable**          | If `false`, cannot change value                                   |
| **Enumerable**        | If `false`, hidden from loops and `Object.keys()`                 |
| **Configurable**      | If `false`, cannot delete or redefine                             |
| **Data Property**     | Stores value directly                                             |
| **Accessor Property** | Defines `get`/`set` for computed access                           |
| **Hidden Class**      | V8’s internal optimization structure for objects                  |
| **Shallow Copy**      | Copies only top-level properties                                  |
| **Deep Copy**         | Copies entire nested structure                                    |
| **Heap**              | Memory area for objects and functions                             |
| **Stack**             | Memory area for primitive values and references                   |

## 11. ⚠️ Common Pitfalls & Best Practices

Pitfalls

1. Accidental shared references
   ```js
   const a = { x: 1 };
   const b = a;
   b.x = 99; // a.x also 99
   ```
2. Modifying frozen or sealed objects silently
3. Confusing shallow and deep clones
4. Breaking hidden class optimization by adding props dynamically

Best Practices

- Use const for object bindings (prevents reassigning the reference)
- Use Object.freeze() for config objects
- Use structuredClone() for deep copies
- Maintain consistent object shapes for performance
- Use Object.getOwnPropertyDescriptors() for detailed reflection/debugging

## 12. 🧩 Practice Tasks

### Task 1 — Property Descriptors

Create an object settings with a mode property that:

- Has value "dark"
- Is read-only
- Doesn’t appear in Object.keys(settings)

Verify with Object.getOwnPropertyDescriptor().

### Task 2 — Shallow vs Deep Copy

Given:

```js
const original = { info: { name: "Ada" } };
```

Predict the effect of:

```js
const copy = Object.assign({}, original);
copy.info.name = "Grace";
console.log(original.info.name);
```

Then fix it with a deep clone.

### Task 3 — Freeze Challenge

```js
const config = Object.freeze({ version: 1 });
config.version = 2;
console.log(config.version); // ?
```

Explain what happens internally.
