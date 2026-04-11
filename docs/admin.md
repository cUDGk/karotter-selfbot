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

例: `https://api.karotter.com/api/admin/analytics`

### 認証

管理エンドポイントには以下が必要です:

1. 有効な `Authorization: Bearer <token>` ヘッダー（すべての認証済みエンドポイントと同様）
2. 認証済みユーザーが `isAdmin === true` であること
3. 変更リクエストには標準の `x-csrf-token` ヘッダー

非管理者ユーザーはすべての管理エンドポイントで `403 Forbidden` を受け取ります。

---

## 管理パネルタブ

管理パネルのフロントエンドは8つのタブで構成されています:

| タブ | ルート | 説明 |
|-----|-------|-------------|
| ダッシュボード | `/control-room-x9k2` | アナリティクス概要 |
| ユーザー | `/control-room-x9k2/users` | ユーザー管理 |
| 投稿 | `/control-room-x9k2/posts` | 投稿モデレーション |
| ストーリー | `/control-room-x9k2/stories` | ストーリーモデレーション |
| おすすめ | `/control-room-x9k2/recommend` | おすすめアルゴリズムテスト |
| トレンド | `/control-room-x9k2/trending` | トレンドアルゴリズムテスト |
| アンケート | `/control-room-x9k2/survey` | アンケート/投票結果 |
| ベータ実験 | `/control-room-x9k2/beta-experiment` | A/Bテスト結果 |

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
| `totalUsers` | `number` | プラットフォームの登録ユーザー総数 |
| `totalPosts` | `number` | 作成された投稿の総数 |
| `activeUsers` | `number` | 直近期間のアクティブユーザー数 |
| `totalReports` | `number` | 保留中/アクティブな通報の総数 |

---

## ユーザー管理

### `GET /admin/users`

検索とカーソルベースページネーションでユーザーを一覧表示します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | 1ページあたりのユーザー数（最大100） |
| `search` | `string` | — | ユーザー名で検索（`@`を先頭に付けると完全一致検索） |
| `cursor` | `string` | — | 前回のレスポンスからのページネーションカーソル |

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
| `id` | `number` | ユーザーID |
| `username` | `string` | ユーザー名 |
| `displayName` | `string` | 表示名 |
| `email` | `string` | メールアドレス |
| `emailVerified` | `boolean` | メールが確認済みかどうか |
| `officialMark` | `string[]` | 公式マークカラーコードの配列 |
| `isParodyAccount` | `boolean` | 自己申告のパロディアカウント |
| `isBotAccount` | `boolean` | 自己申告のBotアカウント |
| `adminForceHidden` | `boolean` | 管理者によるプロフィール非表示の強制 |
| `adminForceParody` | `boolean` | 管理者によるパロディラベルの強制 |
| `adminForceBot` | `boolean` | 管理者によるBotラベルの強制 |
| `showBotAccounts` | `boolean` | ユーザー設定: Botアカウントの表示 |
| `showHiddenPosts` | `boolean` | ユーザー設定: 非表示投稿の表示 |
| `showR18Content` | `boolean` | ユーザー設定: R18コンテンツの表示 |
| `hideProfileFromMinors` | `boolean` | 未成年者からプロフィールを非表示にするかどうか |
| `isBanned` | `boolean` | ユーザーがBANされているかどうか |
| `isRestricted` | `boolean` | ユーザーが制限されているかどうか |
| `followersCount` | `number` | フォロワー数 |
| `postsCount` | `number` | 投稿数 |

---

### `PATCH /admin/users/:id/ban`

ユーザーアカウントをBANします。

**パスパラメータ:**

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | BANするユーザーID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `reason` | `string` | Yes | BANの理由（監査用に保存） |

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

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | BAN解除するユーザーID |

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

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | ユーザーID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `username` | `string` | No | 新しいユーザー名 |
| `displayName` | `string` | No | 新しい表示名 |
| `email` | `string` | No | 新しいメールアドレス |
| `password` | `string` | No | 新しいパスワード（平文、サーバーがハッシュ化） |
| `emailVerified` | `boolean` | No | メール確認ステータスの上書き |

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
- すべてのフィールドは任意です。指定されたフィールドのみ更新されます。
- `password`を変更すると、そのユーザーの既存セッションがすべて無効になります。

---

### `PATCH /admin/users/:id/official-mark`

ユーザーの公式認証マークを設定または更新します。

