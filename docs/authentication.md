# Authentication

Complete reference for the Karotter authentication system, including login, registration, token management, session handling, and multi-account support.

---

## Base URL

| Priority | Domain |
|----------|--------|
| Primary | `https://api.karotter.com/api` |
| Failover 1 | `https://api.karotter.jp/api` |
| Failover 2 | `https://api.karotter.net/api` |
| Failover 3 | `https://apikarotter.karon.jp/api` |

All endpoints documented below are relative to the base URL (e.g. `/auth/login` means `https://api.karotter.com/api/auth/login`).

---

## Required Headers

Every authenticated request must include the following headers:

| Header | Description | Example |
|--------|-------------|---------|
| `Authorization` | Bearer token obtained from login or refresh | `Bearer eyJhbGciOiJIUzI1NiIs...` |
| `x-csrf-token` | CSRF token obtained from `/auth/csrf-token` | `a1b2c3d4-e5f6-7890-abcd-ef1234567890` |
| `x-client-type` | Client platform identifier | `web`, `ios`, `android` |
| `x-device-id` | Unique device identifier (UUID v4 recommended) | `550e8400-e29b-41d4-a716-446655440000` |

---

## Axios Client Configuration

The official client uses the following Axios defaults:

```js
{
  baseURL: "https://api.karotter.com/api",
  timeout: 15000,           // 15 seconds
  withCredentials: true,     // send cookies cross-origin
  headers: {
    "Content-Type": "application/json"
  }
}
```

**Special behavior:**
- When the request body is `FormData`, the `Content-Type` header is **stripped** so the browser can set the correct `multipart/form-data` boundary automatically.

### Response Interceptor Logic

The client implements automatic retry logic on certain error responses:

| Status | Condition | Action |
|--------|-----------|--------|
| `401` | Token expired | Automatically call `/auth/refresh-token`, replace the token, and retry the original request once. |
| `403` | CSRF token invalid (error code `CSRF_INVALID` or similar) | Re-fetch CSRF token via `GET /auth/csrf-token`, update stored token, and retry the original request once. |
| `403` | Account banned detected | Redirect the user to a ban notice page. No retry. |

---

## CSRF Token

### `GET /auth/csrf-token`

Retrieves a CSRF token required for all state-changing requests (POST, PUT, PATCH, DELETE).

**Request Headers:**

| Header | Required | Description |
|--------|----------|-------------|
| `x-client-type` | Yes | Must be one of `web`, `ios`, `android` |

**Request:**

```http
GET /auth/csrf-token HTTP/1.1
Host: api.karotter.com
x-client-type: web
```

**Response `200 OK`:**

```json
{
  "csrfToken": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Notes:**
- The CSRF token should be stored and sent as `x-csrf-token` on all subsequent mutating requests.
- If a `403` response indicates a CSRF mismatch, re-fetch this endpoint and retry.

---

## Login

### `POST /auth/login`

Authenticate a user and receive access/refresh tokens.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `identifier` | `string` | Yes | Username or email address |
| `password` | `string` | Yes | Account password |
| `deviceId` | `string` | Yes | Unique device identifier (UUID v4) |
| `clientType` | `string` | Yes | One of `web`, `ios`, `android` |
| `deviceName` | `string` | Yes | Human-readable device name (e.g. `"Chrome on Windows"`) |

**Request:**

```http
POST /auth/login HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
x-client-type: web

