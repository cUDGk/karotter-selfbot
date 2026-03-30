# Posts

Complete reference for creating, reading, editing, deleting posts, and all post interactions including likes, rekarots, bookmarks, reactions, polls, replies, analytics, and scheduled posts.

---

## Create Post

### `POST /posts`

Create a new post. Uses `multipart/form-data` to support media uploads.

**Request:** `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | `string` | No | Post text content. Can be empty if media is attached. |
| `media` | `File[]` | No | Media files. Use the **same field name** `media` for each file in the FormData. |
| `mediaAlts` | `string` (JSON) | No | JSON-encoded array of alt text strings, one per media item. Example: `'["Alt for image 1","Alt for image 2"]'` |
| `mediaSpoilerFlags` | `string` (JSON) | No | JSON-encoded array of booleans indicating spoiler status per media item. Example: `'[false,true]'` |
| `mediaR18Flags` | `string` (JSON) | No | JSON-encoded array of booleans indicating R18/NSFW status per media item. Example: `'[false,false]'` |
| `isAiGenerated` | `boolean` | No | Flag for AI-generated content. **Note:** This field does not work when sent via the API; it is ignored server-side. |
| `visibility` | `string` | No | Post visibility: `PUBLIC` (default) or `CIRCLE` |
| `replyRestriction` | `string` | No | Who can reply: `EVERYONE` (default), `FOLLOWING`, `MENTIONED`, `CIRCLE` |
| `parentId` | `string` | No | ID of the parent post (makes this a reply) |

**Request Example (text only):**

```http
POST /posts HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="content"

Hello Karotter! This is my first post.
------FormBoundary
Content-Disposition: form-data; name="visibility"

PUBLIC
------FormBoundary--
```

**Request Example (with images):**

```http
POST /posts HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="content"

Check out these photos!
------FormBoundary
Content-Disposition: form-data; name="media"; filename="photo1.jpg"
Content-Type: image/jpeg

(binary data)
------FormBoundary
Content-Disposition: form-data; name="media"; filename="photo2.jpg"
Content-Type: image/jpeg

(binary data)
------FormBoundary
Content-Disposition: form-data; name="mediaAlts"

["Sunset at the beach","Mountain view"]
------FormBoundary
Content-Disposition: form-data; name="mediaSpoilerFlags"

[false,false]
------FormBoundary
Content-Disposition: form-data; name="mediaR18Flags"

