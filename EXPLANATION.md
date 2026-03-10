The bug was in `HttpClient.request()`. When `api` was `true`, the client refreshed the token only if `oauth2Token` was `null` or an expired `OAuth2Token` instance. A plain object token such as `{ accessToken: "stale", expiresAt: 0 }` was treated as truthy, so it skipped refresh, but it also failed the later `instanceof OAuth2Token` check, so no `Authorization` header was added.

It happened because the code mixed two token representations in `TokenState`: real `OAuth2Token` instances and generic plain objects. The refresh condition only recognized one invalid case for object shape: `null`. It did not treat non-`OAuth2Token` objects as unusable tokens.

The fix adds one condition: refresh whenever the stored token is not an `OAuth2Token` instance. That keeps the existing behavior for valid typed tokens, still refreshes expired tokens, and now converts unsupported plain-object tokens into a fresh `OAuth2Token` before building the request headers.

One realistic edge case the tests still do not cover is mutation of the caller-provided `headers` object. `request()` reuses the same object reference, so adding `Authorization` may have side effects outside the client.
