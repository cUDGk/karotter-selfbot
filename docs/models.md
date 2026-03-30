# Data Models

Complete reference for all data model definitions used in the Karotter API. Every field, its type, and its purpose are documented.

---

## Post

Represents a single post ("karot") on the platform.

```ts
interface Post {
  id: number;                              // Unique post identifier
  content: string;                         // Post text content (may contain mentions, hashtags)
  createdAt: string;                       // ISO 8601 timestamp of creation
  parentId: number | null;                 // ID of parent post if this is a reply, null otherwise

  // Author information (embedded subset of User)
  author: {
    id: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
    isPrivate: boolean;
    officialMark: string[];                // Array of mark color codes, e.g. ["BLUE"], or empty
    isParodyAccount: boolean;
    adminForceParody: boolean;             // Admin-forced parody label
    isBotAccount: boolean;
    adminForceBot: boolean;                // Admin-forced bot label
  };

  // Media attachments
  mediaUrls: string[];                     // Relative paths, e.g. ["/uploads/posts/uuid.jpg"]
  mediaAlts: string[];                     // Alt text for each media item
  mediaWidths: number[];                   // Width in pixels for each media item
  mediaHeights: number[];                  // Height in pixels for each media item
  mediaSpoilerFlags: boolean[];            // Whether each media is marked as spoiler
  mediaR18Flags: boolean[];                // Whether each media is marked as R18/NSFW

  // Engagement counts
  likesCount: number;
  rekarotsCount: number;                   // Repost count
  repliesCount: number;
  bookmarksCount: number;
  quoteUsersCount: number;                 // Number of users who quoted this post
  viewsCount: number;

  // Viewer's interaction state
  liked: boolean;                          // Whether the authenticated user liked this post
  rekaroted: boolean;                      // Whether the authenticated user reposted this
  bookmarked: boolean;                     // Whether the authenticated user bookmarked this

  // Interaction permissions
  canInteract: boolean;                    // Whether the viewer can like/react/reply
  canQuote: boolean;                       // Whether the viewer can quote this post

  // Reactions (emoji reactions beyond simple likes)
  reactionSummary: Array<{
    emoji: string;                         // The emoji text (up to 32 characters)
    count: number;                         // Total reaction count for this emoji
    reacted: boolean;                      // Whether the authenticated user reacted with this emoji
  }>;

  // Reply context
  replyToUsers: Array<{
    id: number;
    username: string;
  }>;

  // Reply restriction settings
  replyRestriction: "EVERYONE"             // Anyone can reply
                  | "FOLLOWING"            // Only users the author follows
                  | "MENTIONED"            // Only users mentioned in the post
                  | "CIRCLE";              // Only members of the specified circle
  replyCircleId: number | null;            // Circle ID if replyRestriction is "CIRCLE"

  // Visibility settings
  visibility: "PUBLIC"                     // Visible to everyone
            | "CIRCLE";                    // Visible only to circle members
  viewerCircle: {                          // Present if visibility is "CIRCLE" and viewer is a member
    id: number;
    name: string;
  } | null;

  // Quoted post (embedded, recursive)
  quotedPost: Post | null;                 // The post being quoted, if any (same Post structure)

  // Poll data
  poll: {
    options: Array<{
      id: number;                          // Option identifier
      text: string;                        // Option text
      percentage: number;                  // Vote percentage (0-100)
      votedByMe: boolean;                  // Whether the authenticated user voted for this option
    }>;
    totalVotes: number;
    isExpired: boolean;
    expiresAt: string;                     // ISO 8601 timestamp
  } | null;

  // Moderation / block state
  isMutedByViewer: boolean;                // Whether the viewer has muted this post's author
  hasBlockedAuthor: boolean;               // Whether the viewer has blocked the author
  isBlockedByAuthor: boolean;              // Whether the author has blocked the viewer

  // Miscellaneous
  itemId: number | null;                   // Associated item ID (for shop/marketplace integration)
  type: string;                            // Post type identifier
}
```

### Media URL Resolution

Media URLs in `mediaUrls` are relative paths. To construct the full URL:

```
Full URL = "https://karotter.com" + mediaUrl
Example:  "https://karotter.com/uploads/posts/abc123.jpg"
```

### Reaction Limit

The `emoji` field in reactions accepts any string up to **32 characters**. This is enforced server-side and cannot be bypassed.

---

## User (Full Profile)

