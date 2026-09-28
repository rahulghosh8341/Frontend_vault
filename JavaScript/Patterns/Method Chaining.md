---
aliases:
  - Method Chaining
  - Fluent Interface
---

## Core Idea

Method Chaining (also known as Fluent Interface) is an object-oriented pattern where methods mutate or query instance state and return the instance reference (`return this`) so that subsequent method calls can be chained sequentially in a single expression.

## Recognition

- API design requiring consecutive actions on a single entity (e.g., `turtle.right().forward(5).left()`, jQuery-style `$('div').hide().addClass('active')`, or query builders like Knex/Prisma).
- Class or prototype methods performing state transitions where chaining improves readability.

## Template

```js
class Builder {
  constructor() {
    this.state = 0;
  }

  stepA(val) {
    this.state += val;
    return this; // Enables chaining
  }

  stepB() {
    this.state *= 2;
    return this; // Enables chaining
  }

  build() {
    return this.state; // Terminal method returning final value
  }
}
```

## Variations

1. **Mutating Fluent Interface**: Methods mutate internal state in-place and return `this` (like `Turtle`).
2. **Immutable Fluent Interface**: Methods return a *new* instance of the class with updated state (like `BigDecimal` or functional query builders).
3. **Promise / Asynchronous Chaining**: Each step returns a Promise resolving to the next chainable context.

## Complexity

- **Time Complexity**: $O(1)$ per chained method invocation.
- **Space Complexity**: $O(1)$ for mutable chaining; $O(k)$ for immutable chaining creating new instances.

## Common Mistakes

- Forgetting to `return this` in one method, breaking the chain with `TypeError: Cannot read properties of undefined`.
- Using arrow functions for prototype methods, causing `this` to point to outer lexical scope rather than the instance.
- Mixing terminal methods (methods that return computed data like `position()`) with intermediate chainable methods.

## Interview Tips

- Explicitly call out separating chainable methods (`return this`) from terminal/accessor methods (e.g. `position()`, `build()`, `exec()`).
- Discuss mutable vs immutable chaining tradeoffs (e.g., memory overhead vs branch safety).

## Problems Using This Pattern

- [[Turtle]]

## Related Patterns

- [[Function Chaining]]

## Related Concepts

- `this` binding
- OOP Classes & Prototypes
- Fluent API Design
