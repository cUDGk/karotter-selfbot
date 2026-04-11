# Socket.IO Real-Time Events

Complete reference for all Socket.IO events used in the Karotter platform, including DM messaging, voice calls, radio/spaces, draw-chat, and system events.

---

## Connection

### URL

The Socket.IO server is the same origin as the REST API:

```
https://api.karotter.com
```

There is no custom `path` option -- the default `/socket.io` path is used.

### Connection Configuration

```js
import { io } from "socket.io-client";

const socket = io("https://api.karotter.com", {
  auth: {
    token: accessToken,          // JWT access token (without "Bearer " prefix)
  },
  withCredentials: true,         // send cookies cross-origin
  transports: ["websocket"],     // skip HTTP long-polling, connect via WebSocket only
  reconnection: true,            // enable automatic reconnection
  reconnectionAttempts: Infinity,// never stop trying
  reconnectionDelay: 1000,       // initial delay between attempts (ms)
  reconnectionDelayMax: 8000,    // maximum delay between attempts (ms)
  timeout: 20000,                // connection timeout (ms)
});
```

### Automatic Token Refresh on Auth Failure

When a `connect_error` event fires, the client checks whether the error message matches an authentication-related pattern. If so, it refreshes the access token and reconnects:

```js
socket.on("connect_error", async (error) => {
  if (/unauthorized|authentication|jwt/i.test(error.message)) {
    const newToken = await refreshAccessToken(); // call POST /auth/refresh-token
    socket.auth = { token: newToken };
    socket.connect();
  }
});
```

**Pattern tested:** `/unauthorized|authentication|jwt/i`

This handles cases where the JWT has expired between reconnection attempts. The client should call `POST /api/auth/refresh-token` to obtain a fresh access token, update `socket.auth.token`, and then call `socket.connect()`.

### Connection Lifecycle Events

| Event | Direction | Description |
|-------|-----------|-------------|
| `connect` | Listen | Fired when the connection is established. `socket.id` is now available. |
| `connect_error` | Listen | Fired on connection failure. Check `error.message` for auth issues. |
| `disconnect` | Listen | Fired when disconnected. Receives a `reason` string. |
| `reconnect` | Listen | Fired after a successful reconnection. |
| `reconnect_attempt` | Listen | Fired on each reconnection attempt. Receives attempt number. |
| `reconnect_failed` | Listen | Fired when all reconnection attempts are exhausted (never, with `Infinity`). |

---

## DM (Direct Message) Events

### Events to Listen For

#### `dm:new-message`

Fired when a new message is sent in any DM group the user is a member of.

```ts
socket.on("dm:new-message", (data: {
  groupId: string;
  message: {
    id: number;
    groupId: string;
    senderId: number;
    replyToId: number | null;
    content: string;
    encryptedContent: string | null;
    isEncrypted: boolean;
    systemType: string | null;       // e.g. "MEMBER_JOINED", "MEMBER_LEFT"
    systemMeta: object | null;
    systemActorId: number | null;
    attachmentUrls: string[];
    attachmentTypes: string[];
    attachmentAlts: string[];
    attachmentSpoilerFlags: boolean[];
    attachmentR18Flags: boolean[];
    isDeleted: boolean;
    editedAt: string | null;
    createdAt: string;               // ISO 8601
    updatedAt: string;               // ISO 8601
    sender: {
      id: number;
      username: string;
      displayName: string;
      avatarUrl: string | null;
      officialMark: string[];
    };
    replyTo: object | null;          // nested message object
    reactions: Array<{
      emoji: string;
      userId: number;
    }>;
  };
}) => { /* handle */ });
```

#### `dm:message-updated`

Fired when a message is edited in a DM group.

```ts
socket.on("dm:message-updated", (data: {
  groupId: string;
  message: DmMessage;               // same structure as dm:new-message
}) => { /* handle */ });
```

#### `dm:message-deleted`

Fired when a message is deleted in a DM group.

