# 絵チャ API

> **ベースURL:** `https://karotter.com/api`
>
> すべてのエンドポイントは `Authorization: Bearer {token}` ヘッダーと `X-CSRF-Token` ヘッダーによる認証が必要です。
> 一部の操作ではCookieベース認証（`karotter_at`, `karotter_rt`, `karotter_csrf`）が必要な場合があります。

---

## 目次

- [Overview](#overview)
- [List Rooms](#list-rooms)
- [Create a Room](#create-a-room)
- [Get Room Details](#get-room-details)
- [Delete a Room](#delete-a-room)
- [Join a Room](#join-a-room)
- [Update Layers](#update-layers)
- [Rotate Invite Code](#rotate-invite-code)
- [Get Layers](#get-layers)
- [Send Chat Message (REST)](#send-chat-message-rest)
- [Canvas Specifications](#canvas-specifications)
- [Layer System](#layer-system)
- [Stroke Types](#stroke-types)
- [Text Objects](#text-objects)
- [Chat](#chat)
- [Socket.IO Events](#socketio-events)

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

### Query Parameters

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number (1-indexed) |
| `limit` | number | 5 | Rooms per page |

### Response

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
| `rooms` | Room[] | Array of room summary objects |
| `rooms[].id` | number | Room ID |
| `rooms[].name` | string | Room name |
| `rooms[].visibility` | string | `"PUBLIC"` or `"INVITE"` |
| `rooms[].participantCount` | number | Current number of participants |
| `rooms[].ownerId` | number | Room owner's user ID |
| `rooms[].ownerUsername` | string | Room owner's username |
| `rooms[].inviteCode` | string \| null | Current invite code (null for public rooms) |
| `rooms[].createdAt` | string | ISO 8601 creation timestamp |
| `rooms[].updatedAt` | string | ISO 8601 last update timestamp |

---

## ルーム作成

新しい絵チャルームを作成します。

```
POST /api/draw/rooms
```

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `name` | string | Yes | Room name |
| `visibility` | string | No | `"PUBLIC"` (default) or `"INVITE"` |

### Example

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

### Response

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

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Room ID |

### Response

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
| `room.id` | number | Room ID |
| `room.name` | string | Room name |
| `room.visibility` | string | `"PUBLIC"` or `"INVITE"` |
| `room.participantCount` | number | Number of active participants |
| `room.participants` | object[] | Array of participant objects |
| `room.participants[].userId` | number | Participant's user ID |
| `room.participants[].username` | string | Participant's username |
| `room.participants[].displayName` | string | Participant's display name |
| `room.participants[].avatarUrl` | string \| null | Participant's avatar URL |
| `room.participants[].joinedAt` | string | When the participant joined |
| `room.ownerId` | number | Owner's user ID |
| `room.ownerUsername` | string | Owner's username |
| `room.inviteCode` | string \| null | Current invite code |
| `room.updatedAt` | string | ISO 8601 last update timestamp |
| `room.layers` | Layer[] | Array of canvas layer objects |

---

## ルーム削除

絵チャルームを削除します。ルームオーナーのみ削除可能です。

```
DELETE /api/draw/rooms/:id
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Room ID |

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ルームを削除しました"}` | Room deleted |
| 403 | `{"error": "権限がありません"}` | Not the room owner |
| 404 | `{"error": "ルームが見つかりません"}` | Room not found |

---

## ルームに参加

絵チャルームに参加します。��待制ルームの場合、招待コードが必要です。

```
POST /api/draw/rooms/:id/join
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Room ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `inviteCode` | string | No* | Invite code (required for `INVITE` visibility rooms) |

### Example: Join Public Room

```http
POST /api/draw/rooms/15/join HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

### Example: Join Invite-Only Room

```http
POST /api/draw/rooms/16/join HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "inviteCode": "abc123def456"
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ルームに参加しました", "room": {...}}` | Joined |
| 400 | `{"error": "既に参加しています"}` | Already in room |
| 403 | `{"error": "招待コードが必要です"}` | Invite code required but not provided |
| 403 | `{"error": "招待コードが正しくありません"}` | Invalid invite code |

---

## レイヤー更新

キャンバスのすべてのレイヤーを更新します。これは**完全置換**操作です。

```
PUT /api/draw/rooms/:id/layers
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Room ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `layers` | Layer[] | Yes | Complete array of all layers |

### Layer Object

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `id` | string | Yes | Layer ID (client-generated, e.g., UUID) |
| `name` | string | Yes | Layer name |
| `order` | number | Yes | Layer order (0 = bottom) |
| `visible` | boolean | Yes | Whether the layer is visible |
| `opacity` | number | Yes | Layer opacity (0.0 to 1.0) |
| `dataUrl` | string | Yes | Base64-encoded PNG data URL |

### Example

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

> **CRITICAL WARNING:** This endpoint performs a **full replacement**. You MUST include ALL existing layers in the request. Any layer not included in the `layers` array will be **permanently deleted**. Always fetch the current layers first with `GET /api/draw/rooms/:id` before updating.

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "レイヤーを更新しました"}` | Layers updated |
| 400 | `{"error": "レイヤー数の上限を超えています"}` | Exceeds 7 layer limit |
| 413 | (varies) | Payload too large (~5MB limit exceeded) |

---

## 招待コードのローテーション

ルームの新しい招待コードを生成し、以前のコードを無効化します。

```
POST /api/draw/rooms/:id/invite/rotate
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Room ID |

### リクエストボディ

なし。

### Response

```json
{
  "inviteCode": "new-unique-code-here"
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"inviteCode": "..."}` | New invite code generated |
| 403 | `{"error": "権限がありません"}` | Not the room owner |

---

## レイヤー取得

ルーム詳細なしで絵チャルームのレイヤーのみを返します。レイヤー更新のポーリングに便利です。

```
GET /api/draw/rooms/:id/layers
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Room ID |

### Response

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

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Room ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `content` | string | Yes | Message text (max 400 characters) |

### Example

```http
POST /api/draw/rooms/15/chat HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "content": "Nice drawing!"
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メッセージを送信しました"}` | Message sent |
| 403 | `{"error": "ルームに参加していません"}` | Not a participant |

---

## キャンバス仕様

| プロパティ | 値 |
|----------|-------|
| Canvas size | 2560 x 2560 pixels |
| Color format | RGBA (32-bit color with alpha channel) |
| Image format | PNG |
| Maximum layers | 7 |
| Maximum payload size | ~5 MB (for the entire PUT /layers request) |
| Data encoding | Base64 data URL (`data:image/png;base64,...`) |

### キャンバス座標系

```
(0,0) -------- (2560,0)
  |                |
  |    Canvas      |
  |                |
(0,2560) ---- (2560,2560)
```

Origin is at the top-left corner. X increases to the right, Y increases downward.

---

## レイヤーシステム

Layers are composited in order from `order: 0` (bottom) to `order: N` (top). Each layer is an independent 2560x2560 RGBA PNG image.

### レイヤープロパティ

| プロパティ | 型 | 範囲 | 説明 |
|----------|------|-------|-------------|
| `id` | string | - | Unique identifier (client-generated) |
| `name` | string | - | Human-readable layer name |
| `order` | number | 0-6 | Stacking order (0 = bottom) |
| `visible` | boolean | - | Whether the layer is rendered |
| `opacity` | number | 0.0 - 1.0 | Layer opacity |
| `dataUrl` | string | - | Base64 PNG data URL |

### レイヤー制限

- Maximum 7 layers per room
- Each layer is a full 2560x2560 RGBA PNG
- Total payload for all layers must be under ~5MB
- Transparent layers (all-zero alpha) still count toward the limit

---

## ストロークタイプ

Drawing operations are transmitted via Socket.IO in real-time. Each stroke type has specific properties.

### Line Stroke

Draws a line segment between two points.

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

#### Line Properties

| Property | タイプ | 説明 |
|----------|------|-------------|
| `x1` | number | Start X coordinate |
| `y1` | number | Start Y coordinate |
| `x2` | number | End X coordinate |
| `y2` | number | End Y coordinate |
| `color` | string | Color in hex format (e.g., `"#FF0000"`) |
| `size` | number | Brush size in pixels |
| `opacity` | number | Stroke opacity (0.0 - 1.0) |
| `tool` | string | Drawing tool |

#### Available Tools

| ツール | 説明 |
|------|-------------|
| `pen` | Standard hard-edge pen |
| `pencil` | Soft pencil with texture |
| `marker` | Wide semi-transparent marker |
| `watercolor` | Soft watercolor brush with blending |
| `chalk` | Textured chalk stroke |
| `charcoal` | Dark charcoal texture |
| `spraypaint` | Spray paint with random particle distribution |
| `neon` | Glowing neon effect |
| `eraser` | Standard eraser (removes pixels) |
| `eraser-hard` | Hard-edge eraser with no feathering |
| `eraser-soft` | Soft eraser with feathered edges |

### Bucket Fill Stroke

Fills a contiguous area with a color.

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

#### Bucket Properties

| Property | タイプ | 説明 |
|----------|------|-------------|
| `x` | number | Fill origin X coordinate |
| `y` | number | Fill origin Y coordinate |
| `color` | string | Fill color in hex |
| `opacity` | number | Fill opacity (0.0 - 1.0) |
| `tolerance` | number | Color matching tolerance: `20` (strict) or `60` (loose) |

### Shape Stroke

Draws geometric shapes.

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

#### Shape Properties

| Property | タイプ | 説明 |
|----------|------|-------------|
| `shape` | string | Shape type (see table below) |
| `x1` | number | Start X (top-left for rectangles, center for ellipses) |
| `y1` | number | Start Y |
| `x2` | number | End X (bottom-right for rectangles) |
| `y2` | number | End Y |
| `color` | string | Stroke/fill color in hex |
| `size` | number | Stroke width in pixels (for outline shapes) |
| `opacity` | number | Shape opacity (0.0 - 1.0) |

#### Available Shapes

| シェイプ | 説明 |
|-------|-------------|
| `line` | Straight line from (x1,y1) to (x2,y2) |
| `rect` | Rectangle outline |
| `rect-fill` | Filled rectangle |
| `ellipse` | Ellipse outline |
| `ellipse-fill` | Filled ellipse |
| `triangle` | Triangle outline |
| `triangle-fill` | Filled triangle |
| `arrow` | Arrow from (x1,y1) to (x2,y2) |
| `star` | Five-pointed star |

### Gradient Stroke

Applies a linear gradient fill.

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

#### Gradient Properties

| Property | タイプ | 説明 |
|----------|------|-------------|
| `x1` | number | Gradient start X |
| `y1` | number | Gradient start Y |
| `x2` | number | Gradient end X |
| `y2` | number | Gradient end Y |
| `color` | string | Start color in hex |
| `secondaryColor` | string | End color in hex |
| `opacity` | number | Gradient opacity (0.0 - 1.0) |

---

## テキストオブジェクト

Text can be placed on the canvas as persistent objects.

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

### Text Object Properties

| プロパティ | 型 | 範囲 | 説明 |
|----------|------|-------|-------------|
| `id` | string | - | Unique text object ID |
| `layerId` | string | - | ID of the layer this text belongs to |
| `x` | number | 0-2560 | X position on canvas |
| `y` | number | 0-2560 | Y position on canvas |
| `width` | number | 70-1600 | Text box width in pixels |
| `height` | number | 36-1000 | Text box height in pixels |
| `rotation` | number | 0-360 | Rotation in degrees |
| `color` | string | - | Text color in hex |
| `fontSize` | number | 8-200 | Font size in pixels |
| `content` | string | max 600 chars | Text content |

### Text Constraints

| 制約 | 値 |
|-----------|-------|
| Minimum width | 70 px |
| Maximum width | 1600 px |
| Minimum height | 36 px |
| Maximum height | 1000 px |
| Minimum font size | 8 px |
| Maximum font size | 200 px |
| Maximum content length | 600 characters |

---

## チャット

Each draw room has a built-in text chat for communication between participants.

### チャット Constraints

| 制約 | 値 |
|-----------|-------|
| Maximum messages in memory | 80 |
| Maximum characters per message | 400 |

Chat messages are ephemeral and transmitted via Socket.IO. They are not persisted to the database and will be lost when all participants leave.

---

## Socket.IOイベント

Draw chat uses Socket.IO for real-time synchronization.

### 接続

```javascript
import { io } from "socket.io-client";

const socket = io("https://karotter.com", {
  path: "/socket.io",
  auth: { token: "Bearer eyJ..." }
});
```

### Events

| イベント | 方向 | ペイロード | 説明 |
|-------|-----------|---------|-------------|
| `draw:join` | emit | `{roomId}` | Join a draw room |
| `draw:leave` | emit | `{roomId}` | Leave a draw room |
| `draw:room-state` | receive | `{room, layers, participants}` | Full room state after joining |
| `draw:stroke` | emit/receive | `{roomId, layerId, stroke}` | Real-time stroke data |
| `draw:text` | emit/receive | `{roomId, textObject}` | Text object placement/update |
| `draw:layer-update` | receive | `{roomId, layers}` | Layer data updated |
| `draw:participant-join` | receive | `{roomId, user}` | New participant joined |
| `draw:participant-leave` | receive | `{roomId, userId}` | Participant left |
| `draw:chat` | emit/receive | `{roomId, content, sender}` | Chat message |

### 典型的なフロー

1. Call `POST /api/draw/rooms/:id/join` to join via REST API
2. Emit `draw:join` with `{roomId}` via Socket.IO
3. Receive `draw:room-state` with full canvas state
4. Listen for `draw:stroke` events from other users
5. Emit `draw:stroke` when drawing locally
6. Periodically call `PUT /api/draw/rooms/:id/layers` to persist canvas state

---

## レート制限

すべての絵チャエンドポイントはデフォルトのレート制限を共有しています:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
