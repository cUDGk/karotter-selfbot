# ユーザー

ユーザープロフィール、フォロワー/フォロー中、プロフィール編集、設定、アカウント管理の完全なリファレンスです。

---

## ユーザープロフィール取得

### `GET /users/:usernameOrId`

ユーザー名またはユーザーIDでユーザーの公開プロフィールを取得します。認証済みユーザーとの関係データを含みます。

**パスパラメータ:**

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `usernameOrId` | `string` | ユーザーのユーザー名（例: `myusername`）またはユニークID（例: `clx1abc2d3ef...`） |

**リクエスト:**

```http
GET /users/myusername HTTP/1.1
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
    "avatarUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp",
    "headerUrl": "https://cdn.karotter.com/headers/clx1abc2d3ef4gh5ij6kl7mn8.webp",
    "bio": "Hello world! This is my profile.",
    "location": "Tokyo, Japan",
    "websiteUrl": "https://example.com",
    "birthday": "2000-05-15",
    "birthdayVisibility": "PUBLIC",
    "birthdayBalloonsEnabled": true,
    "createdAt": "2025-01-15T12:00:00.000Z",
    "followersCount": 150,
    "followingCount": 75,
    "postsCount": 320,
    "isPrivate": false,
    "isBanned": false,
    "onlineStatus": "ONLINE",
    "statusMessage": "Working!",
    "showLikedPosts": true,
    "officialMark": [],
    "isParodyAccount": false,
    "adminForceParody": false,
    "isBotAccount": false,
    "adminForceBot": false
  },
  "isFollowing": true,
  "isFollowedBy": false,
  "hasPendingRequest": false,
  "hasBlocked": false,
  "isBlockedBy": false,
  "isMuted": false,
  "mutualFollowersCount": 3,
  "mutualFollowersPreview": [
    {
      "id": "clx_mutual1",
      "username": "mutualfriend1",
      "displayName": "Mutual Friend",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_mutual1.webp"
    }
  ],
  "pinnedPost": {
    "id": "post_pinned123",
    "content": "This is my pinned post!",
    "createdAt": "2026-02-01T00:00:00.000Z"
  },
  "profileUnavailableReason": null,
  "profileUnavailableDetails": []
}
```

**ユーザーオブジェクトフィールド:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | `string` | ユニークユーザー識別子（CUID形式） |
| `username` | `string` | ユニークなユーザー名（1-15文字、英数字+アンダースコア） |
| `displayName` | `string` | 表示名（1-40 Unicode文字） |
| `avatarUrl` | `string \| null` | アバター画像のURL、未設定の場合は `null` |
| `headerUrl` | `string \| null` | ヘッダー/バナー画像のURL、未設定の場合は `null` |
| `bio` | `string` | ユーザー自己紹介（0-200文字） |
| `location` | `string \| null` | 自由テキストの所在地 |
| `websiteUrl` | `string \| null` | ユーザーのウェブサイトURL |
| `birthday` | `string \| null` | 誕生日（`YYYY-MM-DD`形式）、または `null` |
| `birthdayVisibility` | `string` | `PRIVATE` または `PUBLIC` のいずれか |
| `birthdayBalloonsEnabled` | `boolean` | 誕生日バルーンアニメーションが有効かどうか |
| `createdAt` | `string` | ISO 8601 アカウント作成タイムスタンプ |
| `followersCount` | `number` | フォロワー数 |
| `followingCount` | `number` | フォロー中のユーザー数 |
| `postsCount` | `number` | 投稿総数 |
| `isPrivate` | `boolean` | アカウントが非公開かどうか（フォロー承認が必要） |
| `isBanned` | `boolean` | アカウントがBANされているかどうか |
| `onlineStatus` | `string` | `ONLINE`, `OFFLINE`, `DND` のいずれか |
| `statusMessage` | `string \| null` | カスタムステータスメッセージ（最大15文字） |
| `showLikedPosts` | `boolean` | ユーザーのいいねタブが公開されているかどうか |
| `officialMark` | `array` | 公式認証マークの配列（未認証ユーザーは空） |
| `isParodyAccount` | `boolean` | ユーザーがパロディアカウントを自己申告しているかどうか |
| `adminForceParody` | `boolean` | 管理者がパロディラベルを強制しているかどうか |
| `isBotAccount` | `boolean` | ユーザーがBotアカウントを自己申告しているかどうか |
| `adminForceBot` | `boolean` | 管理者がBotラベルを強制しているかどうか |

