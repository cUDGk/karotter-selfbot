# タイムライン

タイムラインフィード、おすすめ投稿、リストフィード、ベータA/Bテスト、フィード設定の完全なリファレンスです。

---

## タイムライン Feed

### `GET /posts/timeline`

ユーザーのメインタイムライン（フォロー中ユーザーの投稿）を取得します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | `number` | `1` | ページ番号（1始まり） |
| `limit` | `number` | `12` | 1ページあたりの結果数 |
| `mode` | `string` | `latest` | フィードソートモード（下記参照） |

**モード値:**

| モード | 説明 |
|------|-------------|
| `latest` | 時系列順（新しい順） |
| `ranked` | エンゲージメントと関連性によるアルゴリズムランキング |
| `trending` | 現在トレンドの投稿 |
| `following` | フォロー中ユーザーの投稿（デフォルト動作のエイリアス） |

**リクエスト:**

```http
GET /posts/timeline?page=1&limit=12&mode=latest HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

## おすすめフィード

### `GET /posts/recommended`

アルゴリズムによるおすすめ投稿を取得します。複数のおすすめモードとベータA/Bテストバリアントをサポートします。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `12` | 1ページあたりの結果数 |
| `mode` | `string` | `algorithm` | おすすめモード（下記参照） |
| `page` | `number` | — | ページベースのページネーション（一部モードで使用） |
| `cursor` | `string` | — | カーソルベースのページネーション（一部モードで使用） |

**モード値:**

| モード | 説明 |
|------|-------------|
| `algorithm` | 標準的なアルゴリズムおすすめ |
| `latest` | 全ユーザーの最新投稿（フォロー中に限定されない） |
| `beta` | A/Bテストバリアント付きのベータおすすめアルゴリズム |

**リクエスト:**

```http
GET /posts/recommended?limit=12&mode=beta HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

**レスポンスフィールド:**

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `posts` | `Post[]` | おすすめ投稿オブジェクトの配列 |
| `pagination` | `object` | ページネーション情報（`page`, `limit`） |
| `betaVariant` | `string \| undefined` | 割り当てられたA/Bテストバリアント（`beta`モード時のみ存在） |

---

## ベータA/Bテストバリアント

`mode=beta`の場合、ユーザーはユーザーIDに基づいて5つのバリアントのいずれかに割り当てられます。割り当て式は:

```
variant = ["A", "B", "C", "D", "E"][userId % 5]
```

各バリアントは異なるスコアリングアルゴリズムを使用して投稿をランク付けします:

| バリアント | 戦略 | ブーストスコア | 説明 |
|---------|----------|-------------|-------------|
| **A** | 新しさ優先 | +22 | 最近の投稿を大幅に優遇。新しい投稿ほど大きなスコアブーストを得る。 |
| **B** | 品質 | +8 | エンゲージメント品質が高い投稿を優遇（いいね対閲覧比率、有意義なリプライ）。 |
| **C** | バランス | +14 | 新しさと品質シグナルのバランスの取れたミックス。 |
| **D** | 会話 | +10 | ディスカッションを生む投稿を優先（高リプライ数、リプライチェーン）。 |
| **E** | ディスカバリー | +12 | 閲覧者の通常のソーシャルグラフ外のユーザーからの投稿をプロモート。 |

**全バリアント共通ルール:**
- **1投稿者1投稿の多様性**: 各ページの結果には1投稿者あたり最大1投稿のみ含まれます。これにより、1人の多作なユーザーがフィードを独占するのを防ぎます。

### `POST /posts/feedback/beta-survey`

ベータおすすめアルゴリズムに対するユーザーフィードバックを送信します。

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `preference` | `string` | Yes | `current`（標準アルゴリズムを好む）または`beta`（ベータバリアントを好む） |
| `variant` | `string` | Yes | ユーザーに表示されたベータバリアント（A-E） |

**リクエスト:**

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

**レスポンス `200 OK`:**

```json
{
  "message": "Feedback submitted"
}
```

---

## リストフィード

### `GET /social/lists/:id/posts`

特定のユーザーリストからの投稿を取得します。ページネーションなしで1ページ分の結果を返します。

**パスパラメータ:**

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `string` | リストID |

**リクエスト:**

```http
GET /social/lists/list_abc123/posts HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**レスポンス `200 OK`:**

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

> **注意:** このエンドポイントは**1ページ分**の結果を返します。ページネーションはなく、利用可能なすべての投稿を1つのレスポンスで受け取ります。

---

## タブ構造

クライアントは以下の構造でフィードをタブに整理しています:

### フォロー中タブ

| サブタブ | エンドポイント | モード |
|---------|----------|------|
| Latest | `GET /posts/timeline` | `mode=latest` |
| Ranked | `GET /posts/timeline` | `mode=ranked` |

### おすすめタブ

| サブタブ | エンドポイント | モード |
|---------|----------|------|
| Algorithm | `GET /posts/recommended` | `mode=algorithm` |
| Latest | `GET /posts/recommended` | `mode=latest` |
| Beta | `GET /posts/recommended` | `mode=beta` |

### リストタブ

| 選択 | エンドポイント |
|-----------|----------|
| 特定のリスト（`listId`指定） | `GET /social/lists/:listId/posts` |

---

## React Query設定

公式クライアントはタイムラインフィードに対して以下の設定でReact Query（TanStack Query）を使用しています:

| 設定 | 値 | 説明 |
|---------|-------|-------------|
| `refetchInterval` | `30000`（30秒） | 30秒ごとにフィードを自動再取得 |
| `staleTime` | `10000`（10秒） | 取得後10秒間はデータを新鮮とみなす |
| `retry` | `2` | 失敗したリクエストを最大2回リトライ |

---

## バッチビュー

クライアントは、ユーザーがタイムラインをスクロールする際に自動的に投稿の閲覧を報告します:

| 設定 | 値 | 説明 |
|---------|-------|-------------|
| エンドポイント | `POST /posts/batch-views` | [Posts > Batch Views](./posts.md#batch-views)を参照 |
| 間隔 | **5秒**ごと | 蓄積された投稿IDが5秒ごとにフラッシュ |
| 滞在時間 | **1秒** | 投稿がカウントされるには最低1秒表示される必要あり |
| 表示閾値 | **50%** | 投稿の少なくとも50%がビューポートに表示される必要あり |

バッチビューのペイロード:

```json
{
  "postIds": ["post_tl001", "post_tl002", "post_tl003"]
}
```