{
  "identifier": "myusername",
  "password": "MySecurePassword123",
  "deviceId": "550e8400-e29b-41d4-a716-446655440000",
  "clientType": "web",
  "deviceName": "Chrome on Windows"
}
```

**Response `200 OK`:**

```json
{
  "user": {
    "id": "clx1abc2d3ef4gh5ij6kl7mn8",
    "username": "myusername",
    "displayName": "My Display Name",
    "email": "user@example.com",
    "avatarUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp",
    "headerUrl": null,
    "bio": "Hello world",
    "isPrivate": false,
    "isBanned": false,
    "createdAt": "2025-01-15T12:00:00.000Z"
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "sessionId": "sess_abc123def456"
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `400` | `INVALID_CREDENTIALS` | Incorrect username/email or password |
| `403` | `ACCOUNT_BANNED` | The account has been banned |
| `403` | `EMAIL_NOT_VERIFIED` | Email verification required before login |
| `429` | `TOO_MANY_REQUESTS` | Rate limited; retry after the specified period |

---

## Registration

### `POST /auth/register`

Create a new user account. Requires Cloudflare Turnstile verification.

**Cloudflare Turnstile Sitekey:** `0x4AAAAAACujb-w-3YVWR1zA`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | `string` | Yes | Valid email address |
| `username` | `string` | Yes | 1-15 characters, alphanumeric and underscores only (`/^[a-zA-Z0-9_]{1,15}$/`) |
| `gender` | `string` | Yes | One of `MALE`, `FEMALE`, `OTHER` |
| `password` | `string` | Yes | 8-72 characters, must pass password policy (see below) |
| `birthday` | `string` | Yes | Date of birth in `YYYY-MM-DD` format |
| `acceptTerms` | `boolean` | Yes | Must be `true` — user accepts terms of service |
| `acceptPrivacy` | `boolean` | Yes | Must be `true` — user accepts privacy policy |
| `turnstileToken` | `string` | Yes | Cloudflare Turnstile verification token |
| `_ts` | `number` | Yes | Current Unix timestamp in milliseconds |

**Request:**

```http
POST /auth/register HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
x-client-type: web

{
  "email": "newuser@example.com",
  "username": "newuser",
  "gender": "OTHER",
  "password": "MySecurePassword123",
  "birthday": "2000-05-15",
  "acceptTerms": true,
  "acceptPrivacy": true,
  "turnstileToken": "0.turnstile_token_value_here",
  "_ts": 1711756800000
}
```

**Response `201 Created`:**

```json
{
  "user": {
    "id": "clx9new0user1id2here3xyz",
    "username": "newuser",
    "displayName": "newuser",
    "email": "newuser@example.com",
    "avatarUrl": null,
    "createdAt": "2026-03-30T12:00:00.000Z"
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "sessionId": "sess_new789"
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `400` | `INVALID_EMAIL` | Email format is invalid |
| `400` | `INVALID_USERNAME` | Username does not match `/^[a-zA-Z0-9_]{1,15}$/` |
| `400` | `WEAK_PASSWORD` | Password fails policy checks |
| `400` | `INVALID_BIRTHDAY` | Birthday is not a valid date or user is too young |
| `400` | `TURNSTILE_FAILED` | Cloudflare Turnstile verification failed |
| `409` | `EMAIL_TAKEN` | Email address is already registered |
| `409` | `USERNAME_TAKEN` | Username is already taken |

### Password Policy

Passwords must satisfy **all** of the following rules:

| Rule | Details |
|------|---------|
| Minimum length | 8 characters |
| Maximum length | 72 characters |
| Blacklist | Rejected passwords: `12345678`, `123456789`, `1234567890`, `password`, `password1`, `qwerty123`, `asdfghjk`, `00000000`, `11111111`, `87654321` |
| All-same-digit | Rejected if every character is the same digit (e.g. `99999999`) |
| Sequential digits | Rejected if the password contains 4 or more sequential ascending digits (e.g. `1234`) or 4 or more sequential descending digits (e.g. `4321`) |

---

## Logout

### `POST /auth/logout`

Invalidate the current session and revoke tokens.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `deviceId` | `string` | Yes | The device ID used during login |

**Request:**

```http
POST /auth/logout HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
x-client-type: web

{
  "deviceId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response `200 OK`:**

```json
{
  "message": "Logged out successfully"
}
```

---

## Token Refresh

### `POST /auth/refresh-token`

Obtain a new access token using the refresh token. Called automatically by the response interceptor on `401` errors.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `deviceId` | `string` | Yes | The device ID used during login |
| `clientType` | `string` | Yes | One of `web`, `ios`, `android` |
| `deviceName` | `string` | Yes | Human-readable device name |
| `refreshToken` | `string` | **Native only** | Required for `ios` and `android` clients. Web clients rely on HTTP-only cookies instead. |

**Request (web client):**

```http
POST /auth/refresh-token HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
x-client-type: web
x-device-id: 550e8400-e29b-41d4-a716-446655440000

{
  "deviceId": "550e8400-e29b-41d4-a716-446655440000",
  "clientType": "web",
  "deviceName": "Chrome on Windows"
}
```

**Request (native client):**

```http
POST /auth/refresh-token HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
x-client-type: android
x-device-id: 550e8400-e29b-41d4-a716-446655440000

{
  "deviceId": "550e8400-e29b-41d4-a716-446655440000",
  "clientType": "android",
  "deviceName": "Pixel 8 Pro",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response `200 OK`:**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...(new token)...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...(new refresh token)...",
  "sessionId": "sess_refreshed456"
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `401` | `INVALID_REFRESH_TOKEN` | Refresh token is expired or invalid |
| `403` | `SESSION_REVOKED` | The session was revoked (e.g. by another device) |

---

## Email Management

### `POST /auth/me/email`

Update the authenticated user's email address. Triggers a verification email to the new address.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | `string` | Yes | New email address |

**Request:**

```http
POST /auth/me/email HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "email": "newemail@example.com"
}
```

**Response `200 OK`:**

```json
{
  "message": "Verification email sent"
}
```

### `POST /auth/me/email/resend`

Resend the email verification link to the pending email address.

**Request:**

```http
POST /auth/me/email/resend HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "Verification email resent"
}
```

**Cooldown:** 600 seconds (10 minutes) between resend requests. Returns `429` if called sooner.

### `POST /auth/verify-email`

Verify an email address using the token from the verification email.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `token` | `string` | Yes | Verification token from the email link |

**Request:**

```http
POST /auth/verify-email HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "token": "verify_abc123def456ghi789"
}
```

**Response `200 OK`:**

```json
{
  "message": "Email verified successfully"
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `400` | `INVALID_TOKEN` | Token is malformed or expired |
| `429` | `TOO_MANY_REQUESTS` | Resend cooldown not yet elapsed (600s) |

---

## Password Reset

### `POST /auth/forgot-password`

Request a password reset email.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | `string` | Yes | The account's registered email address |

**Request:**

```http
POST /auth/forgot-password HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "email": "user@example.com"
}
```

**Response `200 OK`:**

```json
{
  "message": "Password reset email sent"
}
```

> **Note:** This endpoint always returns `200 OK` even if the email is not registered, to prevent email enumeration attacks.

### `POST /auth/reset-password`

Set a new password using the token from the reset email.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `token` | `string` | Yes | Reset token from the email link |
| `password` | `string` | Yes | New password (must pass password policy) |

**Request:**

```http
POST /auth/reset-password HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "token": "reset_xyz789abc123",
  "password": "MyNewSecurePassword456"
}
```

**Response `200 OK`:**

```json
{
  "message": "Password reset successfully"
}
```

---

## Sessions

### `GET /auth/sessions`

List all active sessions for the authenticated user.

**Request:**

```http
GET /auth/sessions HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "sessions": [
    {
      "id": "sess_abc123",
      "deviceId": "550e8400-e29b-41d4-a716-446655440000",
      "deviceName": "Chrome on Windows",
      "clientType": "web",
      "isCurrent": true,
      "createdAt": "2026-03-25T10:00:00.000Z",
      "lastUsedAt": "2026-03-30T08:30:00.000Z",
      "expiresAt": "2026-04-25T10:00:00.000Z"
    },
    {
      "id": "sess_def456",
      "deviceId": "660f9500-f30c-52e5-b827-557766551111",
      "deviceName": "iPhone 15",
      "clientType": "ios",
      "isCurrent": false,
      "createdAt": "2026-03-20T14:00:00.000Z",
      "lastUsedAt": "2026-03-29T22:15:00.000Z",
      "expiresAt": "2026-04-20T14:00:00.000Z"
    }
  ]
}
```

**Session Object Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique session identifier |
| `deviceId` | `string` | Device UUID associated with this session |
| `deviceName` | `string` | Human-readable device description |
| `clientType` | `string` | One of `web`, `ios`, `android` |
| `isCurrent` | `boolean` | Whether this is the session making the request |
| `createdAt` | `string` | ISO 8601 timestamp of session creation |
| `lastUsedAt` | `string` | ISO 8601 timestamp of last activity |
| `expiresAt` | `string` | ISO 8601 timestamp of session expiration |

### `DELETE /auth/sessions/:id`

Revoke a specific session by its ID.

**Request:**

```http
DELETE /auth/sessions/sess_def456 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "Session revoked"
}
```

### `DELETE /auth/sessions/others`

Revoke all sessions except the current one.

**Request Headers:**

| Header | Required | Description |
|--------|----------|-------------|
| `x-device-id` | Yes | Device ID of the current session (to identify which session to keep) |

**Request:**

```http
DELETE /auth/sessions/others HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
x-device-id: 550e8400-e29b-41d4-a716-446655440000
```

**Response `200 OK`:**

```json
{
  "revokedCount": 3
}
```

### `DELETE /auth/sessions/all`

Revoke all sessions including the current one. Effectively logs the user out of all devices.

**Request:**

```http
DELETE /auth/sessions/all HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "All sessions revoked"
}
```

---

## Multi-Account Support

Karotter supports up to **5 accounts** simultaneously on a single device.

### `POST /auth/switch-session`

Switch the active session to a different account.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sessionId` | `string` | Yes | Target session ID to switch to |
| `userId` | `string` | Yes | User ID associated with the target session |
| `deviceId` | `string` | Yes | Current device identifier |

**Request:**

```http
POST /auth/switch-session HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "sessionId": "sess_account2",
  "userId": "clx2second0account1id2here",
  "deviceId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response `200 OK`:**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...(token for switched account)...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "sessionId": "sess_account2",
  "user": {
    "id": "clx2second0account1id2here",
    "username": "myaltaccount",
    "displayName": "Alt Account"
  }
}
```

### `POST /auth/session-unread-snapshots`

Get unread notification/message counts for all sessions on this device. Used to display badge counts for inactive accounts.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `deviceId` | `string` | Yes | Current device identifier |

**Request:**

```http
POST /auth/session-unread-snapshots HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "deviceId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response `200 OK`:**

