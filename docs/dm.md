# Direct Messages (DM) API

> **Base URL:** `https://karotter.com/api`
>
> All endpoints require authentication via `Authorization: Bearer {token}` header and `X-CSRF-Token` header.
> Cookie-based authentication (`karotter_at`, `karotter_rt`, `karotter_csrf`) may be required for some operations.

---

## Table of Contents

- [Age Gate](#age-gate)
- [Start a DM](#start-a-dm)
- [List DM Groups](#list-dm-groups)
- [Create a Group DM](#create-a-group-dm)
- [Get Messages](#get-messages)
- [Send a Message](#send-a-message)
- [Mark as Read](#mark-as-read)
- [Clear DM History](#clear-dm-history)
- [Leave Group](#leave-group)
- [Add Group Member](#add-group-member)
- [Remove Group Member](#remove-group-member)
- [Accept Message Request](#accept-message-request)
- [Reject Message Request](#reject-message-request)
- [Delete a Message](#delete-a-message)
- [Edit a Message](#edit-a-message)
- [React to a Message](#react-to-a-message)
- [Vote on a Poll](#vote-on-a-poll)
- [Calls](#calls)
- [Object Reference: DmMessage](#object-reference-dmmessage)
- [Object Reference: DMGroup](#object-reference-dmgroup)

---

## Age Gate

DM functionality is restricted to users aged **13 or older**. Accounts registered with a birthday indicating they are under 13 cannot access any DM endpoint. Attempting to do so returns:

```json
{
  "error": "年齢制限により利用できません"
}
```

---

## Start a DM

Starts or retrieves a 1-on-1 DM conversation with a target user. If a DM group already exists between you and the target, it returns the existing group.

```
POST /api/dm/start
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `targetUserId` | number | Yes | The numeric ID of the user to DM |

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"group": {...}}` | DM group created or retrieved |
| 400 | `{"error": "自分自身にDMを送ることはできません"}` | Attempted to DM yourself |
| 403 | `{"error": "メールアドレスの確認が必要です", "code": "EMAIL_NOT_VERIFIED"}` | Email not verified (required for DMing non-followers) |
| 403 | `{"error": "このユーザーにDMを送ることはできません"}` | Target has DMs disabled or has blocked you |

### Example

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

> **Note:** If the target user's `dmRequestPolicy` is `"FOLLOWERS"` and you don't follow them, the DM will be created as a **message request**. The recipient must accept it before they see your messages.

---

## List DM Groups

Returns a paginated list of all your DM conversations (both 1-on-1 and group DMs).

```
GET /api/dm/groups
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number (1-indexed) |
| `limit` | number | 30 | Number of groups per page (max 30) |

### Response

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

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `groups` | DMGroup[] | Array of DM group objects |
| `pagination.page` | number | Current page |
| `pagination.hasNext` | boolean | Whether more pages exist |

---

## Create a Group DM

Creates a new group DM with multiple users.

```
POST /api/dm/groups
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `userIds` | number[] | Yes | Array of user IDs to include (minimum 2) |
| `name` | string | Yes | Group name |
| `isGroup` | boolean | Yes | Must be `true` |

### Example

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

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"group": {...}}` | Group created |
| 400 | `{"error": "..."}` | Validation error (e.g., too few users) |
| 403 | `{"error": "メールアドレスの確認が必要です", "code": "EMAIL_NOT_VERIFIED"}` | Email not verified |

---

## Get Messages

Returns a paginated list of messages in a DM group, ordered newest-first.

```
GET /api/dm/groups/:id/messages
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number (1-indexed) |
| `limit` | number | 50 | Messages per page (max 50) |

### Response

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

### Pagination Fields

| Field | Type | Description |
|-------|------|-------------|
| `pagination.page` | number | Current page number |
| `pagination.limit` | number | Items per page |
| `pagination.total` | number | Total number of messages in the group |
| `pagination.pages` | number | Total number of pages |

---

## Send a Message

Sends a message to a DM group. Uses `multipart/form-data` encoding.

```
POST /api/dm/groups/:id/messages
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### Request Body (FormData)

| Field | Type | Required | Description |
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

> **CRITICAL:** The file field name is `attachments`, NOT `media`. Using `media` will result in a 500 error. This differs from the post creation endpoint which uses `media`.

### Poll Rules

| Rule | Constraint |
|------|-----------|
| Minimum options | 2 |
| Maximum options | 4 |
| Max characters per option | 80 |
| Allowed durations (hours) | 1, 6, 12, 24, 48, 72, 168 |
| Combined with attachments | **Not allowed** - sending a poll with attachments will fail |

### Example: Text Message

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

### Example: Message with Attachments

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

### Example: Message with Poll

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

### Example: Reply

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

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": {...}}` | Message sent successfully |
| 400 | `{"error": "..."}` | Validation error |
| 403 | `{"error": "メッセージを送信する権限がありません"}` | Cannot send (blocked, request not accepted, etc.) |

---

## Mark as Read

Marks all messages in a DM group as read up to the current time.

```
POST /api/dm/groups/:id/read
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### Request Body

None.

### Response

```json
{
  "ok": true,
  "lastRead": "2026-03-28T15:30:00.000Z"
}
```

---

## Clear DM History

Removes the DM group from **your** DM list. The other participant(s) still see the conversation. This is a **destructive operation** -- the conversation disappears from your view.

```
POST /api/dm/groups/:id/clear
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### Request Body

None.

### Response

```json
{
  "message": "DM履歴を自分の一覧から削除しました"
}
```

> **Warning:** This is irreversible from the API. The conversation will reappear if the other user sends a new message.

---

## Leave Group

Leaves a group DM. Only works for group DMs (`isGroup: true`).

```
POST /api/dm/groups/:id/leave
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### Request Body

None.

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "グループから退出しました"}` | Successfully left |
| 400 | `{"error": "退出できません。履歴削除を利用してください"}` | Cannot leave a 1-on-1 DM; use clear instead |

> **Note:** For 1-on-1 DMs, you cannot "leave". Use `POST /dm/groups/:id/clear` instead.

---

## Add Group Member

Adds a user to a group DM. Requires email verification.

```
POST /api/dm/groups/:id/members
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `userId` | number | Yes | User ID to add |

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "メンバーを追加しました"}` | Member added |
| 403 | `{"error": "メールアドレスの確認が必要です", "code": "EMAIL_NOT_VERIFIED"}` | Email not verified |

---

## Remove Group Member

Removes a user from a group DM. Requires admin privileges in the group.

```
DELETE /api/dm/groups/:id/members/:userId
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |
| `userId` | number | Yes | User ID to remove |

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "メンバーを削除しました"}` | Member removed |
| 403 | `{"error": "権限がありません"}` | Not a group admin |

---

## Accept Message Request

Accepts a DM message request from a user you don't follow.

```
POST /api/dm/groups/:id/request/accept
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### Request Body

None.

### Response

```json
{
  "message": "メッセージリクエストを承認しました"
}
```

After accepting, `canSend` becomes `true` and messages flow normally.

---

## Reject Message Request

Rejects a DM message request. The conversation is removed from your list.

```
POST /api/dm/groups/:id/request/reject
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | DM group ID |

### Request Body

None.

### Response

```json
{
  "message": "メッセージリクエストを拒否しました"
}
```

---

## Delete a Message

Deletes one of your own messages. You can only delete messages you sent.

```
DELETE /api/dm/messages/:id
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Message ID |

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "メッセージを削除しました"}` | Deleted |
| 403 | `{"error": "権限がありません"}` | Not your message |
| 404 | `{"error": "メッセージが見つかりません"}` | Message not found |

> **Note:** Deleted messages remain in the conversation with `isDeleted: true` and `content` cleared. They are not physically removed.

---

## Edit a Message

Edits the text content of one of your own messages.

```
PATCH /api/dm/messages/:id
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Message ID |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | string | Yes | New text content |

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": {...}}` | Edited message object |
| 400 | `{"error": "暗号化メッセージは編集できません"}` | Cannot edit encrypted messages |
| 400 | `{"error": "システムメッセージは編集できません"}` | Cannot edit system messages |
| 403 | `{"error": "権限がありません"}` | Not your message |

### Restrictions

- **Encrypted messages** (`isEncrypted: true`) cannot be edited
- **System messages** (`systemType` is not null, e.g., `MEMBER_JOINED`, `MEMBER_LEFT`) cannot be edited
- The `editedAt` field is updated to the current timestamp after editing

---

## React to a Message

Adds an emoji reaction to a DM message.

```
POST /api/dm/messages/:id/reactions
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Message ID |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `emoji` | string | Yes | Emoji text (max 32 characters) |

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "リアクションしました"}` | Reaction added |
| 400 | `{"error": "既にこの絵文字でリアクションしています"}` | Already reacted with this emoji |

### Example

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

## Vote on a Poll

Votes on a poll within a DM message. This is a **toggle** -- calling it again with the same `optionId` removes your vote.

```
POST /api/dm/messages/:id/poll/vote
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Message ID containing the poll |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `optionId` | number | Yes | The poll option ID to vote for |

### Example

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

## Calls

DM groups support voice/video calls with WebRTC.

### Start a Call

```
POST /api/dm/groups/:id/call/start
```

Starts a new call in the DM group. Returns call information including ICE server configuration.

### Join a Call

```
POST /api/dm/groups/:id/call/join
```

Joins an existing active call in the DM group.

### Leave a Call

```
POST /api/dm/groups/:id/call/leave
```

Leaves the current call. If you are the last participant, the call ends automatically.

### Socket.IO Events for Calls

| Event | Direction | Payload | Description |
|-------|-----------|---------|-------------|
| `voice:offer` | emit | `{groupId, sdp}` | Send WebRTC offer |
| `voice:answer` | emit | `{groupId, sdp}` | Send WebRTC answer |
| `voice:ice-candidate` | emit | `{groupId, candidate}` | Send ICE candidate |
| `voice:hangup` | emit | `{groupId}` | Hang up |

### Socket.IO Events for DM

| Event | Direction | Payload | Description |
|-------|-----------|---------|-------------|
| `joinDM` | emit | `{groupId}` | Join DM room for real-time updates |
| `leaveDM` | emit | `{groupId}` | Leave DM room |
| `markAsRead` | emit | `{groupId, messageId}` | Mark message as read |
| `startTyping` | emit | `{groupId}` | Start typing indicator |
| `stopTyping` | emit | `{groupId}` | Stop typing indicator |

---

## Object Reference: DmMessage

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

### DmMessage Field Reference

| Field | Type | Description |
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

## Object Reference: DMGroup

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

### DMGroup Field Reference

| Field | Type | Description |
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

### Message Request Status Values

| Value | Description |
|-------|-------------|
| `PENDING` | Request sent but not yet reviewed |
| `ACCEPTED` | Request accepted, normal messaging |
| `REJECTED` | Request rejected |
| `null` | Not applicable (e.g., mutual followers) |

---

## Rate Limiting

All DM endpoints share the default rate limit:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
