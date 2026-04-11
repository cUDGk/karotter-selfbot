# 検索

検索エンドポイント、トレンドトピック、ディスカバリーフィード、検索履歴の完全なリファレンスです。

---

## 統合検索

### `GET /search`

投稿、ユーザー、ハッシュタグを横断的に検索します。すべてのカテゴリの結果を1つのレスポンスで返します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `q` | `string` | — | 検索クエリ文字列（必須） |
| `limit` | `number` | `12` | カテゴリあたりの最大結果数 |

**リクエスト:**

```http
GET /search?q=karotter&limit=12 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**ページネーションオブジェクト:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `hasNext` | `boolean` | さらに結果があるかどうか |
| `nextCursor` | `string \| null` | 次ページの結果用カーソル |
| `page` | `number` | 現在のページ番号 |
| `pages` | `number` | 利用可能な総ページ数 |

---

## 投稿検索

### `GET /search/posts`

追加のフィルタリングとソートオプションで投稿を検索します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `q` | `string` | — | 検索クエリ文字列（必須） |
| `sort` | `string` | `latest` | ソート順（下記参照） |
| `hasMedia` | `boolean` | — | メディア添付の投稿のみにフィルタ |
| `limit` | `number` | `12` | 1ページあたりの結果数 |
| `cursor` | `string` | — | 前回のレスポンスからのページネーションカーソル |

**ソート値:**

| ソート | 説明 |
|------|-------------|
| `latest` | 最新の投稿順（時系列） |
| `topics` | クエリとのトピック関連性でソート |

**リクエスト:**

```http
GET /search/posts?q=sunset&sort=latest&hasMedia=true&limit=12 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**リクエスト（次ページ）:**

```http
GET /search/posts?q=sunset&sort=latest&hasMedia=true&limit=12&cursor=cursor_posts_def456 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

---

## ユーザー検索

### `GET /search/users`

ユーザー名または表示名でユーザーを検索します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `q` | `string` | — | 検索クエリ文字列（必須） |
| `limit` | `number` | `12` | 1ページあたりの結果数。サジェストドロップダウンには`6`、完全な検索結果には`12`を使用。 |

**リクエスト（サジェストモード）:**

```http
GET /search/users?q=karo&limit=6 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**リクエスト（フル検索）:**

```http
GET /search/users?q=karotter&limit=12 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**ユーザー検索のReact Query設定:**

| 設定 | 値 |
|---------|-------|
| `staleTime` | `15000`（15秒） |

---

## ハッシュタグ検索

### `GET /search/hashtags`

名前でハッシュタグを検索します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `q` | `string` | — | ハッシュタグ検索クエリ（`#`プレフィックスなし） |

**リクエスト:**

```http
GET /search/hashtags?q=sunset HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**ハッシュタグオブジェクト:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | `string` | ユニークなハッシュタグ識別子 |
| `name` | `string` | ハッシュタグテキスト（`#`プレフィックスなし） |
| `usageCount` | `number` | このハッシュタグを使用した投稿の総数 |
| `trendScore` | `number` | トレンドスコア（高いほど最近トレンド） |

---

## トレンドトピック

### `GET /search/trending/topics`

現在のトレンドトピック/ハッシュタグを取得します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | 返すトレンドトピックの最大数 |

**リクエスト:**

```http
GET /search/trending/topics?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**トレンドオブジェクト:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `type` | `string` | 常に `HASHTAG`（現在サポートされている唯一のタイプ） |
| `label` | `string` | `#`プレフィックス付きの表示ラベル |
| `postCount` | `number` | トレンド期間中にこのトピックを使用した投稿数 |
| `token` | `string` | 検索クエリで使用する生のトークン値（`#`なしのハッシュタグ名） |

---

## ディスカバーフィード

### `GET /search/discover/latest`

プラットフォーム全体の最新投稿を発見します（フォロー中のユーザーに限定されません）。

### `GET /search/discover/media`

メディア（画像/動画）を含む投稿を発見します。

### `GET /search/discover/topics`

トレンドトピックで整理された投稿を発見します。

**クエリパラメータ（3つのエンドポイント共通）:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `12` | 1ページあたりの結果数 |
| `cursor` | `string` | — | 前回のレスポンスからのページネーションカーソル |

**リクエスト:**

```http
GET /search/discover/latest?limit=12 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`（`/discover/latest`の例）:**

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

**リクエスト（次ページ）:**

```http
GET /search/discover/latest?limit=12&cursor=cursor_discover_ghi789 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`（`/discover/media`の例）:**

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

## 検索履歴

検索履歴は**クライアント側**の `localStorage` に保存されます。サーバーでは管理されません。

**ストレージキーフォーマット:**

```
karotter-search-history-v1:{userId}
```

**例:** ユーザー `clx1abc2d3ef4gh5ij6kl7mn8` の場合、キーは:

```
karotter-search-history-v1:clx1abc2d3ef4gh5ij6kl7mn8
```

**ストレージフォーマット:**

値はJSON形式の検索クエリ文字列の配列で、最新のものから古いものの順に並んでいます。

```json
["karotter", "sunset photos", "gamedev", "minecraft mod"]
```

**制限:**

| 設定 | 値 | 説明 |
|---------|-------|-------------|
| 最大エントリ数 | `12` | 最新の12件の検索クエリのみ保存 |

**動作:**
- 新しい検索が実行されると、配列の先頭に追加されます。
- クエリが既に履歴に存在する場合、先頭に移動されます（重複排除）。
- 配列が12エントリを超えると、最も古いエントリが削除されます。
- 各ユーザーIDには独立した検索履歴があります。
- 検索履歴は`localStorage`を使用するため、ブラウザ/デバイスごとに独立しています。
- サーバーは検索履歴を把握しておらず、読み書き用のAPIエンドポイントはありません。

**実装例:**

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
