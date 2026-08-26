---
title: What is the difference between **double-equals** and **triple-equals** in JavaScript?
aliases:
  - ==_vs_===
  - loose equality vs strict equality
tags:
  - javascript
  - interview
  - equality
  - type-coercion
---

> [!info]
> 🟢 Difficulty: Medium
> 📂 Category: JavaScript Interview Question
> ⏱️ Review Time: ~5 minutes

---

## TL;DR

**double-equals** is the **loose/abstract equality operator**, while **triple-equals** is the **strict equality operator**.

- **double-equals** performs **type coercion** before comparing in many cases.
- **triple-equals** does **not** perform type coercion.
- **triple-equals** is generally preferred in application code because it avoids unexpected coercion.
- A useful intentional exception is `x double-equals null`, which checks for both `null` and `undefined`.

| Behavior                     | **double-equals**         | **triple-equals** |
| ---------------------------- | ------------------------- | ----------------- |
| Name                         | Loose / abstract equality | Strict equality   |
| Type coercion                | Yes                       | No                |
| Different types can be equal | Yes                       | No                |
| General recommendation       | Avoid unless intentional  | Prefer            |

---

## Interview Answer (30–60 sec)

> The main difference is type coercion. **double-equals** is the abstract equality operator, so when the operands have different types, JavaScript may convert one or both values before comparing them. For example, `1 double-equals '1'` is `true`. **triple-equals** is strict equality, so it compares both type and value without coercion, making `1 =double-equals '1'` false. I generally prefer **triple-equals** because it makes comparisons predictable and avoids bugs caused by implicit coercion. One intentional exception is `x double-equals null`, which is a concise way to check for either `null` or `undefined`.

---

## Key Takeaways

### **double-equals**

```js
1 == '1';       // true
true == 1;      // true
null == undefined; // true
```

JavaScript may perform type conversion before the comparison.

### **triple-equals**

```js
1 === '1';      // false
true === 1;     // false
null === undefined; // false
```

No type coercion occurs.

### Prefer **triple-equals**

```js
if (value === expected) {
  // predictable comparison
}
```

### Intentional `double-equals null`

```js
if (value == null) {
  // value is null OR undefined
}
```

This is one of the useful cases where **double-equals** is intentional.

---

## 🧠 Mental Model

```text
'=='
Are the types the same?
       │
   ┌───┴───┐
  YES      NO
   │        │
compare    may coerce
directly   types
              │
           compare
```

```text
  '==='
Compare type
     +
Compare value
     ↓
No coercion
```

Think:

```text
 '=='   → "Can these values become equal?"
 '==='  → "Are these the same type AND value?"
```

---

## Examples You Should Know

```js
1 == '1';       // true
1 === '1';      // false

0 == false;     // true
0 === false;    // false

'' == 0;        // true
'' === 0;       // false

null == undefined;  // true
null === undefined; // false
```

### Important edge cases

```js
NaN == NaN;     // false
NaN === NaN;    // false
Object.is(NaN, NaN); // true

+0 === -0;      // true
Object.is(+0, -0); // false
```

---

## Common Interview Traps

### "**double-equals** always converts both values to strings."

❌ Incorrect.

The Abstract Equality Comparison algorithm performs different conversions depending on the operand types.

---

### "**triple-equals** converts values to the same type before comparing."

❌ Incorrect.

**triple-equals** performs no type coercion.

---

### "`null double-equals false` is true because null is falsy."

❌ Incorrect.

```js
null == false; // false
```

`null` and `undefined` have a special loose-equality relationship.

---

### "`null double-equals 0` is true."

❌ Incorrect.

```js
null == 0; // false
```

---

### "`NaN double-equals NaN` is true."

❌ Incorrect.

```js
NaN == NaN;   // false
NaN === NaN;  // false
```

Use:

```js
Number.isNaN(value);
```

or:

```js
Object.is(value, NaN);
```

---

### "**double-equals** is transitive."

❌ Incorrect.

For example:

```js
0 == '';      // true
0 == '0';     // true
'' == '0';    // false
```

---

## Common Follow-ups

- What is type coercion in JavaScript?
- What is the Abstract Equality Comparison algorithm?
- Why does `1 double-equals '1'` return `true`?
- Why does `null double-equals undefined` return `true`?
- Why does `null double-equals 0` return `false`?
- What is the difference between **double-equals**, **triple-equals**, and `Object.is()`?
- What is special about `NaN`?
- Why is `Object.is(NaN, NaN)` true?
- Why is `Object.is(+0, -0)` false?
- Is **double-equals** ever useful?
- Why is **triple-equals** generally preferred?

