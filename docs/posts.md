# 投稿

投稿の作成、取得、編集、削除、およびいいね、リカロット、ブックマーク、リアクション、投票、リプライ、アナリティクス、予約投稿を含むすべての投稿インタラクションの完全なリファレンスです。

---

## 投稿の作成

### `POST /posts`

新しい投稿を作成します。メディアアップロードをサポートするため `multipart/form-data` を使用します。

**リクエスト:** `multipart/form-data`

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `content` | `string` | いいえ | 投稿のテキスト内容。メディアが添付されている場合は空でも可 |
| `media` | `File[]` | いいえ | メディアファイル。FormDataで各ファイルに同じフィールド名 `media` を使用 |
| `mediaAlts` | `string` (JSON) | いいえ | 各メディアアイテムの代替テキスト文字列のJSON配列。例: `'["画像1の代替テキスト","画像2の代替テキスト"]'` |
| `mediaSpoilerFlags` | `string` (JSON) | いいえ | 各メディアアイテムのスポイラーステータスを示すブール値のJSON配列。例: `'[false,true]'` |
| `mediaR18Flags` | `string` (JSON) | いいえ | 各メディアアイテムのR18/NSFWステータスを示すブール値のJSON配列。例: `'[false,false]'` |
| `isAiGenerated` | `boolean` | いいえ | AI生成コンテンツフラグ。**注意:** このフィールドはAPI経由で送信しても動作せず、サーバー側で無視されます |
| `visibility` | `string` | いいえ | 投稿の公開範囲: `PUBLIC`（デフォルト）または `CIRCLE` |
| `replyRestriction` | `string` | いいえ | リプライ可能な人: `EVERYONE`（デフォルト）, `FOLLOWING`, `MENTIONED`, `CIRCLE` |
| `parentId` | `string` | いいえ | 親投稿のID（これをリプライにする） |

**リクエスト例（テキストのみ）:**

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

**リクエスト例（画像付き）:**

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

**レスポンス `201 Created`:**

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

> **重要: 画像と動画の混在不可。** 同じ投稿に画像ファイルと動画ファイルの両方をアップロードしようとすると、サーバーは `400 Bad Request` エラーを返します。各投稿には画像または動画のいずれかを含められますが、両方は不可です。

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `400` | `MIXED_MEDIA_TYPES` | 1つの投稿に画像と動画の両方を含めることはできない |
| `400` | `CONTENT_REQUIRED` | 投稿にコンテンツもメディアもない |
| `400` | `MEDIA_LIMIT_EXCEEDED` | 添付メディアファイルが多すぎる |
| `413` | `PAYLOAD_TOO_LARGE` | メディアファイルサイズが上限を超過 |

---

## 投稿の取得

### `GET /posts/:id`

IDを指定して単一の投稿を取得します。

**リクエスト:**