```ts
socket.on("dm:message-deleted", (data: {
  groupId: string;
  messageId: number;
}) => { /* handle */ });
```

#### `dm:read`

Fired when a member reads messages in the group. Used to update read receipt indicators.

```ts
socket.on("dm:read", (data: {
  groupId: string;
  userId: number;
  messageId: number;                 // the latest message the user has read up to
}) => { /* handle */ });
```

#### `dm:member-added`

Fired when a new member is added to a group DM.

```ts
socket.on("dm:member-added", (data: {
  groupId: string;
  userId: number;
  username: string;
}) => { /* handle */ });
```

#### `dm:member-left`

Fired when a member voluntarily leaves a group DM.

```ts
socket.on("dm:member-left", (data: {
  groupId: string;
  userId: number;
}) => { /* handle */ });
```

#### `dm:member-removed`

Fired when a member is removed (kicked) from a group DM by the group owner.

```ts
socket.on("dm:member-removed", (data: {
  groupId: string;
  userId: number;
}) => { /* handle */ });
```

#### `dm:request-updated`

Fired when a DM request status changes (accepted or declined).

```ts
socket.on("dm:request-updated", (data: {
  groupId: string;
  status: string;                    // "accepted" | "declined"
}) => { /* handle */ });
```

#### `dm:join`

Fired when the current user is added to a new DM group (either by creating one or being invited).

```ts
socket.on("dm:join", (data: {
  groupId: string;
}) => { /* handle */ });
```

#### `dm:leave`

Fired when the current user is removed from a DM group.

```ts
socket.on("dm:leave", (data: {
  groupId: string;
}) => { /* handle */ });
```

### Typing Indicator Events

#### `typing:user` (Listen)

Fired when another user starts typing in a DM group.

```ts
socket.on("typing:user", (data: {
  groupId: string;
  userId: number;
  username: string;
}) => { /* handle */ });
```

#### `typing:stop` (Listen)

Fired when a user stops typing in a DM group.

```ts
socket.on("typing:stop", (data: {
  groupId: string;
  userId: number;
}) => { /* handle */ });
```

### Events to Emit

#### `typing:start`

Notify the server that you have started typing.

```ts
socket.emit("typing:start", {
  groupId: string;
});
```

**Behavior:** The server broadcasts `typing:user` to all other members in the group. The server automatically emits `typing:stop` after a timeout if no further `typing:start` events are received.

#### `typing:stop`

Notify the server that you have stopped typing.

```ts
socket.emit("typing:stop", {
  groupId: string;
});
```

#### `joinDM`

Join the Socket.IO room for a DM group. This must be emitted when opening a DM conversation to start receiving real-time events for that group.

```ts
socket.emit("joinDM", {
  groupId: string;
});
```

#### `leaveDM`

Leave the Socket.IO room for a DM group. Emit this when navigating away from the conversation.

```ts
socket.emit("leaveDM", {
  groupId: string;
});
```

#### `markAsRead`

Mark messages in a DM group as read up to a specific message ID. Triggers `dm:read` events for other members.

```ts
socket.emit("markAsRead", {
  groupId: string;
  messageId: number;
});
```

---

## Call Events (Voice/Video)

Voice and video calls use WebRTC with Socket.IO as the signaling layer.

### Events to Listen For

#### `call:incoming`

Fired when an incoming call is received.

```ts
socket.on("call:incoming", (data: {
  groupId: string;
  callerId: number;
  callerUsername: string;
  callType: string;                  // "voice" | "video"
}) => { /* handle */ });
```

#### `call:state`

Fired when the call state changes (e.g., someone joins, leaves, or the call ends).

```ts
socket.on("call:state", (data: {
  groupId: string;
  state: string;                     // "ringing" | "active" | "ended"
  participants: Array<{
    userId: number;
    username: string;
    isMuted: boolean;
    isVideoEnabled: boolean;
  }>;
}) => { /* handle */ });
```