The complete user object returned by `GET /api/auth/me` and profile endpoints. Note that `email` is only returned by `/api/auth/me`.

```ts
interface User {
  id: number;                              // Unique user identifier
  username: string;                        // Unique username (1-15 chars, /^[a-zA-Z0-9_]{1,15}$/)
  displayName: string;                     // Display name (may contain any characters)
  email: string;                           // Email address (only from /auth/me)
  avatarUrl: string | null;                // Avatar image URL, or null for default
  avatarFrameId: string | null;            // ID of equipped avatar frame decoration
  headerUrl: string | null;                // Profile header/banner image URL
  bio: string;                             // Profile bio/description text
  birthday: string;                        // Date of birth in "YYYY-MM-DD" format
  birthdayVisibility: "PRIVATE"            // Only the user can see
                    | "PUBLIC";            // Everyone can see
  birthdayBalloonsEnabled: boolean;        // Show balloon animation on birthday
  gender: "MALE" | "FEMALE" | "OTHER";
  officialMark: string[];                  // Array of mark color codes (e.g. ["BLUE", "PURPLE"])
  isBotAccount: boolean;                   // Self-declared bot account
  isParodyAccount: boolean;                // Self-declared parody account
  adminForceHidden: boolean;               // Admin-forced profile hiding
  adminForceBot: boolean;                  // Admin-forced bot label
  adminForceParody: boolean;               // Admin-forced parody label
  location: string;                        // User-provided location string
  websiteUrl: string;                      // User-provided website URL

  // Follower/following counts
  followersCount: number;
  followingCount: number;
  postsCount: number;

  // Privacy settings
  isPrivate: boolean;                      // Protected account (followers-only)
  isBanned: boolean;                       // Whether the user is banned
  isAdmin: boolean;                        // Whether the user has admin privileges
  isRestricted: boolean;                   // Whether the user is restricted (limited functionality)

  // Online status
  onlineStatus: "ONLINE" | "OFFLINE" | "DND";
  statusMessage: string;                   // Custom status message text
  onlineStatusVisibility: "PUBLIC"         // Anyone can see status
                        | "PRIVATE";       // Only the user can see
  showOnlineStatus: boolean;               // Master toggle for online status visibility

  // Account verification
  emailVerified: boolean;

  // Privacy preferences
  showLikedPosts: boolean;                 // Show liked posts on profile
  showReadReceipts: boolean;               // Show read receipts in DMs
  directMessagesEnabled: boolean;          // Whether DMs are enabled at all
  dmRequestPolicy: "EVERYONE"              // Anyone can DM
                 | "FOLLOWING"             // Only people you follow
                 | "CIRCLE";              // Only circle members

  // Content filtering
  mutedKeywords: string[];                 // List of muted keywords

  // Notification preferences (all boolean)
  notifyLikes: boolean;
  notifyReplies: boolean;
  notifyMentions: boolean;
  notifyRekarots: boolean;
  notifyFollows: boolean;
  notifyFollowRequests: boolean;
  notifyReactions: boolean;
  notifyQuotes: boolean;
  notifyDMs: boolean;
  notifySystem: boolean;

  // Display preferences (all boolean)
  showBotAccounts: boolean;                // Show posts from bot accounts in timeline
  showHiddenPosts: boolean;                // Show admin-hidden posts
  showR18Content: boolean;                 // Show NSFW/R18 content

  // Safety
  hideProfileFromMinors: boolean;          // Hide this profile from users flagged as minors

  // Timestamps
  createdAt: string;                       // ISO 8601 account creation timestamp
}
```

### User (Compact / Embedded)

When a user is embedded within other objects (e.g., `Post.author`, notification actors), a subset of fields is used:

```ts
interface UserCompact {
  id: number;
  username: string;
  displayName: string;
  avatarUrl: string | null;
  officialMark: string[];
  isPrivate: boolean;
  isParodyAccount: boolean;
  adminForceParody: boolean;
  isBotAccount: boolean;
  adminForceBot: boolean;
}
```

---

## Notification

Represents a notification for the authenticated user. Notifications may be grouped (e.g., multiple likes on the same post).

