# Promises and async/await

A promise represents a value that isn't available yet. `async`/`await` is syntax on top of promises that makes them read like sequential code.

## What it is

A promise is in one of three states: pending, fulfilled, or rejected. Once settled it never changes.

```js
function getUser(id) {
  return fetch(`/api/users/${id}`).then((res) => {
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  });
}
```

The same thing with `async`/`await`:

```js
async function getUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

- An `async` function always returns a promise.
- `await` pauses that function (not the whole program) until the promise settles.
- `throw` inside an async function becomes a rejected promise.

## Common mistakes

**`fetch` does not reject on HTTP errors.** It rejects on network failure. A 404 or 500 still resolves, so check `res.ok`.

**Forgetting `await`.** You get a promise object instead of the value, and errors slip past your `try/catch`.

**Sequential when it could be parallel.**

```js
// slow: second request waits for the first
const a = await getA();
const b = await getB();

// both start immediately
const [a2, b2] = await Promise.all([getA(), getB()]);
```

**`await` inside `forEach`.** `forEach` ignores the returned promise, so nothing waits.

```js
// doesn't wait
items.forEach(async (item) => { await save(item); });

// waits, one at a time
for (const item of items) { await save(item); }

// waits, all at once
await Promise.all(items.map((item) => save(item)));
```

**Unhandled rejections.** A rejected promise nobody handles will log a warning in browsers and can crash a Node process depending on version and settings.

## Combinators

| Function | Resolves when | Rejects when |
|---|---|---|
| `Promise.all` | all fulfil | any rejects (first rejection) |
| `Promise.allSettled` | all settle | never |
| `Promise.race` | first settles (either way) | first settles as rejected |
| `Promise.any` | first fulfils | all reject (`AggregateError`) |

`Promise.all` does not cancel the other promises when one rejects. They keep running; you just stop waiting.

## Practical notes

- `try/catch` only catches errors from things you `await` inside the `try`.
- `.finally()` runs either way and is good for cleanup.
- Returning a value from `.then` wraps it in a promise; returning a promise flattens it.
- Promise callbacks are microtasks, which affects ordering (see [event loop](event-loop.md)).

## When to use it

- Any asynchronous work: network, files, timers wrapped in promises.
- Prefer `async`/`await` for readability. Use `.then` chains or combinators when composing several promises.

## Remember

- Check `res.ok` with `fetch`.
- Independent work goes in `Promise.all`.
- `forEach` and `await` don't mix.

## Related

- [Event loop](event-loop.md)
- [Reading stack traces](../../debugging/general/reading-stack-traces.md) (async traces can be confusing)