**関係フィールド（認証済み閲覧者との相対値）:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `isFollowing` | `boolean` | 閲覧者がこのユーザーをフォローしているかどうか |
| `isFollowedBy` | `boolean` | このユーザーが閲覧者をフォローしているかどうか |
| `hasPendingRequest` | `boolean` | 閲覧者がこのユーザーへのフォローリクエストを保留中かどうか |
| `hasBlocked` | `boolean` | 閲覧者がこのユーザーをブロックしているかどうか |
| `isBlockedBy` | `boolean` | 閲覧者がこのユーザーにブロックされているかどうか |
| `isMuted` | `boolean` | 閲覧者がこのユーザーをミュートしているかどうか |
| `mutualFollowersCount` | `number` | 共通フォロワー数 |
| `mutualFollowersPreview` | `array` | 共通フォロワーのユーザーオブジェクトの小さなプレビュー配列 |
| `pinnedPost` | `Post \| null` | ユーザーのピン留め投稿、または `null` |

**プロフィール非表示理由:**

`profileUnavailableReason`がnull以外の場合、プロフィールの内容は閲覧者から非表示になっている可能性があります。

| `profileUnavailableReason` | 説明 |
|----------------------------|-------------|
| `FILTERED` | 閲覧者のビューからフィルタリングされたプロフィール |
| `null` | プロフィールは完全に表示 |

`profileUnavailableDetails`配列は具体的な理由を提供します:

| 詳細 | 説明 |
|--------|-------------|
| `ADMIN_HIDDEN` | 管理者がこのプロフィールを非表示にした |
| `PARODY_FILTERED` | 閲覧者がパロディアカウントをフィルタリングしている |
| `BOT_FILTERED` | 閲覧者がBotアカウントをフィルタリングしている |
| `R18_FILTERED` | 閲覧者がR18/アダルトコンテンツをフィルタリングしている |
| `MINOR_RESTRICTED` | 閲覧者が未成年で、このコンテンツは年齢制限付き |

---

## ユーザーの投稿、リプライ、メディア、いいね

### `GET /users/:id/posts`

ユーザーのオリジナル投稿（リプライを除く）を取得します。

### `GET /users/:id/replies`

ユーザーのリプライを取得します。

### `GET /users/:id/media`

ユーザーのメディア投稿（画像や動画を含む投稿）を取得します。

### `GET /users/:id/likes`

ユーザーがいいねした投稿を取得します（ユーザーが `showLikedPosts` を有効にしている場合のみ利用可能）。

**クエリパラメータ（4つのエンドポイント共通）:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | `number` | `1` | ページ番号（1始まり） |
| `limit` | `number` | `20` | 1ページあたりの結果数 |

**リクエスト:**

