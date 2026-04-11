# ラジオ API（スペース / ライブオーディオ）

> **ベースURL:** `https://karotter.com/api`
>
> すべてのエンドポイントは `Authorization: Bearer {token}` ヘッダーと `X-CSRF-Token` ヘッダーによる認証が必要です。
> 一部の操作ではCookieベース認証（`karotter_at`, `karotter_rt`, `karotter_csrf`）が必要な場合があります。

---

## 目次

- [概要](#概要)
- [スペースを作成](#スペースを作成)
- [アクティブなスペース取得](#アクティブなスペース取得)
- [ICEサーバー取得](#iceサーバー取得)
- [スペース詳細取得](#スペース詳細取得)
- [スペースに参加](#スペースに参加)
- [スペースを退出](#スペースを退出)
- [スペースを終了](#スペースを終了)
- [チャットメッセージ取得](#チャットメッセージ取得)
- [チャットメッセージ送信](#チャットメッセージ送信)
- [スペース設定更新](#スペース設定更新)
- [参加者ロール変更](#参加者ロール変更)
- [参加者をミュート](#参加者をミュート)
- [スピーカー招待](#スピーカー招待)
- [スピーカー招待取消](#スピーカー招待取消)
- [スピーカーリクエスト](#スピーカーリクエスト)
- [スピーカー招待承諾](#スピーカー招待承諾)
- [スペースモードとポリシー](#スペースモードとポリシー)
- [参加者ロール](#参加者ロール)

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
| `title` | string | Yes | 1-100文字 | スペースのタイトル |

### 例

```http
POST /api/radio HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "title": "Late Night Coding Session"
}
```

### レスポンス

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

### クエリパラメータ

なし。

### レスポンス

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
| `spaces` | Space[] | アクティブなスペースオブジェクトの配列 |
| `spaces[].id` | number | スペースID |
| `spaces[].title` | string | スペースタイトル |
| `spaces[].hostId` | number | ホストのユーザーID |
| `spaces[].host` | UserBrief | ホストユーザーオブジェクト |
| `spaces[].mode` | string | アクセスモード（`"public"`, `"followers"`, `"invite"`） |
| `spaces[].isActive` | boolean | アクティブなスペースでは常に `true` |
| `spaces[].participantCount` | number | 現在の参加者数 |
| `spaces[].createdAt` | string | ISO 8601 作成タイムスタンプ |

---

## ICEサーバー取得

オーディオ接続を確立するためのWebRTC ICE（STUN/TURN）サーバー設定を返します。

```
GET /api/radio/ice-servers
```

### クエリパラメータ

なし。

### レスポンス

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

> **注意:** TURNの認証情報は時間制限付きです。各接続試行前に新しいICEサーバーを取得してください。

---

## スペース詳細取得

全参加者を含む特定のスペースの詳細情報を返します。

```
GET /api/radio/:id
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### レスポンス

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

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"space": {...}}` | スペース詳細 |
| 404 | `{"error": "スペースが見つかりません"}` | スペースが見つからない |

---

## スペースに参加

アクティブなスペースにリスナーとして参加します。

```
POST /api/radio/:id/join
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### リクエストボディ

なし。

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スペースに参加しました", "space": {...}}` | 参加成功 |
| 400 | `{"error": "既に参加しています"}` | 既にスペースに参加中 |
| 403 | `{"error": "このスペースに参加できません"}` | アクセス制限（招待制、フォロワー限定） |
| 404 | `{"error": "スペースが見つかりません"}` | スペースが見つからないまたは終了済み |

### 例

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

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### リクエストボディ

なし。

### レスポンス

```json
{
  "message": "スペースから退出しました"
}
```

> **注意:** HOSTがスペースを終了せずに退出した場合、ホスト権限は別のスピーカーに移譲される場合があります。スペースを適切に閉じるには、[スペースを終了](#スペースを終了)エンドポイントを使用してください。

---

## スペースを終了

スペースを完全に終了します。HOSTのみスペースを終了できます。すべての参加者が切断されます。

```
POST /api/radio/:id/end
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### リクエストボディ

なし。

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スペースを終了しました"}` | スペース終了完了 |
| 403 | `{"error": "権限がありません"}` | ホストではない |

---

## チャットメッセージ取得

スペースのテキストチャットメッセージを返します。

```
GET /api/radio/:id/messages
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### クエリパラメータ

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | number | 1 | ページ番号 |
| `limit` | number | 50 | 1ページあたりのメッセージ数 |

### レスポンス

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

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `content` | string | Yes | メッセージテキスト |

### 例

```http
POST /api/radio/301/messages HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "content": "Can you talk about TypeScript next?"
}
```

### レスポンス

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

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `title` | string | No | 新しいタイトル（1-100文字） |
| `mode` | string | No | アクセスモード: `"public"`, `"followers"`, `"invite"` |
| `speakerPolicy` | string | No | 発言できる人: `"following"`, `"everyone"`, `"invited"` |

### 例

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

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "設定を更新しました", "space": {...}}` | 設定更新完了 |
| 403 | `{"error": "権限がありません"}` | ホストではない |

---

## 参加者ロール変更

スペースでの参加者のロールを変更します。HOSTのみロールを変更できます。

```
PATCH /api/radio/:id/participants/:userId/role
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |
| `userId` | number | Yes | 対象参加者のユーザーID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `role` | string | Yes | 新しいロール: `"HOST"`, `"SPEAKER"`, または `"LISTENER"` |

### 例

```http
PATCH /api/radio/301/participants/18179/role HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "role": "SPEAKER"
}
```

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ロールを変更しました"}` | ロール変更完了 |
| 403 | `{"error": "権限がありません"}` | ホストではない |
| 404 | `{"error": "参加者が見つかりません"}` | ユーザーがスペースにいない |

> **注意:** 誰かをHOSTに昇格させるとホスト権限が移譲されます。元のホストはSPEAKERになります。

---

## 参加者をミュート

参加者をミュート/ミュート解除します。HOSTは誰でもミュートできます。参加者は自分自身をミュートすることもできます。

```
PATCH /api/radio/:id/participants/:userId/mute
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |
| `userId` | number | Yes | 対象参加者のユーザーID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `isMuted` | boolean | Yes | `true`でミュート、`false`でミュート解除 |

### 例

```http
PATCH /api/radio/301/participants/18179/mute HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "isMuted": true
}
```

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "ミュートを変更しました"}` | ミュート状態変更完了 |
| 403 | `{"error": "権限がありません"}` | ホストではない（かつ自己ミュートでもない） |

---

## スピーカー招待

リスナーにスピーカー招待を送信します。HOSTのみスピーカーを招待できます。

```
POST /api/radio/:id/participants/:userId/invite-speaker
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |
| `userId` | number | Yes | 対象リスナーのユーザーID |

### リクエストボディ

なし。

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を送信しました"}` | 招待送信完了 |
| 400 | `{"error": "既にスピーカーです"}` | ユーザーは既にスピーカー |
| 403 | `{"error": "権限がありません"}` | ホストではない |

---

## スピーカー招待取消

以前に送信したスピーカー招待をキャンセルします。

```
DELETE /api/radio/:id/participants/:userId/invite-speaker
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |
| `userId` | number | Yes | 対象ユーザーID |

### リクエストボディ

なし。

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を取り消しました"}` | 招待取消完了 |
| 403 | `{"error": "権限がありません"}` | ホストではない |

---

## スピーカーリクエスト

スピーカーへの昇格をホストにリクエストします。リスナーが利用可能です。

```
POST /api/radio/:id/request-speaker
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### リクエストボディ

なし。

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スピーカーリクエストを送信しました"}` | リクエスト送信完了 |
| 400 | `{"error": "既にスピーカーです"}` | 既にスピーカー |

---

## スピーカー招待承諾

ホストから送信されたスピーカー招待を承諾します。

```
POST /api/radio/:id/accept-speaker-invite
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | スペースID |

### リクエストボディ

なし。

### レスポンスs

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を承諾しました"}` | 承諾完了、SPEAKERになりました |
| 400 | `{"error": "招待が見つかりません"}` | 保留中のスピーカー招待がない |

---

## スペースモードとポリシー

### アクセスモード (`mode`)

| モード | 説明 |
|------|-------------|
| `public` | 誰でも参加してリッスンできる |
| `followers` | ホストをフォローしている人のみ参加可能 |
| `invite` | 明示的に招待されたユーザーのみ参加可能 |

### スピーカーポリシー (`speakerPolicy`)

| ポリシー | 説明 |
|--------|-------------|
| `everyone` | すべての参加者がスピーカーリクエスト可能で自動承認 |
| `following` | ホストがフォローしているユーザーのみスピーカーリクエスト可能 |
| `invited` | ホストが明示的に招待したユーザーのみ発言可能 |

### モード + ポリシーの組み合わせ

| モード | スピーカーポリシー | 参加可能 | 発言可能 |
|------|---------------|-------------|---------------|
| `public` | `everyone` | 誰でも | リクエストした人 |
| `public` | `following` | 誰でも | ホストのフォロー中でリクエストした人 |
| `public` | `invited` | 誰でも | 招待されたユーザーのみ |
| `followers` | `everyone` | ホストのフォロワー | リクエストしたフォロワー |
| `followers` | `invited` | ホストのフォロワー | 招待されたフォロワーのみ |
| `invite` | `invited` | 招待されたユーザーのみ | スピーカー招待されたユーザーのみ |

---

## 参加者ロール

| ロール | 権限 |
|------|------------|
| `HOST` | 全権限: スペース終了、設定変更、ロール管理、誰でもミュート、スピーカー招待 |
| `SPEAKER` | 発言（音声）、チャットメッセージ送信、セルフミュートが可能 |
| `LISTENER` | 音声のリッスン、チャットメッセージ送信、スピーカーリクエストが可能 |

### ロール階層

```
HOST > SPEAKER > LISTENER
```

- HOSTは同時に1人のみ。別のユーザーをHOSTに昇格させると、現在のホストはSPEAKERに降格されます。
- SPEAKERはHOSTによってLISTENERに降格できます。
- LISTENERはHOSTによって、またはスピーカーリクエスト/招待フローでSPEAKERに昇格できます。

---

## レート制限

すべてのラジオエンドポイントはデフォルトのレート制限を共有しています:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
