---
aliases:
  - Observer
  - Observer Pattern
---

## Core Idea

The Observer pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents (observers/subscribers) are notified and updated automatically.

## Recognition

- Component or model state changes need to trigger side effects in views, logs, or other components without tight coupling.
- Requirement for subscription APIs: `on(event, callback)`, `off(event, callback)`, `trigger(event, ...args)`.
- Event-driven architecture where producers don't know consumer details.

## Template

```js
class Observable {
  constructor() {
    this.listeners = new Map(); // eventName -> Set of callbacks
  }

  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event).add(callback);
    return () => this.off(event, callback);
  }

  off(event, callback) {
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      callbacks.delete(callback);
      if (callbacks.size === 0) {
        this.listeners.delete(event);
      }
    }
  }

  trigger(event, ...args) {
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      for (const cb of callbacks) {
        cb(...args);
      }
    }
  }
}
```

## Variations

1. **Global Event Bus / Mediator**: Event registry is independent of individual model instances.
2. **Model-Bound Observable (Backbone Style)**: Event listeners are scoped directly to individual attributes of a model instance.
3. **Reactive State (Signals / Proxies)**: Fine-grained subscription using automatic dependency tracking via getters and setters instead of explicit `.on()` strings.

## Complexity

- **Subscription (`on`)**: O(1)
- **Unsubscription (`off`)**: O(1) with `Set` / O(N) with `Array`
- **Emission (`trigger`)**: O(K) where K is the number of active listeners for that event
- **Space**: O(N) total stored listeners

## Common Mistakes

- **Memory Leaks**: Forgetting to unsubscribe (`off`) when a consumer component is destroyed.
- **Mutating Listener List During Trigger**: Iterating over an array while callbacks add/remove listeners can skip or duplicate calls.
- **Context Loss**: Invoking callbacks without preserving `this` (or designated context).
- **Scope Leak**: Sharing event maps across instances instead of scoping per model.

## Interview Tips

- Distinguish Observer Pattern (subject directly maintains list of observers) vs Pub-Sub (observers and publishers communicate through an intermediary broker/bus).
- Discuss `Set` vs `Array` for storing callbacks (Set gives O(1) deletion and natural deduplication).
- Mention cleanup: returning an unsubscribe function from `on()` is standard modern practice.

## Problems Using This Pattern

- [[Backbone Model]]
- [[Event Emitter]]
- [[Event Emitter II]]

## Related Patterns

- [[Closure]]
- [[Singleton]]

## Related Concepts

- Event Loop & Callbacks
- Pub-Sub Architecture
- MVC in Frontend
