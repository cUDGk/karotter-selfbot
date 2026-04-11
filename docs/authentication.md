# 認証

Karotterの認証システムの完全なリファレンスです。ログイン、新規登録、トークン管理、セッション管理、マルチアカウント対応を含みます。

---

## ベースURL

| 優先度 | ドメイン |
|--------|--------|
| プライマリ | `https://api.karotter.com/api` |
| フェイルオーバー 1 | `https://api.karotter.jp/api` |
| フェイルオーバー 2 | `https://api.karotter.net/api` |
| フェイルオーバー 3 | `https://apikarotter.karon.jp/api` |

以下のエンドポイントはすべてベースURLからの相対パスです（例: `/auth/login` は `https://api.karotter.com/api/auth/login`）。

---

## 必須ヘッダー

認証済みリクエストには以下のヘッダーが必要です:

| ヘッダー | 説明 | 例 |
|--------|-------------|---------|
| `Authorization` | ログインまたはリフレッシュで取得したBearerトークン | `Bearer eyJhbGciOiJIUzI1NiIs...` |
| `x-csrf-token` | `/auth/csrf-token` から取得したCSRFトークン | `a1b2c3d4-e5f6-7890-abcd-ef1234567890` |
| `x-client-type` | クライアントプラットフォーム識別子 | `web`, `ios`, `android` |
| `x-device-id` | 一意のデバイス識別子（UUID v4推奨） | `550e8400-e29b-41d4-a716-446655440000` |

---

## Axiosクライアント設定

公式クライアントは以下のAxiosデフォルト設定を使用しています:

```js
{
  baseURL: "https://api.karotter.com/api",
  timeout: 15000,           // 15秒
  withCredentials: true,     // クロスオリジンでCookieを送信
  headers: {
    "Content-Type": "application/json"
  }
}
```

**特殊な動作:**
- リクエストボディが `FormData` の場合、ブラウザが正しい `multipart/form-data` バウンダリを自動設定できるよう `Content-Type` ヘッダーは**削除**されます。

### レスポンスインターセプターロジック

クライアントは特定のエラーレスポンスに対する自動リトライロジックを実装しています:

| ステータス | 条件 | アクション |
|--------|-----------|--------|
| `401` | トークン期限切れ | 自動的に `/auth/refresh-token` を呼び出し、トークンを置き換えて、元のリクエストを1回リトライ |
| `403` | CSRFトークン無効（エラーコード `CSRF_INVALID` 等） | `GET /auth/csrf-token` でCSRFトークンを再取得し、保存されたトークンを更新して、元のリクエストを1回リトライ |
| `403` | アカウントBAN検出 | ユーザーをBAN通知ページにリダイレクト。リトライなし |

---

## CSRFトークン

### `GET /auth/csrf-token`

状態変更リクエスト（POST, PUT, PATCH, DELETE）に必要なCSRFトークンを取得します。

**リクエストヘッダー:**

| ヘッダー | 必須 | 説明 |
|--------|----------|-------------|
| `x-client-type` | はい | `web`, `ios`, `android` のいずれか |

**リクエスト:**

```http
GET /auth/csrf-token HTTP/1.1
Host: api.karotter.com
x-client-type: web
```

**レスポンス `200 OK`:**