```http
GET /posts/post_abc123 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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
        "emoji": "\ud83d\udc4d",
        "count": 12,
        "hasReacted": false
      },
      {
        "emoji": "\u2764\ufe0f",
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

### 完全な投稿オブジェクトリファレンス

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | `string` | 一意の投稿識別子 |
| `content` | `string` | 投稿のテキスト内容 |
| `author` | `object` | 投稿者ユーザーオブジェクト (id, username, displayName, avatarUrl, isPrivate, officialMark, isParodyAccount, adminForceParody, isBotAccount, adminForceBot) |
| `visibility` | `string` | `PUBLIC` または `CIRCLE` |
| `replyRestriction` | `string` | `EVERYONE`, `FOLLOWING`, `MENTIONED`, `CIRCLE` |
| `media` | `array` | メディアオブジェクトの配列 |
| `media[].id` | `string` | メディアアイテムID |
| `media[].url` | `string` | メディアファイルのCDN URL |
| `media[].type` | `string` | `IMAGE` または `VIDEO` |
| `media[].width` | `number` | ピクセル単位の幅 |
| `media[].height` | `number` | ピクセル単位の高さ |
| `media[].alt` | `string \| null` | 代替テキストの説明 |
| `media[].isSpoiler` | `boolean` | スポイラーとしてマークされているか |
| `media[].isR18` | `boolean` | R18/NSFWとしてマークされているか |
| `media[].isAiGenerated` | `boolean` | AI生成としてフラグ付けされているか |
| `poll` | `object \| null` | 投稿に投票が含まれる場合の投票オブジェクト |
| `poll.id` | `string` | 投票ID |
| `poll.options` | `array` | `{id, text, votesCount}` の配列 |
| `poll.totalVotes` | `number` | 総投票数 |
| `poll.expiresAt` | `string` | ISO 8601 有効期限タイムスタンプ |
| `poll.hasVoted` | `boolean` | 閲覧者が投票したかどうか |
| `poll.votedOptionId` | `string \| null` | 閲覧者が投票した選択肢 |
| `parentId` | `string \| null` | リプライの場合の親投稿ID |
| `parentPost` | `Post \| null` | 親投稿オブジェクト（含まれる場合） |
| `quotedPost` | `Post \| null` | 引用投稿オブジェクト（引用投稿の場合） |
| `quotedPostId` | `string \| null` | 引用投稿ID |
| `createdAt` | `string` | ISO 8601 作成タイムスタンプ |
| `updatedAt` | `string` | ISO 8601 最終更新タイムスタンプ |
| `editedAt` | `string \| null` | ISO 8601 編集タイムスタンプ。未編集の場合は `null` |
| `likesCount` | `number` | いいね数 |
| `repliesCount` | `number` | 直接リプライ数 |
| `rekarotsCount` | `number` | リカロット（リポスト）数 |
| `quotesCount` | `number` | 引用投稿数 |
| `bookmarksCount` | `number` | ブックマーク数 |
| `viewCount` | `number` | 閲覧数 |
| `isLiked` | `boolean` | 閲覧者がこの投稿にいいねしたか |
| `isRekaroted` | `boolean` | 閲覧者がこの投稿をリカロットしたか |
| `isBookmarked` | `boolean` | 閲覧者がこの投稿をブックマークしたか |
| `myReactions` | `string[]` | 閲覧者がリアクションした絵文字文字列の配列 |
| `reactions` | `array` | `{emoji, count, hasReacted}` オブジェクトの配列 |
| `isMutedByViewer` | `boolean` | 閲覧者がこの投稿の会話をミュートしたか |
| `scheduledFor` | `string \| null` | ISO 8601 予約公開時刻。公開済み投稿は `null` |
| `viewerCircle` | `object \| null` | 公開範囲が `CIRCLE` の場合のサークルオブジェクト |

---

## 投稿の編集

### `PUT /posts/:id`

既存の投稿を編集します。投稿者のみ編集可能です。

**リクエスト:** `POST /posts` と同じ形式（multipart/form-data）。

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "post": {
    "id": "post_abc123",
    "content": "Updated post content here.",
    "editedAt": "2026-03-30T12:30:00.000Z"
  }
}
```

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `403` | `FORBIDDEN` | この投稿の投稿者ではない |
| `404` | `NOT_FOUND` | 投稿が存在しない |

---

## 投稿の削除

### `DELETE /posts/:id`

投稿を削除します。投稿者のみ削除可能です。

**リクエスト:**

```http
DELETE /posts/post_abc123 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "カロートを削除しました"
}
```

> **注意:** 成功メッセージは日本語で返されます: "カロートを削除しました"（Karotが削除されたという意味）。

---

## いいね / いいね解除

### `POST /posts/:id/like`

投稿にいいねします。

### `DELETE /posts/:id/like`

いいねを取り消します。

**リクエスト:**

```http
POST /posts/post_abc123/like HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "Post liked"
}
```

**クライアント動作の注意:**
- クライアントは**楽観的更新**を実行します: UIはいいね/いいね解除を即座に反映します。
- API呼び出し後に**1500ms遅延キャッシュ無効化**がトリガーされ、実際のサーバー状態を同期します。

---

## リカロット / リカロット解除

### `POST /posts/:id/rekarot`

投稿をフォロワーにリカロット（リポスト）します。

