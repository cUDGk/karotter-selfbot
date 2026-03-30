# Admin API

Complete reference for the Karotter admin panel API. Access requires `user.isAdmin === true`.

---

## Overview

### Frontend Access

The admin panel is accessible at an obfuscated URL:

```
https://karotter.com/control-room-x9k2
```

This URL is not linked anywhere in the public UI. It is only known to administrators.

### API Prefix

All admin API endpoints are prefixed with:

```
/api/admin
```

For example: `https://api.karotter.com/api/admin/analytics`

### Authentication

Admin endpoints require:

1. A valid `Authorization: Bearer <token>` header (same as all authenticated endpoints)
2. The authenticated user must have `isAdmin === true`
3. Standard `x-csrf-token` header for mutating requests

Non-admin users receive `403 Forbidden` on all admin endpoints.

---

## Admin Panel Tabs

The admin panel frontend is organized into 8 tabs:

| Tab | Route | Description |
|-----|-------|-------------|
| Dashboard | `/control-room-x9k2` | Analytics overview |
| Users | `/control-room-x9k2/users` | User management |
| Posts | `/control-room-x9k2/posts` | Post moderation |
| Stories | `/control-room-x9k2/stories` | Story moderation |
| Recommend | `/control-room-x9k2/recommend` | Recommendation algorithm testing |
| Trending | `/control-room-x9k2/trending` | Trending algorithm testing |
| Survey | `/control-room-x9k2/survey` | Survey/poll results |
| Beta Experiment | `/control-room-x9k2/beta-experiment` | A/B testing results |

---

## Dashboard

### `GET /admin/analytics`

Retrieve platform-wide analytics summary.

**Response `200 OK`:**

```json
{
  "totalUsers": 15230,
  "totalPosts": 892451,
  "activeUsers": 3847,
  "totalReports": 156
}
```

| Field | Type | Description |
|-------|------|-------------|
| `totalUsers` | `number` | Total registered users on the platform |
| `totalPosts` | `number` | Total posts ever created |
| `activeUsers` | `number` | Users active in the recent period |
| `totalReports` | `number` | Total pending/active reports |

---

## User Management

### `GET /admin/users`

List users with search and cursor-based pagination.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | Number of users per page (max 100) |
| `search` | `string` | — | Search by username (prefix with `@` to search by exact username) |
| `cursor` | `string` | — | Pagination cursor from previous response |

**Request:**

```http
GET /admin/users?limit=30&search=@testuser HTTP/1.1
```

**Response `200 OK`:**

```json
{
  "users": [
    {
      "id": 12345,
      "username": "testuser",
      "displayName": "Test User",
      "email": "test@example.com",
      "emailVerified": true,
      "officialMark": ["BLUE"],
      "isParodyAccount": false,
      "isBotAccount": false,
      "adminForceHidden": false,
      "adminForceParody": false,
      "adminForceBot": false,
      "showBotAccounts": true,
      "showHiddenPosts": false,
      "showR18Content": false,
      "hideProfileFromMinors": false,
      "isBanned": false,
      "isRestricted": false,
      "followersCount": 1520,
      "postsCount": 4830
    }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6MTIzNDV9",
    "total": 15230
  }
}
```

**User Object Fields (Admin View):**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `number` | User ID |
| `username` | `string` | Username |
| `displayName` | `string` | Display name |
| `email` | `string` | Email address |
| `emailVerified` | `boolean` | Whether email is verified |
| `officialMark` | `string[]` | Array of official mark color codes |
| `isParodyAccount` | `boolean` | Self-declared parody account |
| `isBotAccount` | `boolean` | Self-declared bot account |
| `adminForceHidden` | `boolean` | Admin-forced profile hiding |
| `adminForceParody` | `boolean` | Admin-forced parody label |
| `adminForceBot` | `boolean` | Admin-forced bot label |
| `showBotAccounts` | `boolean` | User preference: show bot accounts |
| `showHiddenPosts` | `boolean` | User preference: show hidden posts |
| `showR18Content` | `boolean` | User preference: show R18 content |
| `hideProfileFromMinors` | `boolean` | Whether profile is hidden from minors |
| `isBanned` | `boolean` | Whether the user is banned |
| `isRestricted` | `boolean` | Whether the user is restricted |
| `followersCount` | `number` | Follower count |
| `postsCount` | `number` | Post count |