[false,false]
------FormBoundary--
```

**Response `201 Created`:**

```json
{
  "post": {
    "id": "post_new123abc",
    "content": "Hello Karotter! This is my first post.",
    "author": {
      "id": "clx1abc2d3ef4gh5ij6kl7mn8",
      "username": "myusername",
      "displayName": "My Display Name",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp"
    },
    "visibility": "PUBLIC",
    "replyRestriction": "EVERYONE",
    "media": [],
    "createdAt": "2026-03-30T12:00:00.000Z",
    "updatedAt": "2026-03-30T12:00:00.000Z",
    "likesCount": 0,
    "repliesCount": 0,
    "rekarotsCount": 0,
    "bookmarksCount": 0,
    "viewCount": 0
  }
}
```

> **IMPORTANT: Cannot mix images and videos.** If you attempt to upload both image files and video files in the same post, the server returns a `400 Bad Request` error. Each post may contain either images or a video, but not both.

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `400` | `MIXED_MEDIA_TYPES` | Cannot include both images and videos in one post |
| `400` | `CONTENT_REQUIRED` | Post has no content and no media |
| `400` | `MEDIA_LIMIT_EXCEEDED` | Too many media files attached |
| `413` | `PAYLOAD_TOO_LARGE` | Media file size exceeds limit |

---

## Get Post

### `GET /posts/:id`

Retrieve a single post by its ID.

**Request:**

```http
GET /posts/post_abc123 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "post": {
    "id": "post_abc123",
    "content": "Hello Karotter! This is my post with media.",
    "author": {
      "id": "clx1abc2d3ef4gh5ij6kl7mn8",
      "username": "myusername",
      "displayName": "My Display Name",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp",
      "isPrivate": false,
      "officialMark": [],
      "isParodyAccount": false,
      "adminForceParody": false,
      "isBotAccount": false,
      "adminForceBot": false
    },
    "content": "Hello world!",
    "visibility": "PUBLIC",
    "replyRestriction": "EVERYONE",
    "media": [
      {
        "id": "media_001",
        "url": "https://cdn.karotter.com/media/media_001.webp",
        "type": "IMAGE",
        "width": 1920,
        "height": 1080,
        "alt": "Sunset at the beach",
        "isSpoiler": false,
        "isR18": false,
        "isAiGenerated": false
      }
    ],
    "poll": null,
    "parentId": null,
    "parentPost": null,
    "quotedPost": null,
    "quotedPostId": null,
    "createdAt": "2026-03-30T08:00:00.000Z",
    "updatedAt": "2026-03-30T08:00:00.000Z",
    "editedAt": null,
    "likesCount": 42,
    "repliesCount": 7,
    "rekarotsCount": 3,
    "quotesCount": 1,
    "bookmarksCount": 5,
    "viewCount": 1250,
    "isLiked": false,
    "isRekaroted": false,
    "isBookmarked": false,
    "myReactions": [],
    "reactions": [
      {
        "emoji": "👍",
        "count": 12,
        "hasReacted": false
      },
      {
        "emoji": "❤️",
        "count": 8,
        "hasReacted": true
      }
    ],
    "isMutedByViewer": false,
    "scheduledFor": null,
    "viewerCircle": null
  }
}
```

### Full Post Object Reference

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique post identifier |
| `content` | `string` | Post text content |
| `author` | `object` | Author user object (id, username, displayName, avatarUrl, isPrivate, officialMark, isParodyAccount, adminForceParody, isBotAccount, adminForceBot) |
| `visibility` | `string` | `PUBLIC` or `CIRCLE` |
| `replyRestriction` | `string` | `EVERYONE`, `FOLLOWING`, `MENTIONED`, `CIRCLE` |
| `media` | `array` | Array of media objects |
| `media[].id` | `string` | Media item ID |
| `media[].url` | `string` | CDN URL for the media file |
| `media[].type` | `string` | `IMAGE` or `VIDEO` |
| `media[].width` | `number` | Width in pixels |
| `media[].height` | `number` | Height in pixels |
| `media[].alt` | `string \| null` | Alt text description |
| `media[].isSpoiler` | `boolean` | Whether marked as spoiler |
| `media[].isR18` | `boolean` | Whether marked as R18/NSFW |
| `media[].isAiGenerated` | `boolean` | Whether flagged as AI-generated |
| `poll` | `object \| null` | Poll object if the post contains a poll |
| `poll.id` | `string` | Poll ID |
| `poll.options` | `array` | Array of `{id, text, votesCount}` |
| `poll.totalVotes` | `number` | Total vote count |
| `poll.expiresAt` | `string` | ISO 8601 expiration timestamp |
| `poll.hasVoted` | `boolean` | Whether the viewer has voted |
| `poll.votedOptionId` | `string \| null` | Which option the viewer voted for |
| `parentId` | `string \| null` | Parent post ID if this is a reply |
| `parentPost` | `Post \| null` | Parent post object (when included) |
| `quotedPost` | `Post \| null` | Quoted post object (for quote posts) |
| `quotedPostId` | `string \| null` | Quoted post ID |
| `createdAt` | `string` | ISO 8601 creation timestamp |
| `updatedAt` | `string` | ISO 8601 last update timestamp |
| `editedAt` | `string \| null` | ISO 8601 edit timestamp, `null` if never edited |
| `likesCount` | `number` | Number of likes |
| `repliesCount` | `number` | Number of direct replies |
| `rekarotsCount` | `number` | Number of rekarots (reposts) |
| `quotesCount` | `number` | Number of quote posts |
| `bookmarksCount` | `number` | Number of bookmarks |
| `viewCount` | `number` | Number of views |
| `isLiked` | `boolean` | Whether the viewer has liked this post |
| `isRekaroted` | `boolean` | Whether the viewer has rekaroted this post |
| `isBookmarked` | `boolean` | Whether the viewer has bookmarked this post |
| `myReactions` | `string[]` | Array of emoji strings the viewer has reacted with |
| `reactions` | `array` | Array of `{emoji, count, hasReacted}` objects |
| `isMutedByViewer` | `boolean` | Whether the viewer has muted this post's conversation |
| `scheduledFor` | `string \| null` | ISO 8601 scheduled publication time, `null` for published posts |
| `viewerCircle` | `object \| null` | Circle object if visibility is `CIRCLE` |

---

## Edit Post

### `PUT /posts/:id`

Edit an existing post. Only the post's author can edit it.

**Request:** Same format as `POST /posts` (multipart/form-data).

**Request:**

```http
PUT /posts/post_abc123 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="content"

