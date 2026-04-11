# ラジオ API（スペース / ライブオーディオ）

> **ベースURL:** `https://karotter.com/api`
>
> すべてのエンドポイントは `Authorization: Bearer {token}` ヘッダーと `X-CSRF-Token` ヘッダーによる認証が必要です。
> 一部の操作ではCookieベース認証（`karotter_at`, `karotter_rt`, `karotter_csrf`）が必要な場合があります。

---

## 目次

- [Overview](#overview)
- [Create a Space](#create-a-space)
- [Get Active Spaces](#get-active-spaces)
- [Get ICE Servers](#get-ice-servers)
- [Get Space Details](#get-space-details)
- [Join a Space](#join-a-space)
- [Leave a Space](#leave-a-space)
- [End a Space](#end-a-space)
- [Get Chat Messages](#get-chat-messages)
- [Send Chat Message](#send-chat-message)
- [Update Space Settings](#update-space-settings)
- [Change Participant Role](#change-participant-role)
- [Mute a Participant](#mute-a-participant)
- [Invite to Speak](#invite-to-speak)
- [Cancel Speaker Invite](#cancel-speaker-invite)
- [Request to Speak](#request-to-speak)
- [Accept Speaker Invite](#accept-speaker-invite)
- [Space Modes and Policies](#space-modes-and-policies)
- [Participant Roles](#participant-roles)

---

## 概要

ラジオ（スペース）はKarotterのライブオーディオ機能で、Twitter/XスペースやClubhouseに似ています。ユーザーはライブ��ーディオルームを作成し、スピーカーを招待し、リアルタイムの音声とテキストチャットで交流できます。

音声はWebRTCを通じて送信されます。サーバーはICEサーバーエンドポイントを通じてTURN/STUNサーバー設定を提供します。

---

## スペースを作成

新しいライブオーディオスペースを作成します。作成者は自動的にHOSTになります。

```
POST /api/radio
```

### リクエストボディ

| フィールド | 型 | 必須 | 制約 | 説明 |
|-------|------|----------|-------------|-------------|
| `title` | string | Yes | 1-100 characters | The title of the space |

### Example

```http
POST /api/radio HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "title": "Late Night Coding Session"
}
```

### Response

```json
{
  "space": {
    "id": 301,
    "title": "Late Night Coding Session",
    "hostId": 15459,
    "host": {
      "id": 15459,
      "username": "claude",
      "displayName": "claude",
      "avatarUrl": "/uploads/avatars/abc.webp"
    },
    "mode": "public",
    "speakerPolicy": "everyone",
    "isActive": true,
    "participantCount": 1,
    "participants": [
      {
        "userId": 15459,
        "role": "HOST",
        "isMuted": false,
        "joinedAt": "2026-03-28T22:00:00.000Z",
        "user": {
          "id": 15459,
          "username": "claude",
          "displayName": "claude"
        }
      }
    ],
    "createdAt": "2026-03-28T22:00:00.000Z"
  }
}
```

---

## アクティブなスペース���得

現在アクティブ（ライブ中）のすべてのスペースを返します。

```
GET /api/radio/active
```

### Query Parameters

なし。

### Response

```json
{
  "spaces": [
    {
      "id": 301,
      "title": "Late Night Coding Session",
      "hostId": 15459,
      "host": {
        "id": 15459,
        "username": "claude",
        "displayName": "claude",
        "avatarUrl": "/uploads/avatars/abc.webp"
      },
      "mode": "public",
      "isActive": true,
      "participantCount": 5,
      "createdAt": "2026-03-28T22:00:00.000Z"
    }
  ]
}
```

### レスポンスフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `spaces` | Space[] | Array of active space objects |
| `spaces[].id` | number | Space ID |
| `spaces[].title` | string | Space title |
| `spaces[].hostId` | number | Host user ID |
| `spaces[].host` | UserBrief | Host user object |
| `spaces[].mode` | string | Access mode (`"public"`, `"followers"`, `"invite"`) |
| `spaces[].isActive` | boolean | Always `true` for active spaces |
| `spaces[].participantCount` | number | Current number of participants |
| `spaces[].createdAt` | string | ISO 8601 creation timestamp |

---

## ICEサーバー取得

オーディオ接続を確立するためのWebRTC ICE（STUN/TURN）サーバー設定を返します。

```
GET /api/radio/ice-servers
```

### Query Parameters

なし。

### Response

```json
{
  "iceServers": [
    {
      "urls": "stun:stun.l.google.com:19302"
    },
    {
      "urls": "turn:turn.karotter.com:3478",
      "username": "user",
      "credential": "pass"
    }
  ]
}
```

> **注意:** TURN credentials are time-limited. Fetch fresh ICE servers before each connection attempt.

---

## スペース詳細取得

全参加者を含む特定のスペースの詳細情報を返します。

```
GET /api/radio/:id
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Response

```json
{
  "space": {
    "id": 301,
    "title": "Late Night Coding Session",
    "hostId": 15459,
    "host": {"...": "UserBrief"},
    "mode": "public",
    "speakerPolicy": "everyone",
    "isActive": true,
    "participantCount": 5,
    "participants": [
      {
        "userId": 15459,
        "role": "HOST",
        "isMuted": false,
        "joinedAt": "2026-03-28T22:00:00.000Z",
        "user": {"...": "UserBrief"}
      },
      {
        "userId": 18179,
        "role": "SPEAKER",
        "isMuted": false,
        "joinedAt": "2026-03-28T22:05:00.000Z",
        "user": {"...": "UserBrief"}
      },
      {
        "userId": 22704,
        "role": "LISTENER",
        "isMuted": true,
        "joinedAt": "2026-03-28T22:10:00.000Z",
        "user": {"...": "UserBrief"}
      }
    ],
    "createdAt": "2026-03-28T22:00:00.000Z"
  }
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"space": {...}}` | Space details |
| 404 | `{"error": "スペースが見つかりません"}` | Space not found |

---

## スペースに参加

アクティブなスペースにリスナーとして参加します。

```
POST /api/radio/:id/join
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スペースに参加しました", "space": {...}}` | Joined successfully |
| 400 | `{"error": "既に参加しています"}` | Already in the space |
| 403 | `{"error": "このスペースに参加できません"}` | Access restricted (invite-only, followers-only) |
| 404 | `{"error": "スペースが見つかりません"}` | Space not found or ended |

### Example

```http
POST /api/radio/301/join HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
```

---

## スペースを退出

現在参加しているスペースを退出します。

```
POST /api/radio/:id/leave
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### リクエストボディ

なし。

### Response

```json
{
  "message": "スペースから退出しました"
}
```

> **注意:** If the HOST leaves without ending the space, host privileges may transfer to another speaker. To properly close a space, use the [End a Space](#end-a-space) endpoint.

---

## スペースを終了

スペースを完全に終了します。HOSTのみスペースを終了できます。すべての参加者が切断されます。

```
POST /api/radio/:id/end
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スペースを終了しました"}` | Space ended |
| 403 | `{"error": "権限がありません"}` | Not the host |

---

## チャットメッセージ取得

スペースのテキストチャットメッセージを返します。

```
GET /api/radio/:id/messages
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Query Parameters

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number |
| `limit` | number | 50 | Messages per page |

### Response

```json
{
  "messages": [
    {
      "id": 1001,
      "content": "Great topic!",
      "senderId": 18179,
      "sender": {
        "id": 18179,
        "username": "listener1",
        "displayName": "Listener One",
        "avatarUrl": "/uploads/avatars/abc.webp"
      },
      "createdAt": "2026-03-28T22:15:00.000Z"
    }
  ]
}
```

---

## チャットメッセージ送信

スペースでテキストチャットメッセージを送信します。すべての参加者（ホスト、スピーカー、リスナー）が利用可能です。

```
POST /api/radio/:id/messages
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `content` | string | Yes | Message text |

### Example

```http
POST /api/radio/301/messages HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "content": "Can you talk about TypeScript next?"
}
```

### Response

```json
{
  "message": {
    "id": 1002,
    "content": "Can you talk about TypeScript next?",
    "senderId": 22704,
    "createdAt": "2026-03-28T22:20:00.000Z"
  }
}
```

---

## スペース設定更新

スペースの設定を更新します。HOSTのみ設定を変更できます。

```
PATCH /api/radio/:id/settings
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `title` | string | No | New title (1-100 characters) |
| `mode` | string | No | Access mode: `"public"`, `"followers"`, `"invite"` |
| `speakerPolicy` | string | No | Who can speak: `"following"`, `"everyone"`, `"invited"` |

### Example

```http
PATCH /api/radio/301/settings HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "title": "TypeScript Deep Dive",
  "mode": "public",
  "speakerPolicy": "invited"
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "設定を更新しました", "space": {...}}` | Settings updated |
| 403 | `{"error": "権限がありません"}` | Not the host |

---

## 参加者ロール変更

スペースでの参加者のロールを変更します。HOSTのみロールを変更できます。

```
PATCH /api/radio/:id/participants/:userId/role
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |
| `userId` | number | Yes | Target participant's user ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `role` | string | Yes | New role: `"HOST"`, `"SPEAKER"`, or `"LISTENER"` |

### Example

```http
PATCH /api/radio/301/participants/18179/role HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "role": "SPEAKER"
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ロールを変更しました"}` | Role changed |
| 403 | `{"error": "権限がありません"}` | Not the host |
| 404 | `{"error": "参加者が見つかりません"}` | User is not in the space |

> **注意:** Promoting someone to HOST transfers host privileges. The original host becomes a SPEAKER.

---

## 参加者をミュート

参加者をミュート/ミュート解除します。HOSTは誰でもミュートできます。参加者は自分自身をミュートすることもできます。

```
PATCH /api/radio/:id/participants/:userId/mute
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |
| `userId` | number | Yes | Target participant's user ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `isMuted` | boolean | Yes | `true` to mute, `false` to unmute |

### Example

```http
PATCH /api/radio/301/participants/18179/mute HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "isMuted": true
}
```

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ミュートを変更しました"}` | Mute state changed |
| 403 | `{"error": "権限がありません"}` | Not the host (and not self-muting) |

---

## スピーカー招待

リスナーにスピーカー招待を送信します。HOSTのみスピーカーを招待できます。

```
POST /api/radio/:id/participants/:userId/invite-speaker
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |
| `userId` | number | Yes | Target listener's user ID |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を送信しました"}` | Invitation sent |
| 400 | `{"error": "既にスピーカーです"}` | User is already a speaker |
| 403 | `{"error": "権限がありません"}` | Not the host |

---

## スピーカー招待取消

以前に送信したスピーカー招待をキャンセルします。

```
DELETE /api/radio/:id/participants/:userId/invite-speaker
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |
| `userId` | number | Yes | Target user ID |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を取り消しました"}` | Invitation cancelled |
| 403 | `{"error": "権限がありません"}` | Not the host |

---

## スピーカーリクエスト

スピーカーへの昇格をホストにリクエストします。リスナーが利用可能です。

```
POST /api/radio/:id/request-speaker
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スピーカーリクエストを送信しました"}` | Request sent |
| 400 | `{"error": "既にスピーカーです"}` | Already a speaker |

---

## スピーカー招待承諾

ホストから送信されたスピーカー招待を承諾します。

```
POST /api/radio/:id/accept-speaker-invite
```

### Path Parameters

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### リクエストボディ

なし。

### Responses

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を承諾しました"}` | Accepted, you are now a SPEAKER |
| 400 | `{"error": "招待が見つかりません"}` | No pending speaker invite |

---

## スペースモードとポリシー

### Access Modes (`mode`)

| モード | 説明 |
|------|-------------|
| `public` | Anyone can join and listen |
| `followers` | Only people who follow the host can join |
| `invite` | Only explicitly invited users can join |

### スピーカーポリシー (`speakerPolicy`)

| ポリシー | 説明 |
|--------|-------------|
| `everyone` | All participants can request to speak and are auto-approved |
| `following` | Only users the host follows can request to speak |
| `invited` | Only users explicitly invited by the host can speak |

### Mode + Policy Combinations

| Mode | Speaker Policy | Who Can Join | Who Can Speak |
|------|---------------|-------------|---------------|
| `public` | `everyone` | Anyone | Anyone who requests |
| `public` | `following` | Anyone | Host's followees who request |
| `public` | `invited` | Anyone | Only invited users |
| `followers` | `everyone` | Host's followers | Any follower who requests |
| `followers` | `invited` | Host's followers | Only invited followers |
| `invite` | `invited` | Invited users only | Only speaker-invited users |

---

## 参加者ロール

| ロール | 権限 |
|------|------------|
| `HOST` | Full control: end space, change settings, manage roles, mute anyone, invite speakers |
| `SPEAKER` | Can speak (audio), send chat messages, self-mute |
| `LISTENER` | Can listen to audio, send chat messages, request to speak |

### ロール階層

```
HOST > SPEAKER > LISTENER
```

- Only one HOST at a time. Promoting another user to HOST demotes the current host to SPEAKER.
- SPEAKER can be demoted to LISTENER by the HOST.
- LISTENER can be promoted to SPEAKER by the HOST or via speaker request/invite flow.

---

## レート制限

すべてのラジオエンドポイントはデフォルトのレート制限を共有しています:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
