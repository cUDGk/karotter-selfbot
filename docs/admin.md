# 管理 API

Karotter管理パネルAPIの完全なリファレンスです。アクセスには `user.isAdmin === true` が必要です。

---

## 概要

### フロントエンドアクセス

管理パネルは難読化されたURLでアクセス可能です:

```
https://karotter.com/control-room-x9k2
```

このURLは公開UIのどこにもリンクされていません。管理者のみが知っています。

### APIプレフィックス

すべての管理APIエンドポイントには以下のプレフィックスが付きます:

```
/api/admin
```

For example: `https://api.karotter.com/api/admin/analytics`

### 認証

Admin endpoints require:

1. A valid `Authorization: Bearer <token>` header (same as all authenticated endpoints)
2. The authenticated user must have `isAdmin === true`
3. Standard `x-csrf-token` header for mutating requests

非管理者ユーザーはすべての管理エンドポイントで `403 Forbidden` を受け取ります。

---

## 管理パネルタブ

The admin panel frontend is organized into 8 tabs:

| タブ | ルート | 説明 |
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

## ダッシュボード

### `GET /admin/analytics`

プラットフォーム全体のアナリティクスサマリーを取得します。

**レスポンス `200 OK`:**

```json
{
  "totalUsers": 15230,
  "totalPosts": 892451,
  "activeUsers": 3847,
  "totalReports": 156
}
```

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `totalUsers` | `number` | Total registered users on the platform |
| `totalPosts` | `number` | Total posts ever created |
| `activeUsers` | `number` | Users active in the recent period |
| `totalReports` | `number` | Total pending/active reports |

---

## ユーザー管理

### `GET /admin/users`

検索とカーソルベースページネーションでユーザーを一覧表示します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | Number of users per page (max 100) |
| `search` | `string` | — | Search by username (prefix with `@` to search by exact username) |
| `cursor` | `string` | — | Pagination cursor from previous response |

**リクエスト:**

```http
GET /admin/users?limit=30&search=@testuser HTTP/1.1
```

**レスポンス `200 OK`:**

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

**ユーザーオブジェクトフィールド（管理者ビュー）:**

| フィールド | 型 | 説明 |
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

ユーザーアカウントをBANします。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | User ID to ban |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `reason` | `string` | Yes | Reason for the ban (stored for audit) |

**リクエスト:**

```http
PATCH /admin/users/12345/ban HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "reason": "Repeated harassment violations"
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "User banned successfully"
}
```

---

### `PATCH /admin/users/:id/unban`

以前にBANされたユーザーアカウントのBANを解除します。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | User ID to unban |

**リクエストボディ:** Empty object `{}`

**リクエスト:**

```http
PATCH /admin/users/12345/unban HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{}
```

**レスポンス `200 OK`:**

```json
{
  "message": "User unbanned successfully"
}
```

---

### `PATCH /admin/users/:id/account`

ユーザーのコアアカウントフィールドを編集します。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | User ID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `username` | `string` | No | New username |
| `displayName` | `string` | No | New display name |
| `email` | `string` | No | New email address |
| `password` | `string` | No | New password (plain text, server hashes it) |
| `emailVerified` | `boolean` | No | Override email verification status |

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "message": "User account updated",
  "user": { ... }
}
```

**注意:**
- All fields are optional; only provided fields are updated.
- Changing the `password` invalidates all existing sessions for that user.

---

### `PATCH /admin/users/:id/official-mark`

ユーザーの公式認証マークを設定または更新します。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | User ID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `officialMark` | `string[]` | Yes | Array of mark color codes. Pass `[]` to remove all marks. |

**Important:** The `officialMark` field is an **array**. A user can have multiple marks simultaneously (e.g., `["BLUE", "PURPLE"]` for a verified admin).

**リクエスト:**

```http
PATCH /admin/users/12345/official-mark HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "officialMark": ["BLUE", "PURPLE"]
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "Official mark updated",
  "officialMark": ["BLUE", "PURPLE"]
}
```

**利用可能なマークカラー:**

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

ユーザーアカウントのモデレーションフラグを切り替えます。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | User ID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
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

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "message": "User flags updated"
}
```

---

### `DELETE /admin/users/:id`

ユーザーアカウントと関連するすべてのデータを完全に削除します。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | User ID to delete |

**リクエスト:**

```http
DELETE /admin/users/12345 HTTP/1.1
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>
```

**レスポンス `200 OK`:**

```json
{
  "message": "User deleted successfully"
}
```

**Warning:** This action is irreversible. All posts, DMs, reactions, and other data associated with the user are permanently removed.

---

## 投稿管理

