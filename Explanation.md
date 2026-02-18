# Bug Explanation

## What was the bug?

When making an API request with `api=True`, the client did not refresh the OAuth2 token when `oauth2_token` was a dict (e.g. `{"access_token": "stale", "expires_at": 0}`). It only refreshed when the token was `None` or when it was an `OAuth2Token` instance that was expired. As a result, no `Authorization` header was set for dict-shaped tokens, causing API calls to fail.

## Why did it happen?

The refresh condition was:

```python
if not self.oauth2_token or (isinstance(self.oauth2_token, OAuth2Token) and self.oauth2_token.expired):
```

For a dict, `not self.oauth2_token` is `False` (dicts are truthy), and `isinstance(..., OAuth2Token)` is `False`, so the whole condition was `False` and no refresh occurred. The code then only set the header when `isinstance(self.oauth2_token, OAuth2Token)`, which is also `False` for a dict, so no header was added.

## Why does your fix solve it?

The fix changes the condition to:

```python
if not isinstance(self.oauth2_token, OAuth2Token) or self.oauth2_token.expired:
```

This refreshes whenever the token is not a valid `OAuth2Token` (including `None` and dicts) or when it is expired. After refresh, `oauth2_token` is always an `OAuth2Token`, so the header is set correctly.

## What's one realistic case / edge case your tests still don't cover?

A token that is an `OAuth2Token` with `expires_at` in the past but very close to "now" (e.g. within the same second) could behave differently depending on when `expired` is evaluated vs when the request is sent. Clock skew or minor timing differences could lead to using a token that is technically expired. The tests use fixed timestamps and do not cover this race window.
