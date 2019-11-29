# Core HTTP API

This API provides authentication-related methods (such as generating access tokens and reseting passwords) and other _system-level_ functionality.

## Authentication

[GraphQL API](./graphql.md) is only available to authenticated clients (authenticated via JWT tokens).

Requests MUST include `Authorization` header with the access token, e.g.:

```sh
$ curl -H "Content-Type: application/json" -H "Authorization: <token>" \
-X POST "http://example.dev/api/graphql"
```

### `POST /api/auth`

Generate the access token for a user.

Example:

```sh
$ curl -H "Content-Type: application/json" -X POST "http://example.dev/api/auth" \
-d '{"email":"user@email.com","password":"password"}'
```

If the authentication succeed the response contains a JSON with an access token:

```json
{"token":"very-long-jwt-token"}
```

In case of failure the response status code is 401 and body contains:

```json
{"error":"Not authorized"}
```

### Authentication Errors

When user is not authenticated to perform the action, the server responds with the
`{"error": "<reason">}` JSON and 401 status code.

The _reason_ could be one of the following:

- `"Not authorized"`— authentication attempt failed (token is invalid or missing or authentication credentials are invalid)
- `"Token has expired"`— authentication token has expired.

### `POST /api/auth/refresh`

When request fails with 401 and `{"error":"Token has expired"}` that means that
the provided access token has expired.

To refresh the token we MUST use the same (expired) access token to perform the following request:

```sh
$ curl -H "Content-Type: application/json" -H "Authorization: <old-token>" \
-X POST "http://example.dev/api/auth/refresh"
```

The response contains the new access token if successful.

### `DELETE /api/auth`

Revoke the access token (=_logout_):

```sh
$ curl -H "Content-Type: application/json" -H "Authorization: <token>" \
-X DELETE "http://example.dev/api/auth"
```

## Resetting Password

### `POST /api/password`

Generate reset password token and delivers e-mail to user with reset password link.

Example:

```sh
$ curl -H "Content-Type: application/json" -X POST "http://example.dev/api/password" \
-d '{"email":"user@email.com"}'
```

**NOTE:** this method always returns 201 status code with empty body (even if the email is not valid).
This way we avoid exposing users emails to attackers.

### `PATCH /api/password`

Update user's password using reset password token.

```sh
$ curl -H "Content-Type: application/json" -X PATCH "http://example.dev/api/password" \
-d '{"token":"<token>","password":"new_password","password_confirmation":"new_password"}'
```

Responds with 200 status code and empty body if successful.

Responds with 401 if token is invalid.

Responds with 422 and `{"errors": ["..."]}` when something went wrong (e.g. confirmation mismatch).