```ts
interface Notification {
  id: number;                              // Unique notification identifier
  type: "FOLLOW"                           // Someone followed you
      | "FOLLOW_REQUEST"                   // Someone requested to follow you (private account)
      | "LIKE"                             // Someone liked your post
      | "REPLY"                            // Someone replied to your post
      | "MENTION"                          // Someone mentioned you in a post
      | "REKAROT"                          // Someone reposted your post
      | "REACTION"                         // Someone reacted to your post with an emoji
      | "DM"                               // New direct message
      | "QUOTE"                            // Someone quoted your post
      | "REPORT_UPDATE"                    // Update on a report you filed
      | "SYSTEM";                          // System notification
  createdAt: string;                       // ISO 8601 timestamp
  isRead: boolean;                         // Whether the notification has been read
  message: string | null;                  // Human-readable message (for SYSTEM type)

  // Like/rekarot context for grouped notifications
  likeContext: "REKAROTED_POST" | null;    // Indicates the like was on a post you reposted
  rekarotContext: "OWN_POST" | null;       // Indicates the rekarot was of your own post

  // Actor information (who triggered the notification)
  actor: UserCompact | null;               // Primary actor (most recent)
  actors: UserCompact[];                   // All actors (for grouped notifications)
  actorCount: number;                      // Total number of actors (may exceed actors array length)

  // Associated post(s)
  post: Post | null;                       // Primary associated post
  posts: Post[];                           // All associated posts (for grouped notifications)
  postId: number | null;                   // Primary post ID
  postCount: number;                       // Total number of associated posts

  // Grouping
  notificationIds: number[];               // IDs of all notifications grouped into this one
}
```

---

## DM Message

Represents a single message within a DM group conversation.

```ts
interface DmMessage {
  id: number;                              // Unique message identifier
  groupId: string;                         // ID of the DM group this message belongs to
  senderId: number;                        // User ID of the sender
  replyToId: number | null;                // ID of the message being replied to, or null
  content: string;                         // Message text content
  encryptedContent: string | null;         // Encrypted content (if encryption is enabled)
  isEncrypted: boolean;                    // Whether the message is end-to-end encrypted

  // System messages (e.g., "User joined the group")
  systemType: "MEMBER_JOINED"              // A member was added
             | "MEMBER_LEFT"              // A member left
             | null;                       // Not a system message
  systemMeta: object | null;               // Additional metadata for system messages
  systemActorId: number | null;            // User who triggered the system event

  // Attachments
  attachmentUrls: string[];                // URLs of attached files/images
  attachmentTypes: string[];               // MIME types or type identifiers (e.g. "image", "video")
  attachmentAlts: string[];                // Alt text for each attachment
  attachmentSpoilerFlags: boolean[];       // Whether each attachment is a spoiler
  attachmentR18Flags: boolean[];           // Whether each attachment is R18/NSFW

  // State
  isDeleted: boolean;                      // Whether the message has been deleted (tombstone)
  editedAt: string | null;                 // ISO 8601 timestamp of last edit, or null
  createdAt: string;                       // ISO 8601 timestamp of creation
  updatedAt: string;                       // ISO 8601 timestamp of last update

  // Embedded objects
  sender: {
    id: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
    officialMark: string[];
  };
  replyTo: DmMessage | null;              // The message being replied to (nested)
  reactions: Array<{
    emoji: string;
    userId: number;
  }>;

  // Poll (if the message contains a poll)
  poll: {
    options: Array<{
      id: number;
      text: string;
      percentage: number;
      votedByMe: boolean;
    }>;
    totalVotes: number;
    isExpired: boolean;
    expiresAt: string;
  } | null;
}
```

---

## DM Group

Represents a DM group conversation (1-on-1 or group chat).

```ts
interface DMGroup {
  id: string;                              // Unique group identifier (CUID or UUID)
  name: string | null;                     // Group name (null for 1-on-1 DMs)
  isGroup: boolean;                        // true for group chats, false for 1-on-1
  ownerId: number | null;                  // User ID of the group owner (null for 1-on-1)
  iconUrl: string | null;                  // Custom group icon URL

  // Members
  members: Array<{
    userId: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
    officialMark: string[];
    role: string;                          // "owner" | "member"
    joinedAt: string;
  }>;

  // Last message preview
  lastMessage: {
    id: number;
    content: string;
    senderId: number;
    senderUsername: string;
    createdAt: string;
    isDeleted: boolean;
  } | null;

  // Read state
  unreadCount: number;                     // Number of unread messages for the viewer
  lastReadMessageId: number | null;        // Last message ID the viewer has read

  // DM request state (for non-mutual follows)
  isRequest: boolean;                      // Whether this is a pending DM request
  requestStatus: "pending"                 // Awaiting acceptance
               | "accepted"               // Request accepted
               | "declined"               // Request declined
               | null;                    // Not a request

  // Settings
  isMuted: boolean;                        // Whether the viewer has muted this group
  isPinned: boolean;                       // Whether the viewer has pinned this group

  // Active call state
  activeCall: {
    callId: string;
    startedAt: string;
    participants: Array<{
      userId: number;
      username: string;
    }>;
  } | null;

  // Timestamps
  createdAt: string;
  updatedAt: string;
}
```

