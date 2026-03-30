# Search

Complete reference for search endpoints, trending topics, discovery feeds, and search history.

---

## Universal Search

### `GET /search`

Perform a combined search across posts, users, and hashtags. Returns results from all categories in a single response.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | `string` | — | Search query string (required) |
| `limit` | `number` | `12` | Maximum results per category |

**Request:**

```http
GET /search?q=karotter&limit=12 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "posts": [
    {
      "id": "post_search001",
      "content": "I love using Karotter every day!",
      "author": {
        "id": "clx_author1",
        "username": "karotter_fan",
        "displayName": "Karotter Fan",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_author1.webp"
      },
      "createdAt": "2026-03-30T08:00:00.000Z",
      "likesCount": 10,
      "repliesCount": 2,
      "rekarotsCount": 1
    }
  ],
  "users": [
    {
      "id": "clx_user_karotter",
      "username": "karotter_official",
      "displayName": "Karotter Official",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_user_karotter.webp",
      "bio": "Official Karotter account",
      "officialMark": ["VERIFIED"],
      "isParodyAccount": false,
      "isBotAccount": false
    }
  ],
  "hashtags": [
    {
      "id": "tag_karotter",
      "name": "karotter",
      "usageCount": 15420,
      "trendScore": 85.3
    }
  ],
  "pagination": {
    "hasNext": true,
    "nextCursor": "cursor_search_abc123",
    "page": 1,
    "pages": 5
  }
}
```

**Pagination Object:**

| Field | Type | Description |
|-------|------|-------------|
| `hasNext` | `boolean` | Whether more results are available |
| `nextCursor` | `string \| null` | Cursor for the next page of results |
| `page` | `number` | Current page number |
| `pages` | `number` | Total number of pages available |

---

## Search Posts

### `GET /search/posts`

Search specifically for posts with additional filtering and sorting options.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | `string` | — | Search query string (required) |
| `sort` | `string` | `latest` | Sort order (see below) |
| `hasMedia` | `boolean` | — | Filter to posts with media attachments only |
| `limit` | `number` | `12` | Results per page |
| `cursor` | `string` | — | Pagination cursor from previous response |

**Sort Values:**

| Sort | Description |
|------|-------------|
| `latest` | Most recent posts first (chronological) |
| `topics` | Sorted by topical relevance to the query |

**Request:**

```http
GET /search/posts?q=sunset&sort=latest&hasMedia=true&limit=12 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "posts": [
    {
      "id": "post_sunset001",
      "content": "Beautiful sunset today! #sunset #photography",
      "author": {
        "id": "clx_photog1",
        "username": "sunset_chaser",
        "displayName": "Sunset Chaser",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_photog1.webp"
      },
      "media": [
        {
          "id": "media_sunset001",
          "url": "https://cdn.karotter.com/media/media_sunset001.webp",
          "type": "IMAGE",
          "width": 2048,
          "height": 1536,
          "alt": "Orange and pink sunset over mountains",
          "isSpoiler": false,
          "isR18": false
        }
      ],
      "createdAt": "2026-03-30T17:30:00.000Z",
      "likesCount": 85,
      "repliesCount": 12,
      "rekarotsCount": 8,
      "viewCount": 2400
    }
  ],
  "pagination": {
    "hasNext": true,
    "nextCursor": "cursor_posts_def456"
  }
}
```

**Request (next page):**

```http
GET /search/posts?q=sunset&sort=latest&hasMedia=true&limit=12&cursor=cursor_posts_def456 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

---

## Search Users

### `GET /search/users`

Search for users by username or display name.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | `string` | — | Search query string (required) |
| `limit` | `number` | `12` | Results per page. Use `6` for suggestion dropdowns, `12` for full search results. |

**Request (suggestion mode):**

```http
GET /search/users?q=karo&limit=6 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Request (full search):**

```http
GET /search/users?q=karotter&limit=12 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_user1",
      "username": "karotter_fan",
      "displayName": "Karotter Fan",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_user1.webp",
      "bio": "I love Karotter!",
      "followersCount": 250,
      "isPrivate": false,
      "officialMark": [],
      "isParodyAccount": false,
      "isBotAccount": false
    },
    {
      "id": "clx_user2",
      "username": "karotter_dev",
      "displayName": "Karotter Developer",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_user2.webp",
      "bio": "Building cool things",
      "followersCount": 1200,
      "isPrivate": false,
      "officialMark": ["VERIFIED"],
      "isParodyAccount": false,
      "isBotAccount": false
    }
  ]
}
```

**React Query config for user search:**

| Setting | Value |
|---------|-------|
| `staleTime` | `15000` (15 seconds) |

---

## Search Hashtags

### `GET /search/hashtags`

Search for hashtags by name.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `q` | `string` | — | Hashtag search query (without the `#` prefix) |

**Request:**

```http
GET /search/hashtags?q=sunset HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "hashtags": [
    {
      "id": "tag_sunset",
      "name": "sunset",
      "usageCount": 8750,
      "trendScore": 72.5
    },
    {
      "id": "tag_sunsetphotography",
      "name": "sunsetphotography",
      "usageCount": 3200,
      "trendScore": 45.1
    },
    {
      "id": "tag_sunsetlovers",
      "name": "sunsetlovers",
      "usageCount": 1800,
      "trendScore": 28.9
    }
  ]
}
```

