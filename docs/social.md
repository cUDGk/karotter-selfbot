# ソーシャル API（サークル、リスト、ストーリー）

> **ベースURL:** `https://karotter.com/api`
>
> すべてのエンドポイントは `Authorization: Bearer {token}` ヘッダーと `X-CSRF-Token` ヘッダーによる認証が必要です。
> 一部の操作ではCookieベース認証（`karotter_at`, `karotter_rt`, `karotter_csrf`）が必要な場合があります。

---

## 目次

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

## サークル

サークルはフォロワーの非公開グループです。投稿の公開範囲の制御（投稿を特定のサークルに制限可能）とリプライ権限に使用されます。

### サークル一覧

認証済みユーザーが作成したすべてのサークルを返します。

```
GET /api/social/circles
```

#### クエリパラメータ

なし。

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

#### レスポンスフィールド

| フィールド | 型 | 説明 |
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

### サークル作成

新しいサークルを作成します。

```
POST /api/social/circles
```

#### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
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

### サークル削除

サークルを削除します。作成者のみ削除可能です。このサークルに表示されていた投稿はメンバーからアクセスできなくなります（投稿者のみ表示に戻ります）。

```
DELETE /api/social/circles/:id
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Circle ID |

#### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "サークルを削除しました"}` | Successfully deleted |
| 403 | `{"error": "権限がありません"}` | Not the circle creator |
| 404 | `{"error": "サークルが見つかりません"}` | Circle not found |

---

### サークルメンバー追加

サークルにユーザーを追加します。

```
POST /api/social/circles/:id/members
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Circle ID |

#### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
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

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メンバーを追加しました"}` | Member added |
| 400 | `{"error": "既にメンバーです"}` | Already a member |
| 403 | `{"error": "権限がありません"}` | Not the circle creator |

---

### サークルメンバー削除

サークルからユーザーを削除します。

```
DELETE /api/social/circles/:id/members/:userId
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Circle ID |
| `userId` | number | Yes | User ID to remove |

#### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メンバーを削除しました"}` | Member removed |
| 403 | `{"error": "権限がありません"}` | Not the circle creator |
| 404 | `{"error": "メンバーが見つかりません"}` | User is not a member |

---

## リスト

リストはキュレーションされたユーザーコレクションです。リストの全メンバーの投稿を統合したタイムラインを表示できます。Twitter/Xのリストに似ています。

### リスト一覧

認証済みユーザーが作成したすべてのリストを返します。

```
GET /api/social/lists
```

#### クエリパラメータ

なし。

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

#### レスポンスフィールド

| フィールド | 型 | 説明 |
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

### リスト作成

新しいリストを作成します。

```
POST /api/social/lists
```

#### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
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

### リスト削除

リストを削除します。作成者のみ削除可能です。

```
DELETE /api/social/lists/:id
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | List ID |

#### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "リストを削除しました"}` | Successfully deleted |
| 403 | `{"error": "権限がありません"}` | Not the list creator |
| 404 | `{"error": "リストが見つかりません"}` | List not found |

---

### リストの投稿取得

リストの全メンバーの投稿のタイムラインを返します。

```
GET /api/social/lists/:id/posts
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | List ID |

#### クエリパラメータ

| パラメータ | 型 | デフォルト | 説明 |
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

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"posts": [...], "pagination": {...}}` | Posts retrieved |
| 404 | `{"error": "リストが見つかりません"}` | List not found |

---

### リストメンバー追加

リストにユーザーを追加します。

```
POST /api/social/lists/:id/members
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | List ID |

#### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `userId` | number | Yes | User ID to add |

#### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メンバーを追加しました"}` | Member added |
| 400 | `{"error": "既にメンバーです"}` | Already a member |
| 403 | `{"error": "権限がありません"}` | Not the list creator |

---

### リストメンバー削除

リストからユーザーを削除します。

```
DELETE /api/social/lists/:id/members/:userId
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | List ID |
| `userId` | number | Yes | User ID to remove |

#### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メンバーを削除しました"}` | Member removed |
| 403 | `{"error": "権限がありません"}` | Not the list creator |
| 404 | `{"error": "メンバーが見つかりません"}` | User is not a member |

---

## ストーリー

ストーリーは24時間後に消える一時的なメディア投稿です。タイムライン上部のカルーセルに表示されます。

### 全ストーリー取得

フォローしているユーザーのストーリーを新しい順で返します。

```
GET /api/social/stories
```

#### クエリパラメータ

なし。

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