### WebRTC Signaling Events (Emit and Listen)

These events are both emitted and listened to. The client sends its own offers/answers/candidates and receives them from other participants.

#### `voice:offer`

Send or receive a WebRTC SDP offer.

```ts
// Emit
socket.emit("voice:offer", {
  groupId: string;
  targetUserId: number;
  offer: RTCSessionDescriptionInit;  // { type: "offer", sdp: "..." }
});

// Listen
socket.on("voice:offer", (data: {
  groupId: string;
  fromUserId: number;
  offer: RTCSessionDescriptionInit;
}) => { /* handle */ });
```

#### `voice:answer`

Send or receive a WebRTC SDP answer.

```ts
// Emit
socket.emit("voice:answer", {
  groupId: string;
  targetUserId: number;
  answer: RTCSessionDescriptionInit; // { type: "answer", sdp: "..." }
});

// Listen
socket.on("voice:answer", (data: {
  groupId: string;
  fromUserId: number;
  answer: RTCSessionDescriptionInit;
}) => { /* handle */ });
```

#### `voice:ice-candidate`

Exchange ICE candidates for NAT traversal.

```ts
// Emit
socket.emit("voice:ice-candidate", {
  groupId: string;
  targetUserId: number;
  candidate: RTCIceCandidateInit;
});

// Listen
socket.on("voice:ice-candidate", (data: {
  groupId: string;
  fromUserId: number;
  candidate: RTCIceCandidateInit;
}) => { /* handle */ });
```

#### `voice:hangup`

Signal that the user is hanging up or receive a hangup signal.

```ts
// Emit
socket.emit("voice:hangup", {
  groupId: string;
});

// Listen
socket.on("voice:hangup", (data: {
  groupId: string;
  userId: number;
}) => { /* handle */ });
```

#### `voice:participant-state`

Update or receive participant media state changes (mute/unmute, video on/off).

```ts
// Emit
socket.emit("voice:participant-state", {
  groupId: string;
  isMuted: boolean;
  isVideoEnabled: boolean;
});

// Listen
socket.on("voice:participant-state", (data: {
  groupId: string;
  userId: number;
  isMuted: boolean;
  isVideoEnabled: boolean;
}) => { /* handle */ });
```

---

## Radio Events (Spaces / Live Audio)

Radio is the Karotter equivalent of Twitter Spaces -- live audio rooms with a host, speakers, and listeners.

### Events to Listen For

#### `radio:join`

Confirmation that you have successfully joined a radio room.

```ts
socket.on("radio:join", (data: {
  roomId: string;
  participants: Array<{
    userId: number;
    username: string;
    role: "HOST" | "SPEAKER" | "LISTENER";
    isMuted: boolean;
  }>;
}) => { /* handle */ });
```

#### `radio:leave`

Confirmation that you have left the radio room.

```ts
socket.on("radio:leave", (data: {
  roomId: string;
}) => { /* handle */ });
```

#### `radio:ended`

Fired when the radio room is closed by the host.

```ts
socket.on("radio:ended", (data: {
  roomId: string;
}) => { /* handle */ });
```

#### `radio:user-joined`

Fired when another user joins the radio room.

```ts
socket.on("radio:user-joined", (data: {
  roomId: string;
  userId: number;
  username: string;
  displayName: string;
  avatarUrl: string | null;
  role: "HOST" | "SPEAKER" | "LISTENER";
}) => { /* handle */ });
```

#### `radio:user-left`

Fired when another user leaves the radio room.

```ts
socket.on("radio:user-left", (data: {
  roomId: string;
  userId: number;
}) => { /* handle */ });
```

#### `radio:host-disconnected`

Fired when the host's connection drops. The room stays alive for a grace period.

```ts
socket.on("radio:host-disconnected", (data: {
  roomId: string;
  gracePeriodMs: number;             // how long before the room auto-closes
}) => { /* handle */ });
```

#### `radio:host-reconnected`

