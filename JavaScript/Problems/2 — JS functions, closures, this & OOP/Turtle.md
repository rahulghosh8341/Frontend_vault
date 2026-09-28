---
title: Turtle
aliases:
  - Turtle
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Method Chaining]]"
concepts:
  - "[[Method Chaining]]"
  - "[[Object-Oriented Programming]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-17
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Implement a `Turtle` class simulating 2D grid movement, rotation with circular indexing, and method chaining.

## Problem

Implement a `Turtle` class simulating turtle graphics movement on an x-y plane starting at `(0, 0)` facing North:
- `forward(distance)`: Move forward along current facing direction.
- `backward(distance)`: Move backward along current facing direction.
- `left()`: Rotate in-place 90 degrees left.
- `right()`: Rotate in-place 90 degrees right.
- `position()`: Return coordinates as `[x, y]`.
- All movement/rotation methods must support chaining: `turtle.right().right().forward(5)`.

```js
const turtle = new Turtle();
turtle.position(); // [0, 0]
turtle.forward(1); // [0, 1]
turtle.backward(1); // [0, 0]
turtle.right().right().forward(5); // [-5, 0]
```

## Pattern

- [[Method Chaining]]

## 🤔 Thought Process

The key decision was representing directions as indexes:

```js
const directions = ['N', 'E', 'S', 'W'];
```

with:

```text
0 → N
1 → E
2 → S
3 → W
```

Then rotation becomes simple:

```js
right → +1
left  → -1
```

You correctly handle the circular wraparound with `% 4`.

For movement, you translate the current direction into an `x/y` change.

## 💻 Final Solution

```js
const directions = ['N', 'E', 'S', 'W'];
export default class Turtle {
  constructor() {
    this.x = 0;
    this.y = 0;
    this.direction = 0;
  }

  /**
   * @param {number} distance Distance to move forward while facing the current direction.
   * @returns {Turtle}
   */
  forward(distance) {
    switch (directions[this.direction]) {
      case 'N':
        this.y += distance;
        break;

      case 'E':
        this.x += distance;
        break;

      case 'S':
        this.y -= distance;
        break;

      case 'W':
        this.x -= distance;
        break;
    }
    return this;
  }

  /**
   * @param {number} distance Distance to move backward while facing the current direction.
   * @returns {Turtle}
   */
  backward(distance) {
    switch (directions[this.direction]) {
      case 'N':
        this.y -= distance;
        break;

      case 'E':
        this.x -= distance;
        break;

      case 'S':
        this.y += distance;
        break;
  
      case 'W':
        this.x -= distance;
        break;
    }
    return this;
  }

  /**
   * Turns the turtle left.
   * @returns {Turtle}
   */
  left() {
    this.direction = (this.direction + 3) % 4; // Move one direction counter-clockwise with wraparound
    return this;
  }

  /**
   * Turns the turtle right.
   * @returns {Turtle}
   */
  right() {
    this.direction = (this.direction + 1) % 4; // loop through the direction array
    return this;
  }

  /**
   * @returns {[number, number]} Coordinates [x, y]
   */
  position() {
    return [this.x, this.y];
  }
}
```

## 🤔 Why This Works

### Direction rotation

Your right:

```js
this.direction = (this.direction + 1) % 4;
```

gives:

```text
N → E → S → W → N
```

Your left:

```js
this.direction = (this.direction + 3) % 4;
```

is equivalent to subtracting `1` while avoiding a negative index:

```text
N → W → S → E → N
```

That's a very good implementation.

### Movement

Your mapping is exactly right:

```text
N → y + distance
E → x + distance
S → y - distance
W → x - distance
```

And backward correctly does the opposite.

### Method chaining

You correctly return:

```js
return this;
```

from:

```text
forward()
backward()
left()
right()
```

so this works:

```js
turtle.right().right().forward(5);
```

## 🐞 Bugs I Made

Your implementation doesn't have a functional bug.

One small thing I would change is the comment:

```js
// loop through the direction array
```

For:

```js
(this.direction + 3) % 4
```

I'd make it more precise:

```js
// Move one direction counter-clockwise with wraparound
```

Because `% 4` isn't literally looping through the array; it's **wrapping the direction index**.

## Production Considerations

- Direction offsets can also be stored as coordinate delta tuples `[[0, 1], [1, 0], [0, -1], [-1, 0]]` to eliminate switch statements entirely: `this.x += dx * distance; this.y += dy * distance;`.
- JavaScript `%` operator can return negative values for negative dividends (e.g. `-1 % 4 === -1`). Adding `+ 3` (or `(dir - 1 + 4) % 4`) guarantees non-negative modulo results.

## ⭐ Revision Notes

### Key Facts

* `direction` is an **index**, not the direction string itself.
* `% 4` provides circular wraparound.
* `+1` → turn right.
* `+3` → effectively turn left by one position.
* `forward()` changes position based on direction.
* `backward()` performs the opposite movement.
* `left()` and `right()` don't modify `x` or `y`.
* Returning `this` enables method chaining.
* `position()` returns `[x, y]`.

### Complexity

```text
forward()   O(1)
backward()  O(1)
left()      O(1)
right()     O(1)
position()  O(1)
```

Space:

```text
O(1)
```

The `directions` array is constant-sized.

### 🧠 Mental Model

The whole problem can be reduced to:

```text
          N (0)
           ↑
           |
W (3) ←────┼────→ E (1)
           |
           ↓
          S (2)
```

Rotation:

```text
right = +1 mod 4
left  = -1 mod 4
```

Movement:

```text
N → y+
E → x+
S → y-
W → x-
```

That's essentially the entire problem.

### Common Interview Questions

- How does `(dir + 3) % 4` simulate turning left? Subtracting 1 in modulo 4 arithmetic is congruent to adding 3: `(-1 + 4) % 4 === 3`. Avoids negative modulo pitfall in JS.
- Why return `this`? Enables fluent interface / method chaining where each call returns the calling instance reference.
- How to eliminate switch statements? Use direction vectors `const DELTAS = [[0, 1], [1, 0], [0, -1], [-1, 0]];` indexed directly by `this.direction`.

### ⭐ Interview Takeaway

Your approach is **interview-clean**.

The strongest part is using a numeric direction index instead of writing separate direction state such as:

```js
this.direction = 'N';
```

and manually changing strings.

If asked to explain your solution in an interview:

> “I represent the four directions as a circular array and store the current direction as an index. Right rotation increments the index modulo four, while left rotation decrements it using an equivalent positive offset. Movement then updates either x or y based on the current direction. All mutating methods return `this` to support chaining.”

**No alternative is necessary here unless the interviewer specifically asks for one.**

### Related

- [[Method Chaining]]
- [[Function Chaining]]