---

### `PATCH /admin/users/:id/ban`

Ban a user account.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | User ID to ban |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `reason` | `string` | Yes | Reason for the ban (stored for audit) |

**Request:**

```http
PATCH /admin/users/12345/ban HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "reason": "Repeated harassment violations"
}
```

**Response `200 OK`:**

```json
{
  "message": "User banned successfully"
}
```

---

### `PATCH /admin/users/:id/unban`

Unban a previously banned user account.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | User ID to unban |

**Request Body:** Empty object `{}`

**Request:**

```http
PATCH /admin/users/12345/unban HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{}
```

**Response `200 OK`:**

```json
{
  "message": "User unbanned successfully"
}
```

---

### `PATCH /admin/users/:id/account`

Edit core account fields for a user.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | User ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `username` | `string` | No | New username |
| `displayName` | `string` | No | New display name |
| `email` | `string` | No | New email address |
| `password` | `string` | No | New password (plain text, server hashes it) |
| `emailVerified` | `boolean` | No | Override email verification status |

**Request:**

```http
PATCH /admin/users/12345/account HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "username": "newusername",
  "displayName": "New Display Name",
  "email": "newemail@example.com",
  "emailVerified": true
}
```

**Response `200 OK`:**

```json
{
  "message": "User account updated",
  "user": { ... }
}
```

**Notes:**
- All fields are optional; only provided fields are updated.
- Changing the `password` invalidates all existing sessions for that user.

---

### `PATCH /admin/users/:id/official-mark`

Set or update the official verification marks for a user.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | User ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `officialMark` | `string[]` | Yes | Array of mark color codes. Pass `[]` to remove all marks. |

**Important:** The `officialMark` field is an **array**. A user can have multiple marks simultaneously (e.g., `["BLUE", "PURPLE"]` for a verified admin).

**Request:**

```http
PATCH /admin/users/12345/official-mark HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "officialMark": ["BLUE", "PURPLE"]
}
```

**Response `200 OK`:**

```json
{
  "message": "Official mark updated",
  "officialMark": ["BLUE", "PURPLE"]
}
```

**Available Mark Colors:**

| Code | Hex | Meaning |
|------|-----|---------|
| `BLUE` | `#1d9bf0` | Verified individual (本人確認) |
| `YELLOW` | `#f8c500` | Organization/group (団体・企業) |
| `ORANGE` | `#ff7a00` | Orange mark |
| `PURPLE` | `#8b5cf6` | Platform staff/admin (運営) |
| `GRAY` | `#9ca3af` | Government/official body (政府・公的機関) |
| `BLACK` | `#111827` | Black mark |
| `RED` | `#ef4444` | Red mark |
| `GREEN` | `#22c55e` | Green mark |

---

### `PATCH /admin/users/:id/flags`

Toggle moderation flags on a user account.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | User ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `isParodyAccount` | `boolean` | No | Self-declared parody flag |
| `adminForceHidden` | `boolean` | No | Force-hide the user's profile and posts |
| `adminForceParody` | `boolean` | No | Force the parody label on the user |
| `isBotAccount` | `boolean` | No | Self-declared bot flag |
| `adminForceBot` | `boolean` | No | Force the bot label on the user |
| `showBotAccounts` | `boolean` | No | Override user's bot visibility preference |
| `showHiddenPosts` | `boolean` | No | Override user's hidden posts preference |
| `showR18Content` | `boolean` | No | Override user's R18 content preference |
| `hideProfileFromMinors` | `boolean` | No | Force hide profile from minor accounts |

**Request:**

```http
PATCH /admin/users/12345/flags HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "adminForceHidden": true,
  "adminForceBot": true
}
```

**Response `200 OK`:**

```json
{
  "message": "User flags updated"
}
```

---

### `DELETE /admin/users/:id`

Permanently delete a user account and all associated data.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | User ID to delete |

**Request:**

```http
DELETE /admin/users/12345 HTTP/1.1
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>
```

**Response `200 OK`:**

```json
{
  "message": "User deleted successfully"
}
```