Fired when the host reconnects within the grace period.

```ts
socket.on("radio:host-reconnected", (data: {
  roomId: string;
}) => { /* handle */ });
```

#### `radio:message`

Fired when a chat message is sent in the radio room.

```ts
socket.on("radio:message", (data: {
  roomId: string;
  message: {
    id: number;
    content: string;
    senderId: number;
    sender: {
      id: number;
      username: string;
      displayName: string;
      avatarUrl: string | null;
    };
    createdAt: string;
  };
}) => { /* handle */ });
```

### Events to Emit and Listen (Bidirectional)

#### `radio:signal`

WebRTC signaling for radio audio streams. Used for SDP offers, answers, and ICE candidates between participants.

```ts
// Emit
socket.emit("radio:signal", {
  roomId: string;
  targetUserId: number;
  signal: object;                    // RTCSessionDescriptionInit or RTCIceCandidateInit
});

// Listen
socket.on("radio:signal", (data: {
  roomId: string;
  fromUserId: number;
  signal: object;
}) => { /* handle */ });
```

#### `radio:participant-state`

Update or receive participant state changes in the radio room (mute/unmute, role changes).

```ts
// Emit
socket.emit("radio:participant-state", {
  roomId: string;
  isMuted: boolean;
});

// Listen
socket.on("radio:participant-state", (data: {
  roomId: string;
  userId: number;
  isMuted: boolean;
  role: "HOST" | "SPEAKER" | "LISTENER";
}) => { /* handle */ });
```

#### `radio:renegotiate-request`

Request or receive a WebRTC renegotiation (e.g., when a new speaker joins and streams need to be renegotiated).

```ts
// Emit
socket.emit("radio:renegotiate-request", {
  roomId: string;
  targetUserId: number;
});

// Listen
socket.on("radio:renegotiate-request", (data: {
  roomId: string;
  fromUserId: number;
}) => { /* handle */ });
```

### Emit and Listen

#### `radio:reaction`

Send a floating reaction emoji in the radio room (visible to all participants).

```ts
// Emit
socket.emit("radio:reaction", {
  roomId: string;
  emoji: string;                     // e.g. "fire", "heart", "clap", "100"
});

// Listen
socket.on("radio:reaction", (data: {
  roomId: string;
  userId: number;
  emoji: string;
}) => { /* handle */ });
```

---

## Draw Events (Draw-Chat / Collaborative Canvas)

Draw-chat provides real-time collaborative drawing with layer support, cursors, and in-room chat.

### Canvas Specifications

| Property | Value |
|----------|-------|
| Canvas size | 2560 x 2560 pixels |
| Format | RGBA PNG |
| Max payload | ~5 MB (all layers combined) |

### Events to Emit

#### `draw:join`

Join a draw-chat room. The server responds with `draw:room-state`.

```ts
socket.emit("draw:join", {
  roomId: string;
});
```

#### `draw:leave`

Leave a draw-chat room.

```ts
socket.emit("draw:leave", {
  roomId: string;
});
```

#### `draw:stroke`

Send a drawing stroke to all other participants.

```ts
socket.emit("draw:stroke", {
  roomId: string;
  layerId: string;
  tool: string;                      // "pen" | "eraser" | "bucket" | etc.
  color: string;                     // hex color, e.g. "#ff0000"
  width: number;                     // brush width in pixels
  opacity: number;                   // 0.0 - 1.0
  points: Array<{
    x: number;
    y: number;
    pressure: number;                // 0.0 - 1.0 (for pressure-sensitive input)
  }>;
  compositeOperation: string;        // e.g. "source-over", "destination-out"
});
```

#### `draw:cursor`

Broadcast your cursor position in real time.

```ts
socket.emit("draw:cursor", {
  roomId: string;
  x: number;                         // 0 - 2560
  y: number;                         // 0 - 2560
  drawing: boolean;                  // true if actively drawing
});
```

#### `draw:layer-sync`

