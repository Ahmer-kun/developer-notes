# Reading Stack Traces

A stack trace lists the chain of function calls active when an error was created. It tells you where to look, if you read it in the right order.

## Example (Node.js)

```
TypeError: Cannot read properties of undefined (reading 'name')
    at formatUser (/app/src/format.js:12:20)
    at /app/src/routes/users.js:30:17
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at next (/app/node_modules/express/lib/router/route.js:149:13)
    ...
```

## How to read it

1. **The first line**: error type and message. `TypeError: Cannot read properties of undefined (reading 'name')` says something was `undefined` when the code did `.name` on it.
2. **The first frame in your own code**: `format.js:12:20` is file, line, column. Usually the best starting point.
3. **The frames below it**: how execution got there. `users.js:30` called `formatUser`.
4. **Frames from `node_modules` or framework internals**: usually noise. Skim them; the bug is rarely there.

The error message says *what* went wrong. The trace says *where*. The cause is often one step earlier, where the bad value was created.

## Approach

- Open the file at the first line that's yours.
- Ask: which value is not what I expected? (`undefined`, `null`, wrong type)
- Walk *up* the trace to see where that value came from. Log it there.
- Reproduce with the smallest input possible.

## Things that make traces confusing

- **Async code**: the trace may show only the point where the callback resumed, not who scheduled it. Modern engines add async frames (`at async fn`) in many cases, but not always.
- **Transpiled / bundled code**: line numbers refer to generated output. Source maps map them back; make sure they're enabled (browser devtools use them automatically when available; Node needs `--enable-source-maps` for some setups).
- **Minified production code**: unreadable without source maps.
- **Wrapped errors**: some libraries show "Caused by" or an `error.cause` chain. The *inner* error is usually the real one.
- **Swallowed errors**: `catch (e) {}` or re-throwing a new `Error` without the original loses the trace. Preserve it: `throw new Error("context", { cause: e })`.
- **Unhandled promise rejection**: trace may point at the rejection, not at where the promise was created.

## Common error messages

| Message | Usually means |
|---|---|
| `Cannot read properties of undefined (reading 'x')` | variable is undefined; check where it's assigned or fetched |
| `x is not a function` | wrong import, wrong shape, or called before defined |
| `x is not defined` | typo, missing import, or scope issue |
| `Unexpected token < in JSON` | got HTML (often an error page or a 404) where JSON was expected |
| `EADDRINUSE` | see [port already in use](../node/port-already-in-use.md) |
| `Cannot find module 'x'` | not installed, wrong path, or wrong working directory |

The `Unexpected token <` one is worth remembering: log the raw response text and status before parsing.

## Practical notes

- Copy the full trace when asking for help or searching, not just the last line.
- Search the exact error message in quotes, minus paths specific to your machine.
- Browser devtools: click the file link in the console to jump to the line; use "pause on exceptions".
- `console.trace()` prints a stack trace at any point, handy for "who called this?".

## Remember

- Top line = what. First frame in your code = where.
- The bad value usually originated earlier.
- Keep the original error when wrapping.

## Related

- [Promises and async/await](../../programming/javascript/promises-and-async-await.md)
- [Environment variables not loading](../node/env-variables-not-loading.md)

## References

- MDN: Error and `Error.prototype.stack`
- Node.js docs: `--enable-source-maps`
