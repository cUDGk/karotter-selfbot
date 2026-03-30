# Users

Complete reference for user profiles, followers/following, profile editing, settings, and account management.

---

## Get User Profile

### `GET /users/:usernameOrId`

Retrieve a user's public profile by their username or user ID. Includes relationship data relative to the authenticated user.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `usernameOrId` | `string` | The user's username (e.g. `myusername`) or their unique ID (e.g. `clx1abc2d3ef...`) |

**Request:**

```http
GET /users/myusername HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "user": {
    "id": "clx1abc2d3ef4gh5ij6kl7mn8",
    "username": "myusername",
    "displayName": "My Display Name",
    "avatarUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp",
    "headerUrl": "https://cdn.karotter.com/headers/clx1abc2d3ef4gh5ij6kl7mn8.webp",
    "bio": "Hello world! This is my profile.",
    "location": "Tokyo, Japan",
    "websiteUrl": "https://example.com",
    "birthday": "2000-05-15",
    "birthdayVisibility": "PUBLIC",
    "birthdayBalloonsEnabled": true,
    "createdAt": "2025-01-15T12:00:00.000Z",
    "followersCount": 150,
    "followingCount": 75,
    "postsCount": 320,
    "isPrivate": false,
    "isBanned": false,
    "onlineStatus": "ONLINE",
    "statusMessage": "Working!",
    "showLikedPosts": true,
    "officialMark": [],
    "isParodyAccount": false,
    "adminForceParody": false,
    "isBotAccount": false,
    "adminForceBot": false
  },
  "isFollowing": true,
  "isFollowedBy": false,
  "hasPendingRequest": false,
  "hasBlocked": false,
  "isBlockedBy": false,
  "isMuted": false,
  "mutualFollowersCount": 3,
  "mutualFollowersPreview": [
    {
      "id": "clx_mutual1",
      "username": "mutualfriend1",
      "displayName": "Mutual Friend",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_mutual1.webp"
    }
  ],
  "pinnedPost": {
    "id": "post_pinned123",
    "content": "This is my pinned post!",
    "createdAt": "2026-02-01T00:00:00.000Z"
  },
  "profileUnavailableReason": null,
  "profileUnavailableDetails": []
}
```

**User Object Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique user identifier (CUID format) |
| `username` | `string` | Unique username (1-15 chars, alphanumeric + underscore) |
| `displayName` | `string` | Display name (1-40 Unicode characters) |
| `avatarUrl` | `string \| null` | URL to avatar image, or `null` if not set |
| `headerUrl` | `string \| null` | URL to header/banner image, or `null` if not set |
| `bio` | `string` | User biography (0-200 characters) |
| `location` | `string \| null` | Free-text location |
| `websiteUrl` | `string \| null` | User's website URL |
| `birthday` | `string \| null` | Birthday in `YYYY-MM-DD` format, or `null` |
| `birthdayVisibility` | `string` | One of `PRIVATE`, `PUBLIC` |
| `birthdayBalloonsEnabled` | `boolean` | Whether birthday balloon animation is enabled |
| `createdAt` | `string` | ISO 8601 account creation timestamp |
| `followersCount` | `number` | Number of followers |
| `followingCount` | `number` | Number of users being followed |
| `postsCount` | `number` | Total number of posts |
| `isPrivate` | `boolean` | Whether the account is private (requires follow approval) |
| `isBanned` | `boolean` | Whether the account is banned |
| `onlineStatus` | `string` | One of `ONLINE`, `OFFLINE`, `DND` |
| `statusMessage` | `string \| null` | Custom status message (max 15 characters) |
| `showLikedPosts` | `boolean` | Whether the user's liked posts tab is public |
| `officialMark` | `array` | Array of official verification marks (empty for unverified users) |
| `isParodyAccount` | `boolean` | Whether the user has self-declared as a parody account |
| `adminForceParody` | `boolean` | Whether an admin has forced the parody label |
| `isBotAccount` | `boolean` | Whether the user has self-declared as a bot account |
| `adminForceBot` | `boolean` | Whether an admin has forced the bot label |

**Relationship Fields (relative to the authenticated viewer):**

| Field | Type | Description |
|-------|------|-------------|
| `isFollowing` | `boolean` | Whether the viewer follows this user |
| `isFollowedBy` | `boolean` | Whether this user follows the viewer |
| `hasPendingRequest` | `boolean` | Whether the viewer has a pending follow request to this user |
| `hasBlocked` | `boolean` | Whether the viewer has blocked this user |
| `isBlockedBy` | `boolean` | Whether the viewer is blocked by this user |
| `isMuted` | `boolean` | Whether the viewer has muted this user |
| `mutualFollowersCount` | `number` | Number of followers in common |
| `mutualFollowersPreview` | `array` | Small preview array of mutual follower user objects |
| `pinnedPost` | `Post \| null` | The user's pinned post, or `null` |