---

## Draw Room

Represents a collaborative drawing room.

```ts
interface DrawRoom {
  id: string;                              // Unique room identifier
  name: string;                            // Room name
  ownerId: number;                         // User ID of the room creator
  ownerUsername: string;                    // Username of the room creator
  maxParticipants: number;                 // Maximum number of concurrent participants

  // Current participants
  participants: Array<{
    userId: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
  }>;
  participantCount: number;                // Current number of participants

  // Layer data
  layers: DrawLayer[];                     // All layers in the room

  // Settings
  isPublic: boolean;                       // Whether the room is publicly listed
  allowAnonymous: boolean;                 // Whether non-authenticated users can join

  // Timestamps
  createdAt: string;
  updatedAt: string;
}
```

---

## Draw Layer

Represents a single layer within a draw room canvas.

```ts
interface DrawLayer {
  id: string;                              // Unique layer identifier
  name: string;                            // Layer name (e.g. "Layer 1", "Background")
  order: number;                           // Z-order (0 = bottom)
  visible: boolean;                        // Whether the layer is visible
  opacity: number;                         // Layer opacity (0.0 - 1.0)
  dataUrl: string;                         // Full image data as data URI
                                           // "data:image/png;base64,iVBOR..."
  lockedBy: number | null;                 // User ID who locked this layer, or null
}
```

---

## Board

Represents a topic board (community board / forum-like feature).

```ts
interface Board {
  id: number;                              // Unique board identifier
  name: string;                            // Board name
  description: string;                     // Board description
  iconUrl: string | null;                  // Board icon URL
  bannerUrl: string | null;                // Board banner image URL
  ownerId: number;                         // User ID of the board creator
  isPublic: boolean;                       // Whether the board is publicly visible
  isArchived: boolean;                     // Whether the board is archived (read-only)
  memberCount: number;                     // Total number of members
  threadCount: number;                     // Total number of threads
  category: string;                        // Board category
  rules: string[];                         // List of board rules

  // Viewer state
  isMember: boolean;                       // Whether the viewer is a member
  role: "owner" | "moderator" | "member" | null;

  // Timestamps
  createdAt: string;
  updatedAt: string;
}
```

---

## Thread

Represents a thread within a board.

```ts
interface Thread {
  id: number;                              // Unique thread identifier
  boardId: number;                         // ID of the parent board
  title: string;                           // Thread title
  content: string;                         // Thread body content
  author: UserCompact;                     // Thread author (compact user object)

  // Media
  mediaUrls: string[];
  mediaAlts: string[];
  mediaWidths: number[];
  mediaHeights: number[];

  // Engagement
  repliesCount: number;
  viewsCount: number;
  likesCount: number;
  liked: boolean;                          // Whether the viewer liked this thread

  // State
  isPinned: boolean;                       // Whether the thread is pinned
  isLocked: boolean;                       // Whether replies are disabled
  isArchived: boolean;                     // Whether the thread is archived

  // Timestamps
  createdAt: string;
  updatedAt: string;
  lastActivityAt: string;                  // Timestamp of the most recent reply
}
```

---

## Reply (Board Thread Reply)

Represents a reply within a board thread. Not to be confused with post replies.

```ts
interface Reply {
  id: number;                              // Unique reply identifier
  threadId: number;                        // ID of the parent thread
  content: string;                         // Reply text content
  author: UserCompact;                     // Reply author
  parentReplyId: number | null;            // ID of the reply being replied to (nested replies)

  // Media
  mediaUrls: string[];
  mediaAlts: string[];

  // Engagement
  likesCount: number;
  liked: boolean;

  // State
  isDeleted: boolean;
  editedAt: string | null;

  // Timestamps
  createdAt: string;
  updatedAt: string;
}
```

---

## Session

Represents an active login session on a device.

