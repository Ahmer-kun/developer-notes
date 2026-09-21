# Cookies and Sessions

A cookie is a small piece of data the server asks the browser to store and send back on later requests. A session is server-side state for a user, usually looked up via an ID stored in a cookie.

## How it works

```mermaid
sequenceDiagram
    participant B as Browser
    participant S as Server
    participant DB as Session store
    B->>S: POST /login (credentials)
    S->>DB: create session {id: abc, userId: 42}
    S-->>B: Set-Cookie: sid=abc; HttpOnly; Secure; SameSite=Lax
    B->>S: GET /profile (Cookie: sid=abc)
    S->>DB: look up abc
    DB-->>S: userId 42
    S-->>B: 200 profile data
```

The cookie holds an opaque, unguessable ID. The real data (user id, etc.) stays on the server. The browser automatically attaches the cookie to matching requests.

## Cookie attributes

| Attribute | What it does |
|---|---|
| `HttpOnly` | JavaScript can't read it via `document.cookie` (limits damage from XSS) |
| `Secure` | only sent over HTTPS |
| `SameSite=Strict` | not sent on any cross-site request |
| `SameSite=Lax` | sent on top-level navigations (link clicks), not on cross-site subrequests like `fetch`/forms POST |
| `SameSite=None` | sent cross-site; requires `Secure` |
| `Domain` / `Path` | which requests it applies to |
| `Max-Age` / `Expires` | lifetime; without them it's a session cookie (gone when browser closes, roughly) |

Example header:

```
Set-Cookie: sid=abc123; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=86400
```

## Session storage options

- In-memory (default in many frameworks): fine for development, lost on restart, doesn't work across multiple server instances.
- Database or Redis: survives restarts and can be shared between instances.
- Signed-cookie sessions: data lives in the cookie itself, signed (and sometimes encrypted) by the server. No store needed, but size-limited and harder to revoke.

## Common mistakes

- Using the default in-memory session store in production.
- Not regenerating the session ID after login (session fixation).
- Missing `Secure`/`HttpOnly`, or setting `SameSite=None` without understanding cross-site implications.
- Cookies not being sent in cross-origin `fetch`: needs `credentials: "include"` on the client and proper [CORS](../apis/cors.md) headers on the server.
- Cookies set on `localhost` behaving differently across ports: cookies don't isolate by port, unlike origins.
- Storing sensitive data (passwords, full user objects) in cookie contents.

## Practical notes

- Cookies are sent automatically, which is what makes CSRF possible: another site can cause the browser to send a request with your cookie. `SameSite` helps; CSRF tokens are the classic additional defense. Check current framework guidance for which you need.
- Behind a reverse proxy, frameworks may need a "trust proxy" setting to treat the connection as HTTPS, otherwise `Secure` cookies may not be set.
- To debug: devtools → Application/Storage → Cookies, and look at `Set-Cookie` in the response headers.

## When to use sessions

- Traditional server-rendered apps, or SPAs with a same-site backend.
- When you want easy revocation (delete the session record).

## When not to

- Cross-domain or non-browser clients where cookies are awkward (token-based auth may fit better). See [JWT vs sessions](jwt-vs-sessions.md).

## Remember

- Cookie = browser storage that's auto-sent. Session = server state keyed by an ID.
- `HttpOnly` + `Secure` + `SameSite` should be the default.
- Auto-sending is both the convenience and the CSRF risk.

## Related

- [JWT vs sessions](jwt-vs-sessions.md)
- [Authentication vs authorization](authentication-vs-authorization.md)
- [HTTP basics](../apis/http-basics.md)

## References

- RFC 6265 (cookies) and its updates
- MDN: Using HTTP cookies
- OWASP Session Management Cheat Sheet
