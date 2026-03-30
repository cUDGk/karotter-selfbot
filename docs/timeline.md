# Timeline

Complete reference for timeline feeds, recommended posts, list feeds, beta A/B testing, and feed configuration.

---

## Timeline Feed

### `GET /posts/timeline`

Retrieve the user's main timeline (posts from followed users).

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `number` | `1` | Page number (1-indexed) |
| `limit` | `number` | `12` | Results per page |
| `mode` | `string` | `latest` | Feed sorting mode (see below) |

**Mode Values:**

| Mode | Description |
|------|-------------|
| `latest` | Chronological order (newest first) |
| `ranked` | Algorithmically ranked by engagement and relevance |
| `trending` | Currently trending posts |
| `following` | Posts from users you follow (alias for the default behavior) |

**Request:**

```http
GET /posts/timeline?page=1&limit=12&mode=latest HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "posts": [
    {
      "id": "post_tl001",
      "content": "Good morning everyone!",
      "author": {
        "id": "clx_friend1",
        "username": "morning_person",
        "displayName": "Morning Person",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_friend1.webp",
        "officialMark": [],
        "isParodyAccount": false,
        "isBotAccount": false
      },
      "visibility": "PUBLIC",
      "media": [],
      "createdAt": "2026-03-30T06:00:00.000Z",
      "likesCount": 15,
      "repliesCount": 3,
      "rekarotsCount": 1,
      "viewCount": 200,
      "isLiked": false,
      "isRekaroted": false,
      "isBookmarked": false,
      "reactions": []
    },
    {
      "id": "post_tl002",
      "content": "Check out this sunset!",
      "author": {
        "id": "clx_friend2",
        "username": "photographer",
        "displayName": "Photographer",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_friend2.webp",
        "officialMark": ["VERIFIED"],
        "isParodyAccount": false,
        "isBotAccount": false
      },
      "visibility": "PUBLIC",
      "media": [
        {
          "id": "media_sunset01",
          "url": "https://cdn.karotter.com/media/media_sunset01.webp",
          "type": "IMAGE",
          "width": 1920,
          "height": 1080,
          "alt": "Beautiful sunset over the ocean",
          "isSpoiler": false,
          "isR18": false
        }
      ],
      "createdAt": "2026-03-30T05:30:00.000Z",
      "likesCount": 42,
      "repliesCount": 7,
      "rekarotsCount": 5,
      "viewCount": 580,
      "isLiked": true,
      "isRekaroted": false,
      "isBookmarked": true,
      "reactions": [
        { "emoji": "😍", "count": 8, "hasReacted": false },
        { "emoji": "📸", "count": 3, "hasReacted": true }
      ]
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 12
  }
}
```

---

## Recommended Feed

### `GET /posts/recommended`

Retrieve algorithmically recommended posts. Supports multiple recommendation modes and beta A/B test variants.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `12` | Results per page |
| `mode` | `string` | `algorithm` | Recommendation mode (see below) |
| `page` | `number` | — | Page-based pagination (used with some modes) |
| `cursor` | `string` | — | Cursor-based pagination (used with some modes) |

**Mode Values:**

| Mode | Description |
|------|-------------|
| `algorithm` | Standard algorithmic recommendations |
| `latest` | Latest posts from all users (not just following) |
| `beta` | Beta recommendation algorithm with A/B test variants |

**Request:**