Synchronize layer data with all participants. Used when adding, reordering, or modifying layers.

```ts
socket.emit("draw:layer-sync", {
  roomId: string;
  layers: Array<{
    id: string;
    name: string;
    order: number;
    visible: boolean;
    opacity: number;                 // 0.0 - 1.0
    dataUrl: string;                 // "data:image/png;base64,..." (full layer image)
  }>;
  revision: number;                  // monotonically increasing revision number
  clientId: string;                  // unique client identifier for deduplication
  broadcast: boolean;                // whether to broadcast to other clients
});
```

**Important:** When syncing layers, you must include ALL layers in the array, not just the modified one. Omitted layers will be deleted on other clients.

#### `draw:chat`

Send a chat message within the draw room.

```ts
socket.emit("draw:chat", {
  roomId: string;
  message: string;
});
```

### Events to Listen For

#### `draw:room-state`

Received after emitting `draw:join`. Contains the full state of the room.

```ts
socket.on("draw:room-state", (data: {
  roomId: string;
  room: {
    id: string;
    name: string;
    ownerId: number;
    ownerUsername: string;
    maxParticipants: number;
    participants: Array<{
      userId: number;
      username: string;
      displayName: string;
      avatarUrl: string | null;
    }>;
    layers: Array<{
      id: string;
      name: string;
      order: number;
      visible: boolean;
      opacity: number;
      dataUrl: string;
    }>;
    createdAt: string;
  };
}) => { /* handle */ });
```

#### `draw:stroke` (Listen)

Receive a stroke drawn by another participant.

```ts
socket.on("draw:stroke", (data: {
  roomId: string;
  userId: number;
  layerId: string;
  tool: string;
  color: string;
  width: number;
  opacity: number;
  points: Array<{ x: number; y: number; pressure: number }>;
  compositeOperation: string;
}) => { /* handle */ });
```

#### `draw:cursor` (Listen)

Receive cursor position updates from other participants.

```ts
socket.on("draw:cursor", (data: {
  roomId: string;
  userId: number;
  username: string;
  x: number;
  y: number;
  drawing: boolean;
}) => { /* handle */ });
```

#### `draw:layer-sync` (Listen)

Receive layer synchronization data from another participant. Deduplicate by checking `clientId` + `revision`.

```ts
socket.on("draw:layer-sync", (data: {
  roomId: string;
  userId: number;
  layers: Array<{
    id: string;
    name: string;
    order: number;
    visible: boolean;
    opacity: number;
    dataUrl: string;
  }>;
  revision: number;
  clientId: string;
}) => { /* handle */ });
```

**Deduplication:** If `clientId` matches your own client ID, ignore the event (it is your own sync echoed back). Additionally, only apply syncs where `revision` is greater than your last-seen revision for that `clientId`.

#### `draw:user-left`

Fired when a participant leaves the draw room.

```ts
socket.on("draw:user-left", (data: {
  userId: number;
}) => { /* handle */ });
```

#### `draw:chat` (Listen)

Receive a chat message from within the draw room.

```ts
socket.on("draw:chat", (data: {
  roomId: string;
  userId: number;
  username: string;
  message: string;
  createdAt: string;                 // ISO 8601
}) => { /* handle */ });
```

#### `draw:error`

Fired when an error occurs in the draw room (e.g., room not found, permission denied).

```ts
socket.on("draw:error", (data: {
  message: string;
}) => { /* handle */ });
```

---

## Screen Share Events

### `screen-share:view` (Emit)

Notify the server that you are viewing a screen share.

```ts
socket.emit("screen-share:view", {
  groupId: string;
  targetUserId: number;
});
```

---

## User Status Events

### `user:status` (Listen)

Fired when a user's online status changes. Only received for users you follow or are in a DM group with.

```ts
socket.on("user:status", (data: {
  userId: number;
  status: "ONLINE" | "OFFLINE" | "DND";
  statusMessage: string | null;
}) => { /* handle */ });
```

