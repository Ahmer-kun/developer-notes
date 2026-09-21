# Environment Variables Not Loading

## Symptoms

- `process.env.SOMETHING` is `undefined`.
- Database or API calls fail with missing credentials.
- Works on one machine or terminal but not another.
- Value exists in `.env` but the app behaves as if it doesn't.

## Likely causes

- The `.env` file isn't being loaded at all. Node does not read it by itself in older versions.
- Loaded too late: code that reads `process.env` ran before `dotenv` was configured.
- Wrong working directory: the loader looks for `.env` relative to where the process was **started**, not where the script file is.
- Typo in name, or the file is named `.env.txt` / `env`.
- Frontend tooling requires a prefix and inlines values at build time.
- Server wasn't restarted after editing `.env`.
- Deployment platform doesn't have the variable set (`.env` is usually git-ignored and not deployed).
- Values in `.env` have odd formatting (spaces, quotes, hidden characters).

## Checks

```js
console.log(process.cwd());
console.log(Object.keys(process.env).filter((k) => k.startsWith("DB_")));
```

Print key names, not secret values.

Then check:

1. Is the file actually named `.env` and in the directory printed by `process.cwd()`?
2. Is the loader called first?

```js
// top of the entry file, before anything reads process.env
require("dotenv").config();
```

or, with ES modules:

```js
import "dotenv/config";
```

3. Newer Node versions (v20.6+) can load a file natively:

```bash
node --env-file=.env app.js
```

Version-dependent; check `node --version` and the docs.

4. Using a bundler/framework? Prefixes are usually required for anything exposed to browser code, and the rules are tool-specific. Examples: Vite exposes `VITE_*` via `import.meta.env`; Next.js exposes `NEXT_PUBLIC_*` to the browser. Check the tool's docs.

5. Formatting in `.env`:

```
DB_HOST=localhost
DB_PASSWORD="pass with spaces"
```

No `export` needed (parsers vary), and avoid trailing spaces.

6. Shell variables: `export FOO=bar` only affects that terminal session and its children.

## Fix

Correct the location, load order, name, or prefix. Restart the process. Set the variable in the hosting platform's environment settings for production.

## Why it happened

Environment variables belong to the process that inherits them. A `.env` file is just a text file; something has to read it and copy values into `process.env`. All values end up as strings, so `"false"` is truthy and port numbers need conversion.

## Prevention

- Validate required variables at startup and fail with a clear message:

```js
const required = ["DB_HOST", "DB_PASSWORD"];
const missing = required.filter((k) => !process.env[k]);
if (missing.length) {
  throw new Error(`Missing env vars: ${missing.join(", ")}`);
}
```

- Commit a `.env.example` with names and no secrets. Add `.env` to `.gitignore`.
- Never expose secrets through browser-prefixed variables; they're visible to every visitor.
- Never commit real secrets. If one leaks, rotate it; deleting the commit isn't enough.

## Related

- [Linux cheatsheet](../../devops/linux/linux-cheatsheet.md) (setting variables in a shell)
- [Reading stack traces](../general/reading-stack-traces.md)