```http
GET /users/clx1abc2d3ef4gh5ij6kl7mn8/posts?page=1&limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

```json
{
  "posts": [
    {
      "id": "post_abc123",
      "content": "Hello world!",
      "author": {
        "id": "clx1abc2d3ef4gh5ij6kl7mn8",
        "username": "myusername",
        "displayName": "My Display Name",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp"
      },
      "createdAt": "2026-03-30T08:00:00.000Z",
      "likesCount": 5,
      "repliesCount": 2,
      "rekarotsCount": 1
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20
  }
}
```

---

## フォロワー & フォロー中

### `GET /users/:id/followers`

ユーザーのフォロワーリストを取得します。

### `GET /users/:id/following`

このユーザーがフォローしているユーザーの一覧を取得します。

**クエリパラメータ（両エンドポイント共通）:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | 1ページあたりの結果数 |
| `cursor` | `string` | — | ページネーション用カーソル（前回のレスポンスから） |

**リクエスト:**

```http
GET /users/clx1abc2d3ef4gh5ij6kl7mn8/followers?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_follower1",
      "username": "follower_one",
      "displayName": "Follower One",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_follower1.webp",
      "bio": "I follow people",
      "isPrivate": false,
      "is_following": false,
      "is_followed_by": true,
      "follow_request_sent": false,
      "officialMark": [],
      "isParodyAccount": false,
      "adminForceParody": false,
      "isBotAccount": false,
      "adminForceBot": false
    }
  ],
  "pagination": {
    "nextCursor": "cursor_abc123def456"
  }
}
```

> **重要: スネークケースフィールド。** フォロワー/フォロー中のユーザーオブジェクトは、リレーションシップフィールド（`is_following`, `is_followed_by`, `follow_request_sent`）に **snake_case** を使用しています。API全体では camelCase が使われていますが、これはサーバーレスポンスの不整合です。

**フォロワー/フォロー中ユーザーオブジェクトフィールド:**

| フィールド | 型 | ケース | 説明 |
|-------|------|------|-------------|
| `id` | `string` | camelCase | ユーザーID |
| `username` | `string` | camelCase | ユーザー名 |
| `displayName` | `string` | camelCase | 表示名 |
| `avatarUrl` | `string \| null` | camelCase | アバターURL |
| `bio` | `string` | camelCase | 自己紹介テキスト |
| `isPrivate` | `boolean` | camelCase | アカウントが非公開かどうか |
| `is_following` | `boolean` | **snake_case** | 閲覧者がこのユーザーをフォローしているか |
| `is_followed_by` | `boolean` | **snake_case** | このユーザーが閲覧者をフォローしているか |
| `follow_request_sent` | `boolean` | **snake_case** | 閲覧者がフォローリクエストを送信しているか |
| `officialMark` | `array` | camelCase | 公式認証マーク |
| `isParodyAccount` | `boolean` | camelCase | パロディアカウントフラグ |
| `adminForceParody` | `boolean` | camelCase | 管理者によるパロディラベル強制 |
| `isBotAccount` | `boolean` | camelCase | Botアカウントフラグ |
| `adminForceBot` | `boolean` | camelCase | 管理者によるBotラベル強制 |

**ページネーション:**

カーソルベースのページネーションを使用します。レスポンスの `nextCursor` の値を次のリクエストの `cursor` クエリパラメータとして渡してください。`nextCursor` が `null` または存在しない場合、これ以上の結果はありません。

### `GET /users/:id/mutual-followers`

あなたと指定ユーザーの両方をフォローしているユーザーを取得します（「共通フォロワー」/「知り合いのフォロワー」）。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | 1ページあたりの結果数 |
| `cursor` | `string` | — | ページネーション用カーソル |

**リクエスト:**

```http
GET /users/clx1abc2d3ef4gh5ij6kl7mn8/mutual-followers?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_mutual1",
      "username": "mutualfriend",
      "displayName": "Mutual Friend",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_mutual1.webp",
      "bio": "Hello!",
      "isPrivate": false,
      "officialMark": [],
      "isParodyAccount": false,
      "isBotAccount": false
    }
  ],
  "pagination": {
    "nextCursor": "cursor_mutual123"
  }
}
```

---

## プロフィール編集

### `PATCH /users/profile`

認証済みユーザーのプロフィールフィールドを更新します。すべてのフィールドは任意で、変更したいフィールドのみ含めてください。

**リクエストボディ:**

| フィールド | 型 | 制約 | 説明 |
|-------|------|-------------|-------------|
| `displayName` | `string` | 1-40 Unicode文字 | 表示名 |
| `bio` | `string` | 0-200文字 | 自己紹介テキスト |
| `location` | `string \| null` | — | 所在地テキスト |
| `websiteUrl` | `string \| null` | — | ウェブサイトURL |
| `birthday` | `string \| null` | `YYYY-MM-DD` または `null` | 誕生日（`null`を設定するとクリア） |
| `birthdayVisibility` | `string` | `PRIVATE` または `PUBLIC` | 誕生日の公開設定 |
| `birthdayBalloonsEnabled` | `boolean` | — | 誕生日バルーンアニメーションを有効にする |
| `gender` | `string` | `MALE`, `FEMALE`, または `OTHER` | 性別 |

**リクエスト:**

```http
PATCH /users/profile HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "displayName": "New Display Name",
  "bio": "Updated bio text",
  "birthday": "2000-05-15",
  "birthdayVisibility": "PUBLIC"
}
```

**レスポンス `200 OK`:**

```json
{
  "user": {
    "id": "clx1abc2d3ef4gh5ij6kl7mn8",
    "username": "myusername",
    "displayName": "New Display Name",
    "bio": "Updated bio text",
    "birthday": "2000-05-15",
    "birthdayVisibility": "PUBLIC"
  }
}
```

### テキストサニタイゼーション

すべてのテキストフィールドはサーバー側でサニタイゼーションされます:

1. **HTMLストリッピング** (`stripHtml`): 入力からすべてのHTMLタグが除去されます。
2. **制御文字の除去**: Unicode制御文字（C0、C1範囲）がストリップされます。
3. **文字カウント**: 文字数は `Array.from(text).length` で計算され、バイトではなく **Unicodeコードポイント** をカウントします。つまり、1つの絵文字（例: `😀`）は1文字としてカウントされ、結合絵文字（例: `👨‍👩‍👧‍👦`）は複数コードポイントとしてカウントされる場合があります。

---

## オンラインステータス更新

### `PATCH /users/status`

ユーザーのオンラインステータスとオプションのステータスメッセージを設定します。

**リクエストボディ:**

| フィールド | 型 | 制約 | 説明 |
|-------|------|-------------|-------------|
| `status` | `string` | `ONLINE`, `OFFLINE`, `DND` | オンラインステータス |
| `statusMessage` | `string` | 最大15文字 | カスタムステータスメッセージ |

**リクエスト:**

```http
PATCH /users/status HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "status": "DND",
  "statusMessage": "Do not disturb"
}
```

**レスポンス `200 OK`:**

```json
{
  "status": "DND",
  "statusMessage": "Do not disturb"
}
```

---

## プロフィール画像

### `POST /profile/avatar`

新しいアバター画像をアップロードします。

**リクエスト:** `multipart/form-data`

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `avatar` | `File` | 画像ファイル (JPEG, PNG, WebP, GIF) |

**リクエスト:**

```http
POST /profile/avatar HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="avatar"; filename="avatar.png"
Content-Type: image/png