---

## Notification Events

### `notification` (Listen)

Fired when a new notification is created for the current user (like, reply, follow, mention, etc.).

```ts
socket.on("notification", (data: {
  id: number;
  type: "FOLLOW" | "FOLLOW_REQUEST" | "LIKE" | "REPLY" | "MENTION"
      | "REKAROT" | "REACTION" | "DM" | "QUOTE" | "REPORT_UPDATE" | "SYSTEM";
  createdAt: string;
  isRead: boolean;
  message: string | null;
  actor: {
    id: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
    officialMark: string[];
  } | null;
  post: object | null;
  postId: number | null;
}) => { /* handle */ });
```

---

## Event Summary Table

| Event Name | Direction | Category | Description |
|------------|-----------|----------|-------------|
| `dm:new-message` | Listen | DM | New message in a group |
| `dm:message-updated` | Listen | DM | Message edited |
| `dm:message-deleted` | Listen | DM | Message deleted |
| `dm:read` | Listen | DM | Read receipt |
| `dm:member-added` | Listen | DM | Member added to group |
| `dm:member-left` | Listen | DM | Member left group |
| `dm:member-removed` | Listen | DM | Member kicked from group |
| `dm:request-updated` | Listen | DM | DM request accepted/declined |
| `dm:join` | Listen | DM | You were added to a group |
| `dm:leave` | Listen | DM | You were removed from a group |
| `typing:user` | Listen | DM | Someone started typing |
| `typing:stop` | Listen / Emit | DM | Someone stopped typing / you stop typing |
| `typing:start` | Emit | DM | You started typing |
| `joinDM` | Emit | DM | Join group room for events |
| `leaveDM` | Emit | DM | Leave group room |
| `markAsRead` | Emit | DM | Mark messages as read |
| `call:incoming` | Listen | Call | Incoming call |
| `call:state` | Listen | Call | Call state change |
| `voice:offer` | Emit / Listen | Call | WebRTC SDP offer |
| `voice:answer` | Emit / Listen | Call | WebRTC SDP answer |
| `voice:ice-candidate` | Emit / Listen | Call | WebRTC ICE candidate |
| `voice:hangup` | Emit / Listen | Call | Hang up call |
| `voice:participant-state` | Emit / Listen | Call | Participant media state |
| `radio:join` | Listen | Radio | Joined radio room |
| `radio:leave` | Listen | Radio | Left radio room |
| `radio:ended` | Listen | Radio | Radio room closed |
| `radio:user-joined` | Listen | Radio | User joined room |
| `radio:user-left` | Listen | Radio | User left room |
| `radio:host-disconnected` | Listen | Radio | Host connection lost |
| `radio:host-reconnected` | Listen | Radio | Host reconnected |
| `radio:signal` | Emit / Listen | Radio | WebRTC signaling |
| `radio:participant-state` | Emit / Listen | Radio | Participant state |
| `radio:renegotiate-request` | Emit / Listen | Radio | WebRTC renegotiation |
| `radio:reaction` | Emit / Listen | Radio | Floating emoji reaction |
| `radio:message` | Listen | Radio | Chat message in room |
| `draw:join` | Emit | Draw | Join draw room |
| `draw:leave` | Emit | Draw | Leave draw room |
| `draw:stroke` | Emit / Listen | Draw | Drawing stroke |
| `draw:cursor` | Emit / Listen | Draw | Cursor position |
| `draw:layer-sync` | Emit / Listen | Draw | Layer synchronization |
| `draw:chat` | Emit / Listen | Draw | In-room chat message |
| `draw:room-state` | Listen | Draw | Full room state on join |
| `draw:user-left` | Listen | Draw | User left draw room |
| `draw:error` | Listen | Draw | Error in draw room |
| `screen-share:view` | Emit | Screen | View a screen share |
| `user:status` | Listen | System | User online status change |
| `notification` | Listen | System | New notification |
