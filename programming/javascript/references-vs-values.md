# References vs Values

Primitives are copied when you assign them. Objects are not: you copy a reference to the same object.

## What it is

Primitive types: string, number, boolean, `null`, `undefined`, symbol, bigint. Everything else (objects, arrays, functions) is an object.

```js
let a = 1;
let b = a;
b = 2;
console.log(a); // 1

const x = { n: 1 };
const y = x;
y.n = 2;
console.log(x.n); // 2  (same object)
```

Passing arguments works the same way. A function receives a copy of the value, and for objects that value is a reference. So the function can mutate the object, but reassigning the parameter does not affect the caller.

```js
function rename(user) {
  user.name = "changed";   // visible outside
  user = { name: "new" };  // not visible outside
}
```

## Why it matters

- Explains "I changed one thing and something else changed".
- State management in frameworks relies on detecting *new* references. Mutating in place often means no update happens.
- Equality: `{} === {}` is `false`. `===` on objects compares identity.

## Copying

```js
const original = { name: "a", tags: ["x", "y"] };

const shallow = { ...original };            // new top-level object
shallow.tags.push("z");                     // original.tags changed too

const deep = structuredClone(original);     // fully independent copy
```

- Spread, `Object.assign`, `slice()` and `Array.from` are **shallow** copies.
- `structuredClone` is a deep copy available in modern browsers and recent Node versions. It can't clone functions, and class instances lose their prototype.
- `JSON.parse(JSON.stringify(x))` also works for plain data but drops `undefined`, functions, and turns dates into strings.

## Common mistakes

- Mutating state directly (`state.items.push(x)`) instead of creating a new array.
- Assuming spread makes a deep copy.
- Comparing objects or arrays with `===` and expecting content comparison.
- Using an object as a default parameter or shared constant, then mutating it.

## Practical notes

- `const` stops reassignment, not mutation. `const arr = []; arr.push(1)` is fine.
- `Object.freeze` is shallow too.
- Sorting with `arr.sort()` mutates the array. `toSorted()` returns a copy in newer runtimes; check support before relying on it.

## Remember

- Assignment copies the value; for objects the value is a reference.
- Spread is shallow.
- New reference = "changed" as far as most frameworks are concerned.

## Related

- [Closures](closures.md)