**Warning:** This action is irreversible. All posts, DMs, reactions, and other data associated with the user are permanently removed.

---

## Post Management

### `GET /admin/posts`

List posts with search and cursor-based pagination.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | Number of posts per page |
| `search` | `string` | — | Search by post content |
| `cursor` | `string` | — | Pagination cursor |

**Request:**

```http
GET /admin/posts?limit=30&search=keyword HTTP/1.1
```

**Response `200 OK`:**

```json
{
  "posts": [
    {
      "id": 98765,
      "content": "This is a post containing keyword",
      "author": {
        "username": "someuser",
        "displayName": "Some User",
        "officialMark": []
      },
      "viewsCount": 1250,
      "likesCount": 42,
      "rekarotsCount": 8,
      "repliesCount": 15,
      "isR18": false,
      "adminForceR18": false,
      "adminForceHidden": false
    }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6OTg3NjV9",
    "total": 892451
  }
}
```

**Post Object Fields (Admin View):**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `number` | Post ID |
| `content` | `string` | Post text content |
| `author.username` | `string` | Author's username |
| `author.displayName` | `string` | Author's display name |
| `author.officialMark` | `string[]` | Author's official marks |
| `viewsCount` | `number` | Total views |
| `likesCount` | `number` | Total likes |
| `rekarotsCount` | `number` | Total reposts |
| `repliesCount` | `number` | Total replies |
| `isR18` | `boolean` | Whether the post is marked R18 by the author |
| `adminForceR18` | `boolean` | Whether an admin has force-flagged the post as R18 |
| `adminForceHidden` | `boolean` | Whether an admin has hidden the post |

---

### `PATCH /admin/posts/:id/flags`

Toggle moderation flags on a post.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | Post ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `adminForceR18` | `boolean` | No | Force the post to be marked as R18/NSFW |
| `adminForceHidden` | `boolean` | No | Force-hide the post from all timelines |

**Request:**

```http
PATCH /admin/posts/98765/flags HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "adminForceR18": true,
  "adminForceHidden": false
}
```

**Response `200 OK`:**

```json
{
  "message": "Post flags updated"
}
```

---

### `DELETE /admin/posts/:id`

Permanently delete a post.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | Post ID to delete |

**Request:**

```http
DELETE /admin/posts/98765 HTTP/1.1
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>
```

**Response `200 OK`:**

```json
{
  "message": "Post deleted successfully"
}
```

---

## Story Management

### `GET /admin/stories`

List stories for moderation.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | Number of stories per page |
| `cursor` | `string` | — | Pagination cursor |

**Response `200 OK`:**

```json
{
  "stories": [
    {
      "id": 5678,
      "author": {
        "username": "storyuser",
        "displayName": "Story User",
        "officialMark": []
      },
      "mediaUrl": "/uploads/stories/abc123.jpg",
      "mediaType": "image",
      "caption": "Check this out!",
      "viewsCount": 340,
      "adminForceR18": false,
      "adminForceHidden": false,
      "createdAt": "2026-03-30T10:00:00.000Z",
      "expiresAt": "2026-03-31T10:00:00.000Z"
    }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6NTY3OH0",
    "total": 1250
  }
}
```

---

### `PATCH /admin/stories/:id/flags`

Toggle moderation flags on a story.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `number` | Story ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `adminForceR18` | `boolean` | No | Force-flag the story as R18 |
| `adminForceHidden` | `boolean` | No | Force-hide the story |

**Request:**

```http
PATCH /admin/stories/5678/flags HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "adminForceR18": true
}
```

**Response `200 OK`:**

```json
{
  "message": "Story flags updated"
}
```

---

### `DELETE /admin/stories/:id`

Permanently delete a story.

**Request:**

```http
DELETE /admin/stories/5678 HTTP/1.1
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>
```

**Response `200 OK`:**

```json
{
  "message": "Story deleted successfully"
}
```

---

## Recommendation Testing

### `GET /admin/test-recommend`

Test the recommendation algorithm with full scoring details. This endpoint has an extended timeout due to heavy computation.

**Timeout:** 60 seconds

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `userId` | `number` | — | Optional: test recommendations for a specific user ID |
| `limit` | `number` | `30` | Number of recommended posts to return |

