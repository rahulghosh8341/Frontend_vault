---
title: Backbone Model
aliases:
  - Backbone Model
difficulty: Hard
time: 30 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Observer]]"
concepts:
  - "[[Observer]]"
  - "[[Object-Oriented Programming]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-14
type: coding
---

> [!info]
> **Difficulty:** 🔴 Hard | **Time:** 30 min
> Implement a subset of Backbone.Model supporting isolated attribute state, change/unset lifecycle events, and scoped listeners.

## Problem

Before modern UI libraries like React, Angular, and Vue, Backbone.js provided the MVC structure for web applications.

Implement the `BackboneModel` class with:
- `new BackboneModel([initialValues])`: isolated instance attributes and event callbacks.
- `get(attribute)`: retrieve attribute value.
- `set(attribute, value)`: set attribute value, fire `change` event with `(attribute, newValue, oldValue)`.
- `has(attribute)`: boolean check for attribute existence.
- `unset(attribute)`: remove attribute and its listeners, fire `unset` event with `(attribute)`.
- `on(event, attribute, callback, context)`: register listener for changes to a specific attribute. No-op if attribute does not exist.
- `off(eventName, attribute, callback)`: unregister matching callback.

## Pattern

- [[Observer]]

## 🤔 Thought Process

The biggest thing to understand is where the event listeners belong.

Instead of:

```text
events
 └── change
      └── name
           └── callbacks
```

this challenge uses:

```text
attribute
 ├── value
 └── events
      ├── change
      └── unset
```

So:

```js
this._attributes.get('name')
```

gives you both the value and its listeners.

## 💻 Final Solution

```js
export default class BackboneModel {
  constructor(initialValues = {}) {
    this.attributes = { ...initialValues };
    this.events = new Map();
  }

  get(attribute) {
    return this.attributes[attribute];
  }

  set(attribute, value) {
    const oldValue = this.attributes[attribute];

    // Don't fire change if the value is the same
    if (oldValue === value) {
      return;
    }

    this.attributes[attribute] = value;

    this.trigger('change', attribute, value, oldValue);
  }

  has(attribute) {
    return Object.prototype.hasOwnProperty.call(
      this.attributes,
      attribute,
    );
  }

  unset(attribute) {
    delete this.attributes[attribute];

    // Fire unset event
    this.trigger('unset', attribute);

    // Remove all listeners associated with this attribute
    for (const attributeEvents of this.events.values()) {
      attributeEvents.delete(attribute);
    }
  }

  on(eventName, attribute, callback, context) {
    const attributeData = this._attributes.get(attribute);
    // No-op for non-existent attributes.
    if (attributeData == null) {
      return;
    }

    // Add to the list of callbacks.
    attributeData.events[eventName].push({
      fn: callback,
      context,
    });
  }

  off(eventName, attribute, callback) {
    const attributeEvents = this.events.get(eventName);

    if (!attributeEvents) {
      return;
    }

    const callbacks = attributeEvents.get(attribute);

    if (!callbacks) {
      return;
    }

    attributeEvents.set(
      attribute,
      callbacks.filter((event) => event.callback !== callback),
    );
  }

  trigger(eventName, attribute, ...args) {
    const attributeEvents = this.events.get(eventName);

    if (!attributeEvents) {
      return;
    }

    const callbacks = attributeEvents.get(attribute);

    if (!callbacks) {
      return;
    }

    for (const { callback, context } of callbacks) {
      callback.apply(context ?? this, [attribute, ...args]);
    }
  }
}
```

## 🤔 Why This Works

1. **Initial attributes get records**
```js
new BackboneModel({
  name: 'John',
});
```
creates:
```text
name
├── value: "John"
└── events
    ├── change: []
    └── unset: []
```
Therefore:
```js
person.on('change', 'name', callback);
```
works because `name` exists.