```json
{
  "snapshots": [
    {
      "sessionId": "sess_abc123",
      "userId": "clx1abc2d3ef4gh5ij6kl7mn8",
      "unreadNotifications": 5,
      "unreadMessages": 2
    },
    {
      "sessionId": "sess_account2",
      "userId": "clx2second0account1id2here",
      "unreadNotifications": 12,
      "unreadMessages": 0
    }
  ]
}
```

---

## Current User

### `GET /auth/me`

Retrieve the full profile of the currently authenticated user, including email (not exposed on the public profile endpoint).

**Request:**

```http
GET /auth/me HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "user": {
    "id": "clx1abc2d3ef4gh5ij6kl7mn8",
    "username": "myusername",
    "displayName": "My Display Name",
    "email": "user@example.com",
    "avatarUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp",
    "headerUrl": "https://cdn.karotter.com/headers/clx1abc2d3ef4gh5ij6kl7mn8.webp",
    "bio": "Hello world",
    "location": "Tokyo, Japan",
    "websiteUrl": "https://example.com",
    "birthday": "2000-05-15",
    "birthdayVisibility": "PUBLIC",
    "birthdayBalloonsEnabled": true,
    "gender": "OTHER",
    "isPrivate": false,
    "isBanned": false,
    "createdAt": "2025-01-15T12:00:00.000Z",
    "followersCount": 150,
    "followingCount": 75,
    "postsCount": 320,
    "onlineStatus": "ONLINE",
    "statusMessage": "Working!",
    "showLikedPosts": true,
    "officialMark": [],
    "isParodyAccount": false,
    "adminForceParody": false,
    "isBotAccount": false,
    "adminForceBot": false
  }
}
```

> **Note:** This is the only endpoint that returns the user's `email` field. The public `/users/:usernameOrId` endpoint omits it.
