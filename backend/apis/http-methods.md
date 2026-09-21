# HTTP Methods

Methods tell the server what kind of action the request represents. Two properties matter most: **safe** and **idempotent**.

## Definitions

- **Safe**: not meant to change server state (reads).
- **Idempotent**: sending the same request N times has the same effect on the server as sending it once.

| Method | Typical use | Safe | Idempotent | Body |
|---|---|---|---|---|
| GET | read a resource | yes | yes | no (by convention) |
| HEAD | headers only, no body | yes | yes | no |
| OPTIONS | ask what's allowed (CORS preflight) | yes | yes | rarely |
| POST | create, or trigger an action | no | no | yes |
| PUT | replace a resource fully | no | yes | yes |
| PATCH | partial update | no | not guaranteed | yes |
| DELETE | remove | no | yes | rarely |

(Properties as defined in RFC 9110. They describe what the method is *supposed* to mean; a badly written server can break them.)

## PUT vs PATCH

- `PUT /users/42` with a body means "this is the full new representation". Fields you leave out may be removed or reset.
- `PATCH /users/42` with a body means "apply these changes". Only the fields sent change.

PUT is idempotent because sending the same full replacement twice leaves the same result. PATCH depends on the patch format: "set name to X" is idempotent, "increment counter" is not.

## Why idempotency matters

Networks fail. If a client doesn't get a response, it doesn't know whether the server processed the request. Retrying is safe for GET, PUT, DELETE. Retrying a POST can create duplicates (two orders, two charges).

Common fix for POST: an idempotency key sent by the client (header name varies by API) that the server uses to detect repeats.

## Common mistakes

- Using GET for actions that change data (deleting via a link). Prefetching and crawlers can trigger it.
- Using POST for everything. It works, but you lose caching and clear semantics.
- Assuming DELETE on a missing resource must return an error. Either `404` or `204` is defensible; be consistent.
- Sending a body with GET. Some servers and proxies ignore or reject it.

## Practical notes

- Browsers send an `OPTIONS` preflight before some cross-origin requests. See [CORS](cors.md).
- HTML forms only support GET and POST natively. Frameworks fake the others with a hidden field or use `fetch`.

## Remember

- Safe = read-only intent. Idempotent = repeat-safe.
- PUT replaces, PATCH modifies.
- POST is the one to be careful retrying.

## Related

- [HTTP basics](http-basics.md)
- [HTTP status codes](http-status-codes.md)