---

## My Notes

- 

---

# Detailed Reference

## TL;DR

**double-equals** is the abstract equality operator while **triple-equals** is the strict equality operator. **double-equals** performs type coercion before comparing, following the Abstract Equality Comparison algorithm defined in the ECMAScript specification. **triple-equals** does not perform coercion and returns `false` whenever the operand types differ. **triple-equals** is generally preferred in application code because it eliminates a class of bugs caused by unexpected coercion.

The most common exception is `x double-equals null`, which checks for both `null` and `undefined` in a single comparison.

Operator  | **double-equals**  | **triple-equals**
--- | --- | ---
Name  | Loose (abstract) equality operator  | Strict equality operator
Type coercion  | Yes — per the Abstract Equality Comparison algorithm  | No
Comparison behavior  | Types may be coerced before the value comparison  | Types are compared first

> Don't confuse `=` with **double-equals** and **triple-equals**. `=` is the assignment operator — it sets a variable's value (`x = 5`) and does not compare anything.

* * *

## The Abstract Equality Comparison algorithm

The behavior of **double-equals** is defined by the `IsLooselyEqual` algorithm in ECMA-262 §7.2.15. Given operands `x` and `y`, the algorithm proceeds as follows:

1. If `Type(x)` is the same as `Type(y)`, return the result of `x =double-equals y` (strict equality, without coercion).
2. If `x` is `null` and `y` is `undefined`, return `true`.
3. If `x` is `undefined` and `y` is `null`, return `true`.
4. If `Type(x)` is Number and `Type(y)` is String, return `x double-equals ToNumber(y)`.
5. If `Type(x)` is String and `Type(y)` is Number, return `ToNumber(x) double-equals y`.
6. If `Type(x)` is BigInt and `Type(y)` is String, convert `y` with `StringToBigInt`. Return `false` if the conversion is undefined; otherwise compare the resulting BigInts.
7. If `Type(x)` is String and `Type(y)` is BigInt, swap operands and apply step 6.
8. If `Type(x)` is Boolean, return `ToNumber(x) double-equals y`.
9. If `Type(y)` is Boolean, return `x double-equals ToNumber(y)`.
10. If `Type(x)` is String, Number, BigInt, or Symbol, and `Type(y)` is Object, return `x double-equals ToPrimitive(y)`.
11. If `Type(x)` is Object and `Type(y)` is String, Number, BigInt, or Symbol, return `ToPrimitive(x) double-equals y`.
12. If one operand is a BigInt and the other is a Number, return `true` if the mathematical values are equal; otherwise `false`.
13. Return `false`.

Four properties of the algorithm that are not apparent from a truth table alone:

- Boolean operands are always converted to Number first. This is why `true double-equals '1'` is `true`: `true` becomes `1`, then `'1'` converts to `1`.
- Object operands are reduced to primitives via `ToPrimitive`, which invokes `Symbol.toPrimitive`, then `valueOf`, then `toString`. For example, `[1] double-equals 1` is `true` because `[1].toString()` is `'1'`, which then coerces to `1`.
- `null` and `undefined` are only loosely equal to each other and to themselves. They are not coerced to `0` or `false` elsewhere in the algorithm, which is why `a double-equals null` is a valid idiom for testing "null or undefined".
- `NaN` is not equal to any value, including itself, under any equality operator. Use `Number.isNaN(x)` or `Object.is(x, NaN)` to test for it.

### The coercion helpers used by **double-equals**

**double-equals** dispatches to three type-conversion routines defined in ECMA-262 §7.1:

- `ToPrimitive(input, hint)` — returns the input unchanged if it is already a primitive. Otherwise invokes `input[Symbol.toPrimitive](hint)`, then falls back to `valueOf()` and `toString()`. If none returns a primitive, a `TypeError` is thrown.
- `ToNumber(argument)` — `undefined` becomes `NaN`; `null` becomes `+0`; `true` and `false` become `1` and `+0`; strings are parsed with whitespace trimming, with the empty string becoming `0`; Symbols and BigInts throw `TypeError`; objects are first reduced via `ToPrimitive(argument, "number")` and the result is recursed on.
- `ToString(argument)` — `undefined` becomes `"undefined"`; `null` becomes `"null"`; Booleans become `"true"` and `"false"`; Numbers use `Number::toString`; Symbols throw `TypeError`; objects are first reduced via `ToPrimitive(argument, "string")`.

