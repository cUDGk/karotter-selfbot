# Social API (Circles, Lists, Stories)

> **Base URL:** `https://karotter.com/api`
>
> All endpoints require authentication via `Authorization: Bearer {token}` header and `X-CSRF-Token` header.
> Cookie-based authentication (`karotter_at`, `karotter_rt`, `karotter_csrf`) may be required for some operations.

---

## Table of Contents

- [Circles](#circles)
  - [List Circles](#list-circles)
  - [Create a Circle](#create-a-circle)
  - [Delete a Circle](#delete-a-circle)
  - [Add Circle Member](#add-circle-member)
  - [Remove Circle Member](#remove-circle-member)
- [Lists](#lists)
  - [List All Lists](#list-all-lists)
  - [Create a List](#create-a-list)
  - [Delete a List](#delete-a-list)
  - [Get List Posts](#get-list-posts)
  - [Add List Member](#add-list-member)
  - [Remove List Member](#remove-list-member)
- [Stories](#stories)
  - [Get All Stories](#get-all-stories)
  - [Create a Story](#create-a-story)
  - [Get User Stories](#get-user-stories)
  - [Get Story Viewers](#get-story-viewers)
  - [Get Story Comments](#get-story-comments)
  - [Record Story View](#record-story-view)
  - [Comment on a Story](#comment-on-a-story)
  - [Like a Story](#like-a-story)
  - [Unlike a Story](#unlike-a-story)
  - [Delete a Story](#delete-a-story)
  - [Story Visibility](#story-visibility)
  - [Object Reference: Story](#object-reference-story)
- [Anonymous Questions](#anonymous-questions)
  - [Get Question Inbox](#get-question-inbox)
  - [Send a Question](#send-a-question)
  - [Delete a Question](#delete-a-question)
- [Link Preview](#link-preview)
  - [Get Link Preview](#get-link-preview)
  - [Get Link Preview Image](#get-link-preview-image)

---

## Circles

Circles are private groups of followers. They are used for controlling post visibility (posts can be restricted to a specific circle) and reply permissions.

### List Circles

Returns all circles created by the authenticated user.

```
GET /api/social/circles
```

#### Query Parameters

None.

#### Response

```json
{
  "circles": [
    {
      "id": 42,
      "name": "Close Friends",
      "userId": 15459,
      "memberCount": 12,
      "createdAt": "2026-03-20T10:00:00.000Z",
      "updatedAt": "2026-03-28T15:00:00.000Z",
      "members": [
        {
          "id": 18179,
          "username": "friend1",
          "displayName": "Friend One",
          "avatarUrl": "/uploads/avatars/abc.webp"
        }
      ]
    }
  ]
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `circles` | Circle[] | Array of circle objects |
| `circles[].id` | number | Circle ID |
| `circles[].name` | string | Circle name |
| `circles[].userId` | number | Creator's user ID |
| `circles[].memberCount` | number | Number of members |
| `circles[].createdAt` | string | ISO 8601 creation timestamp |
| `circles[].updatedAt` | string | ISO 8601 last update timestamp |
| `circles[].members` | UserBrief[] | Array of member user objects |

---

### Create a Circle

Creates a new circle.

```
POST /api/social/circles
```

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Circle name |

#### Example

```http
POST /api/social/circles HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "name": "Close Friends"
}
```

#### Response

```json
{
  "circle": {
    "id": 42,
    "name": "Close Friends",
    "userId": 15459,
    "memberCount": 0,
    "createdAt": "2026-03-28T15:00:00.000Z"
  }
}
```

---

### Delete a Circle

Deletes a circle. Only the creator can delete it. Posts previously visible to this circle become inaccessible to those members (they revert to author-only visibility).

```
DELETE /api/social/circles/:id
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Circle ID |

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "サークルを削除しました"}` | Successfully deleted |
| 403 | `{"error": "権限がありません"}` | Not the circle creator |
| 404 | `{"error": "サークルが見つかりません"}` | Circle not found |

---

### Add Circle Member

Adds a user to a circle.

```
POST /api/social/circles/:id/members
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Circle ID |

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `userId` | number | Yes | User ID to add |

#### Example

```http
POST /api/social/circles/42/members HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "userId": 18179
}
```

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "メンバーを追加しました"}` | Member added |
| 400 | `{"error": "既にメンバーです"}` | Already a member |
| 403 | `{"error": "権限がありません"}` | Not the circle creator |

---

### Remove Circle Member

Removes a user from a circle.

```
DELETE /api/social/circles/:id/members/:userId
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Circle ID |
| `userId` | number | Yes | User ID to remove |

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "メンバーを削除しました"}` | Member removed |
| 403 | `{"error": "権限がありません"}` | Not the circle creator |
| 404 | `{"error": "メンバーが見つかりません"}` | User is not a member |

---

## Lists

Lists are curated collections of users. You can view a combined timeline of posts from all members of a list. Lists are similar to Twitter/X lists.

### List All Lists

Returns all lists created by the authenticated user.

```
GET /api/social/lists
```

#### Query Parameters

None.

#### Response

```json
{
  "lists": [
    {
      "id": 7,
      "name": "Tech People",
      "description": "Developers and tech enthusiasts",
      "userId": 15459,
      "isPrivate": false,
      "memberCount": 25,
      "createdAt": "2026-03-15T10:00:00.000Z",
      "updatedAt": "2026-03-28T12:00:00.000Z"
    }
  ]
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `lists` | List[] | Array of list objects |
| `lists[].id` | number | List ID |
| `lists[].name` | string | List name |
| `lists[].description` | string | List description |
| `lists[].userId` | number | Creator's user ID |
| `lists[].isPrivate` | boolean | Whether the list is private |
| `lists[].memberCount` | number | Number of members |
| `lists[].createdAt` | string | ISO 8601 creation timestamp |
| `lists[].updatedAt` | string | ISO 8601 last update timestamp |

---

### Create a List

Creates a new list.

```
POST /api/social/lists
```

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | List name |
| `description` | string | No | List description |
| `isPrivate` | boolean | No | Whether the list is private (default: false) |

#### Example

```http
POST /api/social/lists HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "name": "Tech People",
  "description": "Developers and tech enthusiasts",
  "isPrivate": false
}
```

#### Response

```json
{
  "list": {
    "id": 7,
    "name": "Tech People",
    "description": "Developers and tech enthusiasts",
    "userId": 15459,
    "isPrivate": false,
    "memberCount": 0,
    "createdAt": "2026-03-28T15:00:00.000Z"
  }
}
```

---

### Delete a List

Deletes a list. Only the creator can delete it.

```
DELETE /api/social/lists/:id
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | List ID |

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "リストを削除しました"}` | Successfully deleted |
| 403 | `{"error": "権限がありません"}` | Not the list creator |
| 404 | `{"error": "リストが見つかりません"}` | List not found |

---

### Get List Posts

Returns a timeline of posts from all members of a list.

```
GET /api/social/lists/:id/posts
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | List ID |

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number |
| `limit` | number | 15 | Posts per page |

#### Response

```json
{
  "posts": [
    {
      "id": 119340,
      "content": "Post from a list member",
      "authorId": 18179,
      "author": {"...": "UserBrief"},
      "createdAt": "2026-03-28T23:45:24.948Z",
      "...": "full Post object"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 15,
    "hasNext": true
  }
}
```

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"posts": [...], "pagination": {...}}` | Posts retrieved |
| 404 | `{"error": "リストが見つかりません"}` | List not found |

---

### Add List Member

Adds a user to a list.

```
POST /api/social/lists/:id/members
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | List ID |

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `userId` | number | Yes | User ID to add |

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "メンバーを追加しました"}` | Member added |
| 400 | `{"error": "既にメンバーです"}` | Already a member |
| 403 | `{"error": "権限がありません"}` | Not the list creator |

---

### Remove List Member

Removes a user from a list.

```
DELETE /api/social/lists/:id/members/:userId
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | List ID |
| `userId` | number | Yes | User ID to remove |

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "メンバーを削除しました"}` | Member removed |
| 403 | `{"error": "権限がありません"}` | Not the list creator |
| 404 | `{"error": "メンバーが見つかりません"}` | User is not a member |

---

## Stories

Stories are ephemeral media posts that expire after 24 hours. They appear at the top of the timeline in a carousel.

### Get All Stories

Returns stories from users you follow, ordered by recency.

```
GET /api/social/stories
```

#### Query Parameters

None.

#### Response

```json
{
  "stories": [
    {
      "id": 1184,
      "authorId": 18197,
      "mediaUrl": "/uploads/posts/abc123.jpg",
      "mediaType": "image/jpeg",
      "caption": "Having a great day!",
      "isR18": false,
      "hideFromMinors": false,
      "adminForceHidden": false,
      "adminForceR18": false,
      "visibility": "PUBLIC",
      "viewerCircleId": null,
      "createdAt": "2026-03-28T10:00:00.000Z",
      "expiresAt": "2026-03-29T10:00:00.000Z",
      "author": {
        "id": 18197,
        "username": "photographer",
        "displayName": "Photo Person",
        "avatarUrl": "/uploads/avatars/xyz.webp",
        "avatarFrameId": null,
        "officialMark": [],
        "isBotAccount": false,
        "isParodyAccount": false,
        "adminForceBot": false,
        "adminForceParody": false,
        "isPrivate": false
      },
      "viewerCircle": null,
      "views": [],
      "likes": [],
      "_count": {
        "views": 45,
        "likes": 8
      },
      "hasViewed": false,
      "liked": false,
      "viewsCount": 45,
      "likesCount": 8
    }
  ]
}
```

---

### Create a Story

Creates a new story. Uses `multipart/form-data` encoding for file upload.

```
POST /api/social/stories
```

#### Request Body (FormData)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `media` | File | Yes | Image or video file |
| `caption` | string | No | Caption text |
| `visibility` | string | No | Visibility setting (default: `"PUBLIC"`) |
| `viewerCircleId` | number | No | Circle ID if visibility is `"CIRCLE"` |
| `isR18` | boolean | No | Whether the content is R18/NSFW |

#### Example

```http
POST /api/social/stories HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="media"; filename="sunset.jpg"
Content-Type: image/jpeg

(binary data)
------FormBoundary
Content-Disposition: form-data; name="caption"

Beautiful sunset today!
------FormBoundary
Content-Disposition: form-data; name="visibility"

PUBLIC
------FormBoundary--
```

#### Response

```json
{
  "story": {
    "id": 1200,
    "authorId": 15459,
    "mediaUrl": "/uploads/posts/new-story.jpg",
    "mediaType": "image/jpeg",
    "caption": "Beautiful sunset today!",
    "visibility": "PUBLIC",
    "createdAt": "2026-03-28T15:00:00.000Z",
    "expiresAt": "2026-03-29T15:00:00.000Z"
  }
}
```

---

### Get User Stories

Returns all active (non-expired) stories from a specific user.

```
GET /api/social/stories/user/:userId
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `userId` | number | Yes | User ID |

#### Response

```json
{
  "stories": [
    {
      "id": 1184,
      "authorId": 18197,
      "mediaUrl": "/uploads/posts/abc.jpg",
      "mediaType": "image/jpeg",
      "caption": "",
      "visibility": "PUBLIC",
      "createdAt": "2026-03-28T10:00:00.000Z",
      "expiresAt": "2026-03-29T10:00:00.000Z",
      "author": {"...": "UserBrief"},
      "hasViewed": true,
      "liked": false,
      "viewsCount": 45,
      "likesCount": 8
    }
  ]
}
```

---

### Get Story Viewers

Returns users who have viewed a story. Only accessible by the story author.

```
GET /api/social/stories/:id/viewers
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"viewers": [...]}` | List of viewers |
| 403 | `{"error": "権限がありません"}` | Not the story author |
| 404 | `{"error": "ストーリーが見つかりません"}` | Story not found |

#### Response

```json
{
  "viewers": [
    {
      "id": 18179,
      "username": "viewer1",
      "displayName": "Viewer One",
      "avatarUrl": "/uploads/avatars/abc.webp",
      "viewedAt": "2026-03-28T11:00:00.000Z"
    }
  ]
}
```

---

### Get Story Comments

Returns comments on a story.

```
GET /api/social/stories/:id/comments
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Response

```json
{
  "comments": [
    {
      "id": 501,
      "content": "Nice photo!",
      "userId": 18179,
      "user": {
        "id": 18179,
        "username": "commenter",
        "displayName": "Commenter",
        "avatarUrl": "/uploads/avatars/abc.webp"
      },
      "createdAt": "2026-03-28T11:30:00.000Z"
    }
  ]
}
```

---

### Record Story View

Records that the authenticated user has viewed a story.

```
POST /api/social/stories/:id/views
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Request Body

None.

#### Response

```json
{
  "message": "閲覧を記録しました"
}
```

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "閲覧を記録しました"}` | View recorded |
| 404 | `{"error": "ストーリーが見つかりません"}` | Story not found or expired |

---

### Comment on a Story

Adds a comment to a story.

```
POST /api/social/stories/:id/comments
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | string | Yes | Comment text |

#### Example

```http
POST /api/social/stories/1184/comments HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "content": "This looks amazing!"
}
```

---

### Like a Story

Likes a story.

```
POST /api/social/stories/:id/like
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Request Body

None.

#### Response

```json
{
  "message": "いいねしました"
}
```

---

### Unlike a Story

Removes a like from a story.

```
DELETE /api/social/stories/:id/like
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Request Body

None.

#### Response

```json
{
  "message": "いいねを取り消しました"
}
```

---

### Delete a Story

Deletes a story. Only the story author can delete it.

```
DELETE /api/social/stories/:id
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "ストーリーを削除しました"}` | Deleted |
| 403 | `{"error": "権限がありません"}` | Not the story author |
| 404 | `{"error": "ストーリーが見つかりません"}` | Story not found |

---

### Story Visibility

Stories support three visibility levels:

| Value | Description |
|-------|-------------|
| `PUBLIC` | Visible to all users |
| `FOLLOWERS` | Visible only to your followers |
| `CIRCLE` | Visible only to members of a specific circle (requires `viewerCircleId`) |

When using `CIRCLE` visibility, you must set the `viewerCircleId` to a valid circle ID that you own.

---

### Object Reference: Story

Full Story object structure.

```json
{
  "id": 1184,
  "authorId": 18197,
  "mediaUrl": "/uploads/posts/abc123.jpg",
  "mediaType": "image/jpeg",
  "caption": "Story caption text",
  "isR18": false,
  "hideFromMinors": false,
  "adminForceHidden": false,
  "adminForceR18": false,
  "visibility": "PUBLIC",
  "viewerCircleId": null,
  "createdAt": "2026-03-28T10:00:00.000Z",
  "expiresAt": "2026-03-29T10:00:00.000Z",
  "author": {
    "id": 18197,
    "username": "photographer",
    "displayName": "Photo Person",
    "avatarUrl": "/uploads/avatars/xyz.webp",
    "avatarFrameId": null,
    "officialMark": [],
    "isBotAccount": false,
    "isParodyAccount": false,
    "adminForceBot": false,
    "adminForceParody": false,
    "isPrivate": false
  },
  "viewerCircle": null,
  "views": [],
  "likes": [],
  "_count": {
    "views": 45,
    "likes": 8
  },
  "hasViewed": false,
  "liked": false,
  "viewsCount": 45,
  "likesCount": 8
}
```

### Story Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `id` | number | Story ID |
| `authorId` | number | Author's user ID |
| `mediaUrl` | string | Relative URL to the story media (prepend `https://karotter.com`) |
| `mediaType` | string | MIME type of the media (e.g., `"image/jpeg"`, `"video/mp4"`) |
| `caption` | string | Caption text |
| `isR18` | boolean | Whether the content is marked R18 |
| `hideFromMinors` | boolean | Whether to hide from minor users |
| `adminForceHidden` | boolean | Admin-forced hidden flag |
| `adminForceR18` | boolean | Admin-forced R18 flag |
| `visibility` | string | `"PUBLIC"`, `"FOLLOWERS"`, or `"CIRCLE"` |
| `viewerCircleId` | number \| null | Circle ID for circle-restricted stories |
| `createdAt` | string | ISO 8601 creation timestamp |
| `expiresAt` | string | ISO 8601 expiration timestamp (24 hours after creation) |
| `author` | UserBrief | Author user object |
| `viewerCircle` | Circle \| null | Circle object if visibility is `"CIRCLE"` |
| `views` | View[] | Array of view records (may be empty in list endpoints) |
| `likes` | Like[] | Array of like records (may be empty in list endpoints) |
| `_count.views` | number | Total view count |
| `_count.likes` | number | Total like count |
| `hasViewed` | boolean | Whether the authenticated user has viewed this story |
| `liked` | boolean | Whether the authenticated user has liked this story |
| `viewsCount` | number | Total view count (duplicate of `_count.views`) |
| `likesCount` | number | Total like count (duplicate of `_count.likes`) |

---

## Anonymous Questions

Anonymous questions allow users to receive and answer questions from other users. Questions are sent anonymously and appear in the recipient's inbox.

### Get Question Inbox

Returns all questions received by the authenticated user.

```
GET /api/social/questions/inbox
```

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number |
| `limit` | number | 20 | Questions per page |

#### Response

```json
{
  "questions": [
    {
      "id": 123,
      "content": "What's your favorite anime?",
      "createdAt": "2026-03-28T15:00:00.000Z",
      "isAnswered": false
    }
  ],
  "pagination": {
    "page": 1,
    "hasNext": true
  }
}
```

---

### Send a Question

Sends an anonymous question to a user.

```
POST /api/social/questions/:username
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | string | Yes | Target user's username |

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | string | Yes | Question text |

#### Example

```http
POST /api/social/questions/claude HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "content": "What's your favorite programming language?"
}
```

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "質問を送信しました"}` | Question sent |
| 400 | `{"error": "..."}` | Validation error |
| 404 | `{"error": "ユーザーが見つかりません"}` | User not found |

---

### Delete a Question

Deletes a question from your inbox.

```
DELETE /api/social/questions/:id
```

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Question ID |

#### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "質問を削除しました"}` | Question deleted |
| 403 | `{"error": "権限がありません"}` | Not the question recipient |
| 404 | `{"error": "質問が見つかりません"}` | Question not found |

---

## Link Preview

### Get Link Preview

Fetches Open Graph (OGP) metadata for a given URL.

```
GET /api/social/link-preview
```

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | The URL to fetch preview data for |

#### Example

```http
GET /api/social/link-preview?url=https://example.com HTTP/1.1
Authorization: Bearer eyJ...
```

#### Response

```json
{
  "preview": {
    "url": "https://example.com",
    "title": "Example Domain",
    "description": "This domain is for use in illustrative examples.",
    "image": "https://example.com/og-image.png",
    "siteName": "Example"
  }
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `preview.url` | string | The canonical URL |
| `preview.title` | string \| null | Page title from OGP or `<title>` tag |
| `preview.description` | string \| null | Page description from OGP or meta description |
| `preview.image` | string \| null | OGP image URL |
| `preview.siteName` | string \| null | Site name from OGP |

---

### Get Link Preview Image

Proxies the OGP image for a URL. Used by the frontend to display link preview thumbnails without CORS issues.

```
GET /api/social/link-preview-image
```

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | The URL of the image to proxy |

#### Response

Returns the image binary data with the appropriate `Content-Type` header.

---

## Rate Limiting

All social endpoints share the default rate limit:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
