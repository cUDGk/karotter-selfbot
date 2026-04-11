# フォロー API

> **ベースURL:** `https://karotter.com/api`
>
> すべてのエンドポイントは `Authorization: Bearer {token}` ヘッダーと `X-CSRF-Token` ヘッダーによる認証が必要です。
> Cookieベース認証（`karotter_at`, `karotter_rt`, `karotter_csrf`）もサポートされており、一部の操作で必要な場合があります。

---

## 目次

- [Follow a User](#follow-a-user)
- [Unfollow a User](#unfollow-a-user)
- [Block a User](#block-a-user)
- [Unblock a User](#unblock-a-user)
- [Mute a User](#mute-a-user)
- [Unmute a User](#unmute-a-user)
- [Remove a Follower](#remove-a-follower)
- [Get Block List](#get-block-list)
- [Get Mute List](#get-mute-list)
- [Get Pending Follow Requests](#get-pending-follow-requests)
- [Accept Follow Request](#accept-follow-request)
- [Reject Follow Request](#reject-follow-request)
- [Enable Post Notifications](#enable-post-notifications)
- [Disable Post Notifications](#disable-post-notifications)
- [Hide Rekarots](#hide-rekarots)
- [Unhide Rekarots](#unhide-rekarots)
- [Optimistic Update Logic](#optimistic-update-logic)

---

## ユーザーをフォロー

指定したユーザーをフォローします。対象ユーザーが非公開アカウントの場合、即座にフォローする代わりにフォローリクエストが送信されます。

```
POST /api/follow/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the target user |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "フォローしました"}` | Successfully followed the user |
| 200 | `{"message": "フォローリクエストを送信しました"}` | Target has a private account; a follow request was sent |
| 400 | `{"error": "自分自身をフォローすることはできません"}` | Attempted to follow yourself |
| 400 | `{"error": "既にフォローしています"}` | Already following this user |
| 404 | `{"error": "ユーザーが見つかりません"}` | User ID does not exist |

### Example

```http
POST /api/follow/18179 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "フォローしました"
}
```

### 楽観的更新

クライアントがフォローリクエストを送信した時、UIは以下を行います:
1. Immediately increment the target user's `followersCount` by 1
2. Set `isFollowing` to `true` on the user relationship object
3. Increment the authenticated user's `followingCount` by 1
4. If the target is private, set `hasPendingRequest` to `true` instead of `isFollowing`
5. On error, revert all changes

---

## フォロー解除

指定したユーザーとのフォロー関係を解除します。

```
DELETE /api/follow/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the target user |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "フォローを解除しました"}` | Successfully unfollowed |
| 400 | `{"error": "フォローしていません"}` | Not currently following this user |
| 404 | `{"error": "ユーザーが見つかりません"}` | User ID does not exist |

### Example

```http
DELETE /api/follow/18179 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "フォローを解除しました"
}
```

### 楽観的更新

1. Immediately decrement the target user's `followersCount` by 1
2. Set `isFollowing` to `false`
3. Decrement the authenticated user's `followingCount` by 1
4. On error, revert all changes

---

## ユーザーをブロック

指定したユーザーをブロックします。ブロックすると双方向のフォローが自動的に解除されます（あなたが相手のフォローを解除し、相手もあなたのフォローを解除）。

```
POST /api/follow/block/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the user to block |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ユーザーをブロックしました"}` | Successfully blocked |
| 400 | `{"error": "自分自身をブロックすることはできません"}` | Attempted to block yourself |
| 400 | `{"error": "既にブロックしています"}` | Already blocking this user |
| 404 | `{"error": "ユーザーが見つかりません"}` | User ID does not exist |

### Example

```http
POST /api/follow/block/22704 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "ユーザーをブロックしました"
}
```

### 楽観的更新

1. Set `isBlocked` to `true` on the user relationship
2. Set `isFollowing` to `false` (auto-unfollow)
3. Set `isFollowedBy` to `false` (they are auto-unfollowed)
4. Decrement both `followersCount` and `followingCount` as appropriate
5. Hide posts from this user in timeline/search
6. On error, revert all changes

---

## ブロック解除

指定したユーザーのブロックを解除します。以前のフォロー関係は復元されません。

```
DELETE /api/follow/block/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the user to unblock |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ブロックを解除しました"}` | Successfully unblocked |
| 400 | `{"error": "ブロックしていません"}` | Not currently blocking this user |
| 404 | `{"error": "ユーザーが見つかりません"}` | User ID does not exist |

### Example

```http
DELETE /api/follow/block/22704 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "ブロックを解除しました"
}
```

### 楽観的更新

1. Set `isBlocked` to `false`
2. Remove the user from the local block list cache
3. Show posts from this user again in timeline/search
4. On error, revert

---

## ユーザーをミュート

指定したユーザーをミュートします。そのユーザーの投稿はタイムラインから非表示になりますが、フォロー状態は維持されます。

```
POST /api/follow/mute/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the user to mute |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ユーザーをミュートしました"}` | Successfully muted |
| 400 | `{"error": "自分自身をミュートすることはできません"}` | Attempted to mute yourself |
| 400 | `{"error": "既にミュートしています"}` | Already muting this user |
| 404 | `{"error": "ユーザーが見つかりません"}` | User ID does not exist |

### Example

```http
POST /api/follow/mute/22704 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "ユーザーをミュートしました"
}
```

### 楽観的更新

1. Set `isMuted` to `true` on the user relationship
2. Add user to local mute list cache
3. Filter the user's posts from timeline rendering
4. On error, revert

---

## ミュート解除

指定したユーザーのミュートを解除します。

```
DELETE /api/follow/mute/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the user to unmute |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ミュートを解除しました"}` | Successfully unmuted |
| 400 | `{"error": "ミュートしていません"}` | Not currently muting this user |
| 404 | `{"error": "ユーザーが見つかりません"}` | User ID does not exist |

### Example

```http
DELETE /api/follow/mute/22704 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "ミュートを解除しました"
}
```

### 楽観的更新

1. Set `isMuted` to `false`
2. Remove user from local mute list cache
3. Re-include the user's posts in timeline rendering
4. On error, revert

---

## フォロワーを削除

フォロワーリストからユーザーを強制的に削除します。対象ユーザーはあなたをフォローしなくなります。通知はされません。

```
DELETE /api/follow/follower/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                          |
|-----------|--------|----------|--------------------------------------|
| `userId`  | number | Yes      | The numeric ID of the follower to remove |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "フォロワーを削除しました"}` | Successfully removed follower |
| 400 | `{"error": "このユーザーはフォロワーではありません"}` | The specified user is not your follower |
| 404 | `{"error": "ユーザーが見つかりません"}` | User ID does not exist |

### Example

```http
DELETE /api/follow/follower/22704 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "フォロワーを削除しました"
}
```

### 楽観的更新

1. Decrement your own `followersCount` by 1
2. Set `isFollowedBy` to `false` for that user
3. Remove user from the local followers list cache
4. On error, revert

---

## ブロックリスト取得

ブロックしているすべてのユーザーを返します。

```
GET /api/follow/block
```

### Query Parameters

なし。

### Response

```json
{
  "users": [
    {
      "id": 22704,
      "username": "spammer",
      "displayName": "Spam Bot"
    }
  ]
}
```

### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `users` | array | Array of blocked user objects |
| `users[].id` | number | User ID |
| `users[].username` | string | Username |
| `users[].displayName` | string | Display name |

### Example

```http
GET /api/follow/block HTTP/1.1
Authorization: Bearer eyJ...
```

```json
{
  "users": []
}
```

> **注意:** The response returns a minimal user object with only `id`, `username`, and `displayName`. It does NOT include `avatarUrl`, `bio`, or other profile fields.

---

## ミュートリスト取得

ミュートしているすべてのユーザーを返します。

```
GET /api/follow/mute
```

### Query Parameters

なし。

### Response

```json
{
  "users": [
    {
      "id": 18179,
      "username": "noisyuser",
      "displayName": "Noisy Person"
    }
  ]
}
```

### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `users` | array | Array of muted user objects |
| `users[].id` | number | User ID |
| `users[].username` | string | Username |
| `users[].displayName` | string | Display name |

### Example

```http
GET /api/follow/mute HTTP/1.1
Authorization: Bearer eyJ...
```

```json
{
  "users": []
}
```

---

## 保留中のフォローリクエスト取得

受信してまだ承認/拒否していないフォローリクエストを返します。アカウントが非公開（`isPrivate: true`）の場合にのみ関係します。

```
GET /api/follow/requests/pending
```

### Query Parameters

なし。

### Response

```json
{
  "requests": [
    {
      "id": 4521,
      "sender": {
        "username": "newuser",
        "displayName": "New User",
        "avatarUrl": "/uploads/avatars/abc.webp",
        "bio": "Hello world!",
        "isPrivate": false,
        "officialMark": [],
        "isParodyAccount": false,
        "adminForceParody": false,
        "isBotAccount": false,
        "adminForceBot": false
      }
    }
  ]
}
```

### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `requests` | array | Array of pending follow request objects |
| `requests[].id` | number | The follow request ID (used for accept/reject) |
| `requests[].sender` | object | Information about the user who sent the request |
| `requests[].sender.username` | string | Sender's username |
| `requests[].sender.displayName` | string | Sender's display name |
| `requests[].sender.avatarUrl` | string \| null | Sender's avatar URL (relative path) |
| `requests[].sender.bio` | string | Sender's bio text |
| `requests[].sender.isPrivate` | boolean | Whether the sender's account is private |
| `requests[].sender.officialMark` | string[] | Official mark badges (e.g. `["BLUE"]`, `["PURPLE"]`) |
| `requests[].sender.isParodyAccount` | boolean | Whether the sender is a parody account |
| `requests[].sender.adminForceParody` | boolean | Whether an admin forced the parody flag |
| `requests[].sender.isBotAccount` | boolean | Whether the sender is a bot account |
| `requests[].sender.adminForceBot` | boolean | Whether an admin forced the bot flag |

### Example

```http
GET /api/follow/requests/pending HTTP/1.1
Authorization: Bearer eyJ...
```

```json
{
  "requests": []
}
```

---

## フォローリクエスト承認

保留中のフォローリクエストを承認し、送信者があなたをフォローできるようにします。

```
POST /api/follow/requests/:requestId/accept
```

### Path Parameters

| Parameter | Type   | Required | Description                        |
|-----------|--------|----------|------------------------------------|
| `requestId` | number | Yes   | The follow request ID from the pending requests list |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "フォローリクエストを承認しました"}` | Request accepted |
| 400 | `{"error": "リクエストが見つかりません"}` | Invalid request ID |
| 403 | `{"error": "権限がありません"}` | Not your follow request to accept |

### Example

```http
POST /api/follow/requests/4521/accept HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "フォローリクエストを承認しました"
}
```

### 楽観的更新

1. Remove the request from the local pending requests list
2. Increment your `followersCount` by 1
3. The sender's `followingCount` will increase by 1 on their side
4. On error, restore the request to the pending list

---

## フォローリクエスト拒否

保留中のフォローリクエストを拒否します。送信者には拒否されたことは通知されません（保留状態から静かに消えます）。

```
POST /api/follow/requests/:requestId/reject
```

### Path Parameters

| Parameter | Type   | Required | Description                        |
|-----------|--------|----------|------------------------------------|
| `requestId` | number | Yes   | The follow request ID from the pending requests list |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "フォローリクエストを拒否しました"}` | Request rejected |
| 400 | `{"error": "リクエストが見つかりません"}` | Invalid request ID |
| 403 | `{"error": "権限がありません"}` | Not your follow request to reject |

### Example

```http
POST /api/follow/requests/4521/reject HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "フォローリクエストを拒否しました"
}
```

### 楽観的更新

1. Remove the request from the local pending requests list
2. On error, restore the request to the pending list

---

## 投稿通知を有効化

フォローしている特定のユーザーの新規投稿に対するプッシュ通知を有効にします。

```
POST /api/follow/:userId/post-notify
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the target user |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "投稿通知をオンにしました"}` | Post notifications enabled |
| 400 | `{"error": "フォローしていません"}` | Not following this user |

### Example

```http
POST /api/follow/18179/post-notify HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "投稿通知をオンにしました"
}
```

---

## 投稿通知を無効化

特定のユーザーの新規投稿に対するプッシュ通知を無効にします。

```
DELETE /api/follow/:userId/post-notify
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the target user |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "投稿通知をオフにしました"}` | Post notifications disabled |

### Example

```http
DELETE /api/follow/18179/post-notify HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "投稿通知をオフにしました"
}
```

---

## リカロットを非表示

タイムラインで特定のユーザーのリカロット（リポスト）を非表示にします。オリジナルの投稿は表示されますが、リカロットは表示されません。

```
POST /api/follow/hide-rekarots/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the target user |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "リカロットを非表示にしました"}` | Rekarots hidden |

### Example

```http
POST /api/follow/hide-rekarots/18179 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "リカロットを非表示にしました"
}
```

---

## リカロットの非表示を解除

タイムラインで特定のユーザーのリカロットを再表示します。

```
DELETE /api/follow/hide-rekarots/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the target user |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "リカロットの非表示を解除しました"}` | Rekarots unhidden |

### Example

```http
DELETE /api/follow/hide-rekarots/18179 HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

```json
{
  "message": "リカロットの非表示を解除しました"
}
```

---

## 楽観的更新ロジック

Karotterフロントエンドは、即座のUIフィードバックを提供するためにすべてのフォロー関連操作に楽観的更新を使用しています。一般的なパターンは以下の通りです:

### パターン

```
1. User triggers action (click follow/block/mute button)
2. UI immediately updates to reflect the new state
3. API request is sent in the background
4. On success: no further action needed (UI is already correct)
5. On error: revert UI to previous state and show error toast
```

### 影響を受ける状態フィールド

| アクション | 変更されるフィールド |
|--------|---------------|
| Follow | `isFollowing` -> `true`, target `followersCount` +1, self `followingCount` +1 |
| Follow (private) | `hasPendingRequest` -> `true` |
| Unfollow | `isFollowing` -> `false`, target `followersCount` -1, self `followingCount` -1 |
| Block | `isBlocked` -> `true`, `isFollowing` -> `false`, `isFollowedBy` -> `false` |
| Unblock | `isBlocked` -> `false` |
| Mute | `isMuted` -> `true` |
| Unmute | `isMuted` -> `false` |
| Remove Follower | `isFollowedBy` -> `false`, self `followersCount` -1 |
| Accept Request | Remove from pending list, self `followersCount` +1 |
| Reject Request | Remove from pending list |
| Hide Rekarots | `hideRekarots` -> `true` for that user |
| Unhide Rekarots | `hideRekarots` -> `false` for that user |
| Post Notify On | `postNotify` -> `true` for that user |
| Post Notify Off | `postNotify` -> `false` for that user |

### レート制限

すべてのフォローエンドポイントはデフォルトのレート制限（**60秒あたり100リクエスト**）を共有しています。

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