```json
{
  "csrfToken": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**注意:**
- CSRFトークンは保存して、以降の変更リクエストすべてに `x-csrf-token` として送信する必要があります。
- `403` レスポンスがCSRF不一致を示す場合、このエンドポイントを再取得してリトライしてください。

---

## ログイン

### `POST /auth/login`

ユーザーを認証し、アクセストークン/リフレッシュトークンを受け取ります。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `identifier` | `string` | はい | ユーザー名またはメールアドレス |
| `password` | `string` | はい | アカウントのパスワード |
| `deviceId` | `string` | はい | 一意のデバイス識別子（UUID v4） |
| `clientType` | `string` | はい | `web`, `ios`, `android` のいずれか |
| `deviceName` | `string` | はい | 人間が読めるデバイス名（例: `"Chrome on Windows"`） |

**リクエスト:**

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

**レスポンス `200 OK`:**

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

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `400` | `INVALID_CREDENTIALS` | ユーザー名/メールまたはパスワードが不正 |
| `403` | `ACCOUNT_BANNED` | アカウントがBANされている |
| `403` | `EMAIL_NOT_VERIFIED` | ログイン前にメール確認が必要 |
| `429` | `TOO_MANY_REQUESTS` | レート制限中。指定期間後にリトライ |

---

## 新規登録

### `POST /auth/register`

新しいユーザーアカウントを作成します。Cloudflare Turnstile認証が必要です。

**Cloudflare Turnstile サイトキー:** `0x4AAAAAACujb-w-3YVWR1zA`

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `email` | `string` | はい | 有効なメールアドレス |
| `username` | `string` | はい | 1-15文字、英数字とアンダースコアのみ (`/^[a-zA-Z0-9_]{1,15}$/`) |
| `gender` | `string` | はい | `MALE`, `FEMALE`, `OTHER` のいずれか |
| `password` | `string` | はい | 8-72文字、パスワードポリシーに合格する必要あり（下記参照） |
| `birthday` | `string` | はい | 生年月日 `YYYY-MM-DD` 形式 |
| `acceptTerms` | `boolean` | はい | `true` 必須 — 利用規約に同意 |
| `acceptPrivacy` | `boolean` | はい | `true` 必須 — プライバシーポリシーに同意 |
| `turnstileToken` | `string` | はい | Cloudflare Turnstile認証トークン |
| `_ts` | `number` | はい | 現在のUnixタイムスタンプ（ミリ秒） |

**リクエスト:**

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

**レスポンス `201 Created`:**

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

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `400` | `INVALID_EMAIL` | メール形式が不正 |
| `400` | `INVALID_USERNAME` | ユーザー名が `/^[a-zA-Z0-9_]{1,15}$/` に一致しない |
| `400` | `WEAK_PASSWORD` | パスワードがポリシーチェックに失敗 |
| `400` | `INVALID_BIRTHDAY` | 生年月日が無効または年齢制限未満 |
| `400` | `TURNSTILE_FAILED` | Cloudflare Turnstile認証に失敗 |
| `409` | `EMAIL_TAKEN` | メールアドレスが既に登録済み |
| `409` | `USERNAME_TAKEN` | ユーザー名が既に使用中 |

### パスワードポリシー

パスワードは以下の**すべて**のルールを満たす必要があります:

| ルール | 詳細 |
|------|---------|
| 最小文字数 | 8文字 |
| 最大文字数 | 72文字 |
| ブラックリスト | 拒否されるパスワード: `12345678`, `123456789`, `1234567890`, `password`, `password1`, `qwerty123`, `asdfghjk`, `00000000`, `11111111`, `87654321` |
| 同一数字の繰り返し | すべての文字が同じ数字の場合は拒否（例: `99999999`） |
| 連続数字 | 4つ以上の連続した昇順数字（例: `1234`）または4つ以上の連続した降順数字（例: `4321`）を含む場合は拒否 |

---

## ログアウト

### `POST /auth/logout`

現在のセッションを無効化し、トークンを失効させます。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `deviceId` | `string` | はい | ログイン時に使用したデバイスID |

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "message": "Logged out successfully"
}
```

---

## トークンリフレッシュ

### `POST /auth/refresh-token`

リフレッシュトークンを使用して新しいアクセストークンを取得します。`401` エラー時にレスポンスインターセプターによって自動的に呼び出されます。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `deviceId` | `string` | はい | ログイン時に使用したデバイスID |
| `clientType` | `string` | はい | `web`, `ios`, `android` のいずれか |
| `deviceName` | `string` | はい | 人間が読めるデバイス名 |
| `refreshToken` | `string` | **ネイティブのみ** | `ios` および `android` クライアントで必須。Webクライアントは代わりにHTTP-only Cookieを使用 |

**リクエスト（Webクライアント）:**

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

**リクエスト（ネイティブクライアント）:**

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

**レスポンス `200 OK`:**

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...(new token)...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...(new refresh token)...",
  "sessionId": "sess_refreshed456"
}
```

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `401` | `INVALID_REFRESH_TOKEN` | リフレッシュトークンが期限切れまたは無効 |
| `403` | `SESSION_REVOKED` | セッションが失効済み（例: 他のデバイスにより） |

---

## メール管理

### `POST /auth/me/email`

認証済みユーザーのメールアドレスを更新します。新しいアドレスに確認メールが送信されます。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `email` | `string` | はい | 新しいメールアドレス |

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "message": "Verification email sent"
}
```

### `POST /auth/me/email/resend`

保留中のメールアドレスに確認リンクを再送信します。

**リクエスト:**

```http
POST /auth/me/email/resend HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "Verification email resent"
}
```