### `DELETE /posts/:id/rekarot`

リカロットを取り消します。

**リクエスト:**

```http
POST /posts/post_abc123/rekarot HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "Rekaroted"
}
```

**インタラクションガード:** クライアントはリカロット許可前に以下の条件を確認します:
- `canInteract` が `true`（ユーザーがブロックされていない等）
- 投稿者が非公開アカウントではない（フォローしている場合を除く）
- 投稿の公開範囲が `CIRCLE` ではない（サークル限定投稿はリカロット不可）

---

## ブックマーク / ブックマーク解除

### `POST /posts/:id/bookmark`

投稿をブックマークします。

### `DELETE /posts/:id/bookmark`

ブックマークを解除します。

**リクエスト:**

```http
POST /posts/post_abc123/bookmark HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "Post bookmarked"
}
```

---

## リアクション

### `POST /posts/:id/react`

投稿に絵文字リアクションを追加します。

**リクエストボディ:**

| フィールド | 型 | 必須 | 制約 | 説明 |
|-------|------|----------|-------------|-------------|
| `emoji` | `string` | はい | 最大32文字 | リアクションする絵文字 |

**制限:**
- 1つの投稿あたり最大**20種類のユニークな絵文字**（全ユーザー合計）。
- 絵文字文字列は最大**32文字**（サーバー側で厳密にバリデーション）。

**リクエスト:**

```http
POST /posts/post_abc123/react HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "emoji": "\ud83d\udd25"
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "Reaction added"
}
```

### `DELETE /posts/:id/react/:emoji`

投稿からリアクションを削除します。

**パスパラメータ:**

| パラメータ | 型 | 説明 |
|-----------|------|-------------|
| `emoji` | `string` | URLエンコードされた絵文字文字列 |

**リクエスト:**

```http
DELETE /posts/post_abc123/react/%F0%9F%94%A5 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

> **注意:** 絵文字はパスでURLエンコードする必要があります。例えば、火の絵文字は `%F0%9F%94%A5` になります。

**レスポンス `200 OK`:**

```json
{
  "message": "Reaction removed"
}
```

### `GET /posts/:id/react/:emoji/users`

特定の絵文字でリアクションしたユーザーの一覧を取得します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `10` | ページあたりの結果数 |
| `cursor` | `string` | -- | ページネーションカーソル |

**リクエスト:**

```http
GET /posts/post_abc123/react/%F0%9F%94%A5/users?limit=10 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

## 投票

### `POST /posts/:id/poll/vote`

投票の選択肢に投票します。このエンドポイントは**トグル**として動作し、同じ `optionId` で再度投稿すると投票が取り消されます。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `optionId` | `string` | はい | 投票する選択肢のID |

**リクエスト:**

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

**レスポンス `200 OK`:**

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

## リプライ

### `GET /posts/:id/replies`

投稿へのリプライを取得します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | `number` | `1` | ページ番号 |
| `limit` | `number` | `20` | ページあたりの結果数 |

**リクエスト:**

```http
GET /posts/post_abc123/replies?page=1&limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**クライアント側フィルタリング:** クライアントは以下の条件に該当するリプライをフィルタリングします:
- `isMutedByViewer` が `true`
- リプライの投稿者が閲覧者にブロックされている
- 閲覧者がリプライの投稿者にブロックされている

### `GET /posts/:id/reply-targets`

リプライに至るまでの親投稿チェーン（「リプライターゲット」または上位の会話スレッド）を取得します。

**リクエスト:**

```http
GET /posts/post_reply001/reply-targets HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

## いいねリスト

### `GET /posts/:id/likes`

投稿にいいねしたユーザーの一覧を取得します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | ページあたりの結果数 |
| `cursor` | `string` | -- | ページネーションカーソル |

**リクエスト:**

```http
GET /posts/post_abc123/likes?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

## リカロットユーザー

### `GET /posts/:id/rekarots`

投稿をリカロットしたユーザーの一覧を取得します（追加コメントを含む）。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | ページあたりの結果数 |
| `cursor` | `string` | -- | ページネーションカーソル |

**リクエスト:**

```http
GET /posts/post_abc123/rekarots?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