```http
GET /posts/recommended?limit=12&mode=beta HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "posts": [
    {
      "id": "post_rec001",
      "content": "Trending post from someone you don't follow",
      "author": {
        "id": "clx_stranger1",
        "username": "trending_user",
        "displayName": "Trending User",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_stranger1.webp"
      },
      "createdAt": "2026-03-30T04:00:00.000Z",
      "likesCount": 250,
      "repliesCount": 45,
      "rekarotsCount": 30,
      "viewCount": 5000
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 12
  },
  "betaVariant": "C"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `posts` | `Post[]` | Array of recommended post objects |
| `pagination` | `object` | Pagination info (`page`, `limit`) |
| `betaVariant` | `string \| undefined` | The A/B test variant assigned (only present in `beta` mode) |

---

## Beta A/B Test Variants

When `mode=beta`, users are assigned to one of 5 variants based on their user ID. The assignment formula is:

```
variant = ["A", "B", "C", "D", "E"][userId % 5]
```

Each variant uses a different scoring algorithm to rank posts:

| Variant | Strategy | Boost Score | Description |
|---------|----------|-------------|-------------|
| **A** | Freshness priority | +22 | Heavily favors recent posts. Newer posts get a significant scoring boost. |
| **B** | Quality | +8 | Favors posts with high engagement quality (like-to-view ratio, meaningful replies). |
| **C** | Balanced | +14 | Balanced mix of freshness and quality signals. |
| **D** | Conversation | +10 | Prioritizes posts that generate discussions (high reply counts, reply chains). |
| **E** | Discovery | +12 | Promotes posts from users outside the viewer's normal social graph. |

**Common rules across all variants:**
- **1-author-1-post diversity**: Each page of results will contain at most one post per author. This prevents a single prolific user from dominating the feed.

### `POST /posts/feedback/beta-survey`

Submit user feedback on the beta recommendation algorithm.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `preference` | `string` | Yes | Either `current` (prefers the standard algorithm) or `beta` (prefers the beta variant) |
| `variant` | `string` | Yes | The beta variant the user was shown (A-E) |

**Request:**

```http
POST /posts/feedback/beta-survey HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "preference": "beta",
  "variant": "C"
}
```

**Response `200 OK`:**

```json
{
  "message": "Feedback submitted"
}
```

---

## List Feed

### `GET /social/lists/:id/posts`

Get posts from a specific user list. Returns a single page of results with no pagination support.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `string` | The list ID |

**Request:**

```http
GET /social/lists/list_abc123/posts HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "posts": [
    {
      "id": "post_list001",
      "content": "Post from a list member",
      "author": {
        "id": "clx_listmember1",
        "username": "list_member",
        "displayName": "List Member",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_listmember1.webp"
      },
      "createdAt": "2026-03-30T07:00:00.000Z",
      "likesCount": 5,
      "repliesCount": 1
    }
  ]
}
```

> **Note:** This endpoint returns a **single page** of results. There is no pagination; you receive all available posts in one response.

---

## Tab Structure

The client organizes feeds into tabs with the following structure:

### Following Tab

| Sub-tab | Endpoint | Mode |
|---------|----------|------|
| Latest | `GET /posts/timeline` | `mode=latest` |
| Ranked | `GET /posts/timeline` | `mode=ranked` |

### Recommended Tab

| Sub-tab | Endpoint | Mode |
|---------|----------|------|
| Algorithm | `GET /posts/recommended` | `mode=algorithm` |
| Latest | `GET /posts/recommended` | `mode=latest` |
| Beta | `GET /posts/recommended` | `mode=beta` |

### List Tab

| Selection | Endpoint |
|-----------|----------|
| Specific list (by `listId`) | `GET /social/lists/:listId/posts` |

---

## React Query Configuration

The official client uses React Query (TanStack Query) with the following settings for timeline feeds:

| Setting | Value | Description |
|---------|-------|-------------|
| `refetchInterval` | `30000` (30 seconds) | Automatically refetch the feed every 30 seconds |
| `staleTime` | `10000` (10 seconds) | Data is considered fresh for 10 seconds after fetch |
| `retry` | `2` | Retry failed requests up to 2 times |

---

## Batch Views

The client automatically reports post views as the user scrolls through the timeline:

| Setting | Value | Description |
|---------|-------|-------------|
| Endpoint | `POST /posts/batch-views` | See [Posts > Batch Views](./posts.md#batch-views) |
| Interval | Every **5 seconds** | Accumulated post IDs are flushed every 5s |
| Dwell time | **1 second** | Post must be visible for at least 1s to count |
| Visibility threshold | **50%** | At least 50% of the post must be in the viewport |

The batch views payload is:

```json
{
  "postIds": ["post_tl001", "post_tl002", "post_tl003"]
}
```
