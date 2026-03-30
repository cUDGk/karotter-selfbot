# Follow API

> **Base URL:** `https://karotter.com/api`
>
> All endpoints require authentication via `Authorization: Bearer {token}` header and `X-CSRF-Token` header.
> Cookie-based authentication (`karotter_at`, `karotter_rt`, `karotter_csrf`) is also supported and may be required for some operations.

---

## Table of Contents

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
- [Optimistic Update Logic](#optimistic-update-logic)

---

## Follow a User

Follows the specified user. If the target user has a private account, this sends a follow request instead of immediately following.

```
POST /api/follow/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the target user |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

When the client sends a follow request, the UI should:
1. Immediately increment the target user's `followersCount` by 1
2. Set `isFollowing` to `true` on the user relationship object
3. Increment the authenticated user's `followingCount` by 1
4. If the target is private, set `hasPendingRequest` to `true` instead of `isFollowing`
5. On error, revert all changes

---

## Unfollow a User

Removes the follow relationship with the specified user.

```
DELETE /api/follow/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the target user |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

1. Immediately decrement the target user's `followersCount` by 1
2. Set `isFollowing` to `false`
3. Decrement the authenticated user's `followingCount` by 1
4. On error, revert all changes

---

## Block a User

Blocks the specified user. Blocking automatically unfollows both directions (you unfollow them, they unfollow you).

```
POST /api/follow/block/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the user to block |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

1. Set `isBlocked` to `true` on the user relationship
2. Set `isFollowing` to `false` (auto-unfollow)
3. Set `isFollowedBy` to `false` (they are auto-unfollowed)
4. Decrement both `followersCount` and `followingCount` as appropriate
5. Hide posts from this user in timeline/search
6. On error, revert all changes

---

## Unblock a User

Removes the block on the specified user. Does NOT restore any previous follow relationship.

```
DELETE /api/follow/block/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the user to unblock |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

1. Set `isBlocked` to `false`
2. Remove the user from the local block list cache
3. Show posts from this user again in timeline/search
4. On error, revert

---

## Mute a User

Mutes the specified user. Their posts will be hidden from your timeline but you remain following them (if you were).

```
POST /api/follow/mute/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the user to mute |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

1. Set `isMuted` to `true` on the user relationship
2. Add user to local mute list cache
3. Filter the user's posts from timeline rendering
4. On error, revert

---

## Unmute a User

Removes the mute on the specified user.

```
DELETE /api/follow/mute/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                     |
|-----------|--------|----------|---------------------------------|
| `userId`  | number | Yes      | The numeric ID of the user to unmute |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

1. Set `isMuted` to `false`
2. Remove user from local mute list cache
3. Re-include the user's posts in timeline rendering
4. On error, revert

---

## Remove a Follower

Forcibly removes a user from your followers list. The target user will no longer follow you. They are NOT notified.

```
DELETE /api/follow/follower/:userId
```

### Path Parameters

| Parameter | Type   | Required | Description                          |
|-----------|--------|----------|--------------------------------------|
| `userId`  | number | Yes      | The numeric ID of the follower to remove |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

1. Decrement your own `followersCount` by 1
2. Set `isFollowedBy` to `false` for that user
3. Remove user from the local followers list cache
4. On error, revert

---

## Get Block List

Returns all users you have blocked.

```
GET /api/follow/block
```

### Query Parameters

None.

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

### Response Fields

| Field | Type | Description |
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

> **Note:** The response returns a minimal user object with only `id`, `username`, and `displayName`. It does NOT include `avatarUrl`, `bio`, or other profile fields.

---

## Get Mute List

Returns all users you have muted.

```
GET /api/follow/mute
```

### Query Parameters

None.

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

### Response Fields

| Field | Type | Description |
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

## Get Pending Follow Requests

Returns follow requests that you have received and have not yet accepted or rejected. Only relevant if your account is set to private (`isPrivate: true`).

```
GET /api/follow/requests/pending
```

### Query Parameters

None.

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

### Response Fields

| Field | Type | Description |
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

## Accept Follow Request

Accepts a pending follow request, allowing the sender to follow you.

```
POST /api/follow/requests/:requestId/accept
```

### Path Parameters

| Parameter | Type   | Required | Description                        |
|-----------|--------|----------|------------------------------------|
| `requestId` | number | Yes   | The follow request ID from the pending requests list |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

1. Remove the request from the local pending requests list
2. Increment your `followersCount` by 1
3. The sender's `followingCount` will increase by 1 on their side
4. On error, restore the request to the pending list

---

## Reject Follow Request

Rejects a pending follow request. The sender is NOT notified that their request was rejected (it silently disappears from their pending state).

```
POST /api/follow/requests/:requestId/reject
```

### Path Parameters

| Parameter | Type   | Required | Description                        |
|-----------|--------|----------|------------------------------------|
| `requestId` | number | Yes   | The follow request ID from the pending requests list |

### Request Body

None.

### Responses

| Status | Body | Description |
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

### Optimistic Update

1. Remove the request from the local pending requests list
2. On error, restore the request to the pending list

---

## Optimistic Update Logic

The Karotter frontend uses optimistic updates for all follow-related operations to provide instant UI feedback. The general pattern is:

### Pattern

```
1. User triggers action (click follow/block/mute button)
2. UI immediately updates to reflect the new state
3. API request is sent in the background
4. On success: no further action needed (UI is already correct)
5. On error: revert UI to previous state and show error toast
```

### State Fields Affected

| Action | Fields Changed |
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

### Rate Limiting

All follow endpoints share the default rate limit of **100 requests per 60 seconds**.

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