**Request:**

```http
GET /admin/test-recommend?userId=12345&limit=30 HTTP/1.1
Authorization: Bearer <admin-token>
```

**Response `200 OK`:**

```json
{
  "results": [
    {
      "post": {
        "id": 98765,
        "content": "Recommended post content...",
        "author": {
          "id": 67890,
          "username": "popularuser",
          "displayName": "Popular User"
        },
        "likesCount": 245,
        "rekarotsCount": 32,
        "repliesCount": 18,
        "viewsCount": 5420,
        "createdAt": "2026-03-29T14:00:00.000Z"
      },
      "scores": {
        "engagement": 0.85,
        "freshness": 0.72,
        "authorAffinity": 0.65,
        "graph": 0.58,
        "socialProof": 0.43,
        "tagAffinity": 0.31,
        "inNetwork": 0.90,
        "totalRecommend": 0.74
      }
    }
  ],
  "computedAt": "2026-03-30T12:00:00.000Z",
  "computationTimeMs": 4520
}
```

**Score Fields:**

| Score | Range | Description |
|-------|-------|-------------|
| `engagement` | 0.0 - 1.0 | How much engagement the post has received (likes, reposts, replies) |
| `freshness` | 0.0 - 1.0 | Time decay score (newer = higher) |
| `authorAffinity` | 0.0 - 1.0 | How much the target user interacts with this author |
| `graph` | 0.0 - 1.0 | Social graph proximity (mutual follows, follow chains) |
| `socialProof` | 0.0 - 1.0 | Whether users the target follows have engaged with the post |
| `tagAffinity` | 0.0 - 1.0 | Match between post topics/tags and user's interests |
| `inNetwork` | 0.0 - 1.0 | Whether the post is from someone in the user's direct network |
| `totalRecommend` | 0.0 - 1.0 | Final weighted recommendation score |

---

## Trending Testing

### `GET /admin/test-trending`

Test the trending algorithm with detailed window statistics.

**Timeout:** 60 seconds

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | Number of trending posts to return |

**Request:**

```http
GET /admin/test-trending?limit=30 HTTP/1.1
Authorization: Bearer <admin-token>
```

**Response `200 OK`:**

```json
{
  "results": [
    {
      "post": {
        "id": 87654,
        "content": "This post is trending!",
        "author": {
          "id": 11111,
          "username": "trendinguser",
          "displayName": "Trending User"
        },
        "likesCount": 892,
        "rekarotsCount": 156,
        "repliesCount": 234,
        "viewsCount": 28500,
        "createdAt": "2026-03-30T06:00:00.000Z"
      },
      "windowStats": {
        "likes1h": 145,
        "likes6h": 520,
        "likes24h": 892,
        "rekarots1h": 28,
        "rekarots6h": 98,
        "rekarots24h": 156,
        "replies1h": 45,
        "replies6h": 142,
        "replies24h": 234,
        "uniqueActors1h": 180,
        "uniqueActors6h": 610,
        "uniqueActors24h": 1050
      },
      "trendingScore": 0.94
    }
  ],
  "computedAt": "2026-03-30T12:00:00.000Z"
}
```

**Window Stats Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `likes1h` | `number` | Likes received in the last 1 hour |
| `likes6h` | `number` | Likes received in the last 6 hours |
| `likes24h` | `number` | Likes received in the last 24 hours |
| `rekarots1h` | `number` | Reposts in the last 1 hour |
| `rekarots6h` | `number` | Reposts in the last 6 hours |
| `rekarots24h` | `number` | Reposts in the last 24 hours |
| `replies1h` | `number` | Replies in the last 1 hour |
| `replies6h` | `number` | Replies in the last 6 hours |
| `replies24h` | `number` | Replies in the last 24 hours |
| `uniqueActors1h` | `number` | Unique users who interacted in last 1 hour |
| `uniqueActors6h` | `number` | Unique users who interacted in last 6 hours |
| `uniqueActors24h` | `number` | Unique users who interacted in last 24 hours |

---

## Survey Results

### `GET /admin/survey-results`

Retrieve aggregated survey/poll results across the platform.

**Timeout:** 30 seconds

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `10` | Number of surveys to return |

**Request:**