**Profile Unavailable Reasons:**

When `profileUnavailableReason` is non-null, the profile content may be hidden from the viewer.

| `profileUnavailableReason` | Description |
|----------------------------|-------------|
| `FILTERED` | Profile is filtered from the viewer's view |
| `null` | Profile is fully visible |

The `profileUnavailableDetails` array provides specific reasons:

| Detail | Description |
|--------|-------------|
| `ADMIN_HIDDEN` | An admin has hidden this profile |
| `PARODY_FILTERED` | The viewer has parody accounts filtered out |
| `BOT_FILTERED` | The viewer has bot accounts filtered out |
| `R18_FILTERED` | The viewer has R18/adult content filtered out |
| `MINOR_RESTRICTED` | The viewer is a minor and this content is age-restricted |

---

## User Posts, Replies, Media, Likes

### `GET /users/:id/posts`

Get a user's original posts (excluding replies).

### `GET /users/:id/replies`

Get a user's replies.

### `GET /users/:id/media`

Get a user's media posts (posts with images or videos).

### `GET /users/:id/likes`

Get posts the user has liked (only available if the user has `showLikedPosts` enabled).

**Query Parameters (all four endpoints):**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `number` | `1` | Page number (1-indexed) |
| `limit` | `number` | `20` | Results per page |

**Request:**

```http
GET /users/clx1abc2d3ef4gh5ij6kl7mn8/posts?page=1&limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "posts": [
    {
      "id": "post_abc123",
      "content": "Hello world!",
      "author": {
        "id": "clx1abc2d3ef4gh5ij6kl7mn8",
        "username": "myusername",
        "displayName": "My Display Name",
        "avatarUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp"
      },
      "createdAt": "2026-03-30T08:00:00.000Z",
      "likesCount": 5,
      "repliesCount": 2,
      "rekarotsCount": 1
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20
  }
}
```

---

## Followers & Following

### `GET /users/:id/followers`

Get a user's followers list.

### `GET /users/:id/following`

Get the list of users this user follows.

**Query Parameters (both endpoints):**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `20` | Results per page |
| `cursor` | `string` | — | Cursor for pagination (from previous response) |

**Request:**

```http
GET /users/clx1abc2d3ef4gh5ij6kl7mn8/followers?limit=20 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_follower1",
      "username": "follower_one",
      "displayName": "Follower One",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_follower1.webp",
      "bio": "I follow people",
      "isPrivate": false,
      "is_following": false,
      "is_followed_by": true,
      "follow_request_sent": false,
      "officialMark": [],
      "isParodyAccount": false,
      "adminForceParody": false,
      "isBotAccount": false,
      "adminForceBot": false
    }
  ],
  "pagination": {
    "nextCursor": "cursor_abc123def456"
  }
}
```

> **IMPORTANT: Snake case fields.** The follower/following user objects use **snake_case** for relationship fields (`is_following`, `is_followed_by`, `follow_request_sent`) instead of the camelCase used elsewhere in the API. This is an inconsistency in the server response.

**Follower/Following User Object Fields:**

| Field | Type | Case | Description |
|-------|------|------|-------------|
| `id` | `string` | camelCase | User ID |
| `username` | `string` | camelCase | Username |
| `displayName` | `string` | camelCase | Display name |
| `avatarUrl` | `string \| null` | camelCase | Avatar URL |
| `bio` | `string` | camelCase | Bio text |
| `isPrivate` | `boolean` | camelCase | Whether account is private |
| `is_following` | `boolean` | **snake_case** | Whether the viewer follows this user |
| `is_followed_by` | `boolean` | **snake_case** | Whether this user follows the viewer |
| `follow_request_sent` | `boolean` | **snake_case** | Whether the viewer has a pending follow request |
| `officialMark` | `array` | camelCase | Official verification marks |
| `isParodyAccount` | `boolean` | camelCase | Parody account flag |
| `adminForceParody` | `boolean` | camelCase | Admin-forced parody label |
| `isBotAccount` | `boolean` | camelCase | Bot account flag |
| `adminForceBot` | `boolean` | camelCase | Admin-forced bot label |

**Pagination:**

Use cursor-based pagination. Pass the `nextCursor` value from the response as the `cursor` query parameter in the next request. When `nextCursor` is `null` or absent, there are no more results.

---

## Edit Profile

### `PATCH /users/profile`

Update the authenticated user's profile fields. All fields are optional; only include fields you want to change.

**Request Body:**

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `displayName` | `string` | 1-40 Unicode characters | Display name |
| `bio` | `string` | 0-200 characters | Biography text |
| `location` | `string \| null` | — | Location text |
| `websiteUrl` | `string \| null` | — | Website URL |
| `birthday` | `string \| null` | `YYYY-MM-DD` or `null` | Birthday (set `null` to clear) |
| `birthdayVisibility` | `string` | `PRIVATE` or `PUBLIC` | Birthday visibility |
| `birthdayBalloonsEnabled` | `boolean` | — | Enable birthday balloon animation |
| `gender` | `string` | `MALE`, `FEMALE`, or `OTHER` | Gender |

**Request:**

```http
PATCH /users/profile HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "displayName": "New Display Name",
  "bio": "Updated bio text",
  "birthday": "2000-05-15",
  "birthdayVisibility": "PUBLIC"
}
```

**Response `200 OK`:**

```json
{
  "user": {
    "id": "clx1abc2d3ef4gh5ij6kl7mn8",
    "username": "myusername",
    "displayName": "New Display Name",
    "bio": "Updated bio text",
    "birthday": "2000-05-15",
    "birthdayVisibility": "PUBLIC"
  }
}
```

### Text Sanitization

All text fields undergo server-side sanitization:

1. **HTML stripping** (`stripHtml`): Any HTML tags are removed from the input.
2. **Control character removal**: Unicode control characters (C0, C1 ranges) are stripped.
3. **Character counting**: Character count is calculated as `Array.from(text).length`, which counts **Unicode codepoints**, not bytes. This means one emoji (e.g. `😀`) counts as 1 character, and combined emoji (e.g. `👨‍👩‍👧‍👦`) may count as multiple codepoints.

---

## Update Online Status

### `PATCH /users/status`

Set the user's online status and optional status message.

**Request Body:**

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `status` | `string` | `ONLINE`, `OFFLINE`, `DND` | Online status |
| `statusMessage` | `string` | Max 15 characters | Custom status text |

**Request:**

```http
PATCH /users/status HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "status": "DND",
  "statusMessage": "Do not disturb"
}
```

**Response `200 OK`:**

```json
{
  "status": "DND",
  "statusMessage": "Do not disturb"
}
```

---

## Profile Images

### `POST /profile/avatar`

Upload a new avatar image.

**Request:** `multipart/form-data`

| Field | Type | Description |
|-------|------|-------------|
| `avatar` | `File` | Image file (JPEG, PNG, WebP, GIF) |

**Request:**

```http
POST /profile/avatar HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="avatar"; filename="avatar.png"
Content-Type: image/png

(binary data)
------FormBoundary--
```

**Response `200 OK`:**

```json
{
  "imageUrl": "https://cdn.karotter.com/avatars/clx1abc2d3ef4gh5ij6kl7mn8.webp"
}
```

### `POST /profile/header`

Upload a new header/banner image.

**Request:** `multipart/form-data`

| Field | Type | Description |
|-------|------|-------------|
| `header` | `File` | Image file (JPEG, PNG, WebP, GIF) |

**Response `200 OK`:**

```json
{
  "imageUrl": "https://cdn.karotter.com/headers/clx1abc2d3ef4gh5ij6kl7mn8.webp"
}
```

---

## Change Password

### `PATCH /users/password`

Change the authenticated user's password.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `currentPassword` | `string` | Yes | Current password for verification |
| `newPassword` | `string` | Yes | New password (must pass password policy) |

**Request:**

```http
PATCH /users/password HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "currentPassword": "MyOldPassword123",
  "newPassword": "MyNewPassword456"
}
```

**Response `200 OK`:**

