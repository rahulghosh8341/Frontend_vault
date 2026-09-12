---
title: What are the various ways to create objects in JavaScript?
aliases: ways to create objects in JavaScript
tags:
  - objects
  - classes
  - prototypes
  - constructors
section: "1 — JavaScript core"
solved: true
solvedDate: 2026-09-07
type: quiz
---

> [!info] 🟢 Difficulty: Easy 📂 Category: JavaScript Interview Question ⏱️ Review Time: ~5 minutes

## TL;DR

There are several ways to create objects in JavaScript:

1. **Object literal** — simplest and most common.
2. **`Object()` constructor** — create an object using the built-in constructor.
3. **`Object.create()`** — create an object with a specific prototype.
4. **Constructor functions** — create reusable object instances with `new`.
5. **ES2015 classes** — structured syntax for creating objects and supporting inheritance.

For most everyday objects, object literals are the simplest choice. Constructor functions and classes are useful when you need reusable blueprints for multiple objects.

## Interview Answer (30–60 sec)

> There are several ways to create objects in JavaScript. The most common is an object literal using curly braces, which is ideal for a single object with fixed properties. We can also use the built-in `Object` constructor, although object literals are usually simpler. `Object.create()` lets us create an object with a specific prototype. Constructor functions can be used with `new` to create multiple instances that share a common structure, and ES2015 classes provide a more structured syntax for constructor-and-prototype patterns and inheritance. In modern code, I would generally use object literals for simple objects and classes or other reusable patterns when I need multiple related instances.

## Key Takeaways

- `{}` → simplest and most common.
- `new Object()` → built-in `Object` constructor.
- `Object.create(proto)` → explicitly controls the new object's prototype.
- `Object.create(null)` → creates an object with **no prototype**.
- Constructor function + `new` → reusable object blueprint.
- `class` + `new` → modern structured syntax for reusable instances.
- `class` syntax is built on JavaScript's prototype-based object model.

## 🧠 Mental Model

Think about the **kind of object you need**:

```text
Need one simple object?
        ↓
      {}

Need a specific prototype?
        ↓
 Object.create(proto)

Need many similar objects?
        ↓
Constructor function / class
        ↓
      new Person(...)
```

The important distinction is:

```text
{}                  → create an object directly
Object.create(proto) → create object with chosen prototype
new Person(...)     → create an instance from a constructor/class
```

## Common Interview Traps

### "There is only one way to create objects in JavaScript."

❌ Incorrect.

JavaScript provides several approaches, including object literals, `Object.create()`, constructor functions, and classes.

### "`Object.create()` copies all properties from the prototype."

❌ Not exactly.

The new object gets the supplied object as its **prototype**. Properties are inherited through the prototype chain rather than copied onto the new object.

```js
const proto = {
  greet() {
    console.log('Hello');
  },
};

const person = Object.create(proto);

person.greet();
```

### "`Object.create(null)` creates a normal empty object."

❌ Not exactly.

It creates an object with **no prototype**:

```js
const obj = Object.create(null);

Object.getPrototypeOf(obj); // null
```

### "Classes are completely separate from prototypes."

❌ Incorrect.

JavaScript classes provide a more structured syntax over the language's existing prototype-based object model.

### "`new Object()` is preferred over `{}`."

❌ Generally no.

For a normal object literal, `{}` is simpler and more idiomatic.

## Common Follow-ups

- What is the difference between `Object.create()` and `new`?
- What is a prototype in JavaScript?
- What is the prototype chain?
- What happens when you use `new` with a constructor function?
- How are JavaScript classes related to prototypes?
- What is the difference between an object literal and a class instance?
- What does `Object.create(null)` do?
- Why would you use a constructor function instead of an object literal?

## My Notes


# Detailed Reference

## TL;DR

Creating objects in JavaScript offers several methods:

- **Object literals (`{}`):** Simplest and most popular approach. Define key-value pairs within curly braces.
- **`Object()` constructor:** Use `new Object()` with dot notation to add properties.
- **`Object.create()`:** Create new objects using existing objects as prototypes, inheriting properties and methods.
- **Constructor functions:** Define blueprints for objects using functions, creating instances with `new`.
- **ES2015 classes:** Structured syntax similar to other languages, using `class` and `constructor` keywords.

---

## Objects in JavaScript

There are several methods for creating objects in JavaScript. Here are the various ways to do so.

## Object literals (`{}`)

This is the simplest and most popular way to create objects in JavaScript. It involves defining a collection of key-value pairs within curly braces (`{}`). It can be used when you need to create a single object with a fixed set of properties.

```js
const person = {
  firstName: 'John',
  lastName: 'Doe',
  age: 50,
  eyeColor: 'blue',
};

console.log(person);
// { firstName: "John", lastName: "Doe", age: 50, eyeColor: "blue" }
```

## `Object()` constructor

This method involves using the `new` keyword with the built-in `Object` constructor to create an object. You can then add properties to the object using dot notation. It can be used when you need to create an object from a primitive value or to create an empty object.

```js
const person = new Object();
person.firstName = 'John';
person.lastName = 'Doe';

console.log(person);
// { firstName: "John", lastName: "Doe" }
```

## `Object.create()` method

This method allows you to create a new object using an existing object as a prototype. The new object inherits properties and methods from the prototype object. It can be used when you need to create a new object with a specific prototype.

```js
const personPrototype = {
  greet() {
    console.log(
      `Hello, my name is ${this.name} and I'm ${this.age} years old.`,
    );
  },
};

const person = Object.create(personPrototype);
person.name = 'John';
person.age = 30;

person.greet();
// Output: Hello, my name is John and I'm 30 years old.
```

An object without a prototype can be created by doing:

```js
const dictionary = Object.create(null);
```

## ES2015 classes

Classes provide a more structured and familiar syntax for creating objects. They define a blueprint and use methods to interact with the object's properties. It can be used when you need to create complex objects with inheritance and encapsulation.

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet = function () {
    console.log(
      `Hello, my name is ${this.name} and I'm ${this.age} years old.`,
    );
  };
}

const person1 = new Person('John', 30);
const person2 = new Person('Alice', 25);

person1.greet();
person2.greet();
```

## Constructor functions

Constructor functions are used to create reusable blueprints for objects. They define the properties and behaviors shared by all objects of that type. You use the `new` keyword to create instances of the object. It can be used when you need to create multiple objects with similar properties and methods.

ES2015 class syntax is often clearer for constructor-and-prototype patterns, but constructor functions remain part of the language and are still encountered in older code and function-oriented APIs.

```js
function Person(name, age) {
  this.name = name;
  this.age = age;

  this.greet = function () {
    console.log(
      `Hello, my name is ${this.name} and I'm ${this.age} years old.`,
    );
  };
}

const person1 = new Person('John', 30);
const person2 = new Person('Alice', 25);

person1.greet();
person2.greet();
```

## Quick Comparison

| Method | Best understood as | Typical use |
|---|---|---|
| `{}` | Direct object creation | Simple one-off objects |
| `new Object()` | Built-in constructor | Explicit `Object` construction |
| `Object.create(proto)` | Prototype-based creation | Specific prototype/inheritance behavior |
| Constructor function | Reusable blueprint | Multiple related instances, especially older code |
| `class` | Structured constructor syntax | Modern reusable object instances and inheritance |