**Hashtag Object:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique hashtag identifier |
| `name` | `string` | Hashtag text (without `#` prefix) |
| `usageCount` | `number` | Total number of posts using this hashtag |
| `trendScore` | `number` | Trending score (higher = more trending recently) |

---

## Trending Topics

### `GET /search/trending/topics`

Get currently trending topics/hashtags.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | Maximum number of trending topics to return |

**Request:**

```http
GET /search/trending/topics?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "trends": [
    {
      "type": "HASHTAG",
      "label": "#karotter",
      "postCount": 1523,
      "token": "karotter"
    },
    {
      "type": "HASHTAG",
      "label": "#gamedev",
      "postCount": 892,
      "token": "gamedev"
    },
    {
      "type": "HASHTAG",
      "label": "#illustration",
      "postCount": 756,
      "token": "illustration"
    },
    {
      "type": "HASHTAG",
      "label": "#morningpost",
      "postCount": 634,
      "token": "morningpost"
    },
    {
      "type": "HASHTAG",
      "label": "#minecraft",
      "postCount": 521,
      "token": "minecraft"
    }
  ]
}
```

**Trend Object:**

| Field | Type | Description |
|-------|------|-------------|
| `type` | `string` | Always `HASHTAG` (currently the only supported type) |
| `label` | `string` | Display label with `#` prefix |
| `postCount` | `number` | Number of posts using this topic in the trending period |
| `token` | `string` | Raw token value (hashtag name without `#`) for use in search queries |

---

## Discover Feeds

### `GET /search/discover/latest`

Discover the latest posts from across the platform (not limited to followed users).

### `GET /search/discover/media`

Discover posts that contain media (images/videos).

### `GET /search/discover/topics`

Discover posts organized by trending topics.

**Query Parameters (all three endpoints):**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `12` | Results per page |
| `cursor` | `string` | — | Pagination cursor from previous response |

**Request:**

```http
GET /search/discover/latest?limit=12 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK` (example for `/discover/latest`):**

```json
{
  "posts": [
    {
      "id": "post_disc001",
      "content": "Discovering new content on Karotter!",
      "author": {
        "id": "clx_newauthor1",
        "username": "new_creator",
        "displayName": "New Creator",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_newauthor1.webp"
      },
      "createdAt": "2026-03-30T11:00:00.000Z",
      "likesCount": 20,
      "repliesCount": 4,
      "rekarotsCount": 2,
      "viewCount": 350
    }
  ],
  "pagination": {
    "hasNext": true,
    "nextCursor": "cursor_discover_ghi789"
  }
}
```

**Request (next page):**

```http
GET /search/discover/latest?limit=12&cursor=cursor_discover_ghi789 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK` (example for `/discover/media`):**

```json
{
  "posts": [
    {
      "id": "post_media_disc001",
      "content": "My latest artwork #art",
      "author": {
        "id": "clx_artist1",
        "username": "digital_artist",
        "displayName": "Digital Artist",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_artist1.webp"
      },
      "media": [
        {
          "id": "media_art001",
          "url": "https://cdn.karotter.com/media/media_art001.webp",
          "type": "IMAGE",
          "width": 2048,
          "height": 2048,
          "alt": "Digital painting of a forest",
          "isSpoiler": false,
          "isR18": false
        }
      ],
      "createdAt": "2026-03-30T10:00:00.000Z",
      "likesCount": 150,
      "repliesCount": 20,
      "viewCount": 3000
    }
  ],
  "pagination": {
    "hasNext": true,
    "nextCursor": "cursor_media_jkl012"
  }
}
```

---

## Search History

Search history is stored **client-side** in `localStorage`. It is not managed by the server.

**Storage Key Format:**

```
karotter-search-history-v1:{userId}
```

**Example:** For user `clx1abc2d3ef4gh5ij6kl7mn8`, the key is:

```
karotter-search-history-v1:clx1abc2d3ef4gh5ij6kl7mn8
```

**Storage Format:**

The value is a JSON-encoded array of search query strings, ordered from most recent to oldest.

```json
["karotter", "sunset photos", "gamedev", "minecraft mod"]
```

**Limits:**

| Setting | Value | Description |
|---------|-------|-------------|
| Maximum entries | `12` | Only the 12 most recent search queries are stored |

**Behavior:**
- When a new search is performed, it is added to the front of the array.
- If the query already exists in the history, it is moved to the front (deduplicated).
- If the array exceeds 12 entries, the oldest entry is removed.
- Each user ID has its own independent search history.
- Search history is per-browser/per-device since it uses `localStorage`.
- The server has no knowledge of search history; there is no API endpoint to read or write it.

**Example Implementation:**

```js
const STORAGE_KEY = `karotter-search-history-v1:${userId}`;
const MAX_HISTORY = 12;

function addSearchHistory(query) {
  const history = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
  const filtered = history.filter(h => h !== query);
  filtered.unshift(query);
  if (filtered.length > MAX_HISTORY) filtered.pop();
  localStorage.setItem(STORAGE_KEY, JSON.stringify(filtered));
}

function getSearchHistory() {
  return JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
}

function clearSearchHistory() {
  localStorage.removeItem(STORAGE_KEY);
}
```
