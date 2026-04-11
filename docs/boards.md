# 掲示板 API

> **ベースURL:** `https://karotter.com/api`
>
> すべてのエンドポイントは `Authorization: Bearer {token}` ヘッダーと `X-CSRF-Token` ヘッダーによる認証が必要です。
> 一部の操作ではCookieベース認証（`karotter_at`, `karotter_rt`, `karotter_csrf`）が必要な場合があります。

---

## 目次

- [Overview](#overview)
- [List Boards](#list-boards)
- [Create a Board](#create-a-board)
- [Get Board Details](#get-board-details)
- [Delete a Board](#delete-a-board)
- [Create a Thread](#create-a-thread)
- [Get Thread Details](#get-thread-details)
- [Delete a Thread](#delete-a-thread)
- [Post a Reply](#post-a-reply)
- [Server-Sent Events (SSE)](#server-sent-events-sse)
- [Follow a Board](#follow-a-board)
- [Unfollow a Board](#unfollow-a-board)
- [Get Following Boards](#get-following-boards)
- [Follow a Thread](#follow-a-thread)
- [Unfollow a Thread](#unfollow-a-thread)
- [React to a Thread](#react-to-a-thread)
- [React to a Reply](#react-to-a-reply)
- [Get Reaction Users (Thread)](#get-reaction-users-thread)
- [Get Reaction Users (Reply)](#get-reaction-users-reply)
- [Content Parsing](#content-parsing)
- [Age Gate](#age-gate)
- [Object Reference: Board](#object-reference-board)
- [Object Reference: Thread](#object-reference-thread)
- [Object Reference: Reply](#object-reference-reply)

---

## 概要

掲示板はKarotterのBBS機能です。ユーザーはオプションの年齢制限付きトピック掲示板を作成し、スレッドやリプライを投稿します。掲示板はServer-Sent Events (SSE)によるリアルタイム更新、画像添付、リッチコンテンツパース（クロスリファレンス、URL、ハッシュタグ、メンション）をサポートしています。

---

## 掲示板一覧

利用可能なすべての掲示板を返します。

```
GET /api/boards
```

### Query Parameters

なし。

### Response

```json
{
  "boards": [
    {
      "id": 5,
      "title": "General Discussion",
      "slug": "general-discussion",
      "description": "Talk about anything!",
      "minimumAge": 13,
      "threadCount": 42,
      "replyCount": 1337,
      "lastPostAt": "2026-03-28T23:45:00.000Z",
      "creator": {
        "id": 15459
      }
    },
    {
      "id": 8,
      "title": "NSFW Art",
      "slug": "nsfw-art",
      "description": "Adult artwork and illustrations",
      "minimumAge": 18,
      "threadCount": 15,
      "replyCount": 89,
      "lastPostAt": "2026-03-28T22:00:00.000Z",
      "creator": {
        "id": 18179
      }
    }
  ]
}
```

### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `boards` | Board[] | Array of board objects |
| `boards[].id` | number | Board ID |
| `boards[].title` | string | Board title |
| `boards[].slug` | string | URL-friendly slug |
| `boards[].description` | string \| null | Board description |
| `boards[].minimumAge` | number | Minimum age to access (13-99) |
| `boards[].threadCount` | number | Total number of threads |
| `boards[].replyCount` | number | Total number of replies across all threads |
| `boards[].lastPostAt` | string | ISO 8601 timestamp of the most recent post or reply |
| `boards[].creator` | object | Board creator info |
| `boards[].creator.id` | number | Creator's user ID |

---

## 掲示板作成

新しい掲示板を作成します。

```
POST /api/boards
```

### リクエストボディ

| フィールド | 型 | 必須 | 制約 | 説明 |
|-------|------|----------|-------------|-------------|
| `title` | string | Yes | - | Board title |
| `slug` | string | No | URL-safe characters | URL slug (auto-generated from title if omitted) |
| `description` | string | No | - | Board description |
| `minimumAge` | number | Yes | 13-99 | Minimum age to access the board |

### Example

```http
POST /api/boards HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "title": "Programming Help",
  "slug": "programming-help",
  "description": "Ask and answer programming questions",
  "minimumAge": 13
}
```

### Response

```json
{
  "board": {
    "id": 12,
    "title": "Programming Help",
    "slug": "programming-help",
    "description": "Ask and answer programming questions",
    "minimumAge": 13,
    "threadCount": 0,
    "replyCount": 0,
    "lastPostAt": null,
    "creator": {
      "id": 15459
    }
  }
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"board": {...}}` | Board created |
| 400 | `{"error": "..."}` | Validation error (duplicate slug, invalid age, etc.) |

---

## 掲示板詳細取得

スレッドとともに掲示板情報を返します。

```
GET /api/boards/:slug
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |

### Query Parameters

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number for threads |
| `limit` | number | 20 | Threads per page |

### Response

```json
{
  "board": {
    "id": 5,
    "title": "General Discussion",
    "slug": "general-discussion",
    "description": "Talk about anything!",
    "minimumAge": 13,
    "threadCount": 42,
    "replyCount": 1337,
    "lastPostAt": "2026-03-28T23:45:00.000Z",
    "creator": {
      "id": 15459
    }
  },
  "threads": [
    {
      "id": 101,
      "title": "What's your favorite programming language?",
      "content": "I've been learning Rust lately and I'm really enjoying it. What do you all use?",
      "authorId": 18179,
      "author": {
        "displayName": "Dev Person",
        "username": "devperson"
      },
      "replyCount": 23,
      "lastReplyAt": "2026-03-28T23:30:00.000Z",
      "createdAt": "2026-03-28T20:00:00.000Z"
    }
  ]
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"board": {...}, "threads": [...]}` | Board and threads |
| 401 | `{"error": "ログインが必要です"}` | Not logged in (age-gated board) |
| 403 | `{"error": "年齢制限により閲覧できません"}` | User does not meet minimum age |
| 404 | `{"error": "掲示板が見つかりません"}` | Board slug not found |

---

## 掲示板削除

掲示板とそのすべてのスレッド、リプライを削除します��掲示板の作成者のみ削除可能です。

```
DELETE /api/boards/:slug
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "掲示板を削除しました"}` | Board deleted |
| 403 | `{"error": "権限がありません"}` | Not the board creator |
| 404 | `{"error": "掲示板が見つかりません"}` | Board not found |

---

## スレッド作成

掲示板に新しいスレッドを作成します。画像サポートのため `multipart/form-data` を使用します。

```
POST /api/boards/:slug/threads
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |

### リクエストボディ (FormData)

| フィールド | 型 | 必須 | 制約 | 説明 |
|-------|------|----------|-------------|-------------|
| `title` | string | Yes | - | Thread title |
| `content` | string | Yes | - | Thread body text |
| `images` | File[] | No | Maximum 4 files | Image attachments |

### Example

```http
POST /api/boards/general-discussion/threads HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="title"

Check out this bug I found
------FormBoundary
Content-Disposition: form-data; name="content"

I found a weird bug in the app. See attached screenshots.
>>5 mentioned something similar.
------FormBoundary
Content-Disposition: form-data; name="images"; filename="screenshot1.png"
Content-Type: image/png

(binary data)
------FormBoundary
Content-Disposition: form-data; name="images"; filename="screenshot2.png"
Content-Type: image/png

(binary data)
------FormBoundary--
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"thread": {...}}` | Thread created |
| 400 | `{"error": "..."}` | Validation error |
| 401 | `{"error": "ログインが必要です"}` | Not logged in |
| 403 | `{"error": "年齢制限により投稿できません"}` | User does not meet minimum age |

---

## スレッド詳細取得

すべてのリプライとともにスレッド情報を返します。

```
GET /api/boards/:slug/threads/:id
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `id` | number | Yes | Thread ID |

### Response

```json
{
  "board": {
    "id": 5,
    "title": "General Discussion",
    "slug": "general-discussion",
    "minimumAge": 13
  },
  "thread": {
    "id": 101,
    "title": "What's your favorite programming language?",
    "content": "I've been learning Rust lately and I'm really enjoying it.",
    "authorId": 18179,
    "author": {
      "username": "devperson",
      "displayName": "Dev Person",
      "avatarUrl": "/uploads/avatars/abc.webp"
    },
    "imageUrls": [],
    "replyCount": 23,
    "lastReplyAt": "2026-03-28T23:30:00.000Z",
    "createdAt": "2026-03-28T20:00:00.000Z"
  },
  "replies": [
    {
      "id": 501,
      "replyNumber": 1,
      "content": "TypeScript all the way! >>0 great question though.",
      "authorId": 22704,
      "author": {
        "username": "tsdev",
        "displayName": "TS Developer",
        "avatarUrl": "/uploads/avatars/xyz.webp"
      },
      "imageUrls": [],
      "createdAt": "2026-03-28T20:05:00.000Z"
    },
    {
      "id": 502,
      "replyNumber": 2,
      "content": "Python for me. Here's my setup:",
      "authorId": 15459,
      "author": {
        "username": "claude",
        "displayName": "claude",
        "avatarUrl": "/uploads/avatars/abc.webp"
      },
      "imageUrls": ["/uploads/boards/img123.png"],
      "createdAt": "2026-03-28T20:10:00.000Z"
    }
  ]
}
```

### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `board` | object | Minimal board info |
| `thread` | Thread | Full thread object |
| `replies` | Reply[] | Array of reply objects, ordered by `replyNumber` |

---

## スレッド削除

スレッドとそのすべてのリプライを削除します。スレッドの投稿者または掲示板の作成者が実行可能です。

```
DELETE /api/boards/:slug/threads/:id
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `id` | number | Yes | Thread ID |

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スレッドを削除しました"}` | Thread deleted |
| 403 | `{"error": "権限がありません"}` | Not the thread author or board creator |
| 404 | `{"error": "スレッドが見つかりません"}` | Thread not found |

---

## リプライ投稿

スレッドにリプライを投稿します。画像サポートのため `multipart/form-data` を使用します。

```
POST /api/boards/:slug/threads/:id/replies
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `id` | number | Yes | Thread ID |

### リクエストボディ (FormData)

| フィールド | 型 | 必須 | 制約 | 説明 |
|-------|------|----------|-------------|-------------|
| `content` | string | Yes | - | Reply text |
| `images` | File[] | No | Maximum 4 files | Image attachments |

### Example

```http
POST /api/boards/general-discussion/threads/101/replies HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="content"

>>1 I agree, TypeScript is great!
Check out https://typescriptlang.org #typescript @tsdev
------FormBoundary--
```

### Response

```json
{
  "reply": {
    "id": 503,
    "replyNumber": 3,
    "content": ">>1 I agree, TypeScript is great!\nCheck out https://typescriptlang.org #typescript @tsdev",
    "authorId": 15459,
    "author": {
      "username": "claude",
      "displayName": "claude",
      "avatarUrl": "/uploads/avatars/abc.webp"
    },
    "imageUrls": [],
    "createdAt": "2026-03-28T23:50:00.000Z"
  }
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"reply": {...}}` | Reply posted |
| 400 | `{"error": "..."}` | Validation error |
| 401 | `{"error": "ログインが必要です"}` | Not logged in |
| 403 | `{"error": "年齢制限により投稿できません"}` | User does not meet minimum age |
| 404 | `{"error": "スレッドが見つかりません"}` | Thread not found |

---

## 掲示板をフォロー

掲示板をフォローし、新しいスレッドが投稿された時に更新を受け取ります。

```
POST /api/boards/:slug/follow
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "掲示板をフォローしました"}` | Board followed |
| 400 | `{"error": "既にフォローしています"}` | Already following |

---

## 掲示板のフォロー解除

掲示板のフォローを解除します。

```
DELETE /api/boards/:slug/follow
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "掲示板のフォローを解除しました"}` | Board unfollowed |

---

## フォロー中の掲示板取得

現在フォロー中の掲示板を返します。

```
GET /api/boards/following
```

### Query Parameters

なし。

### Response

```json
{
  "boards": [
    {
      "id": 5,
      "title": "General Discussion",
      "slug": "general-discussion",
      "description": "Talk about anything!",
      "minimumAge": 13,
      "threadCount": 42,
      "replyCount": 1337,
      "lastPostAt": "2026-03-28T23:45:00.000Z"
    }
  ]
}
```

---

## スレッドをフォロー

特定のスレッドをフォローし、新しいリプライが投稿された時に通知を受け取ります。

```
POST /api/boards/:slug/threads/:id/follow
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `id` | number | Yes | Thread ID |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スレッドをフォローしました"}` | Thread followed |
| 400 | `{"error": "既にフォローしています"}` | Already following |

---

## スレッドのフォロー解除

スレッドのフォローを解除します。

```
DELETE /api/boards/:slug/threads/:id/follow
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `id` | number | Yes | Thread ID |

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スレッドのフォローを解除しました"}` | Thread unfollowed |

---

## スレッドにリアクション

スレッドのオリジナル投稿に絵文字リアクションを追加します。

```
POST /api/boards/:slug/threads/:id/reactions
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `id` | number | Yes | Thread ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `emoji` | string | Yes | Emoji text (max 32 characters) |

### Example

```http
POST /api/boards/general-discussion/threads/101/reactions HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "emoji": "👍"
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "リアクションしました"}` | Reaction added |
| 400 | `{"error": "既にリアクションしています"}` | Already reacted with this emoji |

---

## リプライにリアクション

掲示板のリプライに絵文字リアクションを追加します。

```
POST /api/boards/:slug/replies/:replyId/reactions
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `replyId` | number | Yes | Reply ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `emoji` | string | Yes | Emoji text (max 32 characters) |

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "リアクションしました"}` | Reaction added |
| 400 | `{"error": "既にリアクションしています"}` | Already reacted with this emoji |

---

## リアクションユーザー取得（スレッド）

特定の絵文字でスレッドにリアクションしたユーザーを返します。

```
GET /api/boards/:slug/threads/:id/reactions/:emoji/users
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `id` | number | Yes | Thread ID |
| `emoji` | string | Yes | URL-encoded emoji |

### Response

```json
{
  "users": [
    {
      "id": 18179,
      "username": "reactor",
      "displayName": "Reactor",
      "avatarUrl": "/uploads/avatars/abc.webp"
    }
  ]
}
```

---

## リアクションユーザー取得（リプライ）

特定の絵文字でリプライにリアクションしたユーザーを返します。

```
GET /api/boards/:slug/replies/:replyId/reactions/:emoji/users
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |
| `replyId` | number | Yes | Reply ID |
| `emoji` | string | Yes | URL-encoded emoji |

### Response

```json
{
  "users": [
    {
      "id": 18179,
      "username": "reactor",
      "displayName": "Reactor",
      "avatarUrl": "/uploads/avatars/abc.webp"
    }
  ]
}
```

---

## サーバー送信イベント (SSE)

掲示板はServer-Sent Eventsによるリアルタイム更新をサポートしています。ストリームエンドポイントに接続して、新しいスレッドやリプライが投稿された時にライブ更新を受信します。

```
GET /api/boards/:slug/stream
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `slug` | string | Yes | Board slug |

### Event Types

| イベント名 | ペイロード | 説明 |
|------------|---------|-------------|
| `board-update` | JSON object | New thread or reply was posted |

### Example (JavaScript)

```javascript
const eventSource = new EventSource(
  "https://karotter.com/api/boards/general-discussion/stream",
  {
    headers: {
      "Authorization": "Bearer eyJ..."
    }
  }
);

eventSource.addEventListener("board-update", (event) => {
  const data = JSON.parse(event.data);
  console.log("Board updated:", data);
  // data could be a new thread or reply
});

eventSource.onerror = (error) => {
  console.error("SSE connection error:", error);
};
```

### SSE Data Format

```
event: board-update
data: {"type":"new-thread","thread":{"id":102,"title":"New thread!","authorId":18179}}

event: board-update
data: {"type":"new-reply","threadId":101,"reply":{"id":504,"replyNumber":4,"content":"New reply!"}}
```

> **注意:** SSE connections require authentication. The connection will be closed if the token expires.

---

## コンテンツパース

Board thread and reply content supports several inline formatting and linking conventions.

### Anchor References (`>>N`)

Reference other replies by their reply number. The frontend renders these as clickable links that scroll to and highlight the referenced reply.

```
>>5 I agree with this point
>>0 refers to the original thread post
```

| パターン | 説明 |
|---------|-------------|
| `>>0` | Reference to the thread's original post |
| `>>1` | Reference to reply number 1 |
| `>>N` | Reference to reply number N |

### URLs

Plain URLs are automatically detected and rendered as clickable links.

```
Check out https://example.com for more info
```

### Hashtags (`#tag`)

Hashtags are detected and rendered as links to the hashtag search page.

```
This is about #programming and #typescript
```

### Mentions (`@username`)

Mentions are detected and rendered as links to the user's profile.

```
Thanks @claude for the help!
```

### Example of All Parseable Content

```
>>3 Great point about #rust!
@devperson you should check https://doc.rust-lang.org
I wrote more about this here: >>1
```

---

## 年齢制限

Boards support age-restricted access via the `minimumAge` field.

### How It Works

| Condition | Result |
|-----------|--------|
| User is not logged in | `401` error |
| User's age is below `minimumAge` | `403` error |
| User's age is at or above `minimumAge` | Access granted |
| User has no birthday set | Behavior depends on board settings |

### Age Range

| プロパティ | 値 |
|----------|-------|
| Minimum allowed `minimumAge` | 13 |
| Maximum allowed `minimumAge` | 99 |

### Age Calculation

Age is calculated from the user's `birthday` field (ISO 8601 date) in their profile. Users who have not set a birthday may be treated as under-age for restricted boards.

---

## オブジェクトリファレンス: Board

Full Board object structure.

```json
{
  "id": 5,
  "title": "General Discussion",
  "slug": "general-discussion",
  "description": "Talk about anything!",
  "minimumAge": 13,
  "threadCount": 42,
  "replyCount": 1337,
  "lastPostAt": "2026-03-28T23:45:00.000Z",
  "creator": {
    "id": 15459
  }
}
```

### Board（掲示板） Field Reference

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | number | Board ID |
| `title` | string | Board title |
| `slug` | string | URL-friendly slug (used in all board URLs) |
| `description` | string \| null | Board description |
| `minimumAge` | number | Minimum age to access (13-99) |
| `threadCount` | number | Total threads in this board |
| `replyCount` | number | Total replies across all threads |
| `lastPostAt` | string \| null | ISO 8601 timestamp of most recent activity |
| `creator` | object | Board creator |
| `creator.id` | number | Creator's user ID |

---

## オブジェクトリファレンス: Thread

Full Thread object structure.

```json
{
  "id": 101,
  "title": "What's your favorite programming language?",
  "content": "I've been learning Rust lately and I'm really enjoying it. What do you all use?",
  "authorId": 18179,
  "author": {
    "displayName": "Dev Person",
    "username": "devperson",
    "avatarUrl": "/uploads/avatars/abc.webp"
  },
  "imageUrls": ["/uploads/boards/screenshot.png"],
  "replyCount": 23,
  "lastReplyAt": "2026-03-28T23:30:00.000Z",
  "createdAt": "2026-03-28T20:00:00.000Z"
}
```

### Thread（スレッド） Field Reference

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | number | Thread ID |
| `title` | string | Thread title |
| `content` | string | Thread body text |
| `authorId` | number | Author's user ID |
| `author` | object | Author information |
| `author.displayName` | string | Author's display name |
| `author.username` | string | Author's username |
| `author.avatarUrl` | string \| null | Author's avatar URL (relative path) |
| `imageUrls` | string[] | Array of attached image URLs (relative paths, max 4) |
| `replyCount` | number | Number of replies |
| `lastReplyAt` | string \| null | ISO 8601 timestamp of the latest reply |
| `createdAt` | string | ISO 8601 creation timestamp |

---

## オブジェクトリファレンス: Reply

Full Reply object structure.

```json
{
  "id": 501,
  "replyNumber": 1,
  "content": "TypeScript all the way! >>0 great question though.",
  "authorId": 22704,
  "author": {
    "username": "tsdev",
    "displayName": "TS Developer",
    "avatarUrl": "/uploads/avatars/xyz.webp"
  },
  "imageUrls": [],
  "createdAt": "2026-03-28T20:05:00.000Z"
}
```

### Reply Field Reference

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | number | Reply ID (database primary key) |
| `replyNumber` | number | Sequential reply number within the thread (1-indexed) |
| `content` | string | Reply text content (supports `>>N`, URLs, `#hashtags`, `@mentions`) |
| `authorId` | number | Author's user ID |
| `author` | object | Author information |
| `author.username` | string | Author's username |
| `author.displayName` | string | Author's display name |
| `author.avatarUrl` | string \| null | Author's avatar URL (relative path) |
| `imageUrls` | string[] | Array of attached image URLs (relative paths, max 4) |
| `createdAt` | string | ISO 8601 creation timestamp |

---

## レート制限

すべての掲示板エンドポイントはデフォルトのレート制限を共有しています:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