### `GET /admin/posts`

検索とカーソルベースページネーションで投稿を一覧表示します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | Number of posts per page |
| `search` | `string` | — | Search by post content |
| `cursor` | `string` | — | Pagination cursor |

**リクエスト:**

```http
GET /admin/posts?limit=30&search=keyword HTTP/1.1
```

**レスポンス `200 OK`:**

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

**投稿オブジェクトフィールド（管理者ビュー）:**

| フィールド | 型 | 説明 |
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

投稿のモデレーションフラグを切り替えます。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | Post ID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `adminForceR18` | `boolean` | No | Force the post to be marked as R18/NSFW |
| `adminForceHidden` | `boolean` | No | Force-hide the post from all timelines |

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "message": "Post flags updated"
}
```

---

### `DELETE /admin/posts/:id`

投稿を完全に削除します。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | Post ID to delete |

**リクエスト:**

```http
DELETE /admin/posts/98765 HTTP/1.1
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>
```

**レスポンス `200 OK`:**

```json
{
  "message": "Post deleted successfully"
}
```

---

## ストーリー管理

### `GET /admin/stories`

モデレーション用のストーリーを一覧表示します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | Number of stories per page |
| `cursor` | `string` | — | Pagination cursor |

**レスポンス `200 OK`:**

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

ストーリーのモデレーションフラグを切り替えます。

**パスパラメータ:**

| Parameter | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | Story ID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `adminForceR18` | `boolean` | No | Force-flag the story as R18 |
| `adminForceHidden` | `boolean` | No | Force-hide the story |

**リクエスト:**

```http
PATCH /admin/stories/5678/flags HTTP/1.1
Content-Type: application/json
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>

{
  "adminForceR18": true
}
```

**レスポンス `200 OK`:**

```json
{
  "message": "Story flags updated"
}
```

---

### `DELETE /admin/stories/:id`

ストーリーを完全に削除します。

**リクエスト:**

```http
DELETE /admin/stories/5678 HTTP/1.1
Authorization: Bearer <admin-token>
x-csrf-token: <csrf-token>
```

**レスポンス `200 OK`:**

```json
{
  "message": "Story deleted successfully"
}
```

---

## おすすめアルゴリズムテスト

### `GET /admin/test-recommend`

完全なスコアリング詳細でおすすめアルゴリズムをテストします。このエンドポイントは重い計算のためタイムアウトが延長されています。

**タイムアウト:** 60 seconds

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `userId` | `number` | — | Optional: test recommendations for a specific user ID |
| `limit` | `number` | `30` | Number of recommended posts to return |

**リクエスト:**

```http
GET /admin/test-recommend?userId=12345&limit=30 HTTP/1.1
Authorization: Bearer <admin-token>
```

**レスポンス `200 OK`:**

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

**スコアフィールド:**

| スコア | 範囲 | 説明 |
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

## トレンドアルゴリズムテスト

### `GET /admin/test-trending`

詳細なウィンドウ統計でトレンドアルゴリズムをテストします。

**タイムアウト:** 60 seconds

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | Number of trending posts to return |

**リクエスト:**

```http
GET /admin/test-trending?limit=30 HTTP/1.1
Authorization: Bearer <admin-token>
```

**レスポンス `200 OK`:**

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

**ウィンドウ統計フィールド:**

| フィールド | 型 | 説明 |
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

## アンケート結果

### `GET /admin/survey-results`

プラットフォーム全体の集計されたアンケート/投票結果を取得します。

**タイムアウト:** 30 seconds

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `10` | Number of surveys to return |

**リクエスト:**

```http
GET /admin/survey-results?limit=10 HTTP/1.1
Authorization: Bearer <admin-token>
```

**レスポンス `200 OK`:**

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

## ベータ実験結果

### `GET /admin/beta-experiment`

バリアント別統計付きのA/Bテスト実験結果を取得します。

**リクエスト:**

```http
GET /admin/beta-experiment HTTP/1.1
Authorization: Bearer <admin-token>
```

**レスポンス `200 OK`:**

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

**バリアントオブジェクトフィールド:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `variant` | `string` | Variant identifier: `"A"`, `"B"`, `"C"`, `"D"`, or `"E"` |
| `label` | `string` | Human-readable description of the variant |
| `uniqueUsers` | `number` | Number of unique users assigned to this variant |
| `impressions` | `number` | Total post impressions shown under this variant |
| `survey.preferBeta` | `number` | Users who preferred the beta/new variant in the survey |
| `survey.preferCurrent` | `number` | Users who preferred the current/control variant |

---

## 公式マークカラーリファレンス

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
