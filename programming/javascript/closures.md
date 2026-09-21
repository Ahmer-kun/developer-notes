# Closures

A closure is a function that keeps access to the variables from the scope where it was created, even after that outer scope has finished running.

## What it is

Every function in JavaScript remembers the lexical environment it was defined in. If the function is returned or passed somewhere else, that environment stays alive as long as the function does.

```js
function makeCounter() {
  let count = 0;
  return function () {
    count += 1;
    return count;
  };
}

const a = makeCounter();
const b = makeCounter();

a(); // 1
a(); // 2
b(); // 1
```

`count` is not a global and not a property on anything. It lives in the environment captured by the returned function. Each call to `makeCounter()` creates a new environment, so `a` and `b` have separate counters.

## Why it matters

- Private state without classes (`count` above cannot be touched from outside).
- Callbacks and event handlers that need context from where they were registered.
- Function factories, memoization, debounce/throttle helpers.
- Most "why is this value wrong inside my callback" bugs come from misunderstanding what was captured.

## Common mistakes

**`var` in loops.** `var` is function-scoped, so every callback sees the same variable.

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 3 3 3
```

With `let`, each iteration gets its own binding:

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 0 1 2
```

**Capturing the variable, not the value.** A closure holds a reference to the variable. If the variable changes later, the closure sees the new value.

**Stale closures in React.** A function created during one render captures that render's state and props. If an effect or timer keeps using an old function, it keeps seeing old values. See the React notes once they exist (roadmap).

## Practical notes

- Closures keep captured variables alive. A long-lived closure that captures a large object can hold memory. Usually fine, but worth remembering when debugging leaks (event listeners that are never removed are the classic case).
- Engines optimize this heavily. Don't avoid closures for performance reasons without measuring.
- Closures are about scope, not about `this`. Arrow functions and `this` are a separate topic.

## When to use it

- You want state tied to a function without exposing it.
- You need to configure a function once and reuse it (`makeValidator(min, max)`).

## When not to use it

- If you need many instances with shared methods and inspectable state, a class or plain object may be clearer.

## Remember

- A closure = function + the scope it was created in.
- It captures variables, not snapshots of values.
- `let` in a loop creates a fresh binding per iteration; `var` does not.

## Related

- [Event loop](event-loop.md) (why timers see the values they do)
- [References vs values](references-vs-values.md)
