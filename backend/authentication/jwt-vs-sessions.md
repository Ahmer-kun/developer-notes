# JWT vs Sessions

Two ways to remember who a user is between requests. Neither is universally better.

## What each is

**Session**: server stores state, client holds an opaque ID (usually in a cookie). See [cookies and sessions](cookies-and-sessions.md).

**JWT** (JSON Web Token): the token itself contains claims (user id, expiry, etc.) and is signed by the server. The server can verify it without a lookup.

A JWT has three base64url-encoded parts separated by dots:

```
header.payload.signature
```

```json
// payload example
{ "sub": "42", "role": "editor", "exp": 1767225600 }
```

Important: a normal signed JWT is **not encrypted**. Anyone holding it can decode and read the payload. The signature only guarantees it wasn't modified.

## Comparison

| | Sessions | JWT |
|---|---|---|
| State | server-side | in the token (stateless verification) |
| Client holds | opaque ID | full signed token |
| Revocation | easy (delete session) | hard until expiry (needs a denylist or short lifetimes) |
| Scaling | needs shared store across servers | any server with the key can verify |
| Size per request | small | larger |
| Typical transport | cookie | `Authorization: Bearer` header or cookie |
| Data changes (role change) | effective immediately | stale until token expires |

## Where each fits

- Sessions: browser apps with a backend you control. Simple, easy to revoke.
- JWT: APIs consumed by multiple clients, service-to-service auth, or when several independent services need to verify identity without a shared session store.

"Stateless" is often oversold. Once you add refresh tokens, revocation lists, or rotation, you have state again.

## Common mistakes

- Putting sensitive data in the payload, assuming it is encrypted.
- Long-lived access tokens with no way to revoke them.
- Storing tokens in `localStorage`: readable by any script on the page, so XSS can steal them. Cookies with `HttpOnly` avoid that but bring CSRF considerations.
- Not verifying the signature and the algorithm explicitly. Libraries have had issues where the token's own `alg` header was trusted (including `none`). Configure the accepted algorithms on the server side.
- Not checking `exp`, `iss`, `aud` where relevant.
- Using JWT just because it's popular in tutorials.

## Practical notes

- Common pattern: short-lived access token (minutes) + longer-lived refresh token stored server-side so it can be revoked.
- Symmetric signing (HS256) shares one secret between issuer and verifier. Asymmetric (RS256/ES256) lets others verify with a public key without being able to issue tokens.
- Debug by pasting a token into a decoder locally (not a random site, if the token is real).

## Remember

- JWT = signed, not secret.
- Sessions = easy revocation; JWT = easy distributed verification.
- The hard part of JWT is logout and revocation.

## Related

- [Cookies and sessions](cookies-and-sessions.md)
- [Authentication vs authorization](authentication-vs-authorization.md)
- [Hashing vs encryption](../../cybersecurity/fundamentals/hashing-vs-encryption.md)

## References

- RFC 7519 (JWT), RFC 8725 (JWT best current practices)
- OWASP JWT and Session Management cheat sheets