**クールダウン:** 再送信リクエスト間は600秒（10分）。早すぎる場合は `429` を返します。

### `POST /auth/verify-email`

確認メールのトークンを使用してメールアドレスを確認します。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `token` | `string` | はい | 確認メールリンクのトークン |

**リクエスト:**

```http
POST /auth/verify-email HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "token": "verify_abc123def456ghi789"
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "Email verified successfully"
}
```

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `400` | `INVALID_TOKEN` | トークンが不正または期限切れ |
| `429` | `TOO_MANY_REQUESTS` | 再送信クールダウンが未経過（600秒） |

---

## パスワードリセット

### `POST /auth/forgot-password`

パスワードリセットメールを要求します。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `email` | `string` | はい | アカウントに登録されたメールアドレス |

**リクエスト:**

```http
POST /auth/forgot-password HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "email": "user@example.com"
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "Password reset email sent"
}
```

> **注意:** このエンドポイントはメールが未登録でも常に `200 OK` を返します。これはメール列挙攻撃を防ぐためです。

### `POST /auth/reset-password`

リセットメールのトークンを使用して新しいパスワードを設定します。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `token` | `string` | はい | リセットメールリンクのトークン |
| `password` | `string` | はい | 新しいパスワード（パスワードポリシーに合格する必要あり） |

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "message": "Password reset successfully"
}
```

---

## セッション

### `GET /auth/sessions`

認証済みユーザーのすべてのアクティブセッションを一覧表示します。

**リクエスト:**

```http
GET /auth/sessions HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**セッションオブジェクトのフィールド:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | `string` | 一意のセッション識別子 |
| `deviceId` | `string` | このセッションに紐づくデバイスUUID |
| `deviceName` | `string` | 人間が読めるデバイスの説明 |
| `clientType` | `string` | `web`, `ios`, `android` のいずれか |
| `isCurrent` | `boolean` | リクエストを行っているセッションかどうか |
| `createdAt` | `string` | ISO 8601 セッション作成タイムスタンプ |
| `lastUsedAt` | `string` | ISO 8601 最終アクティビティのタイムスタンプ |
| `expiresAt` | `string` | ISO 8601 セッション有効期限のタイムスタンプ |

### `DELETE /auth/sessions/:id`

IDを指定して特定のセッションを失効させます。

**リクエスト:**

```http
DELETE /auth/sessions/sess_def456 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "Session revoked"
}
```

### `DELETE /auth/sessions/others`

現在のセッション以外のすべてのセッションを失効させます。

**リクエストヘッダー:**

| ヘッダー | 必須 | 説明 |
|--------|----------|-------------|
| `x-device-id` | はい | 現在のセッションのデバイスID（保持するセッションを特定するため） |

**リクエスト:**

```http
DELETE /auth/sessions/others HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
x-device-id: 550e8400-e29b-41d4-a716-446655440000
```

**レスポンス `200 OK`:**

```json
{
  "revokedCount": 3
}
```

### `DELETE /auth/sessions/all`

現在のセッションを含むすべてのセッションを失効させます。実質的にすべてのデバイスからログアウトします。

**リクエスト:**

```http
DELETE /auth/sessions/all HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "All sessions revoked"
}
```

---

## マルチアカウント対応

Karotterは1つのデバイスで最大**5アカウント**を同時にサポートしています。

### `POST /auth/switch-session`

アクティブセッションを別のアカウントに切り替えます。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `sessionId` | `string` | はい | 切り替え先のセッションID |
| `userId` | `string` | はい | 切り替え先セッションに紐づくユーザーID |
| `deviceId` | `string` | はい | 現在のデバイス識別子 |

**リクエスト:**

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

**レスポンス `200 OK`:**

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

このデバイスのすべてのセッションの未読通知/メッセージ数を取得します。非アクティブアカウントのバッジカウント表示に使用されます。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `deviceId` | `string` | はい | 現在のデバイス識別子 |

**リクエスト:**

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

**レスポンス `200 OK`:**

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

## 現在のユーザー

### `GET /auth/me`

現在の認証済みユーザーの完全なプロフィールを取得します。メールアドレスを含みます（公開プロフィールエンドポイントでは公開されません）。

**リクエスト:**

```http
GET /auth/me HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

> **注意:** ユーザーの `email` フィールドを返す唯一のエンドポイントです。公開の `/users/:usernameOrId` エンドポイントではメールは省略されます。