(binary data)
------FormBoundary--
```

**レスポンス `200 OK`:**

```json
{
  "imageUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp"
}
```

### `POST /profile/header`

新しいヘッダー/バナー画像をアップロードします。

**リクエスト:** `multipart/form-data`

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `header` | `File` | 画像ファイル (JPEG, PNG, WebP, GIF) |

**レスポンス `200 OK`:**

```json
{
  "imageUrl": "https://cdn.karotter.com/headers/clx1abc2d3ef4gh5ij6kl7mn8.webp"
}
```

---

## パスワード変更

### `PATCH /users/password`

認証済みユーザーのパスワードを変更します。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `currentPassword` | `string` | Yes | 確認用の現在のパスワード |
| `newPassword` | `string` | Yes | 新しいパスワード（パスワードポリシーを満たす必要あり） |

**リクエスト:**

```http
PATCH /users/password HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "currentPassword": "MyOldPassword123",
  "newPassword": "MyNewPassword456"
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "Password updated successfully"
}
```

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `400` | `WEAK_PASSWORD` | 新しいパスワードがパスワードポリシーを満たしていない |
| `401` | `INVALID_PASSWORD` | 現在のパスワードが正しくない |

---

## ユーザー名変更

### `PATCH /users/username`

認証済みユーザーのユーザー名を変更します。クォータシステムの対象です。

**リクエストボディ:**

| フィールド | 型 | 必須 | 制約 | 説明 |
|-------|------|----------|-------------|-------------|
| `username` | `string` | Yes | `/^[a-zA-Z0-9_]{1,15}$/` | 新しいユーザー名 |

**リクエスト:**

```http
PATCH /users/username HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "username": "mynewusername"
}
```

**レスポンス `200 OK`:**

```json
{
  "user": {
    "id": "clx1abc2d3ef4gh5ij6kl7mn8",
    "username": "mynewusername"
  }
}
```

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `400` | `INVALID_USERNAME` | 正規表現パターンに一致しない |
| `409` | `USERNAME_TAKEN` | ユーザー名は既に使用されている |
| `429` | `QUOTA_EXCEEDED` | 14日間ウィンドウでのユーザー名変更回数の残りがない |

### `GET /users/username/quota`

現在の14日間ウィンドウで残りのユーザー名変更回数を確認します。

**リクエスト:**

```http
GET /users/username/quota HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

```json
{
  "remainingChanges": 2,
  "usedChanges": 1
}
```

---

## ユーザー設定

### `PATCH /users/settings`

ユーザー設定を更新します。部分更新のため、変更したいフィールドのみ含めてください。

