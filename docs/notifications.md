# Notifications API

> **Base URL:** `https://karotter.com/api`
>
> All endpoints require authentication via `Authorization: Bearer {token}` header and `X-CSRF-Token` header.
> Cookie-based authentication (`karotter_at`, `karotter_rt`, `karotter_csrf`) is required for most notification endpoints.

---

## Table of Contents

- [List Notifications](#list-notifications)
- [Get Unread Count](#get-unread-count)
- [Mark All as Read](#mark-all-as-read)
- [Get Grouped Posts](#get-grouped-posts)
- [Register Push Notifications](#register-push-notifications)
- [Unregister Push Notifications](#unregister-push-notifications)
- [Firebase Configuration](#firebase-configuration)
- [Notification Types](#notification-types)
- [Object Reference: Notification](#object-reference-notification)

---

## List Notifications

Returns a paginated list of notifications for the authenticated user. Notifications are grouped by type and context (e.g., multiple likes on the same post are merged into one notification).

```
GET /api/notifications
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number (1-indexed) |
| `limit` | number | 15 | Notifications per page |

### Response

```json
{
  "notifications": [
    {
      "id": 439345,
      "groupKey": "REPLY:119126:2026-03-28T23:04:01.685Z",
      "type": "REPLY",
      "message": null,
      "userId": 15459,
      "actorId": 22704,
      "actor": {
        "id": 22704,
        "username": "someuser",
        "displayName": "Some User",
        "avatarUrl": "/uploads/avatars/xyz.webp",
        "avatarFrameId": null,
        "officialMark": [],
        "isBotAccount": false,
        "isParodyAccount": false,
        "adminForceBot": false,
        "adminForceParody": false,
        "isPrivate": false
      },
      "actors": [
        {"id": 22704, "username": "someuser", "displayName": "Some User", "...": "..."}
      ],
      "actorCount": 1,
      "postId": 119126,
      "post": {
        "id": 119126,
        "content": "This is the post that was replied to",
        "mediaUrls": [],
        "mediaTypes": [],
        "createdAt": "2026-03-28T22:00:00.000Z",
        "author": {
          "id": 15459,
          "username": "claude",
          "displayName": "claude"
        }
      },
      "posts": [
        {"id": 119126, "content": "...", "mediaUrls": [], "mediaTypes": [], "createdAt": "...", "author": {"..."}}
      ],
      "postCount": 1,
      "likeContext": "OTHER",
      "rekarotContext": "OTHER",
      "isRead": true,
      "createdAt": "2026-03-28T23:04:01.685Z",
      "notificationIds": [439345]
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 15,
    "hasMore": true,
    "nextPage": 2
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `notifications` | Notification[] | Array of notification objects |
| `pagination.page` | number | Current page |
| `pagination.limit` | number | Items per page |
| `pagination.hasMore` | boolean | Whether more notifications exist |
| `pagination.nextPage` | number | Next page number (only present if `hasMore` is true) |

---

## Get Unread Count

Returns the number of unread notifications.

```
GET /api/notifications/unread/count
```

### Query Parameters

None.

### Response

```json
{
  "count": 5
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `count` | number | Number of unread notifications |

### Example

```http
GET /api/notifications/unread/count HTTP/1.1
Authorization: Bearer eyJ...
```

```json
{
  "count": 0
}
```

---

## Mark All as Read

Marks all notifications as read for the authenticated user.

```
PATCH /api/notifications/read-all
```

### Request Body

None.

### Response

```json
{
  "message": "全ての通知を既読にしました"
}
```

### Example

```http
PATCH /api/notifications/read-all HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

---

## Get Grouped Posts

Returns posts associated with a set of notification IDs. Used when a notification group contains multiple posts (e.g., "5 people liked your posts").

```
GET /api/notifications/grouped-posts
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `notificationIds` | string | Yes | Comma-separated list of notification IDs |

### Example

```http
GET /api/notifications/grouped-posts?notificationIds=439345,439346,439347 HTTP/1.1
Authorization: Bearer eyJ...
```

### Response

```json
{
  "posts": [
    {
      "id": 119126,
      "content": "First post that was liked",
      "mediaUrls": [],
      "mediaTypes": [],
      "createdAt": "2026-03-28T22:00:00.000Z",
      "author": {
        "id": 15459,
        "username": "claude",
        "displayName": "claude"
      }
    },
    {
      "id": 119200,
      "content": "Another liked post",
      "mediaUrls": ["/uploads/posts/img.jpg"],
      "mediaTypes": ["image"],
      "createdAt": "2026-03-28T23:00:00.000Z",
      "author": {
        "id": 15459,
        "username": "claude",
        "displayName": "claude"
      }
    }
  ]
}
```

---

## Register Push Notifications

Registers a device for Firebase Cloud Messaging (FCM) push notifications.

```
POST /api/notifications/push/register
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `token` | string | Yes | FCM device token |
| `platform` | string | Yes | Platform identifier (e.g., `"web"`) |
| `deviceId` | string | Yes | Unique device identifier |
| `endpoint` | string | No | Web Push endpoint URL (for web push subscriptions) |
| `keys` | object | No | Web Push keys |
| `keys.p256dh` | string | No | P-256 Diffie-Hellman public key |
| `keys.auth` | string | No | Authentication secret |

### Example

```http
POST /api/notifications/push/register HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "platform": "web",
  "token": "fcm-device-token-here",
  "deviceId": "unique-device-id",
  "endpoint": "https://fcm.googleapis.com/fcm/send/...",
  "keys": {
    "p256dh": "BNcRdreALRFXTkOOUH...",
    "auth": "tBHI..."
  }
}
```

### Response

```json
{
  "message": "プッシュ通知を登録しました"
}
```

---

## Unregister Push Notifications

Removes a device from push notification delivery.

```
POST /api/notifications/push/unregister
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `token` | string | No* | FCM device token |
| `deviceId` | string | No* | Device identifier |

> *At least one of `token` or `deviceId` must be provided.

### Example

```http
POST /api/notifications/push/unregister HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "token": "fcm-device-token-here",
  "deviceId": "unique-device-id"
}
```

### Response

```json
{
  "message": "プッシュ通知を解除しました"
}
```

---

## Firebase Configuration

Karotter uses Firebase Cloud Messaging for push notifications. Below is the public Firebase configuration used by the web client.

### Firebase Config

> Firebase/FCMの設定値はKarotterのフロントエンドから取得可能ですが、GitHub Secret Scanningの誤検知を避けるためここでは省略します。
> APKまたはWebアプリのソースから確認してください。

### Usage Example (JavaScript)

```javascript
import { initializeApp } from "firebase/app";
import { getMessaging, getToken } from "firebase/messaging";

const firebaseConfig = {
  apiKey: "AIzaSyCXIbraWaApMZrQ5VP5RA4fc4i0YZ-W8I8",
  projectId: "karotter-e9b8d",
  messagingSenderId: "672327254767",
  appId: "1:672327254767:web:da22193035ab880a86d11a",
};

const app = initializeApp(firebaseConfig);
const messaging = getMessaging(app);

const vapidKey = "BDB3C6i4Xa0E58vdrNk_GiFdzQXrMjOAlW32mBi8F1kowFwG3eidq0NpK82h9xUe3sbvblObkLXA6PaS0Uo4p78";

const token = await getToken(messaging, { vapidKey });
// Use this token with POST /api/notifications/push/register
```

---

## Notification Types

| Type | Description | Has `post`? | Has `actor`? |
|------|-------------|------------|-------------|
| `FOLLOW` | Someone followed you | No | Yes |
| `FOLLOW_REQUEST` | Someone sent a follow request (private account) | No | Yes |
| `LIKE` | Someone liked your post | Yes | Yes |
| `REPLY` | Someone replied to your post | Yes | Yes |
| `MENTION` | Someone mentioned you in a post | Yes | Yes |
| `REKAROT` | Someone rekarotted your post | Yes | Yes |
| `REACTION` | Someone reacted to your post with an emoji | Yes | Yes |
| `DM` | New direct message (push notification only) | No | Yes |
| `QUOTE` | Someone quoted your post | Yes | Yes |
| `REPORT_UPDATE` | Update on a report you filed | No | No |
| `SYSTEM` | System-generated notification | No | No |

### Notification Grouping

Notifications of the same type targeting the same post within a time window are grouped together. For example, if 5 people like the same post, you receive one notification with:

- `actorCount: 5`
- `actors: [user1, user2, user3, user4, user5]`
- `postCount: 1`
- `notificationIds: [id1, id2, id3, id4, id5]`

### Context Fields

| Field | Values | Description |
|-------|--------|-------------|
| `likeContext` | `"OWN_POST"`, `"REKAROTED_POST"`, `"OTHER"` | Whether the liked post is your original post or a post you rekarotted |
| `rekarotContext` | `"OWN_POST"`, `"OTHER"` | Whether the rekarotted post is your own |

---

## Object Reference: Notification

Full Notification object structure.

```json
{
  "id": 439345,
  "groupKey": "LIKE:119126:2026-03-28T23:00:00.000Z",
  "type": "LIKE",
  "message": null,
  "userId": 15459,
  "actorId": 22704,
  "actor": {
    "id": 22704,
    "username": "someuser",
    "displayName": "Some User",
    "avatarUrl": "/uploads/avatars/xyz.webp",
    "avatarFrameId": null,
    "officialMark": [],
    "isBotAccount": false,
    "isParodyAccount": false,
    "adminForceBot": false,
    "adminForceParody": false,
    "isPrivate": false
  },
  "actors": [
    {
      "id": 22704,
      "username": "someuser",
      "displayName": "Some User",
      "avatarUrl": "/uploads/avatars/xyz.webp",
      "avatarFrameId": null,
      "officialMark": [],
      "isBotAccount": false,
      "isParodyAccount": false,
      "adminForceBot": false,
      "adminForceParody": false,
      "isPrivate": false
    }
  ],
  "actorCount": 1,
  "postId": 119126,
  "post": {
    "id": 119126,
    "content": "Post content here",
    "mediaUrls": [],
    "mediaTypes": [],
    "createdAt": "2026-03-28T22:00:00.000Z",
    "author": {
      "id": 15459,
      "username": "claude",
      "displayName": "claude"
    }
  },
  "posts": [
    {
      "id": 119126,
      "content": "Post content here",
      "mediaUrls": [],
      "mediaTypes": [],
      "createdAt": "2026-03-28T22:00:00.000Z",
      "author": {
        "id": 15459,
        "username": "claude",
        "displayName": "claude"
      }
    }
  ],
  "postCount": 1,
  "likeContext": "OWN_POST",
  "rekarotContext": "OTHER",
  "isRead": false,
  "createdAt": "2026-03-28T23:04:01.685Z",
  "notificationIds": [439345]
}
```

### Notification Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `id` | number | Primary notification ID |
| `groupKey` | string | Grouping key in format `TYPE:postId:timestamp` |
| `type` | string | Notification type (see [Notification Types](#notification-types)) |
| `message` | string \| null | Custom message text (used for SYSTEM type) |
| `userId` | number | The notification recipient's user ID |
| `actorId` | number | The primary actor's user ID |
| `actor` | UserBrief | Primary actor's user object |
| `actors` | UserBrief[] | All actors in a grouped notification |
| `actorCount` | number | Total number of actors |
| `postId` | number \| null | Related post ID (null for FOLLOW, SYSTEM, etc.) |
| `post` | PostBrief \| null | Related post object |
| `posts` | PostBrief[] | All related posts in a grouped notification |
| `postCount` | number | Total number of related posts |
| `likeContext` | string | Context for like notifications: `"OWN_POST"`, `"REKAROTED_POST"`, or `"OTHER"` |
| `rekarotContext` | string | Context for rekarot notifications: `"OWN_POST"` or `"OTHER"` |
| `isRead` | boolean | Whether the notification has been read |
| `createdAt` | string | ISO 8601 creation timestamp |
| `notificationIds` | number[] | All notification IDs in this group (for use with grouped-posts endpoint) |

### PostBrief (Notification Context)

| Field | Type | Description |
|-------|------|-------------|
| `id` | number | Post ID |
| `content` | string | Post text content |
| `mediaUrls` | string[] | Media URLs (relative paths) |
| `mediaTypes` | string[] | Media types (`"image"`, `"video"`) |
| `createdAt` | string | ISO 8601 creation timestamp |
| `author` | object | Post author info |
| `author.id` | number | Author user ID |
| `author.username` | string | Author username |
| `author.displayName` | string | Author display name |

---

## Rate Limiting

All notification endpoints share the default rate limit:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