## 引用投稿

### `GET /posts/:id/quotes`

この投稿を引用している投稿を取得します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | ページあたりの結果数 |
| `cursor` | `string` | -- | ページネーションカーソル |

**リクエスト:**

```http
GET /posts/post_abc123/quotes?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

## 投稿アナリティクス

### `GET /posts/:id/analytics`

投稿のオーディエンスアナリティクスを取得します。投稿者のみ利用可能です。

**リクエスト:**

```http
GET /posts/post_abc123/analytics HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**フィールド:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `audience.knownViewerCount` | `number` | 属性情報が判明している閲覧者の合計 |
| `audience.genderCounts.MALE` | `number` | 男性閲覧者数 |
| `audience.genderCounts.FEMALE` | `number` | 女性閲覧者数 |
| `audience.genderCounts.OTHER` | `number` | その他の性別の閲覧者数 |
| `audience.ageBuckets["13-17"]` | `number` | 13-17歳の閲覧者数 |
| `audience.ageBuckets["18-24"]` | `number` | 18-24歳の閲覧者数 |
| `audience.ageBuckets["25-34"]` | `number` | 25-34歳の閲覧者数 |
| `audience.ageBuckets["35-44"]` | `number` | 35-44歳の閲覧者数 |
| `audience.ageBuckets["45-54"]` | `number` | 45-54歳の閲覧者数 |
| `audience.ageBuckets["55+"]` | `number` | 55歳以上の閲覧者数 |
| `audience.ageBuckets["unknown"]` | `number` | 年齢不明の閲覧者数 |

**エラーレスポンス:**

| ステータス | エラー | 説明 |
|--------|-------|-------------|
| `403` | `FORBIDDEN` | この投稿の投稿者ではない |

---

## 会話を離脱

### `POST /posts/:id/conversation/leave`

会話スレッドをミュート/離脱します。このスレッドのリプライ通知を受け取らなくなります。

**リクエスト:**

```http
POST /posts/post_abc123/conversation/leave HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "Left conversation"
}
```

---

## ピン留め / ピン解除

### `PATCH /users/profile/pinned-post`

投稿をプロフィールにピン留め、または現在のピン留めを解除します。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `postId` | `string \| null` | はい | ピン留めする投稿ID、またはピン解除の場合は `null` |

**リクエスト（ピン留め）:**

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

**リクエスト（ピン解除）:**

```json
{
  "postId": null
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "Pinned post updated"
}
```

---

## バッチビュー

### `POST /posts/batch-views`

ユーザーが投稿のバッチを閲覧したことを報告します。クライアントがこのエンドポイントを自動的に呼び出します。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `postIds` | `string[]` | はい | 閲覧した投稿IDの配列 |

**クライアント動作:**
- 蓄積された投稿IDを**5秒ごと**に送信します。
- 投稿が少なくとも**1秒間**表示され（滞在時間）、投稿の少なくとも**50%**がビューポート内にある（可視性しきい値）場合に「閲覧済み」とカウントされます。

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "message": "Views recorded"
}
```

---

## 予約投稿

### `GET /posts/scheduled/me`

認証済みユーザーの予約（未公開）投稿を取得します。

**リクエスト:**

```http
GET /posts/scheduled/me HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**予約投稿オブジェクト:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | `string` | 予約投稿ID |
| `content` | `string` | 投稿テキスト内容 |
| `scheduledFor` | `string` | ISO 8601 投稿が公開される予定のタイムスタンプ |
| `visibility` | `string` | `PUBLIC` または `CIRCLE` |
| `viewerCircle` | `object \| null` | 公開範囲が `CIRCLE` の場合のサークルオブジェクト |
| `pollOptions` | `array \| null` | 投稿に投票が含まれる場合の `{text}` オブジェクトの配列 |

### `DELETE /posts/scheduled/:id`

公開前の予約投稿を削除します。

**リクエスト:**

```http
DELETE /posts/scheduled/post_sched001 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**レスポンス `200 OK`:**

```json
{
  "message": "Scheduled post deleted"
}
```
