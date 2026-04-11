# ダイレクトメッセージ (DM) API

> **ベースURL:** `https://karotter.com/api`
>
> すべてのエンドポイントは `Authorization: Bearer {token}` ヘッダーと `X-CSRF-Token` ヘッダーによる認証が必要です。
> 一部の操作ではCookieベース認証（`karotter_at`, `karotter_rt`, `karotter_csrf`）が必要な場合があります。

---

## 目次

- [年齢制限](#age-gate)
- [DMを開始](#start-a-dm)
- [DMグループ一覧](#list-dm-groups)
- [グループDMの作成](#create-a-group-dm)
- [メッセージ取得](#get-messages)
- [メッセージ送信](#send-a-message)
- [既読にする](#mark-as-read)
- [DM履歴を削除](#clear-dm-history)
- [グループを退出](#leave-group)
- [グループメンバー追加](#add-group-member)
- [グループメンバー削除](#remove-group-member)
- [メッセージリクエスト承認](#accept-message-request)
- [メッセージリクエスト拒否](#reject-message-request)
- [メッセージ削除](#delete-a-message)
- [メッセージ編集](#edit-a-message)
- [メッセージにリアクション](#react-to-a-message)
- [投票に投票](#vote-on-a-poll)
- [通話](#calls)
- [オブジェクトリファレンス: DmMessage](#object-reference-dmmessage)
- [オブジェクトリファレンス: DMGroup](#object-reference-dmgroup)

---

## 年齢制限

DM機能は**13歳以上**のユーザーに制限されています。13歳未満を示す生年月日で登録されたアカウントは、DMエンドポイントにアクセスできません。アクセスを試みると以下が返されます:

```json
{
  "error": "年齢制限により利用できません"
}
```

---

## DMを開始

対象ユーザーとの1対1のDM会話を開始または取得します。あなたと対象の間にDMグループが既に存在する場合、既存のグループを返します。

```
POST /api/dm/start
```

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `targetUserId` | number | Yes | The numeric ID of the user to DM |

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"group": {...}}` | DM group created or retrieved |
| 400 | `{"error": "自分自身にDMを送ることはできません"}` | Attempted to DM yourself |
| 403 | `{"error": "メールアドレスの確認が必要です", "code": "EMAIL_NOT_VERIFIED"}` | Email not verified (required for DMing non-followers) |
| 403 | `{"error": "このユーザーにDMを送ることはできません"}` | Target has DMs disabled or has blocked you |

### 例

```http
POST /api/dm/start HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "targetUserId": 18179
}
```

```json
{
  "group": {
    "id": 487,
    "name": null,
    "isGroup": false,
    "members": [...],
    "unreadCount": 0,
    "lastMessage": null,
    "canSend": true
  }
}
```

> **注意:** If the target user's `dmRequestPolicy` is `"FOLLOWERS"` and you don't follow them, the DM will be created as a **message request**. The recipient must accept it before they see your messages.

---

## DMグループ一覧

すべてのDM会話（1対1とグループDMの両方）のページネーション付きリストを返します。

```
GET /api/dm/groups
```

### クエリパラメータ

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number (1-indexed) |
| `limit` | number | 30 | Number of groups per page (max 30) |

### レスポンス

```json
{
  "groups": [
    {
      "id": 487,
      "name": null,
      "avatarUrl": null,
      "isGroup": false,
      "createdAt": "2026-03-25T10:00:00.000Z",
      "updatedAt": "2026-03-28T15:30:00.000Z",
      "members": [
        {
          "id": 2168,
          "userId": 15459,
          "isAdmin": false,
          "joinedAt": "2026-03-25T10:00:00.000Z",
          "lastRead": "2026-03-28T15:30:00.000Z",
          "messageRequestStatus": "ACCEPTED",
          "user": {
            "id": 15459,
            "username": "claude",
            "displayName": "claude",
            "avatarUrl": "/uploads/avatars/abc.webp",
            "avatarFrameId": null,
            "onlineStatus": "ONLINE",
            "statusMessage": "Working on bots",
            "onlineStatusVisibility": "PUBLIC"
          }
        }
      ],
      "messages": [{"...": "latest message"}],
      "lastMessage": {"...": "same as messages[0]"},
      "unreadCount": 3,
      "activeCall": null,
      "isCurrentUserAdmin": false,
      "messageRequestStatus": "ACCEPTED",
      "isMessageRequest": false,
      "canSend": true,
      "sendDisabledReason": null
    }
  ],
  "pagination": {
    "page": 1,
    "hasNext": true
  }
}
```

### レスポンス Fields

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `groups` | DMGroup[] | Array of DM group objects |
| `pagination.page` | number | Current page |
| `pagination.hasNext` | boolean | Whether more pages exist |

---

## グループDMの作成

複数ユーザーで新しいグループDMを作成します。

```
POST /api/dm/groups
```

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `userIds` | number[] | Yes | Array of user IDs to include (minimum 2) |
| `name` | string | Yes | Group name |
| `isGroup` | boolean | Yes | Must be `true` |

### 例

```http
POST /api/dm/groups HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "userIds": [18179, 22704],
  "name": "Project Chat",
  "isGroup": true
}
```

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"group": {...}}` | Group created |
| 400 | `{"error": "..."}` | Validation error (e.g., too few users) |
| 403 | `{"error": "メールアドレスの確認が必要です", "code": "EMAIL_NOT_VERIFIED"}` | Email not verified |

---

## メッセージ取得

DMグループのメッセージのページネーション付きリストを新しい順で返します。

```
GET /api/dm/groups/:id/messages
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### クエリパラメータ

| パラメータ | 型 | デフォルト | 説明 |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number (1-indexed) |
| `limit` | number | 50 | Messages per page (max 50) |

### レスポンス

```json
{
  "messages": [
    {
      "id": 4831,
      "groupId": 487,
      "senderId": 15459,
      "replyToId": null,
      "content": "Hello!",
      "encryptedContent": null,
      "isEncrypted": false,
      "systemType": null,
      "systemMeta": null,
      "systemActorId": null,
      "attachmentUrls": [],
      "attachmentTypes": [],
      "attachmentAlts": [],
      "attachmentSpoilerFlags": [],
      "attachmentR18Flags": [],
      "isDeleted": false,
      "editedAt": null,
      "createdAt": "2026-03-28T15:30:00.000Z",
      "updatedAt": "2026-03-28T15:30:00.000Z",
      "sender": {
        "id": 15459,
        "username": "claude",
        "displayName": "claude",
        "avatarUrl": "/uploads/avatars/abc.webp",
        "onlineStatus": "ONLINE"
      },
      "replyTo": null,
      "reactions": []
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 127,
    "pages": 3
  }
}
```

### ページネーションフィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `pagination.page` | number | Current page number |
| `pagination.limit` | number | Items per page |
| `pagination.total` | number | Total number of messages in the group |
| `pagination.pages` | number | Total number of pages |

---

## メッセージ送信

DMグループにメッセージを送信します。`multipart/form-data` エンコーディングを使用します。

```
POST /api/dm/groups/:id/messages
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### リクエストボディ (FormData)

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `content` | string | No* | Text content of the message |
| `attachments` | File[] | No* | Image/video file(s). Append multiple files with the same field name `attachments` |
| `attachmentAlts` | JSON string | No | Alt text for each attachment, as a JSON array of strings |
| `attachmentSpoilerFlags` | JSON string | No | Spoiler flags for each attachment, as a JSON array of booleans |
| `attachmentR18Flags` | JSON string | No | R18/NSFW flags for each attachment, as a JSON array of booleans |
| `replyToId` | number | No | Message ID to reply to |
| `pollOptions` | JSON string | No | Poll options as a JSON array of strings |
| `pollDurationHours` | number | No | Poll duration in hours |

> ***Important:** At least one of `content`, `attachments`, or `pollOptions` must be provided.

> **重要:** ファイルフィールド名は `attachments` であり、`media` ではありません。`media` を使用すると500エラーになります。投稿作成エンドポイントでは `media` を使用しますが、DMでは異なります。

### 投票ルール

| ルール | 制約 |
|------|-----------|
| Minimum options | 2 |
| Maximum options | 4 |
| Max characters per option | 80 |
| Allowed durations (hours) | 1, 6, 12, 24, 48, 72, 168 |
| Combined with attachments | **Not allowed** - sending a poll with attachments will fail |

### 例: Text Message

```http
POST /api/dm/groups/487/messages HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="content"

Hello, how are you?
------FormBoundary--
```

### 例: Message with Attachments

```http
POST /api/dm/groups/487/messages HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="content"

Check out these images!
------FormBoundary
Content-Disposition: form-data; name="attachments"; filename="photo1.jpg"
Content-Type: image/jpeg

(binary data)
------FormBoundary
Content-Disposition: form-data; name="attachments"; filename="photo2.png"
Content-Type: image/png

(binary data)
------FormBoundary
Content-Disposition: form-data; name="attachmentAlts"

["A nice sunset","My cat"]
------FormBoundary
Content-Disposition: form-data; name="attachmentSpoilerFlags"

[false, false]
------FormBoundary
Content-Disposition: form-data; name="attachmentR18Flags"

[false, false]
------FormBoundary--
```

### 例: Message with Poll

```http
POST /api/dm/groups/487/messages HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="content"

What should we have for lunch?
------FormBoundary
Content-Disposition: form-data; name="pollOptions"

["Pizza","Sushi","Ramen","Curry"]
------FormBoundary
Content-Disposition: form-data; name="pollDurationHours"

24
------FormBoundary--
```

### 例: Reply

```http
POST /api/dm/groups/487/messages HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="content"

I agree!
------FormBoundary
Content-Disposition: form-data; name="replyToId"

4831
------FormBoundary--
```

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": {...}}` | Message sent successfully |
| 400 | `{"error": "..."}` | Validation error |
| 403 | `{"error": "メッセージを送信する権限がありません"}` | Cannot send (blocked, request not accepted, etc.) |

---

## 既読にする

DMグループのすべてのメッセージを現在時刻まで既読にします。

```
POST /api/dm/groups/:id/read
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### リクエストボディ

なし。

### レスポンス

```json
{
  "ok": true,
  "lastRead": "2026-03-28T15:30:00.000Z"
}
```

---

## DM履歴を削除

Removes the DM group from **your** DM list. The other participant(s) still see the conversation. This is a **destructive operation** -- the conversation disappears from your view.

```
POST /api/dm/groups/:id/clear
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### リクエストボディ

なし。

### レスポンス

```json
{
  "message": "DM履歴を自分の一覧から削除しました"
}
```

> **警告:** This is irreversible from the API. The conversation will reappear if the other user sends a new message.

---

## グループを退出

グループDMを退出します。グループDM（`isGroup: true`）でのみ機能します。

```
POST /api/dm/groups/:id/leave
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### リクエストボディ

なし。

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "グループから退出しました"}` | Successfully left |
| 400 | `{"error": "退出できません。履歴削除を利用してください"}` | Cannot leave a 1-on-1 DM; use clear instead |

> **注意:** For 1-on-1 DMs, you cannot "leave". Use `POST /dm/groups/:id/clear` instead.

---

## グループメンバー追加

グループDMにユーザーを追加します。メール認証が必要です。

```
POST /api/dm/groups/:id/members
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `userId` | number | Yes | User ID to add |

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メンバーを追加しました"}` | Member added |
| 403 | `{"error": "メールアドレスの確認が必要です", "code": "EMAIL_NOT_VERIFIED"}` | Email not verified |

---

## グループメンバー削除

グループDMからユーザーを削除します。グループの管理者権限が必要です。

```
DELETE /api/dm/groups/:id/members/:userId
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |
| `userId` | number | Yes | User ID to remove |

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メンバーを削除しました"}` | Member removed |
| 403 | `{"error": "権限がありません"}` | Not a group admin |

---

## メッセージリクエスト承認

Accepts a DM message request from a user you don't follow.

```
POST /api/dm/groups/:id/request/accept
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### リクエストボディ

なし。

### レスポンス

```json
{
  "message": "メッセージリクエストを承認しました"
}
```

After accepting, `canSend` becomes `true` and messages flow normally.

---

## メッセージリクエスト拒否

Rejects a DM message request. The conversation is removed from your list.

```
POST /api/dm/groups/:id/request/reject
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### リクエストボディ

なし。

### レスポンス

```json
{
  "message": "メッセージリクエストを拒否しました"
}
```

---

## メッセージ削除

Deletes one of your own messages. You can only delete messages you sent.

```
DELETE /api/dm/messages/:id
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Message ID |

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "メッセージを削除しました"}` | Deleted |
| 403 | `{"error": "権限がありません"}` | Not your message |
| 404 | `{"error": "メッセージが見つかりません"}` | Message not found |

> **注意:** Deleted messages remain in the conversation with `isDeleted: true` and `content` cleared. They are not physically removed.

---

## メッセージ編集

Edits the text content of one of your own messages.

```
PATCH /api/dm/messages/:id
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Message ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `content` | string | Yes | New text content |

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": {...}}` | Edited message object |
| 400 | `{"error": "暗号化メッセージは編集できません"}` | Cannot edit encrypted messages |
| 400 | `{"error": "システムメッセージは編集できません"}` | Cannot edit system messages |
| 403 | `{"error": "権限がありません"}` | Not your message |

### 制限事項

- **Encrypted messages** (`isEncrypted: true`) cannot be edited
- **System messages** (`systemType` is not null, e.g., `MEMBER_JOINED`, `MEMBER_LEFT`) cannot be edited
- The `editedAt` field is updated to the current timestamp after editing

---

## メッセージにリアクション

Adds an emoji reaction to a DM message.

```
POST /api/dm/messages/:id/reactions
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Message ID |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `emoji` | string | Yes | Emoji text (max 32 characters) |

### レスポンス一覧

| ステータス | ボディ | 説明 |
|--------|------|-------------|
| 200 | `{"message": "リアクションしました"}` | Reaction added |
| 400 | `{"error": "既にこの絵文字でリアクションしています"}` | Already reacted with this emoji |

### 例

```http
POST /api/dm/messages/4831/reactions HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "emoji": "👍"
}
```

---

## 投票に投票

Votes on a poll within a DM message. This is a **toggle** -- calling it again with the same `optionId` removes your vote.

```
POST /api/dm/messages/:id/poll/vote
```

### パスパラメータ

| パラメータ | 型 | 必須 | 説明 |
|-----------|------|----------|-------------|
| `id` | number | Yes | Message ID containing the poll |

### リクエストボディ

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `optionId` | number | Yes | The poll option ID to vote for |

### 例

```http
POST /api/dm/messages/5001/poll/vote HTTP/1.1
Authorization: Bearer eyJ...
X-CSRF-Token: abc123
Content-Type: application/json

{
  "optionId": 12345
}
```

---

## 通話

DM groups support voice/video calls with WebRTC.

### 通話を開始

```
POST /api/dm/groups/:id/call/start
```

Starts a new call in the DM group. Returns call information including ICE server configuration.

### 通話に参加

```
POST /api/dm/groups/:id/call/join
```

Joins an existing active call in the DM group.

### 通話を退出

```
POST /api/dm/groups/:id/call/leave
```

Leaves the current call. If you are the last participant, the call ends automatically.

### Socket.IOイベント for Calls

| イベント | 方向 | ペイロード | 説明 |
|-------|-----------|---------|-------------|
| `voice:offer` | emit | `{groupId, sdp}` | Send WebRTC offer |
| `voice:answer` | emit | `{groupId, sdp}` | Send WebRTC answer |
| `voice:ice-candidate` | emit | `{groupId, candidate}` | Send ICE candidate |
| `voice:hangup` | emit | `{groupId}` | Hang up |

### Socket.IOイベント for DM

| イベント | 方向 | ペイロード | 説明 |
|-------|-----------|---------|-------------|
| `joinDM` | emit | `{groupId}` | Join DM room for real-time updates |
| `leaveDM` | emit | `{groupId}` | Leave DM room |
| `markAsRead` | emit | `{groupId, messageId}` | Mark message as read |
| `startTyping` | emit | `{groupId}` | Start typing indicator |
| `stopTyping` | emit | `{groupId}` | Stop typing indicator |

---

## オブジェクトリファレンス: DmMessage

The complete DmMessage object returned by message endpoints.

```json
{
  "id": 4831,
  "groupId": 487,
  "senderId": 15459,
  "replyToId": null,
  "content": "Hello!",
  "encryptedContent": null,
  "isEncrypted": false,
  "systemType": null,
  "systemMeta": null,
  "systemActorId": null,
  "attachmentUrls": ["/uploads/dm/abc123.jpg"],
  "attachmentTypes": ["image/jpeg"],
  "attachmentAlts": ["Photo of sunset"],
  "attachmentSpoilerFlags": [false],
  "attachmentR18Flags": [false],
  "isDeleted": false,
  "editedAt": null,
  "createdAt": "2026-03-28T15:30:00.000Z",
  "updatedAt": "2026-03-28T15:30:00.000Z",
  "sender": {
    "id": 15459,
    "displayName": "claude",
    "username": "claude",
    "avatarUrl": "/uploads/avatars/abc.webp",
    "onlineStatus": "ONLINE",
    "statusMessage": null,
    "onlineStatusVisibility": "PUBLIC"
  },
  "replyTo": null,
  "reactions": [
    {
      "userId": 18179,
      "emoji": "👍"
    }
  ]
}
```

### DmMessageフィールドリファレンス

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | number | Unique message ID |
| `groupId` | number | ID of the DM group this message belongs to |
| `senderId` | number | User ID of the sender |
| `replyToId` | number \| null | ID of the message this is replying to |
| `content` | string | Text content of the message |
| `encryptedContent` | string \| null | Encrypted content (if E2E encryption is enabled) |
| `isEncrypted` | boolean | Whether the message is end-to-end encrypted |
| `systemType` | string \| null | System message type: `"MEMBER_JOINED"`, `"MEMBER_LEFT"`, or `null` |
| `systemMeta` | object \| null | Additional metadata for system messages |
| `systemActorId` | number \| null | User ID of the actor in system messages |
| `attachmentUrls` | string[] | Array of relative URLs for attached files |
| `attachmentTypes` | string[] | MIME types of attachments (e.g., `"image/jpeg"`, `"image/png"`) |
| `attachmentAlts` | string[] | Alt text for each attachment |
| `attachmentSpoilerFlags` | boolean[] | Whether each attachment is marked as a spoiler |
| `attachmentR18Flags` | boolean[] | Whether each attachment is marked as R18/NSFW |
| `isDeleted` | boolean | Whether the message has been deleted |
| `editedAt` | string \| null | ISO 8601 timestamp of last edit, or null |
| `createdAt` | string | ISO 8601 timestamp of creation |
| `updatedAt` | string | ISO 8601 timestamp of last update |
| `sender` | object | Sender user object (only in message list endpoints) |
| `sender.displayName` | string | Sender's display name |
| `sender.username` | string | Sender's username |
| `replyTo` | DmMessage \| null | The message being replied to (nested object) |
| `reactions` | Reaction[] | Array of reactions on this message |
| `reactions[].userId` | number | ID of the user who reacted |
| `reactions[].emoji` | string | The emoji used for the reaction |

### Attachment Metadata (Alternative Format)

In some contexts, attachments may be returned as an `attachmentMeta` array instead of separate arrays:

```json
{
  "attachmentMeta": [
    {
      "isSensitive": false,
      "isR18": false,
      "alt": "Photo description",
      "type": "image/jpeg",
      "url": "/uploads/dm/abc123.jpg"
    }
  ]
}
```

---

## オブジェクトリファレンス: DMGroup

The complete DMGroup object returned by group endpoints.

```json
{
  "id": 487,
  "name": null,
  "avatarUrl": null,
  "isGroup": false,
  "createdAt": "2026-03-25T10:00:00.000Z",
  "updatedAt": "2026-03-28T15:30:00.000Z",
  "members": [
    {
      "id": 2168,
      "userId": 15459,
      "isAdmin": false,
      "joinedAt": "2026-03-25T10:00:00.000Z",
      "lastRead": "2026-03-28T15:30:00.000Z",
      "messageRequestStatus": "ACCEPTED",
      "user": {
        "id": 15459,
        "username": "claude",
        "displayName": "claude",
        "avatarUrl": "/uploads/avatars/abc.webp",
        "avatarFrameId": null,
        "onlineStatus": "ONLINE",
        "statusMessage": "Working on bots",
        "onlineStatusVisibility": "PUBLIC"
      }
    }
  ],
  "messages": [],
  "lastMessage": null,
  "unreadCount": 0,
  "activeCall": null,
  "isCurrentUserAdmin": false,
  "messageRequestStatus": "ACCEPTED",
  "isMessageRequest": false,
  "canSend": true,
  "sendDisabledReason": null
}
```

### DMGroupフィールドリファレンス

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `id` | number | Unique group ID |
| `name` | string \| null | Group name (null for 1-on-1 DMs) |
| `avatarUrl` | string \| null | Group avatar URL |
| `isGroup` | boolean | `true` for group DMs, `false` for 1-on-1 |
| `createdAt` | string | ISO 8601 creation timestamp |
| `updatedAt` | string | ISO 8601 last update timestamp |
| `members` | Member[] | Array of group members |
| `members[].id` | number | Membership ID (not user ID) |
| `members[].userId` | number | User ID of the member |
| `members[].isAdmin` | boolean | Whether this member is a group admin |
| `members[].joinedAt` | string | ISO 8601 timestamp of when they joined |
| `members[].lastRead` | string | ISO 8601 timestamp of their last read message |
| `members[].messageRequestStatus` | string | `"PENDING"`, `"ACCEPTED"`, or `"REJECTED"` |
| `members[].user` | object | User profile data |
| `messages` | DmMessage[] | Latest message(s) in the group |
| `lastMessage` | DmMessage \| null | Most recent message |
| `unreadCount` | number | Number of unread messages for the authenticated user |
| `activeCall` | object \| null | Active call info if a call is in progress |
| `activeCall.participantCount` | number | Number of participants in the call |
| `activeCall.participants` | object[] | Array of call participants |
| `isCurrentUserAdmin` | boolean | Whether the authenticated user is a group admin |
| `messageRequestStatus` | string | Your message request status in this group |
| `isMessageRequest` | boolean | Whether this is an unaccepted message request |
| `canSend` | boolean | Whether you can send messages in this group |
| `sendDisabledReason` | string \| null | Reason why sending is disabled, if applicable |

### メッセージリクエストステータス値

| 値 | 説明 |
|-------|-------------|
| `PENDING` | Request sent but not yet reviewed |
| `ACCEPTED` | Request accepted, normal messaging |
| `REJECTED` | Request rejected |
| `null` | Not applicable (e.g., mutual followers) |

---

## レート制限

すべてのDMエンドポイントはデフォルトのレート制限を共有しています:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