2. **`on()` on a nonexistent attribute does nothing**
```js
const person = new BackboneModel();

person.on('change', 'name', callback);
```
`name` doesn't exist:
`this._attributes.get('name')` returns `undefined`.
Therefore: `return;`.
Then:
```js
person.set('name', 'John');
```
creates `name` with no callbacks.
Hence: `times === 0`.
This is exactly the failing test you were getting.

3. **`set()` doesn't fire when there is no change**
```js
person.set('name', 'John');
person.set('name', 'John');
```
The second call has `attributeData.value === value`, so `if (attributeData.value !== value)` is false. No callback.

4. **`set()` fires before overwriting**
Suppose `name = "John"`, then `set('name', 'Johnny')`.
The callback receives:
- attribute → `"name"`
- new value → `"Johnny"`
- old value → `"John"`
That's why the callback is executed before `attributeData.value = value;`.

5. **`unset()` destroys the entire attribute record**
`this._attributes.delete(attribute);`
So both `value` and `listeners` are removed together.

## 🐞 Bugs I Made

### ❌ Don't use a separate global event map
My earlier approach:
```js
this.events = new Map();
```
was not the right model for this GFE question.

### ❌ Don't make `on()` create a new attribute
This would be wrong:
```js
if (!attributeData) {
  // create attribute
}
```
The tests explicitly expect `on(nonExistingAttribute)` to do nothing.

### ✅ Listeners belong to existing attributes
```js
attributeData.events[eventName].push(...)
```

## Production Considerations

- In real Backbone.js, event emission also supports model-wide events (`change` without attribute specifier) and silent updates (`{ silent: true }`).
- Modern JavaScript implements reactivity via ES6 `Proxy` (as in Vue 3 / MobX) or explicit Signals, avoiding manual `.on()` subscriptions per field.
- Unsubscribing should guard against callback iteration mutation during `trigger`.

## ⭐ Revision Notes

### Key Facts

- `_attributes` is a Map.
- Each attribute stores:
  - value
  - change callbacks
  - unset callbacks
- `on()` only works for an already-existing attribute.
- `set()` creates a missing attribute.
- Creating a new attribute does not fire change.
- Setting the same value does not fire change.
- `change` callback arguments: `attribute, newValue, oldValue`.
- `unset` callback argument: `attribute`.
- `unset()` removes the whole attribute record, including listeners.
- `off()` removes callbacks matching the supplied function.
- Each BackboneModel instance has its own `_attributes` Map.

### 🧠 Mental Model

```text
                _attributes
                    │
             ┌──────┴──────┐
             ↓             ↓
           name           age
             │             │
       ┌─────┴─────┐ ┌─────┴─────┐
       ↓           ↓ ↓           ↓
     value       events       value/events
                   │
             ┌─────┴─────┐
             ↓           ↓
          change        unset
             │
          callbacks
```

And the lifecycle is:

```text
Attribute doesn't exist
        │
        ├── on() → ignored
        │
        └── set() → creates attribute
                         │
                         ↓
                   on() can work
                         │
                         ↓
                   set(new value)
                         │
                         ↓
                    change fires
                         │
                         ↓
                       unset()
                         │
                         ↓
                  entire record gone
```

### Common Interview Questions

- Why store listeners per-attribute rather than per-event? Simplifies lifecycle cleanup during `unset()`, ensuring listeners die with the attribute record.
- How does `context` work in event callbacks? `callback.apply(context ?? this, args)` allows callers to retain their component/service scope.
- Why compare `oldValue === value` in `set`? Prevents unnecessary rendering and infinite loops in two-way data bindings.

### ⭐ Interview Takeaway

The key design decision is:

> Store each attribute's value and event listeners together. `on()` only subscribes to an existing attribute, while `set()` creates attributes and only emits `change` when the value actually changes.

The important thing was understanding the unusual `on()` rule from the tests.

### Related

- [[Observer]]
- [[Event Emitter]]
- [[Event Emitter II]]
- [[Closure]]