**リクエストボディ（すべてのフィールド任意）:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `isPrivate` | `boolean` | アカウントを非公開にする（フォロー承認が必要） |
| `showLikedPosts` | `boolean` | プロフィールにいいねタブを表示 |
| `showReadReceipts` | `boolean` | DMで既読表示を表示 |
| `directMessagesEnabled` | `boolean` | ダイレクトメッセージを許可 |
| `dmRequestPolicy` | `string` | DMリクエストポリシー: `EVERYONE`, `FOLLOWING`, `CIRCLE` |
| `showReactions` | `boolean` | 投稿にリアクションを表示。**特別:** `false`に設定すると`notifyReactions`も`false`になる |
| `notifyLikes` | `boolean` | いいね時に通知 |
| `notifyReplies` | `boolean` | リプライ時に通知 |
| `notifyRekarots` | `boolean` | リカロット時に通知 |
| `notifyFollows` | `boolean` | 新規フォロー時に通知 |
| `notifyMentions` | `boolean` | メンション時に通知 |
| `notifyQuotes` | `boolean` | 引用投稿時に通知 |
| `notifyReactions` | `boolean` | リアクション時に通知 |
| `notifyDMs` | `boolean` | ダイレクトメッセージ時に通知 |
| `notifyDMRequests` | `boolean` | DMリクエスト時に通知 |
| `notifyPollResults` | `boolean` | 投票結果更新時に通知 |
| `showOnlineStatus` | `boolean` | オンラインステータスを他のユーザーに表示 |
| `showFollowerCount` | `boolean` | プロフィールにフォロワー数を表示 |
| `showFollowingCount` | `boolean` | プロフィールにフォロー中数を表示 |
| `showPostCount` | `boolean` | プロフィールに投稿数を表示 |
| `showBirthday` | `boolean` | プロフィールに誕生日を表示 |
| `mutedKeywords` | `string[]` | ミュートキーワード文字列の配列 |

**リクエスト:**

```http
PATCH /users/settings HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "isPrivate": true,
  "dmRequestPolicy": "FOLLOWING",
  "notifyLikes": false,
  "showReactions": false,
  "mutedKeywords": ["spoiler", "leak"]
}
```

**レスポンス `200 OK`:**

```json
{
  "settings": {
    "isPrivate": true,
    "showLikedPosts": true,
    "showReadReceipts": true,
    "directMessagesEnabled": true,
    "dmRequestPolicy": "FOLLOWING",
    "showReactions": false,
    "notifyLikes": false,
    "notifyReplies": true,
    "notifyRekarots": true,
    "notifyFollows": true,
    "notifyMentions": true,
    "notifyQuotes": true,
    "notifyReactions": false,
    "notifyDMs": true,
    "notifyDMRequests": true,
    "notifyPollResults": true,
    "showOnlineStatus": true,
    "showFollowerCount": true,
    "showFollowingCount": true,
    "showPostCount": true,
    "showBirthday": true,
    "mutedKeywords": ["spoiler", "leak"]
  }
}
```

> **注意:** `showReactions`を`false`に設定すると、`notifyReactions`に送信された値に関係なく、サーバーは自動的に`notifyReactions`も`false`に設定します。

---

## アカウント削除

### `DELETE /users/account`

認証済みユーザーのアカウントを完全に削除します。この操作は元に戻せません。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `password` | `string` | Yes | 確認用の現在のパスワード |

**リクエスト:**

```http
DELETE /users/account HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "password": "MyPassword123"
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "Account deleted successfully"
}
```

---

## おすすめユーザー

### `GET /users/recommended`

フォローするおすすめユーザーの一覧を取得します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `3` | 返すおすすめユーザーの数 |

**リクエスト:**

```http
GET /users/recommended?limit=3 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_rec1",
      "username": "popular_user",
      "displayName": "Popular User",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_rec1.webp",
      "bio": "Content creator",
      "followersCount": 5000,
      "isPrivate": false,
      "officialMark": ["VERIFIED"],
      "isParodyAccount": false,
      "isBotAccount": false
    },
    {
      "id": "clx_rec2",
      "username": "trending_creator",
      "displayName": "Trending Creator",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_rec2.webp",
      "bio": "Artist & designer",
      "followersCount": 3200,
      "isPrivate": false,
      "officialMark": [],
      "isParodyAccount": false,
      "isBotAccount": false
    }
  ]
}
```