## Examples

### Array compared with boolean

```js
console.log([] == false); // true
console.log([0] == false); // true
console.log([1] == true); // true
console.log([1, 2] == '1,2'); // true
```

Walking through `[] double-equals false`:

1. `false` is coerced to `0`, producing `[] double-equals 0`.
2. The array is coerced via `ToPrimitive`. `Array.prototype.toString` joins elements with commas, so `[].toString()` is `''`, producing `'' double-equals 0`.
3. `'' double-equals 0` coerces the string to `0`, producing `0 double-equals 0`.
4. Same types, so strict equality returns `true`.

### `[] double-equals ![]`

```js
console.log([] == ![]); // true
```

Evaluation order:

1. `![]` is evaluated first. Applying `ToBoolean` to any object yields `true`, so `![]` is `false`.
2. The expression is now `[] double-equals false`, which evaluates to `true`.

### Object compared with boolean

```js
console.log({} == false); // false
```

Walking through:

1. `false` becomes `0`, producing `{} double-equals 0`.
2. The plain object is coerced via `ToPrimitive`. `Object.prototype.toString` returns `'[object Object]'`.
3. `'[object Object]' double-equals 0` calls `ToNumber('[object Object]')`, which produces `NaN`.
4. `NaN double-equals 0` returns `false`.

### `null` and `undefined`

```js
console.log(null == undefined); // true
console.log(null == 0); // false
console.log(null == false); // false
console.log(null >= 0); // true
```

- `null double-equals undefined` is `true` by the special case in the loose equality algorithm.
- `null double-equals 0` is `false` because **double-equals** does not convert `null` to `0`.
- `null double-equals false` is `false` for the same reason.
- `null >= 0` is `true` because relational operators do not use the Abstract Equality algorithm. They apply `ToNumber` directly, converting `null` to `0`.

### Same-type string comparison does not coerce

```js
console.log(0 == ''); // true
console.log(0 == '0'); // true
console.log('' == '0'); // false
```

`0 double-equals ''` and `0 double-equals '0'` both convert the string to a number. `'' double-equals '0'` compares two strings, so their contents are compared directly.

A consequence is that **double-equals** is not transitive.

### Symbol equality

```js
const s = Symbol('x');

console.log(s == s); // true
console.log(s == 'x'); // false
console.log(s === s); // true
```

Two Symbols are loosely equal only if they are the same value. A Symbol compared with a string returns `false` without attempting a coercion that would throw.

## Common misconceptions

1. `{} double-equals false` is `true`: It is `false`. `{}` coerces to `'[object Object]'`, which coerces to `NaN`.
2. `[] double-equals ![]` is `false`: It is `true`. `![]` is `false`, and `[] double-equals false` evaluates to `true`.
3. `null double-equals false` is `true` because `null` is falsy: It is `false`. The **double-equals** algorithm does not coerce `null` to a boolean or number.
4. **double-equals** is transitive: It is not. `0 double-equals ''` and `0 double-equals '0'` are both `true`, but `'' double-equals '0'` is `false`.
5. `NaN double-equals NaN` is `true`: `NaN` is not equal to any value under **double-equals** or **triple-equals**.

## `Object.is()`

`Object.is(x, y)` returns the same result as **triple-equals** with two exceptions:

- `Object.is(NaN, NaN)` is `true`, whereas `NaN =double-equals NaN` is `false`.
- `Object.is(+0, -0)` is `false`, whereas `+0 =double-equals -0` is `true`.

Use it when those two cases need to be distinguished from the behavior of **triple-equals**.

## Conclusion

- Using **triple-equals** (strict equality) is generally recommended to avoid the pitfalls of type coercion, which can lead to unexpected behavior and bugs in your code. It makes the intent of your comparisons clearer and ensures that you are comparing both the value and the type.
- Use `x double-equals null` when a single check for `null` or `undefined` is required.
- Use `Object.is` when `NaN` equality or distinguishing `+0` from `-0` is required.
- When questioned about an unexpected **double-equals** result, work through the algorithm steps rather than relying on memorized truth tables. The algorithm fully specifies the behavior.

## Further reading

- ECMA-262 §7.2.15 `IsLooselyEqual`
- ECMA-262 §7.1 Type conversion
- Equality (**double-equals**) | MDN
- Strict equality (**triple-equals**) | MDN
- `Object.is()` | MDN
- ESLint `eqeqeq` rule