**パスパラメータ:**

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | ユーザーID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `officialMark` | `string[]` | Yes | マークカラーコードの配列。`[]`を渡すとすべてのマークを削除します。 |

**重要:** `officialMark`フィールドは**配列**です。ユーザーは複数のマークを同時に持つことができます（例: 認証済み管理者の場合 `["BLUE", "PURPLE"]`）。

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

| コード | Hex | 意味 |
|------|-----|---------|
| `BLUE` | `#1d9bf0` | 本人確認（個人） |
| `YELLOW` | `#f8c500` | 団体・企業 |
| `ORANGE` | `#ff7a00` | オレンジマーク |
| `PURPLE` | `#8b5cf6` | 運営・管理者 |
| `GRAY` | `#9ca3af` | 政府・公的機関 |
| `BLACK` | `#111827` | ブラックマーク |
| `RED` | `#ef4444` | レッドマーク |
| `GREEN` | `#22c55e` | グリーンマーク |

---

### `PATCH /admin/users/:id/flags`

ユーザーアカウントのモデレーションフラグを切り替えます。

**パスパラメータ:**

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | ユーザーID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `isParodyAccount` | `boolean` | No | 自己申告のパロディフラグ |
| `adminForceHidden` | `boolean` | No | ユーザーのプロフィールと投稿を強制非表示 |
| `adminForceParody` | `boolean` | No | ユーザーにパロディラベルを強制付与 |
| `isBotAccount` | `boolean` | No | 自己申告のBotフラグ |
| `adminForceBot` | `boolean` | No | ユーザーにBotラベルを強制付与 |
| `showBotAccounts` | `boolean` | No | ユーザーのBot表示設定を上書き |
| `showHiddenPosts` | `boolean` | No | ユーザーの非表示投稿表示設定を上書き |
| `showR18Content` | `boolean` | No | ユーザーのR18コンテンツ表示設定を上書き |
| `hideProfileFromMinors` | `boolean` | No | 未成年アカウントからプロフィールを強制非表示 |

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

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | 削除するユーザーID |

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

**警告:** この操作は元に戻せません。ユーザーに関連するすべての投稿、DM、リアクション、その他のデータが完全に削除されます。

---

## 投稿管理

### `GET /admin/posts`

検索とカーソルベースページネーションで投稿を一覧表示します。

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | 1ページあたりの投稿数 |
| `search` | `string` | — | 投稿内容で検索 |
| `cursor` | `string` | — | ページネーションカーソル |

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
| `id` | `number` | 投稿ID |
| `content` | `string` | 投稿テキスト内容 |
| `author.username` | `string` | 投稿者のユーザー名 |
| `author.displayName` | `string` | 投稿者の表示名 |
| `author.officialMark` | `string[]` | 投稿者の公式マーク |
| `viewsCount` | `number` | 合計閲覧数 |
| `likesCount` | `number` | 合計いいね数 |
| `rekarotsCount` | `number` | 合計リポスト数 |
| `repliesCount` | `number` | 合計リプライ数 |
| `isR18` | `boolean` | 投稿者がR18マークを付けたかどうか |
| `adminForceR18` | `boolean` | 管理者がR18として強制フラグしたかどうか |
| `adminForceHidden` | `boolean` | 管理者が投稿を非表示にしたかどうか |

---

### `PATCH /admin/posts/:id/flags`

投稿のモデレーションフラグを切り替えます。

**パスパラメータ:**

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | 投稿ID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `adminForceR18` | `boolean` | No | 投稿をR18/NSFWとして強制マーク |
| `adminForceHidden` | `boolean` | No | すべてのタイムラインから投稿を強制非表示 |

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

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | 削除する投稿ID |

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
| `limit` | `number` | `30` | 1ページあたりのストーリー数 |
| `cursor` | `string` | — | ページネーションカーソル |

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

| パラメータ | タイプ | 説明 |
|-----------|------|-------------|
| `id` | `number` | ストーリーID |

**リクエストボディ:**

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `adminForceR18` | `boolean` | No | ストーリーをR18として強制フラグ |
| `adminForceHidden` | `boolean` | No | ストーリーを強制非表示 |

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