### ストーリー作成

新しいストーリーを作成します。ファイルアップロードのため `multipart/form-data` エンコーディングを使用します。

```
POST /api/social/stories
```

#### リクエストボディ (FormData)

| フィールド | 型 | 必須 | 説明 |
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

### ユーザーのストーリー取得

特定のユーザーのすべてのアクティブ（未期限切れ）ストーリーを返します。

```
GET /api/social/stories/user/:userId
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
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

### ストーリー閲覧者取得

ストーリーを閲覧したユーザーを返します。ストーリーの投稿者のみアクセス可能です。

```
GET /api/social/stories/:id/viewers
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Responses

| ステータス | ボディ | 説明 |
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

### ストーリーコメント取得

ストーリーのコメントを返します。

```
GET /api/social/stories/:id/comments
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
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

### ストーリー閲覧記録

認証済みユーザーがストーリーを閲覧したことを記録します。

```
POST /api/social/stories/:id/views
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### リクエストボディ

なし。

#### Response

```json
{
  "message": "閲覧を記録しました"
}
```

#### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "閲覧を記録しました"}` | View recorded |
| 404 | `{"error": "ストーリーが見つかりません"}` | Story not found or expired |

---

### ストーリーにコメント

ストーリーにコメントを追加します。

```
POST /api/social/stories/:id/comments
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
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

### ストーリーにいいね

ストーリーにいいねします。

```
POST /api/social/stories/:id/like
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### リクエストボディ

なし。

#### Response

```json
{
  "message": "いいねしました"
}
```

---

### ストーリーのいいね取消

ストーリーのいいねを取り消します。

```
DELETE /api/social/stories/:id/like
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### リクエストボディ

なし。

#### Response

```json
{
  "message": "いいねを取り消しました"
}
```

---

### ストーリー削除

ストーリーを削除します。ストーリーの投稿者のみ削除可能です。

```
DELETE /api/social/stories/:id
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Story ID |

#### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ストーリーを削除しました"}` | Deleted |
| 403 | `{"error": "権限がありません"}` | Not the story author |
| 404 | `{"error": "ストーリーが見つかりません"}` | Story not found |

---

### Story（ストーリー） Visibility

Stories support three visibility levels:

| 値 | 説明 |
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

### Story（ストーリー） Field Reference

| フィールド | 型 | 説明 |
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

## 匿名質問

匿名質問は、ユーザーが他のユーザーから匿名で質問を受け取り回答できる機能です。質問は匿名で送信され、受信者の受信箱に表示されます。

### 質問受信箱取得

認証済みユーザーが受信したすべての質問を返します。

```
GET /api/social/questions/inbox
```

#### クエリパラメータ

| パラメータ | 型 | デフォルト | 説明 |
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

### 質問を送信

ユーザーに匿名質問を送信します。

```
POST /api/social/questions/:username
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `username` | string | Yes | Target user's username |

#### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
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

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "質問を送信しました"}` | Question sent |
| 400 | `{"error": "..."}` | Validation error |
| 404 | `{"error": "ユーザーが見つかりません"}` | User not found |

---

### 質問を削除

受信箱から質問を削除します。

```
DELETE /api/social/questions/:id
```

#### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Question ID |

#### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "質問を削除しました"}` | Question deleted |
| 403 | `{"error": "権限がありません"}` | Not the question recipient |
| 404 | `{"error": "質問が見つかりません"}` | Question not found |

---

## リンクプレビュー

### リンクプレビュー取得

指定URLのOpen Graph (OGP) メタデータを取得します。

```
GET /api/social/link-preview
```

#### クエリパラメータ

| パラメータ | 型 | 必須 | 説明 |
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

#### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `preview.url` | string | The canonical URL |
| `preview.title` | string \| null | Page title from OGP or `<title>` tag |
| `preview.description` | string \| null | Page description from OGP or meta description |
| `preview.image` | string \| null | OGP image URL |
| `preview.siteName` | string \| null | Site name from OGP |

---

### リンクプレビュー取得 Image

URLのOGP画像をプロキシします。フロントエンドがCORS問題なしでリンクプレビューサムネイルを表示するために使用されます。

```
GET /api/social/link-preview-image
```

#### クエリパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `url` | string | Yes | The URL of the image to proxy |

#### Response

適切な `Content-Type` ヘッダーとともに画像バイナリデータを返します。

---

## レート制限

すべてのソーシャルエンドポイントはデフォルトのレート制限を共有しています:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
