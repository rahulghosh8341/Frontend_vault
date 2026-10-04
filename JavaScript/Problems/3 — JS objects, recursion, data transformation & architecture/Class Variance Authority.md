---
title: "Class Variance Authority"
aliases:
  - "cva"
  - "Class Variance Authority"
difficulty: "Medium"
source: GreatFrontEnd
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Class Variance Authority

> [!info] Problem
> Implement a simplified version of cva for generating variant-based class names

## Problem

## Class Variance Authority

[`class-variance-authority`](https://github.com/joe-bell/cva) is a utility for generating class names from a base class list and a set of variants.

Implement a simplified `cva` function. This version only needs to support `base`, `variants`, and `defaultVariants`. You do not need to support `compoundVariants`, `class`, or `className`.

## Examples

```javascript
const button = cva('btn', {
  variants: {
    intent: {
      primary: 'btn-primary',
      secondary: 'btn-secondary',
    },
    size: {
      small: 'btn-small',
      medium: 'btn-medium',
    },
    disabled: {
      true: 'btn-disabled',
      false: null,
    },
  },
  defaultVariants: {
    intent: 'primary',
    size: 'medium',
    disabled: false,
  },
});

button(); // 'btn btn-primary btn-medium'
button({ size: 'small' }); // 'btn btn-primary btn-small'
button({ intent: 'secondary', disabled: true }); // 'btn btn-secondary btn-medium btn-disabled'
```

## Arguments

`cva(base, config)`

- `base` (`string | null | undefined`): Base classes that are always included.
- `config` (`object`, optional): Configuration for the generated function.
  - `variants`: An object whose keys are variant names and whose values map variant values to class names.
  - `defaultVariants`: Default variant values to use when the returned function is called without that variant.

## Returns

Returns a function that accepts an object of selected variant values and returns the final class string.

## Notes

- Ignore unknown variant names in the returned function's argument.
- Ignore resolved variant values that do not exist in the corresponding variant map.
- Join all truthy class names with a single space and no leading or trailing whitespace.

## Resources

- [`class-variance-authority` on GitHub](https://github.com/joe-bell/cva)

## Hints

### Hint 1 : Which value selects each variant class?

### Hint 2 : Which object should drive iteration?

## 🤔 Thought Process

- **Immediate Recognition:** Component styling utility mimicking the popular `cva` package used in modern Tailwind UI libraries (e.g. `shadcn/ui`).
- **Signature & Behavior:**
  - `cva(base, config)` returns a function `(props) => string`.
  - `base`: Can be string, array of strings, or nested falsy values.
  - `config`: Contains `variants` and optional `defaultVariants`.
- **Execution Flow of Returned Function:**
  1. Initialize class list with flattened, normalized `base` classes.
  2. For each variant defined in `config.variants`:
     - Determine the selected variant value: `props?.[variantKey] ?? config.defaultVariants?.[variantKey]`.
     - Look up corresponding classes: `config.variants[variantKey]?.[selectedVal]`.
     - Append to class list if non-empty.
  3. Filter out falsy/empty values, flatten, and join with a single space.

---

## 🧠 Mental Model

Think of a **CSS Class Resolver Factory**:
```
Base Classes ('btn font-bold')
       +
Variant Choices (intent: 'primary' -> 'bg-blue-500', size: 'lg' -> 'h-12 px-6')
       │
       ▼
Flattened, de-falsed CSS String: 'btn font-bold bg-blue-500 h-12 px-6'
```

---

## 🔑 Key Concepts

- Higher-Order Functions / Closure configuration
- Design System variant management
- Dynamic property resolution with fallback (`props?.[k] ?? defaultVariants?.[k]`)
- Array flattening and class string normalization (similar to `classnames`)

---

## ⚠️ Edge Cases / Traps

- **Boolean Variants:** Variants often have boolean keys like `disabled: { true: 'opacity-50', false: null }`. In props, `disabled` is passed as a boolean `true`, but object keys in JS are stringified to `'true'`. Ensure type coercion or string conversion: `String(val)`.
- **Undefined vs Null vs Empty String:** If a variant prop is `undefined`, it should fall back to `defaultVariants`. If it is `null`, it should NOT fall back if explicitly passed as null.
- **Base Classes Format:** `base` can be a single string or an array of strings. Handle both uniformly.

---

## ⭐ Interview Takeaway

- Return a closure that merges `props` over `defaultVariants`:
  `const val = props?.[key] !== undefined ? props[key] : defaultVariants?.[key];`
- Always convert variant values to strings (`String(val)`) when indexing into the `variants[key]` map to seamlessly handle boolean flags (`true` -> `'true'`).
- Filter and join cleanly: `classes.filter(Boolean).join(' ')`.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why is `cva` preferred over manual ternary strings in component design systems?" (Keeps variant mappings centralized, type-safe, and decoupled from JSX template rendering).
- "How do you handle boolean props in variant objects?" (Convert prop values to string keys).

### Follow-up Questions
- "How would you implement `compoundVariants`?" (Iterate through compound rules and append classes only when all specified variant conditions match).
- "How would you handle conflicting Tailwind classes?" (Integrate with `tailwind-merge`).

### Conceptual Questions
- "How does `cva` contrast with CSS-in-JS solutions like styled-components?" (`cva` generates static class names at zero runtime CSS-parsing cost).

---

## 🔄 Variations

- **Classnames / clsx:** General-purpose class composition utility without variant schemas.
- **Compound Variants CVA:** Extending `cva` with multi-variant condition matching.

---

## 📝 Revision Notes

- Clean implementation:
```javascript
export function cva(base, config = {}) {
  const { variants = {}, defaultVariants = {} } = config;

  return function (props = {}) {
    const classList = [];

    if (Array.isArray(base)) {
      classList.push(...base);
    } else if (base) {
      classList.push(base);
    }

    for (const [variantName, variantOptions] of Object.entries(variants)) {
      const propVal = props[variantName];
      const selectedVal = propVal !== undefined ? propVal : defaultVariants[variantName];

      if (selectedVal !== undefined && selectedVal !== null) {
        const matchedClass = variantOptions[String(selectedVal)] || variantOptions[selectedVal];
        if (matchedClass) {
          classList.push(matchedClass);
        }
      }
    }

    return classList.filter(Boolean).join(' ');
  };
}
```

---

## Official Solution

## Class Variance Authority ( Official solution )

Premium
Languages
After [Classnames](/questions/javascript/classnames), this question builds on the same idea with a fixed configuration object and a returned resolver function. Keep that configuration in a closure and resolve classes on demand instead of precomputing every possible variant combination.

## Solution

`cva()` is a small class-name resolver factory. The outer function captures the static configuration, and the returned function combines that configuration with one set of runtime props.

Split the work into two jobs:

1. `cva()` captures `base`, `variants`, and `defaultVariants`.
2. The returned resolver walks through the configured variants, decides the effective value for each one, and appends the matching class names.

The resolver's class-building order is:

1. Start with `base` when it is truthy.
2. Iterate only over the configured `variants`.
3. For each variant, read from `props` first and fall back to `defaultVariants` only when that prop is missing or nullish.
4. Skip nullish resolved values.
5. Convert the resolved value with `String()` before looking it up in the variant map.
6. Append the variant class only when the mapped class value is truthy.

Order comes from the configuration, not from the runtime props object. That keeps output deterministic and lets defaults and provided props share the same variant iteration path.

The boolean conversion is important because object keys are strings. A variant value of `false` should look up the configured `'false'` key, not be treated as "no value". The class value found at that key can still be falsy, and falsy class values should be skipped so they do not add extra spaces or words such as `"false"`.

Using nullish fallback is also deliberate. A prop of `false` should override a default of `true`, while a missing prop or `null`/`undefined` should fall back to the default variant for that variant name.

| Variant | Prop value | Default value | Effective key | Appended class |
| --- | --- | --- | --- | --- |
| `intent` | missing | `'primary'` | `'primary'` | `'btn-primary'` |
| `size` | `'small'` | `'medium'` | `'small'` | `'btn-small'` |
| `disabled` | `false` | `true` | `'false'` | skipped when mapped to `null` |
| `tone` | `'ghost'` | missing | `'ghost'` | skipped when key is unknown |

```jsx
/**
 * @typedef {string | false | null | undefined} ClassValue
 * @typedef {string | boolean | null | undefined} VariantValue
 * @typedef {Record<string, ClassValue>} VariantClasses
 * @typedef {Record<string, VariantClasses>} VariantSchema
 * @typedef {Record<string, VariantValue>} VariantSelection
 */

/**
 * @param {ClassValue} [base]
 * @param {{
 *   variants?: VariantSchema,
 *   defaultVariants?: VariantSelection,
 * }} [config]
 * @return {(props?: VariantSelection) => string}
 */
export default function cva(base, config = {}) {
  const { variants = {}, defaultVariants = {} } = config;

  return function getClassName(props = {}) {
    const classes = [];

    if (base) {
      classes.push(base);
    }

    for (const variantName in variants) {
      if (!Object.hasOwn(variants, variantName)) {
        continue;
      }

      const variantMap = variants[variantName];
      // Props win when present; otherwise fall back to the configured
      // default for that single variant.
      const variantValue = props[variantName] ?? defaultVariants[variantName];

      if (variantValue == null) {
        continue;
      }

      // Variant maps are object-keyed, so booleans like true/false must be
      // normalized into string keys before lookup.
      const variantClass = variantMap[String(variantValue)];

      if (variantClass) {
        classes.push(variantClass);
      }
    }

    return classes.join(' ');
  };
}
```

## Common pitfalls

- Do not iterate over `props`. Extra runtime keys are ignored unless they correspond to configured variants.
- Do not skip boolean `false` before lookup. `false` is a valid variant value and should become the key `'false'`.
- Do skip nullish variant values because they mean there is no selected value after props and defaults are combined.
- Do not append falsy class values. The config type allows `false`, `null`, and `undefined` to mean "no class for this value".
- Defaults are per variant. A provided prop for one variant should not prevent defaults for other variants from applying.

## Notes

- Only iterate over the configured `variants`. Extra keys in the props object should be ignored.
- Convert boolean values such as `true` and `false` to strings before using them as object keys.
- If a resolved variant value does not exist in the variant map, skip it.

## Techniques

- Closures
- Working with objects and dynamic keys
- Handling defaults and falsy values

## Resources

- [`class-variance-authority` on GitHub](https://github.com/joe-bell/cva)

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
Variant selection uses `props[name] ?? defaultVariants[name]`. A refactor instead creates `{ ...defaultVariants, ...props }` and reads the resulting property directly.

Give a prop value that distinguishes the two approaches when the default `size` is `'md'`. Why is an explicit `false` a different case?

Your notes (optional)