**タイムアウト:** 60秒

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `userId` | `number` | — | 任意: 特定のユーザーIDでおすすめをテスト |
| `limit` | `number` | `30` | 返すおすすめ投稿の数 |

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
| `engagement` | 0.0 - 1.0 | 投稿が受けたエンゲージメント量（いいね、リポスト、リプライ） |
| `freshness` | 0.0 - 1.0 | 時間減衰スコア（新しいほど高い） |
| `authorAffinity` | 0.0 - 1.0 | 対象ユーザーがこの投稿者とどの程度インタラクションしているか |
| `graph` | 0.0 - 1.0 | ソーシャルグラフの近接度（相互フォロー、フォローチェーン） |
| `socialProof` | 0.0 - 1.0 | 対象のフォロー中ユーザーがこの投稿にエンゲージしているか |
| `tagAffinity` | 0.0 - 1.0 | 投稿のトピック/タグとユーザーの興味の一致度 |
| `inNetwork` | 0.0 - 1.0 | 投稿がユーザーの直接ネットワーク内の人からのものか |
| `totalRecommend` | 0.0 - 1.0 | 最終的な重み付きおすすめスコア |

---

## トレンドアルゴリズムテスト

### `GET /admin/test-trending`

詳細なウィンドウ統計でトレンドアルゴリズムをテストします。

**タイムアウト:** 60秒

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `30` | 返すトレンド投稿の数 |

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
| `likes1h` | `number` | 過去1時間のいいね数 |
| `likes6h` | `number` | 過去6時間のいいね数 |
| `likes24h` | `number` | 過去24時間のいいね数 |
| `rekarots1h` | `number` | 過去1時間のリポスト数 |
| `rekarots6h` | `number` | 過去6時間のリポスト数 |
| `rekarots24h` | `number` | 過去24時間のリポスト数 |
| `replies1h` | `number` | 過去1時間のリプライ数 |
| `replies6h` | `number` | 過去6時間のリプライ数 |
| `replies24h` | `number` | 過去24時間のリプライ数 |
| `uniqueActors1h` | `number` | 過去1時間にインタラクションしたユニークユーザー数 |
| `uniqueActors6h` | `number` | 過去6時間にインタラクションしたユニークユーザー数 |
| `uniqueActors24h` | `number` | 過去24時間にインタラクションしたユニークユーザー数 |

---

## アンケート結果

### `GET /admin/survey-results`

プラットフォーム全体の集計されたアンケート/投票結果を取得します。

**タイムアウト:** 30秒

**クエリパラメータ:**

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `limit` | `number` | `10` | 返すアンケートの数 |

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
| `variant` | `string` | バリアント識別子: `"A"`, `"B"`, `"C"`, `"D"`, または `"E"` |
| `label` | `string` | バリアントの人間が読める説明 |
| `uniqueUsers` | `number` | このバリアントに割り当てられたユニークユーザー数 |
| `impressions` | `number` | このバリアントで表示された投稿インプレッション総数 |
| `survey.preferBeta` | `number` | アンケートでベータ/新バリアントを好んだユーザー数 |
| `survey.preferCurrent` | `number` | 現行/コントロールバリアントを好んだユーザー数 |

---

## 公式マークカラーリファレンス

公式マークカラーパレットの完全なクイックリファレンス:

| コード | Hex カラー | CSSプレビュー | 日本語ラベル | 一般的な用途 |
|------|-----------|-------------|----------------|---------------|
| `BLUE` | `#1d9bf0` | ![#1d9bf0](https://via.placeholder.com/12/1d9bf0/1d9bf0) | 本人確認 | 本人確認 |
| `YELLOW` | `#f8c500` | ![#f8c500](https://via.placeholder.com/12/f8c500/f8c500) | 団体・企業 | 団体・企業 |
| `ORANGE` | `#ff7a00` | ![#ff7a00](https://via.placeholder.com/12/ff7a00/ff7a00) | オレンジ | 特別指定 |
| `PURPLE` | `#8b5cf6` | ![#8b5cf6](https://via.placeholder.com/12/8b5cf6/8b5cf6) | 運営 | 運営・管理者 |
| `GRAY` | `#9ca3af` | ![#9ca3af](https://via.placeholder.com/12/9ca3af/9ca3af) | 政府・公的機関 | 政府・公的機関 |
| `BLACK` | `#111827` | ![#111827](https://via.placeholder.com/12/111827/111827) | ブラック | 特別指定 |
| `RED` | `#ef4444` | ![#ef4444](https://via.placeholder.com/12/ef4444/ef4444) | レッド | 特別指定 |
| `GREEN` | `#22c55e` | ![#22c55e](https://via.placeholder.com/12/22c55e/22c55e) | グリーン | 特別指定 |
