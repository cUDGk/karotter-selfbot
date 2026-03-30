# Radio API (Spaces / Live Audio)

> **Base URL:** `https://karotter.com/api`
>
> All endpoints require authentication via `Authorization: Bearer {token}` header and `X-CSRF-Token` header.
> Cookie-based authentication (`karotter_at`, `karotter_rt`, `karotter_csrf`) may be required for some operations.

---

## Table of Contents

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

## Overview

Radio (Spaces) is Karotter's live audio feature, similar to Twitter/X Spaces or Clubhouse. Users can create live audio rooms, invite speakers, and interact through real-time voice and text chat.

Audio is transmitted via WebRTC. The server provides TURN/STUN server configurations via the ICE servers endpoint.

---

## Create a Space

Creates a new live audio space. The creator automatically becomes the HOST.

```
POST /api/radio
```

### Request Body

| Field | Type | Required | Constraints | Description |
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

## Get Active Spaces

Returns all currently active (live) spaces.

```
GET /api/radio/active
```

### Query Parameters

None.

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

### Response Fields

| Field | Type | Description |
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

## Get ICE Servers

Returns WebRTC ICE (STUN/TURN) server configurations for establishing audio connections.

```
GET /api/radio/ice-servers
```

### Query Parameters

None.

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

> **Note:** TURN credentials are time-limited. Fetch fresh ICE servers before each connection attempt.

---

## Get Space Details

Returns detailed information about a specific space, including all participants.

```
GET /api/radio/:id
```

### Path Parameters

| Parameter | Type | Required | Description |
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

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"space": {...}}` | Space details |
| 404 | `{"error": "スペースが見つかりません"}` | Space not found |

---

## Join a Space

Joins an active space as a listener.

```
POST /api/radio/:id/join
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Request Body

None.

### Responses

| Status | Body | Description |
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

## Leave a Space

Leaves a space you are currently in.

```
POST /api/radio/:id/leave
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Request Body

None.

### Response

```json
{
  "message": "スペースから退出しました"
}
```

> **Note:** If the HOST leaves without ending the space, host privileges may transfer to another speaker. To properly close a space, use the [End a Space](#end-a-space) endpoint.

---

## End a Space

Ends a space permanently. Only the HOST can end a space. All participants are disconnected.

```
POST /api/radio/:id/end
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Request Body

None.

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "スペースを終了しました"}` | Space ended |
| 403 | `{"error": "権限がありません"}` | Not the host |

---

## Get Chat Messages

Returns text chat messages from a space.

```
GET /api/radio/:id/messages
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Query Parameters

| Parameter | Type | Default | Description |
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

## Send Chat Message

Sends a text chat message in a space. Available to all participants (hosts, speakers, and listeners).

```
POST /api/radio/:id/messages
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Request Body

| Field | Type | Required | Description |
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

## Update Space Settings

Updates the settings of a space. Only the HOST can modify settings.

```
PATCH /api/radio/:id/settings
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Request Body

| Field | Type | Required | Description |
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

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "設定を更新しました", "space": {...}}` | Settings updated |
| 403 | `{"error": "権限がありません"}` | Not the host |

---

## Change Participant Role

Changes a participant's role in the space. Only the HOST can change roles.

```
PATCH /api/radio/:id/participants/:userId/role
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |
| `userId` | number | Yes | Target participant's user ID |

### Request Body

| Field | Type | Required | Description |
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

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "ロールを変更しました"}` | Role changed |
| 403 | `{"error": "権限がありません"}` | Not the host |
| 404 | `{"error": "参加者が見つかりません"}` | User is not in the space |

> **Note:** Promoting someone to HOST transfers host privileges. The original host becomes a SPEAKER.

---

## Mute a Participant

Mutes or unmutes a participant. The HOST can mute anyone. Participants can also self-mute.

```
PATCH /api/radio/:id/participants/:userId/mute
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |
| `userId` | number | Yes | Target participant's user ID |

### Request Body

| Field | Type | Required | Description |
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

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "ミュートを変更しました"}` | Mute state changed |
| 403 | `{"error": "権限がありません"}` | Not the host (and not self-muting) |

---

## Invite to Speak

Sends a speaker invitation to a listener. Only the HOST can invite speakers.

```
POST /api/radio/:id/participants/:userId/invite-speaker
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |
| `userId` | number | Yes | Target listener's user ID |

### Request Body

None.

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を送信しました"}` | Invitation sent |
| 400 | `{"error": "既にスピーカーです"}` | User is already a speaker |
| 403 | `{"error": "権限がありません"}` | Not the host |

---

## Cancel Speaker Invite

Cancels a previously sent speaker invitation.

```
DELETE /api/radio/:id/participants/:userId/invite-speaker
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |
| `userId` | number | Yes | Target user ID |

### Request Body

None.

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を取り消しました"}` | Invitation cancelled |
| 403 | `{"error": "権限がありません"}` | Not the host |

---

## Request to Speak

Sends a request to the host asking to be promoted to speaker. Available to listeners.

```
POST /api/radio/:id/request-speaker
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Request Body

None.

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "スピーカーリクエストを送信しました"}` | Request sent |
| 400 | `{"error": "既にスピーカーです"}` | Already a speaker |

---

## Accept Speaker Invite

Accepts a speaker invitation that was sent to you by the host.

```
POST /api/radio/:id/accept-speaker-invite
```

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | number | Yes | Space ID |

### Request Body

None.

### Responses

| Status | Body | Description |
|--------|------|-------------|
| 200 | `{"message": "スピーカー招待を承諾しました"}` | Accepted, you are now a SPEAKER |
| 400 | `{"error": "招待が見つかりません"}` | No pending speaker invite |

---

## Space Modes and Policies

### Access Modes (`mode`)

| Mode | Description |
|------|-------------|
| `public` | Anyone can join and listen |
| `followers` | Only people who follow the host can join |
| `invite` | Only explicitly invited users can join |

### Speaker Policies (`speakerPolicy`)

| Policy | Description |
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

## Participant Roles

| Role | Permissions |
|------|------------|
| `HOST` | Full control: end space, change settings, manage roles, mute anyone, invite speakers |
| `SPEAKER` | Can speak (audio), send chat messages, self-mute |
| `LISTENER` | Can listen to audio, send chat messages, request to speak |

### Role Hierarchy

```
HOST > SPEAKER > LISTENER
```

- Only one HOST at a time. Promoting another user to HOST demotes the current host to SPEAKER.
- SPEAKER can be demoted to LISTENER by the HOST.
- LISTENER can be promoted to SPEAKER by the HOST or via speaker request/invite flow.

---

## Rate Limiting

All radio endpoints share the default rate limit:

```
ratelimit-policy: 100;w=60
ratelimit-limit: 100
ratelimit-remaining: 99
ratelimit-reset: 58
```