```ts
interface Session {
  id: string;                              // Unique session identifier (e.g. "sess_abc123")
  deviceId: string;                        // UUID of the device
  deviceName: string;                      // Human-readable device name (e.g. "Chrome on Windows")
  clientType: "web" | "ios" | "android";   // Platform type
  isCurrent: boolean;                      // Whether this is the session making the request
  createdAt: string;                       // ISO 8601 session creation timestamp
  lastUsedAt: string;                      // ISO 8601 last activity timestamp
  expiresAt: string;                       // ISO 8601 session expiration timestamp
}
```

---

## Account (Multi-Account)

Represents a stored account entry for multi-account support. Up to 5 accounts can be stored per device.

```ts
interface Account {
  userId: string;                          // User ID (CUID format)
  username: string;                        // Username
  displayName: string;                     // Display name
  avatarUrl: string | null;                // Avatar URL
  sessionId: string;                       // Session ID for this account
  accessToken: string;                     // Current JWT access token
  refreshToken: string;                    // Current refresh token
  deviceId: string;                        // Device ID used for this session
  isActive: boolean;                       // Whether this is the currently active account
  unreadNotifications: number;             // Badge count for notifications
  unreadMessages: number;                  // Badge count for DMs
  lastSwitchedAt: string;                  // ISO 8601 timestamp of last switch to this account
}
```

---

## Pagination

Most list endpoints use cursor-based pagination.

### Cursor-Based Pagination

```ts
interface CursorPagination {
  hasNext: boolean;                        // Whether more results exist
  nextCursor: string | null;               // Cursor to pass as ?cursor= for the next page
  total: number;                           // Total count (not always present)
}
```

### Page-Based Pagination

Some older endpoints use traditional page-based pagination.

```ts
interface PagePagination {
  page: number;                            // Current page number (1-indexed)
  limit: number;                           // Items per page
  total: number;                           // Total number of items
  pages: number;                           // Total number of pages
}
```

---

## Story

Represents a temporary story (disappearing post, similar to Instagram Stories).

```ts
interface Story {
  id: number;                              // Unique story identifier
  author: UserCompact;                     // Story author
  mediaUrl: string;                        // Story media URL (image or video)
  mediaType: "image" | "video";            // Type of media
  caption: string | null;                  // Optional caption text
  viewsCount: number;                      // Number of views

  // Viewer state
  viewed: boolean;                         // Whether the authenticated user has viewed this

  // Admin flags
  adminForceR18: boolean;
  adminForceHidden: boolean;

  // Timestamps
  createdAt: string;                       // ISO 8601
  expiresAt: string;                       // ISO 8601 (typically 24 hours after creation)
}
```

---

## Circle

Represents a user-defined circle for restricting post visibility and reply permissions.

```ts
interface Circle {
  id: number;                              // Unique circle identifier
  name: string;                            // Circle name
  ownerId: number;                         // User ID of the circle creator
  memberCount: number;                     // Number of members in the circle
  members: Array<{
    id: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
  }>;
  createdAt: string;
  updatedAt: string;
}
```

---

## Report

Represents a user-filed report against content or a user.

```ts
interface Report {
  id: number;
  reporterId: number;                      // User who filed the report
  targetType: "post" | "user" | "story" | "dm_message";
  targetId: number | string;               // ID of the reported content
  reason: string;                          // Report reason category
  details: string;                         // Additional details provided by reporter
  status: "pending" | "reviewed" | "resolved" | "dismissed";
  adminNotes: string | null;               // Notes from admin review
  createdAt: string;
  updatedAt: string;
}
```

---

## Survey (Poll on Posts)

Represents a poll attached to a post.

```ts
interface Survey {
  postId: number;                          // The post this poll is attached to
  isActive: boolean;                       // Whether voting is still open
  totalVotes: number;
  satisfactionScore: number;               // Computed satisfaction score (0-100)
  options: Array<{
    text: string;                          // Option display text
    votes: number;                         // Number of votes
    percentage: number;                    // Vote percentage (0-100)
  }>;
}
```

---

## Contact Form Submission

```ts
interface ContactSubmission {
  name: string;                            // Submitter's name
  email: string;                           // Submitter's email
  category: "general"                      // General inquiry
            | "legal"                      // Legal matter
            | "privacy"                    // Privacy concern
            | "rights"                     // Rights/IP issue
            | "safety"                     // Safety concern
            | "bug"                        // Bug report
            | "business";                  // Business inquiry
  subject: string;                         // Subject line
  message: string;                         // Message body
  attachments: string[];                   // Uploaded file URLs
}
```
