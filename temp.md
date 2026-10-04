## Function.prototype.bind (Official solution)

Premium

[Zhenghao He](https://www.zhenghao.io/)Engineering Manager, Robinhood

[](https://www.zhenghao.io/)

Languages

Recommended duration to spend during interviews

15 mins

This question asks for an implementation of the `Function.prototype.bind` method, which should behave exactly like the native one. It is common to implement newer JavaScript APIs using older, more mature language features so these APIs still work on older browsers. This practice is called [polyfilling](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill).

However, writing polyfills that are faithful to the specification and fully compatible with legacy browsers is not easy. It is unrealistic to expect candidates to write one in interview settings. For this question, interviewers are typically more interested in familiarity with the native `bind` method and the `this` keyword.

For this exercise, keep a binding record: the original function, the fixed `thisArg`, and any pre-applied arguments live in a closure. The returned function later combines those saved arguments with call-time arguments and invokes the original function with the fixed receiver.

|Saved when `myBind` is called|Used when bound function is called|
|---|---|
|original function|call it with `apply` or `Reflect.apply`|
|`thisArg`|pass as the receiver|
|bound arguments|prepend before new call-time arguments|

## Clarification questions

- Can I use other related native methods such as `Function.prototype.apply` and `Function.prototype.call`?
    - Yes.
- Can I use other modern JavaScript features?
    - Yes, as long as `Function.prototype.bind` is not used.

## Solution

### Refresher on `Function.prototype.bind` and `this`

The native `bind` is a method on `Function.prototype`, so every declared function automatically inherits that method from the prototype chain.

One common use case for `bind` is to preserve a method's binding when it is called as a function. A method call invokes a function as a property of an object. For example:
