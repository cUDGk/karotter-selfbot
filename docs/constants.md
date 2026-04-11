# Constants and Enums

Complete reference for all enums, constants, magic strings, and configuration values used throughout the Karotter platform.

---

## Official Mark Colors

Verification badge colors that can be assigned to users by administrators. A user may have multiple marks simultaneously.

| Code | Hex | RGB | Japanese Label | Description |
|------|-----|-----|----------------|-------------|
| `BLUE` | `#1d9bf0` | `rgb(29, 155, 240)` | 本人確認 | Verified individual identity |
| `YELLOW` | `#f8c500` | `rgb(248, 197, 0)` | 団体・企業 | Organization or company |
| `ORANGE` | `#ff7a00` | `rgb(255, 122, 0)` | オレンジ | Special designation |
| `PURPLE` | `#8b5cf6` | `rgb(139, 92, 246)` | 運営 | Platform staff / administrator |
| `GRAY` | `#9ca3af` | `rgb(156, 163, 175)` | 政府・公的機関 | Government / official body |
| `BLACK` | `#111827` | `rgb(17, 24, 39)` | ブラック | Special designation |
| `RED` | `#ef4444` | `rgb(239, 68, 68)` | レッド | Special designation |
| `GREEN` | `#22c55e` | `rgb(34, 197, 94)` | グリーン | Special designation |

**Type:** `string[]` (array -- users can hold multiple marks)

```ts
const OFFICIAL_MARK_COLORS = ["BLUE", "YELLOW", "ORANGE", "PURPLE", "GRAY", "BLACK", "RED", "GREEN"] as const;
type OfficialMarkColor = typeof OFFICIAL_MARK_COLORS[number];
```

---

## Gender

User gender selection, required during registration.

| Value | Description |
|-------|-------------|
| `MALE` | Male |
| `FEMALE` | Female |
| `OTHER` | Other / prefer not to say |

```ts
type Gender = "MALE" | "FEMALE" | "OTHER";
```

---

## Post Visibility

Controls who can see a post.

| Value | Description |
|-------|-------------|
| `PUBLIC` | Visible to everyone, appears in timelines and search |
| `CIRCLE` | Visible only to members of the specified circle |

```ts
type PostVisibility = "PUBLIC" | "CIRCLE";
```

---

## Reply Restriction

Controls who can reply to a post.

| Value | Description |
|-------|-------------|
| `EVERYONE` | Anyone can reply |
| `FOLLOWING` | Only users the author follows can reply |
| `MENTIONED` | Only users mentioned in the post can reply |
| `CIRCLE` | Only members of the specified circle can reply (requires `replyCircleId`) |

```ts
type ReplyRestriction = "EVERYONE" | "FOLLOWING" | "MENTIONED" | "CIRCLE";
```

---

## Online Status

User's current online status, displayed to others based on privacy settings.

| Value | Description |
|-------|-------------|
| `ONLINE` | Currently active on the platform |
| `OFFLINE` | Not currently active |
| `DND` | Do Not Disturb -- online but does not want to be contacted |

```ts
type OnlineStatus = "ONLINE" | "OFFLINE" | "DND";
```

---

## Online Status Visibility

Controls who can see the user's online status.

| Value | Description |
|-------|-------------|
| `PUBLIC` | Everyone can see the online status |
| `PRIVATE` | Only the user themselves can see the status |

```ts
type OnlineStatusVisibility = "PUBLIC" | "PRIVATE";
```

---

## Birthday Visibility

Controls who can see the user's birthday.

| Value | Description |
|-------|-------------|
| `PUBLIC` | Everyone can see the birthday on the profile |
| `PRIVATE` | Only the user can see their own birthday |

```ts
type BirthdayVisibility = "PUBLIC" | "PRIVATE";
```

---

## DM Request Policy

Controls who can send direct messages to the user without a prior mutual follow.

| Value | Description |
|-------|-------------|
| `EVERYONE` | Anyone can send a DM (may appear as a request if not following) |
| `FOLLOWING` | Only people the user follows can DM directly |
| `CIRCLE` | Only members of the user's circle can DM directly |

```ts
type DmRequestPolicy = "EVERYONE" | "FOLLOWING" | "CIRCLE";
```

---

## Notification Types

All possible notification types in the system.

| Value | Description | Typical Trigger |
|-------|-------------|-----------------|
| `FOLLOW` | Someone followed you | `POST /api/follow/:userId` |
| `FOLLOW_REQUEST` | Someone requested to follow you | Following a private account |
| `LIKE` | Someone liked your post | `POST /api/posts/:id/like` |
| `REPLY` | Someone replied to your post | Creating a post with `parentId` |
| `MENTION` | Someone mentioned you in a post | `@username` in post content |
| `REKAROT` | Someone reposted your post | `POST /api/posts/:id/rekarot` |
| `REACTION` | Someone reacted to your post | `POST /api/posts/:id/react` |
| `DM` | New direct message received | New message in a DM group |
| `QUOTE` | Someone quoted your post | Creating a post with `quotedPostId` |
| `REPORT_UPDATE` | Update on a report you filed | Admin reviews your report |
| `SYSTEM` | System notification | Platform announcements, maintenance |

```ts
type NotificationType =
  | "FOLLOW"
  | "FOLLOW_REQUEST"
  | "LIKE"
  | "REPLY"
  | "MENTION"
  | "REKAROT"
  | "REACTION"
  | "DM"
  | "QUOTE"
  | "REPORT_UPDATE"
  | "SYSTEM";
```

---

## Space (Radio) Roles

Roles within a live audio room (radio/space).

| Value | Description |
|-------|-------------|
| `HOST` | Room creator; can manage speakers, end the room |
| `SPEAKER` | Can broadcast audio to all participants |
| `LISTENER` | Can only listen; must request to become a speaker |

```ts
type SpaceRole = "HOST" | "SPEAKER" | "LISTENER";
```

---

## Space Modes

Visibility/access modes for radio rooms.

| Value | Description |
|-------|-------------|
| `public` | Anyone can join the room |
| `followers` | Only the host's followers can join |
| `invite` | Only explicitly invited users can join |

```ts
type SpaceMode = "public" | "followers" | "invite";
```

---

## Speaker Policies

Controls who can become a speaker in a radio room.

| Value | Description |
|-------|-------------|
| `following` | Only users the host follows can request to speak |
| `everyone` | Any participant can request to speak |
| `invited` | Only users explicitly invited as speakers can speak |

```ts
type SpeakerPolicy = "following" | "everyone" | "invited";
```

---

## DM System Message Types

Types for system-generated messages within DM groups.

| Value | Description |
|-------|-------------|
| `MEMBER_JOINED` | A user was added to the group |
| `MEMBER_LEFT` | A user left the group |

```ts
type DmSystemMessageType = "MEMBER_JOINED" | "MEMBER_LEFT";
```

---

## Contact Form Categories

Categories for the contact/support form.

| Value | Description |
|-------|-------------|
| `general` | General inquiry |
| `legal` | Legal matter |
| `privacy` | Privacy concern or data request |
| `rights` | Intellectual property / rights issue |
| `safety` | Safety concern (harassment, threats) |
| `bug` | Bug report |
| `business` | Business inquiry / partnership |

```ts
type ContactCategory = "general" | "legal" | "privacy" | "rights" | "safety" | "bug" | "business";
```

---

## Admin Flags

### User Admin Flags

Boolean flags that administrators can set on user accounts.

| Flag | Description |
|------|-------------|
| `adminForceHidden` | Hide the user's profile and all posts from public view |
| `adminForceParody` | Display a "parody account" label regardless of user's own setting |
| `adminForceBot` | Display a "bot account" label regardless of user's own setting |

### Post Admin Flags

Boolean flags that administrators can set on individual posts.

| Flag | Description |
|------|-------------|
| `adminForceR18` | Force the post to be treated as R18/NSFW content |
| `adminForceHidden` | Hide the post from all timelines and search results |

### Story Admin Flags

Boolean flags that administrators can set on stories.

| Flag | Description |
|------|-------------|
| `adminForceR18` | Force the story to be treated as R18/NSFW content |
| `adminForceHidden` | Hide the story from all users |

---

## Profile Unavailable Reasons

Reasons why a user's profile may not be viewable.

| Reason | Description |
|--------|-------------|
| `BANNED` | The user has been banned from the platform |
| `SUSPENDED` | The user's account is temporarily suspended |
| `DEACTIVATED` | The user has deactivated their own account |
| `PRIVATE` | The user's account is private and you do not follow them |
| `BLOCKED` | The user has blocked you |
| `BLOCKED_BY_YOU` | You have blocked the user |
| `ADMIN_HIDDEN` | An administrator has hidden the user's profile (`adminForceHidden`) |
| `NOT_FOUND` | The user does not exist |
| `MINOR_RESTRICTED` | The profile is hidden from minor accounts (`hideProfileFromMinors`) |

```ts
type ProfileUnavailableReason =
  | "BANNED"
  | "SUSPENDED"
  | "DEACTIVATED"
  | "PRIVATE"
  | "BLOCKED"
  | "BLOCKED_BY_YOU"
  | "ADMIN_HIDDEN"
  | "NOT_FOUND"
  | "MINOR_RESTRICTED";
```

---

## Call Settings Defaults

Default configuration values for voice/video calls.

| Setting | Default Value | Description |
|---------|---------------|-------------|
| Audio codec | `opus` | Preferred audio codec |
| Video codec | `VP8` | Preferred video codec |
| Max bitrate (audio) | `64000` bps | Maximum audio bitrate |
| Max bitrate (video) | `1500000` bps | Maximum video bitrate |
| ICE transport policy | `all` | Use all available ICE candidates |
| Bundle policy | `max-bundle` | Bundle all media on a single transport |
| RTCP mux policy | `require` | Require RTCP multiplexing |

### STUN/TURN Server Configuration

```ts
const ICE_SERVERS = [
  { urls: "stun:stun.l.google.com:19302" },
  { urls: "stun:stun1.l.google.com:19302" },
];
```

---

## Client Type

Platform identifiers sent in the `x-client-type` header.

| Value | Description |
|-------|-------------|
| `web` | Web browser client |
| `ios` | iOS native app |
| `android` | Android native app |

```ts
type ClientType = "web" | "ios" | "android";
```

---

## localStorage Keys

All `localStorage` keys used by the Karotter web client. All keys are prefixed with `karotter:`.

| Key | Type | Description |
|-----|------|-------------|
| `karotter:token` | `string` | Current JWT access token |
| `karotter:refreshToken` | `string` | Current refresh token |
| `karotter:csrfToken` | `string` | Current CSRF token |
| `karotter:deviceId` | `string` | UUID v4 device identifier (generated once, persisted) |
| `karotter:sessionId` | `string` | Current session ID |
| `karotter:userId` | `string` | Current user's ID |
| `karotter:username` | `string` | Current user's username |
| `karotter:accounts` | `string` (JSON) | Serialized array of `Account` objects for multi-account support |
| `karotter:active-account-id` | `string` | ID of the currently active account |
| `karotter:activeAccountIndex` | `string` (number) | Index of the active account in the accounts array |
| `karotter:theme` | `string` | UI theme: `"light"`, `"dark"`, or `"system"` |
| `karotter:accentColor` | `string` | Accent color hex code |
| `karotter:fontSize` | `string` (number) | Font size preference |
| `karotter:language` | `string` | UI language code (e.g. `"ja"`, `"en"`) |
| `karotter:sidebarCollapsed` | `string` (boolean) | Whether the sidebar is collapsed |
| `karotter:notificationPermission` | `string` | Browser notification permission state |
| `karotter:lastNotificationCheck` | `string` (ISO date) | Timestamp of last notification poll |
| `karotter:drafts` | `string` (JSON) | Saved post drafts |
| `karotter:recentSearches` | `string` (JSON) | Array of recent search queries |
| `karotter:recentEmojis` | `string` (JSON) | Array of recently used emoji |
| `karotter:mutedKeywords` | `string` (JSON) | Local cache of muted keywords |
| `karotter:timelineMode` | `string` | Timeline display mode: `"latest"`, `"recommend"` |
| `karotter:autoplayVideos` | `string` (boolean) | Whether to autoplay videos in timeline |
| `karotter:reduceMotion` | `string` (boolean) | Whether to reduce UI animations |
| `karotter:contentWarningExpanded` | `string` (boolean) | Whether spoiler/CW content is expanded by default |
| `karotter:showR18Content` | `string` (boolean) | Local cache of R18 content preference |
| `karotter:betaExperimentVariant` | `string` | Assigned A/B test variant (A-E) |
| `karotter:onboardingCompleted` | `string` (boolean) | Whether onboarding flow has been completed |
| `karotter:cookieConsent` | `string` (boolean) | Whether the user accepted the cookie banner |
| `karotter:dmEncryptionKey` | `string` | Client-side encryption key for E2E encrypted DMs |
| `karotter:callSettings` | `string` (JSON) | Saved call audio/video preferences |
| `karotter:drawToolSettings` | `string` (JSON) | Last-used draw tool configuration (color, width, tool) |

---

## Native Preferences Keys

Keys used by the native (iOS/Android) apps, stored in platform-specific secure storage (Keychain / SharedPreferences).

| Key | Type | Description |
|-----|------|-------------|
| `karotter_native_accessToken` | `string` | JWT access token (stored securely) |
| `karotter_native_refreshToken` | `string` | Refresh token (stored securely) |
| `karotter_native_deviceId` | `string` | Device UUID |
| `karotter_native_active_account_id` | `string` | ID of the currently active account |
| `karotter_native_sessionId` | `string` | Current session ID |
| `karotter_native_userId` | `string` | Current user ID |
| `karotter_native_biometricEnabled` | `boolean` | Whether biometric unlock is enabled |
| `karotter_native_pushToken` | `string` | Firebase/APNs push notification token |
| `karotter_native_pushEnabled` | `boolean` | Whether push notifications are enabled |
| `karotter_native_accounts` | `string` (JSON) | Multi-account data (encrypted) |
| `karotter_native_lastSyncTimestamp` | `string` (ISO date) | Last data sync timestamp |
| `karotter_native_offlineCache` | `boolean` | Whether offline caching is enabled |
| `karotter_native_dataUsage` | `string` | Data usage mode: `"wifi_only"`, `"always"`, `"never"` |
| `karotter_native_videoQuality` | `string` | Preferred video quality: `"auto"`, `"low"`, `"medium"`, `"high"` |
| `karotter_native_autoDownloadMedia` | `boolean` | Whether to auto-download media attachments |
| `karotter_native_hapticFeedback` | `boolean` | Whether haptic feedback is enabled |
| `karotter_native_appBadgeCount` | `number` | Current app badge number |

---

## HTTP Status Codes

Common status codes returned by the API and their meanings in context.

| Status | Meaning |
|--------|---------|
| `200` | Success |
| `201` | Created (new resource) |
| `400` | Bad request (validation error, invalid parameters) |
| `401` | Unauthorized (token expired or missing) -- triggers auto-refresh |
| `403` | Forbidden (CSRF invalid, banned, insufficient permissions) |
| `404` | Not found (resource does not exist or route does not exist) |
| `409` | Conflict (duplicate resource, e.g. username/email taken) |
| `429` | Too many requests (rate limited) |
| `500` | Internal server error |

---

## Rate Limits

Known rate limit windows.

| Endpoint / Action | Limit | Window |
|-------------------|-------|--------|
| Login attempts | 5 | per 15 minutes |
| Registration | 3 | per hour |
| Email verification resend | 1 | per 10 minutes (600s cooldown) |
| Post creation | 30 | per hour |
| DM messages | 60 | per minute |
| Reactions | 30 | per minute |
| Follow/Unfollow | 30 | per hour |
| General API | 300 | per minute |

---

## Media Constraints

| Constraint | Value |
|------------|-------|
| Max images per post | 4 |
| Max video per post | 1 |
| Images and videos cannot be mixed | N/A |
| Max image file size | 10 MB |
| Max video file size | 100 MB |
| Supported image formats | JPEG, PNG, GIF, WebP |
| Supported video formats | MP4, WebM |
| Draw canvas size | 2560 x 2560 px |
| Draw layer payload max | ~5 MB (all layers combined) |
| Reaction emoji max length | 32 characters |
| Username max length | 15 characters |
| Username pattern | `/^[a-zA-Z0-9_]{1,15}$/` |
| Password length | 8-72 characters |
| Max accounts per device | 5 |
| Post field name for media | `media` |
| DM field name for media | `attachments` |

---

## API Base URLs

| Priority | Domain | Notes |
|----------|--------|-------|
| Primary | `https://api.karotter.com/api` | Main API endpoint |
| Failover 1 | `https://api.karotter.jp/api` | Japanese domain failover |
| Failover 2 | `https://api.karotter.net/api` | .net domain failover |
| Failover 3 | `https://apikarotter.karon.jp/api` | Legacy domain failover |

### CDN / Media URLs

Media URLs in API responses are relative paths. Construct full URLs:

```
Full URL = "https://karotter.com" + mediaUrl
```

---

## Cloudflare Turnstile

Used for bot protection on registration.

| Property | Value |
|----------|-------|
| Sitekey | `0x4AAAAAACujb-w-3YVWR1zA` |
| Required on | `POST /api/auth/register` |
| Field name | `turnstileToken` |