Updated post content here.
------FormBoundary--
```

**Response `200 OK`:**

```json
{
  "post": {
    "id": "post_abc123",
    "content": "Updated post content here.",
    "editedAt": "2026-03-30T12:30:00.000Z"
  }
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `403` | `FORBIDDEN` | You are not the author of this post |
| `404` | `NOT_FOUND` | Post does not exist |

---

## Delete Post

### `DELETE /posts/:id`

Delete a post. Only the post's author can delete it.

**Request:**

```http
DELETE /posts/post_abc123 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "カロートを削除しました"
}
```

> **Note:** The success message is returned in Japanese: "カロートを削除しました" (meaning "Karot has been deleted").

---

## Like / Unlike

### `POST /posts/:id/like`

Like a post.

### `DELETE /posts/:id/like`

Unlike a post.

**Request:**

```http
POST /posts/post_abc123/like HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "Post liked"
}
```

**Client behavior notes:**
- The client performs an **optimistic update**: the UI reflects the like/unlike immediately.
- A **1500ms delayed cache invalidation** is triggered after the API call to sync the actual server state.

---

## Rekarot / Un-rekarot

### `POST /posts/:id/rekarot`

Rekarot (repost) a post to your followers.

### `DELETE /posts/:id/rekarot`

Remove your rekarot.

**Request:**

```http
POST /posts/post_abc123/rekarot HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "Rekaroted"
}
```

**Interaction guards:** The client checks the following conditions before allowing a rekarot:
- `canInteract` must be `true` (user is not blocked, etc.)
- The post author is not a private account (unless you follow them)
- The post visibility is not `CIRCLE` (circle-only posts cannot be rekaroted)

---

## Bookmark / Unbookmark

### `POST /posts/:id/bookmark`

Bookmark a post.

### `DELETE /posts/:id/bookmark`

Remove a bookmark.

**Request:**

```http
POST /posts/post_abc123/bookmark HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "Post bookmarked"
}
```

---

## Reactions

### `POST /posts/:id/react`

Add an emoji reaction to a post.

**Request Body:**

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `emoji` | `string` | Yes | Max 32 characters | The emoji to react with |

**Limits:**
- Maximum **20 unique emoji types** per post (across all users).
- Emoji string must be at most **32 characters** (hard server validation).

**Request:**

```http
POST /posts/post_abc123/react HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "emoji": "🔥"
}
```

**Response `200 OK`:**

```json
{
  "message": "Reaction added"
}
```

### `DELETE /posts/:id/react/:emoji`

Remove your reaction from a post.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `emoji` | `string` | URL-encoded emoji string |

**Request:**

```http
DELETE /posts/post_abc123/react/%F0%9F%94%A5 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

> **Note:** The emoji must be URL-encoded in the path. For example, `🔥` becomes `%F0%9F%94%A5`.

**Response `200 OK`:**

```json
{
  "message": "Reaction removed"
}
```

### `GET /posts/:id/react/:emoji/users`

Get the list of users who reacted with a specific emoji.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `10` | Results per page |
| `cursor` | `string` | — | Pagination cursor |

**Request:**

```http
GET /posts/post_abc123/react/%F0%9F%94%A5/users?limit=10 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_reactor1",
      "username": "fire_fan",
      "displayName": "Fire Fan",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_reactor1.webp"
    }
  ],
  "pagination": {
    "nextCursor": "cursor_react123"
  }
}
```

---

## Poll Voting

### `POST /posts/:id/poll/vote`

Vote on a poll option. This endpoint acts as a **toggle**: posting the same `optionId` again will remove your vote (unvote).

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `optionId` | `string` | Yes | ID of the poll option to vote for |

**Request:**

```http
POST /posts/post_abc123/poll/vote HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "optionId": "option_001"
}
```

**Response `200 OK`:**

```json
{
  "poll": {
    "id": "poll_abc123",
    "options": [
      { "id": "option_001", "text": "Option A", "votesCount": 15 },
      { "id": "option_002", "text": "Option B", "votesCount": 8 }
    ],
    "totalVotes": 23,
    "hasVoted": true,
    "votedOptionId": "option_001"
  }
}
```

---

## Replies

### `GET /posts/:id/replies`

Get replies to a post.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `number` | `1` | Page number |
| `limit` | `number` | `20` | Results per page |

**Request:**

```http
GET /posts/post_abc123/replies?page=1&limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "replies": [
    {
      "id": "post_reply001",
      "content": "Great post!",
      "author": {
        "id": "clx_replier1",
        "username": "replier",
        "displayName": "Replier",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx_replier1.webp"
      },
      "parentId": "post_abc123",
      "createdAt": "2026-03-30T09:00:00.000Z",
      "likesCount": 2,
      "repliesCount": 0,
      "isMutedByViewer": false
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20
  }
}
```

**Client-side filtering:** The client filters out replies where any of the following are true:
- `isMutedByViewer` is `true`
- The reply author has been blocked by the viewer
- The viewer is blocked by the reply author

### `GET /posts/:id/reply-targets`

Get the chain of parent posts leading up to a reply (the "reply targets" or conversation thread above).

**Request:**

```http
GET /posts/post_reply001/reply-targets HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "posts": [
    {
      "id": "post_abc123",
      "content": "Original post",
      "author": { "id": "clx1abc2d3ef4gh5ij6kl7mn8", "username": "myusername" },
      "createdAt": "2026-03-30T08:00:00.000Z"
    }
  ]
}
```

---

## Likes List

### `GET /posts/:id/likes`

Get the list of users who liked a post.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | Results per page |
| `cursor` | `string` | — | Pagination cursor |

**Request:**

```http
GET /posts/post_abc123/likes?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_liker1",
      "username": "liker_one",
      "displayName": "Liker One",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_liker1.webp"
    }
  ],
  "pagination": {
    "nextCursor": "cursor_likes123"
  }
}
```

---

## Rekarot Users

### `GET /posts/:id/rekarots`

Get the list of users who rekaroted a post, including any comment they added.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | Results per page |
| `cursor` | `string` | — | Pagination cursor |

**Request:**

```http
GET /posts/post_abc123/rekarots?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_rekaroter1",
      "username": "rekaroter",
      "displayName": "Rekaroter",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_rekaroter1.webp",
      "comment": "This is so good!"
    },
    {
      "id": "clx_rekaroter2",
      "username": "silent_rekaroter",
      "displayName": "Silent Rekaroter",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_rekaroter2.webp",
      "comment": null
    }
  ],
  "pagination": {
    "nextCursor": "cursor_rekarots123"
  }
}
```

---

## Quote Posts

### `GET /posts/:id/quotes`

Get posts that quote this post.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | Results per page |
| `cursor` | `string` | — | Pagination cursor |

**Request:**

```http
GET /posts/post_abc123/quotes?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "quotes": [
    {
      "id": "post_quote001",
      "content": "Quoting this because it's important",
      "author": {
        "id": "clx_quoter1",
        "username": "quoter",
        "displayName": "Quoter"
      },
      "quotedPostId": "post_abc123",
      "createdAt": "2026-03-30T10:00:00.000Z",
      "likesCount": 3
    }
  ],
  "pagination": {
    "nextCursor": "cursor_quotes123"
  }
}
```

---

## Post Analytics

### `GET /posts/:id/analytics`

Get audience analytics for a post. Only available for the post's author.

**Request:**

```http
GET /posts/post_abc123/analytics HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "audience": {
    "knownViewerCount": 450,
    "genderCounts": {
      "MALE": 180,
      "FEMALE": 200,
      "OTHER": 70
    },
    "ageBuckets": {
      "13-17": 25,
      "18-24": 150,
      "25-34": 180,
      "35-44": 60,
      "45-54": 20,
      "55+": 5,
      "unknown": 10
    }
  }
}
```

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `audience.knownViewerCount` | `number` | Total viewers with known demographics |
| `audience.genderCounts.MALE` | `number` | Male viewers |
| `audience.genderCounts.FEMALE` | `number` | Female viewers |
| `audience.genderCounts.OTHER` | `number` | Other gender viewers |
| `audience.ageBuckets["13-17"]` | `number` | Viewers aged 13-17 |
| `audience.ageBuckets["18-24"]` | `number` | Viewers aged 18-24 |
| `audience.ageBuckets["25-34"]` | `number` | Viewers aged 25-34 |
| `audience.ageBuckets["35-44"]` | `number` | Viewers aged 35-44 |
| `audience.ageBuckets["45-54"]` | `number` | Viewers aged 45-54 |
| `audience.ageBuckets["55+"]` | `number` | Viewers aged 55+ |
| `audience.ageBuckets["unknown"]` | `number` | Viewers with unknown age |

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `403` | `FORBIDDEN` | You are not the author of this post |

---

## Leave Conversation

### `POST /posts/:id/conversation/leave`

Mute/leave a conversation thread. You will no longer receive notifications for replies in this thread.

**Request:**

```http
POST /posts/post_abc123/conversation/leave HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "Left conversation"
}
```

---

## Pin / Unpin Post

### `PATCH /users/profile/pinned-post`

Pin a post to your profile, or unpin the current pinned post.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `postId` | `string \| null` | Yes | Post ID to pin, or `null` to unpin |

**Request (pin):**

```http
PATCH /users/profile/pinned-post HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "postId": "post_abc123"
}
```

**Request (unpin):**

```json
{
  "postId": null
}
```

**Response `200 OK`:**

```json
{
  "message": "Pinned post updated"
}
```

---

## Batch Views

### `POST /posts/batch-views`

Report that the user has viewed a batch of posts. The client calls this endpoint automatically.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `postIds` | `string[]` | Yes | Array of post IDs that were viewed |

**Client behavior:**
- Called every **5 seconds** with accumulated post IDs.
- A post is counted as "viewed" when it has been visible for at least **1 second** (dwell time) and at least **50% of the post** is within the viewport (visibility threshold).

**Request:**

```http
POST /posts/batch-views HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "postIds": ["post_abc123", "post_def456", "post_ghi789"]
}
```

**Response `200 OK`:**

```json
{
  "message": "Views recorded"
}
```

---

## Scheduled Posts

### `GET /posts/scheduled/me`

Get the authenticated user's scheduled (unpublished) posts.

**Request:**

```http
GET /posts/scheduled/me HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "scheduledPosts": [
    {
      "id": "post_sched001",
      "content": "This will be posted later!",
      "scheduledFor": "2026-04-01T09:00:00.000Z",
      "visibility": "PUBLIC",
      "viewerCircle": null,
      "pollOptions": null
    },
    {
      "id": "post_sched002",
      "content": "Another scheduled post with a poll",
      "scheduledFor": "2026-04-02T12:00:00.000Z",
      "visibility": "PUBLIC",
      "viewerCircle": null,
      "pollOptions": [
        { "text": "Yes" },
        { "text": "No" },
        { "text": "Maybe" }
      ]
    }
  ]
}
```

**Scheduled Post Object:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Scheduled post ID |
| `content` | `string` | Post text content |
| `scheduledFor` | `string` | ISO 8601 timestamp for when the post will be published |
| `visibility` | `string` | `PUBLIC` or `CIRCLE` |
| `viewerCircle` | `object \| null` | Circle object if visibility is `CIRCLE` |
| `pollOptions` | `array \| null` | Array of `{text}` objects if the post includes a poll |

### `DELETE /posts/scheduled/:id`

Delete a scheduled post before it is published.

**Request:**

```http
DELETE /posts/scheduled/post_sched001 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Response `200 OK`:**

```json
{
  "message": "Scheduled post deleted"
}
```
