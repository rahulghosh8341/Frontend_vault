---
title: Singleton
aliases:
  - Singleton
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Singleton]]"
  - "[[Closure]]"
concepts:
  - "[[Closure]]"
  - "[[ES Modules]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-14
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Implement the `GlobalMap` module using the Singleton pattern to return a single shared `Map` instance.

## Problem

The Singleton pattern ensures that a class has only one instance and provides a global point of access to that instance.

Implement the `GlobalMap` module in JavaScript using the Singleton pattern. The `GlobalMap` module exports an object that has a single `getInstance()` method that returns a `Map` object that can be used as a key/value store for global caching/memoization.

```js
// fileA.js
import GlobalMap from './GlobalMap';

const gbMap = GlobalMap.getInstance();
gbMap.set('count', 42);

// fileB.js
import GlobalMap from './GlobalMap';

const gbMap = GlobalMap.getInstance();
console.log(gbMap.get('count')); // 42
```

## Pattern

- [[Singleton]]
- [[Closure]]

## 🤔 Thought Process

The key is that **the `Map` itself must be the Singleton**.

We want:

```js
GlobalMap.getInstance()
```

to always return the **same `Map` object**.

```text
First call
GlobalMap.getInstance()
        ↓
     new Map()
        ↓
    save Map

Second call
GlobalMap.getInstance()
        ↓
    return same Map
```

## 💻 Final Solution

```js
let instance;

const GlobalMap = {
  getInstance() {
    if (!instance) {
      instance = new Map();
    }

    return instance;
  },
};

export default GlobalMap;
```

## 🤔 Why This Works

Keep the singleton instance outside the exported object:

```js
let instance;

const GlobalMap = {
  getInstance() {
    if (!instance) {
      instance = new Map();
    }

    return instance;
  },
};

export default GlobalMap;
```

The module-level `instance` persists because ES modules are evaluated once and their bindings are retained.

So:

```js
const a = GlobalMap.getInstance();
const b = GlobalMap.getInstance();
```

gives:

```js
a === b // true
```

And because both are the same `Map`:

```js
a.set('count', 42);

b.get('count'); // 42
```

## 🐞 Bugs I Made

### Returned `GlobalMap` instead of `Map`

My code did:

```js
GlobalMap.instance = new GlobalMap();
```

Therefore:

```js
GlobalMap.getInstance()
```

returned a `GlobalMap`.

But the test requires:

```js
GlobalMap.getInstance() instanceof Map
```

So the returned object must be the actual `Map`.

### Added unnecessary wrapper methods

I created:

```js
set(key, value) {
  this.map.set(key, value);
}

get(key) {
  return this.map.get(key);
}
```

These aren't needed because the returned object is already a `Map`:

```js
const map = GlobalMap.getInstance();

map.set(...);
map.get(...);
map.has(...);
map.delete(...);
```

## Production Considerations

- Modern ES modules execute as singletons automatically. An eager singleton could simply export `export default new Map()`.
- Lazy singleton (`getInstance()`) delays allocation until first use, beneficial if initialisation is costly.
- In unit testing, singletons risk state leakage across tests. Provide a reset method (`_resetInstanceForTesting()`) when building production singletons.

## ⭐ Revision Notes

### Key Facts

* Singleton = **only one instance exists**.
* Here, the singleton instance is the **`Map`**.
* `getInstance()` is the access point.
* Every call to `getInstance()` returns the same Map.
* `Map` methods should work directly.
* `instance` should be created only once.
* The module itself provides the private scope for `instance`.

### Common Interview Questions

- What is the difference between eager and lazy singletons? Eager creates instance on module load; lazy defers creation until `getInstance()` is called.
- Why are ES modules singletons by default? Engine evaluates module code once and caches module namespace object for subsequent imports.
- How do you handle concurrency in JS singletons? JS runs on a single thread event loop, eliminating multithreaded race conditions found in languages like Java/C++.

### 🧠 Mental Model

```text
             GlobalMap
                 │
           getInstance()
                 │
          ┌──────┴──────┐
          │             │
     instance exists?   │
       /       \        │
     YES        NO      │
      ↓          ↓      │
   return      new Map  │
   instance       ↓     │
              save it   │
                  ↓     │
              return it
```

The most important distinction:

```text
❌ GlobalMap → GlobalMap instance → Map

✅ GlobalMap → Map instance
```

### Interview Takeaways

For this specific problem:

> **`GlobalMap` is the access API, while the `Map` is the Singleton.**

The simplest implementation is:

```js
let instance;

const GlobalMap = {
  getInstance() {
    if (!instance) {
      instance = new Map();
    }

    return instance;
  },
};

export default GlobalMap;
```

The test:

```js
expect(GlobalMap.getInstance()).toBeInstanceOf(Map);
```

is the clue that **you shouldn't create a `GlobalMap` instance at all**.

### Related

- [[Singleton]]
- [[Closure]]
- [[Memoize]]
