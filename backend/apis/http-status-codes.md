# HTTP Status Codes

The first digit gives the class. The rest is detail. Know the classes, then the handful below.

| Class | Meaning |
|---|---|
| 1xx | informational (rarely seen directly) |
| 2xx | success |
| 3xx | redirection |
| 4xx | client error: the request is the problem |
| 5xx | server error: the server failed to handle a valid-looking request |

## 2xx

| Code | Use |
|---|---|
| 200 OK | normal success with a body |
| 201 Created | resource created; usually a `Location` header points to it |
| 204 No Content | success, nothing to return (common for DELETE) |

## 3xx

| Code | Meaning | Method preserved? |
|---|---|---|
| 301 | permanent redirect | clients may change POST to GET |
| 302 | temporary redirect | clients may change POST to GET |
| 307 | temporary redirect | yes |
| 308 | permanent redirect | yes |
| 304 | not modified (cache is still valid) | n/a |

## 4xx

| Code | Meaning | Note |
|---|---|---|
| 400 | malformed request | bad JSON, missing required data |
| 401 | not authenticated | name is misleading; it means "unauthenticated" |
| 403 | authenticated but not allowed | see [authentication vs authorization](../authentication/authentication-vs-authorization.md) |
| 404 | not found | also used to hide existence of resources |
| 405 | method not allowed | route exists, method doesn't |
| 409 | conflict | duplicate, or version conflict |
| 422 | well-formed but semantically invalid | commonly used for validation errors |
| 429 | too many requests | rate limiting; may include `Retry-After` |

400 vs 422 is a convention choice. Pick one for validation errors and keep it consistent across the API.

## 5xx

| Code | Meaning | Usual cause |
|---|---|---|
| 500 | generic server error | unhandled exception |
| 502 | bad gateway | proxy got an invalid response from upstream |
| 503 | service unavailable | overloaded or down for maintenance |
| 504 | gateway timeout | proxy gave up waiting on upstream |

## Common mistakes

- Returning `200` with `{"error": ...}` in the body. Clients and monitoring tools can't tell it failed.
- Returning `500` for user mistakes (like bad input). That's a 4xx.
- Returning `403` when the user isn't logged in. That should be `401`.
- Leaking stack traces in 500 bodies.

## Practical notes

- A `502`/`504` means look at the layer behind the proxy (app crashed, too slow), not the proxy config first.
- A CORS failure has no status code visible to JavaScript; see [CORS errors](../../debugging/frontend/cors-errors.md).
- Custom error bodies should have a consistent shape (`{"error": {"code": "...", "message": "..."}}` or similar).

## Remember

- 4xx: fix the request. 5xx: fix the server.
- 401 = who are you? 403 = you can't do that.
- 307/308 keep the method, 301/302 may not.

## Related

- [HTTP basics](http-basics.md)
- [HTTP methods](http-methods.md)

## References

- RFC 9110, section on status codes
- MDN: HTTP response status codes