```json
{
  "message": "Password updated successfully"
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `400` | `WEAK_PASSWORD` | New password fails password policy |
| `401` | `INVALID_PASSWORD` | Current password is incorrect |

---

## Change Username

### `PATCH /users/username`

Change the authenticated user's username. Subject to a quota system.

**Request Body:**

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `username` | `string` | Yes | `/^[a-zA-Z0-9_]{1,15}$/` | New username |

**Request:**

```http
PATCH /users/username HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "username": "mynewusername"
}
```

**Response `200 OK`:**

```json
{
  "user": {
    "id": "clx1abc2d3ef4gh5ij6kl7mn8",
    "username": "mynewusername"
  }
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| `400` | `INVALID_USERNAME` | Does not match regex pattern |
| `409` | `USERNAME_TAKEN` | Username is already in use |
| `429` | `QUOTA_EXCEEDED` | No remaining username changes in the 14-day window |

### `GET /users/username/quota`

Check how many username changes remain in the current 14-day window.

**Request:**

```http
GET /users/username/quota HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "remainingChanges": 2,
  "usedChanges": 1
}
```

---

## User Settings

### `PATCH /users/settings`

Update user settings. This is a partial update; only include the fields you want to change.

**Request Body (all fields optional):**

| Field | Type | Description |
|-------|------|-------------|
| `isPrivate` | `boolean` | Make account private (require follow approval) |
| `showLikedPosts` | `boolean` | Show liked posts tab on profile |
| `showReadReceipts` | `boolean` | Show read receipts in DMs |
| `directMessagesEnabled` | `boolean` | Allow direct messages |
| `dmRequestPolicy` | `string` | DM request policy: `EVERYONE`, `FOLLOWING`, `CIRCLE` |
| `showReactions` | `boolean` | Show reactions on posts. **Special:** setting to `false` also forces `notifyReactions` to `false` |
| `notifyLikes` | `boolean` | Notify on likes |
| `notifyReplies` | `boolean` | Notify on replies |
| `notifyRekarots` | `boolean` | Notify on rekarots |
| `notifyFollows` | `boolean` | Notify on new followers |
| `notifyMentions` | `boolean` | Notify on mentions |
| `notifyQuotes` | `boolean` | Notify on quote posts |
| `notifyReactions` | `boolean` | Notify on reactions |
| `notifyDMs` | `boolean` | Notify on direct messages |
| `notifyDMRequests` | `boolean` | Notify on DM requests |
| `notifyPollResults` | `boolean` | Notify on poll result updates |
| `showOnlineStatus` | `boolean` | Show online status to others |
| `showFollowerCount` | `boolean` | Show follower count on profile |
| `showFollowingCount` | `boolean` | Show following count on profile |
| `showPostCount` | `boolean` | Show post count on profile |
| `showBirthday` | `boolean` | Show birthday on profile |
| `mutedKeywords` | `string[]` | Array of muted keyword strings |

**Request:**

```http
PATCH /users/settings HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "isPrivate": true,
  "dmRequestPolicy": "FOLLOWING",
  "notifyLikes": false,
  "showReactions": false,
  "mutedKeywords": ["spoiler", "leak"]
}
```

**Response `200 OK`:**

```json
{
  "settings": {
    "isPrivate": true,
    "showLikedPosts": true,
    "showReadReceipts": true,
    "directMessagesEnabled": true,
    "dmRequestPolicy": "FOLLOWING",
    "showReactions": false,
    "notifyLikes": false,
    "notifyReplies": true,
    "notifyRekarots": true,
    "notifyFollows": true,
    "notifyMentions": true,
    "notifyQuotes": true,
    "notifyReactions": false,
    "notifyDMs": true,
    "notifyDMRequests": true,
    "notifyPollResults": true,
    "showOnlineStatus": true,
    "showFollowerCount": true,
    "showFollowingCount": true,
    "showPostCount": true,
    "showBirthday": true,
    "mutedKeywords": ["spoiler", "leak"]
  }
}
```

> **Note:** When `showReactions` is set to `false`, the server automatically sets `notifyReactions` to `false` as well, regardless of what value was sent for `notifyReactions`.

---

## Delete Account

### `DELETE /users/account`

Permanently delete the authenticated user's account. This action is irreversible.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `password` | `string` | Yes | Current password for confirmation |

**Request:**

```http
DELETE /users/account HTTP/1.1
Host: api.karotter.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
x-csrf-token: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{
  "password": "MyPassword123"
}
```

**Response `200 OK`:**

```json
{
  "message": "Account deleted successfully"
}
```

---

## Recommended Users

### `GET /users/recommended`

Get a list of recommended users to follow.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `number` | `3` | Number of recommendations to return |

**Request:**

```http
GET /users/recommended?limit=3 HTTP/1.1
Host: api.karotter.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Response `200 OK`:**

```json
{
  "users": [
    {
      "id": "clx_rec1",
      "username": "popular_user",
      "displayName": "Popular User",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_rec1.webp",
      "bio": "Content creator",
      "followersCount": 5000,
      "isPrivate": false,
      "officialMark": ["VERIFIED"],
      "isParodyAccount": false,
      "isBotAccount": false
    },
    {
      "id": "clx_rec2",
      "username": "trending_creator",
      "displayName": "Trending Creator",
      "avatarUrl": "https://cdn.karotter.com/avatars/clx_rec2.webp",
      "bio": "Artist & designer",
      "followersCount": 3200,
      "isPrivate": false,
      "officialMark": [],
      "isParodyAccount": false,
      "isBotAccount": false
    }
  ]
}
```
