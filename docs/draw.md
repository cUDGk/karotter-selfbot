# 絵チャ API

> **ベースURL:** `https://karotter.com/api`
>
> すべてのエンドポイントは `Authorization: Bearer {token}` ヘッダーと `X-CSRF-Token` ヘッダーによる認証が必要です。
> 一部の操作ではCookieベース認証（`karotter_at`, `karotter_rt`, `karotter_csrf`）が必要な場合があります。

---

## 目次

- [概要](#概要)
- [ルーム一覧](#ルーム一覧)
- [ルーム作成](#ルーム作成)
- [ルーム詳細取得](#ルーム詳細取得)
- [ルーム削除](#ルーム削除)
- [ルームに参加](#ルームに参加)
- [レイヤー更新](#レイヤー更新)
- [招待コードのローテーション](#招待コードのローテーション)
- [レイヤー取得](#レイヤー取得)
- [チャットメッセージ送信 (REST)](#チャットメッセージ送信-rest)
- [キャンバス仕様](#キャンバス仕様)
- [レイヤーシステム](#レイヤーシステム)
- [ストロークタイプ](#ストロークタイプ)
- [テキストオブジェクト](#テキストオブジェクト)
- [チャット](#チャット)
- [Socket.IOイベント](#socketioイベント)

---

## 概要

絵チャはKarotterの共同お絵かき機能です。ユーザーはルームを作成し、2560x2560の共有キャンバス上で複数人がリアルタイムに一緒に描くことができます。キャンバスは最大7レイヤーをサポートし、すべての描画操作はSocket.IOを通じて同期されます。

ルームは公開（誰でも参加可能）または招待制（招待コードが必要）にできます。

---

## ルーム一覧

利用可能な絵チャルームのページネーション付きリストを返します。

```
GET /api/draw/rooms
```

### クエリパラメータ

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | number | 1 | ページ番号（1始まり） |
| `limit` | number | 5 | 1ページあたりのルーム数 |

### レスポンス

```json
{
  "rooms": [
    {
      "id": 15,
      "name": "Free Drawing Room",
      "visibility": "PUBLIC",
      "participantCount": 3,
      "ownerId": 15459,
      "ownerUsername": "claude",
      "inviteCode": null,
      "createdAt": "2026-03-28T20:00:00.000Z",
      "updatedAt": "2026-03-28T22:30:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 5,
    "total": 12,
    "pages": 3
  }
}
```

### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `rooms` | Room[] | ルームサマリーオブジェクトの配列 |
| `rooms[].id` | number | ルームID |
| `rooms[].name` | string | ルーム名 |
| `rooms[].visibility` | string | `"PUBLIC"` または `"INVITE"` |
| `rooms[].participantCount` | number | 現在の参加者数 |
| `rooms[].ownerId` | number | ルームオーナーのユーザーID |
| `rooms[].ownerUsername` | string | ルームオーナーのユーザー名 |
| `rooms[].inviteCode` | string \| null | 現在の招待コード（公開ルームの場合はnull） |
| `rooms[].createdAt` | string | ISO 8601 作成タイムスタンプ |
| `rooms[].updatedAt` | string | ISO 8601 最終更新タイムスタンプ |

---

## ルーム作成

新しい絵チャルームを作成します。

```
POST /api/draw/rooms
```

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `name` | string | Yes | ルーム名 |
| `visibility` | string | No | `"PUBLIC"`（デフォルト）または `"INVITE"` |

### 例

```http
POST /api/draw/rooms HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "name": "Art Collab Room",
  "visibility": "PUBLIC"
}
```

### レスポンス

```json
{
  "room": {
    "id": 16,
    "name": "Art Collab Room",
    "visibility": "PUBLIC",
    "participantCount": 1,
    "ownerId": 15459,
    "ownerUsername": "claude",
    "inviteCode": null,
    "createdAt": "2026-03-28T23:00:00.000Z",
    "updatedAt": "2026-03-28T23:00:00.000Z",
    "layers": []
  }
}
```

---

## ルーム詳細取得

すべてのレイヤーデータを含む絵チャルームの完全な詳細を返します。

```
GET /api/draw/rooms/:id
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | ルームID |

### レスポンス

```json
{
  "room": {
    "id": 15,
    "name": "Free Drawing Room",
    "visibility": "PUBLIC",
    "participantCount": 3,
    "participants": [
      {
        "userId": 15459,
        "username": "claude",
        "displayName": "claude",
        "avatarUrl": "/uploads/avatars/abc.webp",
        "joinedAt": "2026-03-28T20:00:00.000Z"
      }
    ],
    "ownerId": 15459,
    "ownerUsername": "claude",
    "inviteCode": null,
    "updatedAt": "2026-03-28T22:30:00.000Z",
    "layers": [
      {
        "id": "layer-001",
        "name": "Background",
        "order": 0,
        "visible": true,
        "opacity": 1,
        "dataUrl": "data:image/png;base64,iVBOR..."
      },
      {
        "id": "layer-002",
        "name": "Sketch",
        "order": 1,
        "visible": true,
        "opacity": 0.8,
        "dataUrl": "data:image/png;base64,iVBOR..."
      }
    ]
  }
}
```

### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `room.id` | number | ルームID |
| `room.name` | string | ルーム名 |
| `room.visibility` | string | `"PUBLIC"` または `"INVITE"` |
| `room.participantCount` | number | アクティブな参加者数 |
| `room.participants` | object[] | 参加者オブジェクトの配列 |
| `room.participants[].userId` | number | 参加者のユーザーID |
| `room.participants[].username` | string | 参加者のユーザー名 |
| `room.participants[].displayName` | string | 参加者の表示名 |
| `room.participants[].avatarUrl` | string \| null | 参加者のアバターURL |
| `room.participants[].joinedAt` | string | 参加者が参加した日時 |
| `room.ownerId` | number | オーナーのユーザーID |
| `room.ownerUsername` | string | オーナーのユーザー名 |
| `room.inviteCode` | string \| null | 現在の招待コード |
| `room.updatedAt` | string | ISO 8601 最終更新タイムスタンプ |
| `room.layers` | Layer[] | キャンバスレイヤーオブジェクトの配列 |

---

## ルーム削除

絵チャルームを削除します。ルームオーナーのみ削除可能です。

```
DELETE /api/draw/rooms/:id
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | ルームID |

### レスポンス

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ルームを削除しました"}` | ルーム削除完了 |
| 403 | `{"error": "権限がありません"}` | ルームオーナーではない |
| 404 | `{"error": "ルームが見つかりません"}` | ルームが見つからない |

---

## ルームに参加

絵チャルームに参加します。��待制ルームの場合、招待コードが必要です。

```
POST /api/draw/rooms/:id/join
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | ルームID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `inviteCode` | string | No* | 招待コード（`INVITE`公開設定のルームでは必須） |

### 例: 公開ルームに参加

```http
POST /api/draw/rooms/15/join HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

### 例: 招待制ルームに参加

```http
POST /api/draw/rooms/16/join HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "inviteCode": "abc123def456"
}
```

### レスポンス

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ルームに参加しました", "room": {...}}` | 参加完了 |
| 400 | `{"error": "既に参加しています"}` | 既にルームに参加中 |
| 403 | `{"error": "招待コードが必要です"}` | 招待コードが必要だが未提供 |
| 403 | `{"error": "招待コードが正しくありません"}` | 無効な招待コード |

---

## レイヤー更新

キャンバスのすべてのレイヤーを更新します。これは**完全置換**操作です。

```
PUT /api/draw/rooms/:id/layers
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | ルームID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `layers` | Layer[] | Yes | すべてのレイヤーの完全な配列 |

### レイヤーオブジェクト

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `id` | string | Yes | レイヤーID（クライアント生成、例: UUID） |
| `name` | string | Yes | レイヤー名 |
| `order` | number | Yes | レイヤー順序（0 = 最下層） |
| `visible` | boolean | Yes | レイヤーが表示されるかどうか |
| `opacity` | number | Yes | レイヤー不透明度（0.0 から 1.0） |
| `dataUrl` | string | Yes | Base64エンコードされたPNGデータURL |

### 例

```http
PUT /api/draw/rooms/15/layers HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "layers": [
    {
      "id": "layer-001",
      "name": "Background",
      "order": 0,
      "visible": true,
      "opacity": 1,
      "dataUrl": "data:image/png;base64,iVBORw0KGgo..."
    },
    {
      "id": "layer-002",
      "name": "Sketch",
      "order": 1,
      "visible": true,
      "opacity": 0.8,
      "dataUrl": "data:image/png;base64,iVBORw0KGgo..."
    }
  ]
}
```

> **重大な警告:** このエンドポイントは**完全置換**を実行します。リクエストにすべての既存レイヤーを含める必要があります。`layers`配列に含まれないレイヤーは**完全に削除**されます。更新前に必ず `GET /api/draw/rooms/:id` で現在のレイヤーを取得してください。

### レスポンス

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "レイヤーを更新しました"}` | レイヤー更新完了 |
| 400 | `{"error": "レイヤー数の上限を超えています"}` | 7レイヤーの上限を超過 |
| 413 | (様々) | ペイロードが大きすぎる（~5MB制限超過） |

---

## 招待コードのローテーション

ルームの新しい招待コードを生成し、以前のコードを無効化します。

```
POST /api/draw/rooms/:id/invite/rotate
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | ルームID |

### リクエストボディ

なし。

### レスポンス

```json
{
  "inviteCode": "new-unique-code-here"
}
```

### レスポンス

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"inviteCode": "..."}` | 新しい招待コード生成完了 |
| 403 | `{"error": "権限がありません"}` | ルームオーナーではない |

---

## レイヤー取得

ルーム詳細なしで絵チャルームのレイヤーのみを返します。レイヤー更新のポーリングに便利です。

```
GET /api/draw/rooms/:id/layers
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | ルームID |

### レスポンス

```json
{
  "layers": [
    {
      "id": "layer-001",
      "name": "Background",
      "order": 0,
      "visible": true,
      "opacity": 1,
      "dataUrl": "data:image/png;base64,iVBOR..."
    }
  ]
}
```

---

## チャットメッセージ送信 (REST)

REST API経由で絵チャルームにテキストチャットメッセージを送信します。`draw:chat` Socket.IOイベントのエミットの代替手段です。

```
POST /api/draw/rooms/:id/chat
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | ルームID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `content` | string | Yes | メッセージテキスト（最大400文字） |

### 例

```http
POST /api/draw/rooms/15/chat HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "content": "Nice drawing!"
}
```

### レスポンス

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メッセージを送信しました"}` | メッセージ送信完了 |
| 403 | `{"error": "ルームに参加していません"}` | 参加者ではない |

---

## キャンバス仕様

| プロパティ | 値 |
|----------|-------|
| キャンバスサイズ | 2560 x 2560 ピクセル |
| カラーフォーマット | RGBA（32ビットカラー、アルファチャンネル付き） |
| 画像フォーマット | PNG |
| 最大レイヤー数 | 7 |
| 最大ペイロードサイズ | 約5 MB（PUT /layersリクエスト全体） |
| データエンコーディング | Base64データURL (`data:image/png;base64,...`) |

### キャンバス座標系

```
(0,0) -------- (2560,0)
  |                |
  |    Canvas      |
  |                |
(0,2560) ---- (2560,2560)
```

原点は左上隅です。Xは右方向に増加し、Yは下方向に増加します。

---

## レイヤーシステム

レイヤーは `order: 0`（最下層）から `order: N`（最上層）の順に合成されます。各レイヤーは独立した2560x2560 RGBA PNG画像です。

### レイヤープロパティ

| プロパティ | 型 | 範囲 | 説明 |
|----------|------|-------|-------------|
| `id` | string | - | ユニーク識別子（クライアント生成） |
| `name` | string | - | 人間が読めるレイヤー名 |
| `order` | number | 0-6 | 重ね順序（0 = 最下層） |
| `visible` | boolean | - | レイヤーが描画されるかどうか |
| `opacity` | number | 0.0 - 1.0 | レイヤー不透明度 |
| `dataUrl` | string | - | Base64 PNGデータURL |

### レイヤー制限

- 1ルームあたり最大7レイヤー
- 各レイヤーは完全な2560x2560 RGBA PNG
- すべてのレイヤーの合計ペイロードは約5MB以下
- 透明レイヤー（すべてゼロアルファ）も制限にカウントされる

---

## ストロークタイプ

描画操作はSocket.IOを介してリアルタイムに送信されます。各ストロークタイプには固有のプロパティがあります。

### ラインストローク

2点間に線分を描画します。

```json
{
  "type": "line",
  "x1": 100,
  "y1": 200,
  "x2": 300,
  "y2": 400,
  "color": "#FF0000",
  "size": 5,
  "opacity": 1.0,
  "tool": "pen"
}
```

#### ラインプロパティ

| プロパティ | タイプ | 説明 |
|----------|------|-------------|
| `x1` | number | 開始X座標 |
| `y1` | number | 開始Y座標 |
| `x2` | number | 終了X座標 |
| `y2` | number | 終了Y座標 |
| `color` | string | hex形式の色（例: `"#FF0000"`） |
| `size` | number | ブラシサイズ（ピクセル） |
| `opacity` | number | ストローク不透明度（0.0 - 1.0） |
| `tool` | string | 描画ツール |

#### 利用可能なツール

| ツール | 説明 |
|------|-------------|
| `pen` | 標準のハードエッジペン |
| `pencil` | テクスチャ付きのソフト鉛筆 |
| `marker` | 幅広の半透明マーカー |
| `watercolor` | ブレンド付きのソフト水彩ブラシ |
| `chalk` | テクスチャ付きチョークストローク |
| `charcoal` | ダークな木炭テクスチャ |
| `spraypaint` | ランダムパーティクル分布のスプレーペイント |
| `neon` | 光るネオンエフェクト |
| `eraser` | 標準消しゴム（ピクセルを消去） |
| `eraser-hard` | フェザリングなしのハードエッジ消しゴム |
| `eraser-soft` | フェザリング付きのソフト消しゴム |

### バケツ塗りストローク

隣接する領域を色で塗りつぶします。

```json
{
  "type": "bucket",
  "x": 500,
  "y": 500,
  "color": "#00FF00",
  "opacity": 1.0,
  "tolerance": 20
}
```

#### バケツプロパティ

| プロパティ | タイプ | 説明 |
|----------|------|-------------|
| `x` | number | 塗りつぶし開始X座標 |
| `y` | number | 塗りつぶし開始Y座標 |
| `color` | string | hex形式の塗りつぶし色 |
| `opacity` | number | 塗りつぶし不透明度（0.0 - 1.0） |
| `tolerance` | number | 色のマッチング許容値: `20`（厳密）または `60`（緩い） |

### シェイプストローク

幾何学的な図形を描画します。

```json
{
  "type": "shape",
  "shape": "rect",
  "x1": 100,
  "y1": 100,
  "x2": 400,
  "y2": 300,
  "color": "#0000FF",
  "size": 3,
  "opacity": 1.0
}
```

#### シェイププロパティ

| プロパティ | タイプ | 説明 |
|----------|------|-------------|
| `shape` | string | シェイプタイプ（下表参照） |
| `x1` | number | 開始X（矩形の場合は左上、楕円の場合は中心） |
| `y1` | number | 開始Y |
| `x2` | number | 終了X（矩形の場合は右下） |
| `y2` | number | 終了Y |
| `color` | string | hex形式のストローク/塗りつぶし色 |
| `size` | number | ストローク幅（ピクセル、アウトライン図形用） |
| `opacity` | number | シェイプ不透明度（0.0 - 1.0） |

#### 利用可能なシェイプ

| シェイプ | 説明 |
|-------|-------------|
| `line` | (x1,y1)から(x2,y2)への直線 |
| `rect` | 矩形アウトライン |
| `rect-fill` | 塗りつぶし矩形 |
| `ellipse` | 楕円アウトライン |
| `ellipse-fill` | 塗りつぶし楕円 |
| `triangle` | 三角形アウトライン |
| `triangle-fill` | 塗りつぶし三角形 |
| `arrow` | (x1,y1)から(x2,y2)への矢印 |
| `star` | 五芒星 |

### グラデーションストローク

リニアグラデーション塗りつぶしを適用します。

```json
{
  "type": "gradient",
  "x1": 0,
  "y1": 0,
  "x2": 2560,
  "y2": 2560,
  "color": "#FF0000",
  "secondaryColor": "#0000FF",
  "opacity": 1.0
}
```

#### グラデーションプロパティ

| プロパティ | タイプ | 説明 |
|----------|------|-------------|
| `x1` | number | グラデーション開始X |
| `y1` | number | グラデーション開始Y |
| `x2` | number | グラデーション終了X |
| `y2` | number | グラデーション終了Y |
| `color` | string | hex形式の開始色 |
| `secondaryColor` | string | hex形式の終了色 |
| `opacity` | number | グラデーション不透明度（0.0 - 1.0） |

---

## テキストオブジェクト

テキストはキャンバス上に永続的なオブジェクトとして配置できます。

```json
{
  "id": "text-001",
  "layerId": "layer-002",
  "x": 200,
  "y": 150,
  "width": 400,
  "height": 100,
  "rotation": 0,
  "color": "#000000",
  "fontSize": 24,
  "content": "Hello World!"
}
```

### テキストオブジェクトプロパティ

| プロパティ | 型 | 範囲 | 説明 |
|----------|------|-------|-------------|
| `id` | string | - | ユニークなテキストオブジェクトID |
| `layerId` | string | - | このテキストが属するレイヤーのID |
| `x` | number | 0-2560 | キャンバス上のX位置 |
| `y` | number | 0-2560 | キャンバス上のY位置 |
| `width` | number | 70-1600 | テキストボックスの幅（ピクセル） |
| `height` | number | 36-1000 | テキストボックスの高さ（ピクセル） |
| `rotation` | number | 0-360 | 回転角度（度） |
| `color` | string | - | hex形式のテキスト色 |
| `fontSize` | number | 8-200 | フォントサイズ（ピクセル） |
| `content` | string | 最大600文字 | テキスト内容 |

### テキスト制約

| 制約 | 値 |
|-----------|-------|
| 最小幅 | 70 px |
| 最大幅 | 1600 px |
| 最小高さ | 36 px |
| 最大高さ | 1000 px |
| 最小フォントサイズ | 8 px |
| 最大フォントサイズ | 200 px |
| 最大コンテンツ長 | 600文字 |

---

## チャット

各絵チャルームには参加者間のコミュニケーション用のテキストチャットが組み込まれています。

### チャット制約

| 制約 | 値 |
|-----------|-------|
| メモリ内の最大メッセージ数 | 80 |
| 1メッセージあたりの最大文字数 | 400 |

チャットメッセージは一時的で、Socket.IOを介して送信されます。データベースには永続化されず、すべての参加者が退出すると失われます。

---

## Socket.IOイベント

絵チャはSocket.IOを使用してリアルタイム同期を行います。

### 接続

```javascript
import { io } from "socket.io-client";

const socket = io("https://karotter.com", {
  path: "/socket.io",
  auth: { token: "Bearer eyJ..." }
});
```

### イベント

| イベント | 方向 | ペイロード | 説明 |
|-------|-----------|---------|-------------|
| `draw:join` | emit | `{roomId}` | 絵チャルームに参加 |
| `draw:leave` | emit | `{roomId}` | 絵チャルームを退出 |
| `draw:room-state` | receive | `{room, layers, participants}` | 参加後の完全なルーム状態 |
| `draw:stroke` | emit/receive | `{roomId, layerId, stroke}` | リアルタイムストロークデータ |
| `draw:text` | emit/receive | `{roomId, textObject}` | テキストオブジェクトの配置/更新 |
| `draw:layer-update` | receive | `{roomId, layers}` | レイヤーデータ更新 |
| `draw:participant-join` | receive | `{roomId, user}` | 新しい参加者が参加 |
| `draw:participant-leave` | receive | `{roomId, userId}` | 参加者が退出 |
| `draw:chat` | emit/receive | `{roomId, content, sender}` | チャットメッセージ |

### 典型的なフロー

1. `POST /api/draw/rooms/:id/join` を呼び出してREST API経由で参加
2. Socket.IO経由で `draw:join` を `{roomId}` でemit
3. 完全なキャンバス状態を含む `draw:room-state` を受信
4. 他のユーザーからの `draw:stroke` イベントをリッスン
5. ローカルで描画時に `draw:stroke` をemit
6. 定期的に `PUT /api/draw/rooms/:id/layers` を呼び出してキャンバス状態を永続化

---

## レート制限

すべての絵チャエンドポイントはデフォルトのレート制限を共有しています:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
