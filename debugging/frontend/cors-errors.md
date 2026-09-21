# CORS Errors

## Symptoms

Console message like:

```
Access to fetch at 'http://localhost:5000/api/items' from origin 'http://localhost:3000'
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

Other variants: "Response to preflight request doesn't pass access control check", "The value of the 'Access-Control-Allow-Origin' header must not be the wildcard '*' when credentials mode is 'include'".

In JavaScript you typically only see `TypeError: Failed to fetch`, with no status code. That's on purpose: the browser hides the details from scripts.

## Likely causes

- Server doesn't send `Access-Control-Allow-Origin` for your origin.
- Preflight (`OPTIONS`) is not handled: returns 404/405, or needs auth and gets rejected.
- Sending JSON or an `Authorization` header triggers a preflight the server doesn't allow (`Access-Control-Allow-Headers` missing).
- Credentials are used with `*`.
- CORS middleware registered *after* routes, or only on some routes.
- Error responses (4xx/5xx) lack CORS headers, so the real error is masked.
- Redirect in the middle of the request.
- Mismatch in origin: `localhost` vs `127.0.0.1`, http vs https, different port.
- A proxy/CDN strips or overrides headers.

## Checks

1. Devtools → Network. Find the request. Is there an `OPTIONS` request before it? What status did it return, and which `Access-Control-*` headers are in the response?
2. Compare the request's `Origin` header to the `Access-Control-Allow-Origin` in the response. They must match exactly (or the server sends `*` for non-credentialed requests).
3. Reproduce with curl to see what the server sends:

```bash
curl -i -X OPTIONS http://localhost:5000/api/items \
  -H "Origin: http://localhost:3000" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: content-type"
```

Look for `Access-Control-Allow-Origin`, `-Methods`, `-Headers` in the response.

4. Check whether the request reaches the server at all (server logs). If the server logged a 200 and the browser still complains, it's a header issue, not a network one.
5. Using cookies? Client needs `credentials: "include"`; server needs `Access-Control-Allow-Credentials: true` and a specific origin.

## Fix

On the server, send the right headers for the right origin (allowlist), handle `OPTIONS`, and make sure middleware runs first and applies to error responses too. See the Express example in [CORS](../../backend/apis/cors.md).

For local development, a dev-server proxy avoids the problem entirely: the browser talks to the same origin, and the dev server forwards `/api` to the backend.

## Why it happened

Browsers block scripts from reading cross-origin responses unless the server opts in. The check is done by the browser using response headers, so no amount of client-side code can override it.

## What doesn't work

- Adding CORS headers to the *request* on the client.
- `mode: "no-cors"` in `fetch`: gives an opaque response you can't read. It hides the error rather than fixing it.
- Browser extensions that disable CORS: only affects your machine.

## Prevention

- Decide early whether frontend and API share an origin (reverse proxy) or not.
- Keep an allowlist of origins per environment in config.
- Test with the real frontend origin, not just curl or Postman (they don't enforce CORS).

## Related

- [CORS](../../backend/apis/cors.md)
- [HTTP methods](../../backend/apis/http-methods.md)
- [Cookies and sessions](../../backend/authentication/cookies-and-sessions.md)