```http
GET /admin/survey-results?limit=10 HTTP/1.1
Authorization: Bearer <admin-token>
```

**Response `200 OK`:**

```json
{
  "surveys": [
    {
      "postId": 76543,
      "isActive": true,
      "totalVotes": 1250,
      "satisfactionScore": 78.5,
      "options": [
        {
          "text": "Very satisfied",
          "votes": 580,
          "percentage": 46.4
        },
        {
          "text": "Somewhat satisfied",
          "votes": 402,
          "percentage": 32.2
        },
        {
          "text": "Neutral",
          "votes": 150,
          "percentage": 12.0
        },
        {
          "text": "Dissatisfied",
          "votes": 118,
          "percentage": 9.4
        }
      ]
    }
  ]
}
```

---

## Beta Experiment Results

### `GET /admin/beta-experiment`

Retrieve A/B testing experiment results with per-variant statistics.

**Request:**

```http
GET /admin/beta-experiment HTTP/1.1
Authorization: Bearer <admin-token>
```

**Response `200 OK`:**

```json
{
  "variants": [
    {
      "variant": "A",
      "label": "Control (current design)",
      "uniqueUsers": 5000,
      "impressions": 125000,
      "survey": {
        "preferBeta": 1200,
        "preferCurrent": 2800
      }
    },
    {
      "variant": "B",
      "label": "New timeline layout",
      "uniqueUsers": 5000,
      "impressions": 118000,
      "survey": {
        "preferBeta": 3100,
        "preferCurrent": 1400
      }
    },
    {
      "variant": "C",
      "label": "Card-style posts",
      "uniqueUsers": 5000,
      "impressions": 121000,
      "survey": {
        "preferBeta": 2800,
        "preferCurrent": 1600
      }
    },
    {
      "variant": "D",
      "label": "Compact view",
      "uniqueUsers": 5000,
      "impressions": 115000,
      "survey": {
        "preferBeta": 2200,
        "preferCurrent": 2300
      }
    },
    {
      "variant": "E",
      "label": "Media-first layout",
      "uniqueUsers": 5000,
      "impressions": 120000,
      "survey": {
        "preferBeta": 3400,
        "preferCurrent": 1100
      }
    }
  ]
}
```

**Variant Object Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `variant` | `string` | Variant identifier: `"A"`, `"B"`, `"C"`, `"D"`, or `"E"` |
| `label` | `string` | Human-readable description of the variant |
| `uniqueUsers` | `number` | Number of unique users assigned to this variant |
| `impressions` | `number` | Total post impressions shown under this variant |
| `survey.preferBeta` | `number` | Users who preferred the beta/new variant in the survey |
| `survey.preferCurrent` | `number` | Users who preferred the current/control variant |

---

## Official Mark Color Reference

For quick reference, the complete official mark color palette:

| Code | Hex Color | CSS Preview | Japanese Label | Typical Usage |
|------|-----------|-------------|----------------|---------------|
| `BLUE` | `#1d9bf0` | ![#1d9bf0](https://via.placeholder.com/12/1d9bf0/1d9bf0) | 本人確認 | Verified individual identity |
| `YELLOW` | `#f8c500` | ![#f8c500](https://via.placeholder.com/12/f8c500/f8c500) | 団体・企業 | Organization or company |
| `ORANGE` | `#ff7a00` | ![#ff7a00](https://via.placeholder.com/12/ff7a00/ff7a00) | オレンジ | Special designation |
| `PURPLE` | `#8b5cf6` | ![#8b5cf6](https://via.placeholder.com/12/8b5cf6/8b5cf6) | 運営 | Platform staff / admin |
| `GRAY` | `#9ca3af` | ![#9ca3af](https://via.placeholder.com/12/9ca3af/9ca3af) | 政府・公的機関 | Government / official body |
| `BLACK` | `#111827` | ![#111827](https://via.placeholder.com/12/111827/111827) | ブラック | Special designation |
| `RED` | `#ef4444` | ![#ef4444](https://via.placeholder.com/12/ef4444/ef4444) | レッド | Special designation |
| `GREEN` | `#22c55e` | ![#22c55e](https://via.placeholder.com/12/22c55e/22c55e) | グリーン | Special designation |
