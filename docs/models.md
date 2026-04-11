# データモデル

Karotter APIで使用されるすべてのデータモデル定義の完全なリファレンスです。すべてのフィールド、型、目的が文書化されています。

---

## Post（投稿）

プラットフォーム上の1つの投稿（"カロート"）を表します。

```ts
interface Post {
  id: number;                              // 一意の投稿識別子
  content: string;                         // 投稿テキスト（メンション、ハッシュタグを含む場合あり）
  createdAt: string;                       // ISO 8601形式の作成日時
  parentId: number | null;                 // リプライの場合は親投稿のID、それ以外はnull

  // 投稿者情報（Userの埋め込みサブセット）
  author: {
    id: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
    isPrivate: boolean;
    officialMark: string[];                // マークカラーコードの配列（例: ["BLUE"]）、空の場合もあり
    isParodyAccount: boolean;
    adminForceParody: boolean;             // 管理者による強制パロディラベル
    isBotAccount: boolean;
    adminForceBot: boolean;                // 管理者による強制Botラベル
  };

  // メディア添付
  mediaUrls: string[];                     // 相対パス（例: ["/uploads/posts/uuid.jpg"]）
  mediaAlts: string[];                     // 各メディアの代替テキスト
  mediaWidths: number[];                   // 各メディアの幅（ピクセル）
  mediaHeights: number[];                  // 各メディアの高さ（ピクセル）
  mediaSpoilerFlags: boolean[];            // 各メディアがスポイラー指定かどうか
  mediaR18Flags: boolean[];                // 各メディアがR18/NSFW指定かどうか

  // エンゲージメント数
  likesCount: number;
  rekarotsCount: number;                   // リポスト数
  repliesCount: number;
  bookmarksCount: number;
  quoteUsersCount: number;                 // この投稿を引用したユーザー数
  viewsCount: number;

  // 閲覧者のインタラクション状態
  liked: boolean;                          // 認証ユーザーがこの投稿をいいねしたか
  rekaroted: boolean;                      // 認証ユーザーがこの投稿をリポストしたか
  bookmarked: boolean;                     // 認証ユーザーがこの投稿をブックマークしたか

  // インタラクション権限
  canInteract: boolean;                    // 閲覧者がいいね/リアクション/リプライできるか
  canQuote: boolean;                       // 閲覧者がこの投稿を引用できるか

  // リアクション（単純ないいね以外の絵文字リアクション）
  reactionSummary: Array<{
    emoji: string;                         // 絵文字テキスト（最大32文字）
    count: number;                         // この絵文字の合計リアクション数
    reacted: boolean;                      // 認証ユーザーがこの絵文字でリアクションしたか
  }>;

  // リプライコンテキスト
  replyToUsers: Array<{
    id: number;
    username: string;
  }>;

  // リプライ制限設定
  replyRestriction: "EVERYONE"             // 誰でもリプライ可能
                  | "FOLLOWING"            // 投稿者がフォローしているユーザーのみ
                  | "MENTIONED"            // 投稿内でメンションされたユーザーのみ
                  | "CIRCLE";              // 指定サークルのメンバーのみ
  replyCircleId: number | null;            // replyRestrictionが"CIRCLE"の場合のサークルID

  // 公開範囲設定
  visibility: "PUBLIC"                     // 全員に公開
            | "CIRCLE";                    // サークルメンバーのみに公開
  viewerCircle: {                          // visibilityが"CIRCLE"で閲覧者がメンバーの場合に存在
    id: number;
    name: string;
  } | null;

  // 引用投稿（埋め込み、再帰的）
  quotedPost: Post | null;                 // 引用している投稿（同じPost構造）、なければnull

  // アンケートデータ
  poll: {
    options: Array<{
      id: number;                          // 選択肢の識別子
      text: string;                        // 選択肢のテキスト
      percentage: number;                  // 投票割合（0-100）
      votedByMe: boolean;                  // 認証ユーザーがこの選択肢に投票したか
    }>;
    totalVotes: number;
    isExpired: boolean;
    expiresAt: string;                     // ISO 8601形式のタイムスタンプ
  } | null;

  // モデレーション / ブロック状態
  isMutedByViewer: boolean;                // 閲覧者がこの投稿の作者をミュートしているか
  hasBlockedAuthor: boolean;               // 閲覧者が作者をブロックしているか
  isBlockedByAuthor: boolean;              // 作者が閲覧者をブロックしているか

  // その他
  itemId: number | null;                   // 関連アイテムID（ショップ/マーケットプレイス連携用）
  type: string;                            // 投稿タイプ識別子
}
```

### メディアURL解決

Media URLs in `mediaUrls` are relative paths. To construct the full URL:

```
Full URL = "https://karotter.com" + mediaUrl
Example:  "https://karotter.com/uploads/posts/abc123.jpg"
```

### リアクション制限

The `emoji` field in reactions accepts any string up to **32 characters**. This is enforced server-side and cannot be bypassed.

---

## User（完全プロフィール）

The complete user object returned by `GET /api/auth/me` and profile endpoints. Note that `email` is only returned by `/api/auth/me`.

```ts
interface User {
  id: number;                              // 一意のユーザー識別子
  username: string;                        // 一意のユーザー名（1-15文字、/^[a-zA-Z0-9_]{1,15}$/）
  displayName: string;                     // 表示名（任意の文字を含む可能性あり）
  email: string;                           // メールアドレス（/auth/meからのみ取得可能）
  avatarUrl: string | null;                // アバター画像URL、デフォルトの場合はnull
  avatarFrameId: string | null;            // 装着中のアバターフレームデコレーションID
  headerUrl: string | null;                // プロフィールヘッダー/バナー画像URL
  bio: string;                             // プロフィール自己紹介テキスト
  birthday: string;                        // 生年月日（"YYYY-MM-DD"形式）
  birthdayVisibility: "PRIVATE"            // 本人のみ閲覧可能
                    | "PUBLIC";            // 全員が閲覧可能
  birthdayBalloonsEnabled: boolean;        // 誕生日にバルーンアニメーションを表示するか
  gender: "MALE" | "FEMALE" | "OTHER";
  officialMark: string[];                  // マークカラーコードの配列（例: ["BLUE", "PURPLE"]）
  isBotAccount: boolean;                   // 自己申告のBotアカウント
  isParodyAccount: boolean;                // 自己申告のパロディアカウント
  adminForceHidden: boolean;               // 管理者による強制プロフィール非表示
  adminForceBot: boolean;                  // 管理者による強制Botラベル
  adminForceParody: boolean;               // 管理者による強制パロディラベル
  location: string;                        // ユーザーが設定した所在地
  websiteUrl: string;                      // ユーザーが設定したWebサイトURL

  // フォロワー/フォロー数
  followersCount: number;
  followingCount: number;
  postsCount: number;

  // プライバシー設定
  isPrivate: boolean;                      // 非公開アカウント（フォロワー限定）
  isBanned: boolean;                       // ユーザーがBANされているか
  isAdmin: boolean;                        // ユーザーが管理者権限を持っているか
  isRestricted: boolean;                   // ユーザーが制限されているか（機能制限あり）

  // オンラインステータス
  onlineStatus: "ONLINE" | "OFFLINE" | "DND";
  statusMessage: string;                   // カスタムステータスメッセージ
  onlineStatusVisibility: "PUBLIC"         // 誰でもステータスを閲覧可能
                        | "PRIVATE";       // 本人のみ閲覧可能
  showOnlineStatus: boolean;               // オンラインステータス表示のマスタートグル

  // アカウント認証
  emailVerified: boolean;

  // プライバシー設定
  showLikedPosts: boolean;                 // プロフィールにいいねした投稿を表示するか
  showReadReceipts: boolean;               // DMで既読を表示するか
  directMessagesEnabled: boolean;          // DMが有効かどうか
  dmRequestPolicy: "EVERYONE"              // 誰でもDM可能
                 | "FOLLOWING"             // フォローしている人のみ
                 | "CIRCLE";              // サークルメンバーのみ

  // コンテンツフィルタリング
  mutedKeywords: string[];                 // ミュートキーワードのリスト

  // 通知設定（すべてboolean）
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

  // 表示設定（すべてboolean）
  showBotAccounts: boolean;                // タイムラインにBotアカウントの投稿を表示するか
  showHiddenPosts: boolean;                // 管理者が非表示にした投稿を表示するか
  showR18Content: boolean;                 // NSFW/R18コンテンツを表示するか

  // セーフティ
  hideProfileFromMinors: boolean;          // 未成年フラグのあるユーザーからプロフィールを非表示にするか

  // タイムスタンプ
  createdAt: string;                       // ISO 8601形式のアカウント作成日時
}
```

### User（コンパクト / 埋め込み）

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

## Notification（通知）

Represents a notification for the authenticated user. Notifications may be grouped (e.g., multiple likes on the same post).

```ts
interface Notification {
  id: number;                              // 一意の通知識別子
  type: "FOLLOW"                           // 誰かにフォローされた
      | "FOLLOW_REQUEST"                   // 誰かにフォローリクエストされた（非公開アカウント）
      | "LIKE"                             // 誰かに投稿をいいねされた
      | "REPLY"                            // 誰かに投稿にリプライされた
      | "MENTION"                          // 誰かに投稿内でメンションされた
      | "REKAROT"                          // 誰かに投稿をリポストされた
      | "REACTION"                         // 誰かに投稿に絵文字リアクションされた
      | "DM"                               // 新着ダイレクトメッセージ
      | "QUOTE"                            // 誰かに投稿を引用された
      | "REPORT_UPDATE"                    // 提出した通報の更新
      | "SYSTEM";                          // システム通知
  createdAt: string;                       // ISO 8601形式のタイムスタンプ
  isRead: boolean;                         // 通知が既読かどうか
  message: string | null;                  // 人間が読めるメッセージ（SYSTEMタイプ用）

  // グループ化通知のいいね/リカロットコンテキスト
  likeContext: "REKAROTED_POST" | null;    // リポストした投稿へのいいねであることを示す
  rekarotContext: "OWN_POST" | null;       // 自分の投稿のリポストであることを示す

  // アクター情報（通知を発生させたユーザー）
  actor: UserCompact | null;               // 主要アクター（最新）
  actors: UserCompact[];                   // 全アクター（グループ化通知用）
  actorCount: number;                      // アクターの合計数（actors配列の長さを超える場合あり）

  // 関連投稿
  post: Post | null;                       // 主要な関連投稿
  posts: Post[];                           // 全関連投稿（グループ化通知用）
  postId: number | null;                   // 主要な投稿ID
  postCount: number;                       // 関連投稿の合計数

  // グループ化
  notificationIds: number[];               // この通知にグループ化された全通知のID
}
```

---

## DM Message（DMメッセージ）

Represents a single message within a DM group conversation.

```ts
interface DmMessage {
  id: number;                              // 一意のメッセージ識別子
  groupId: string;                         // このメッセージが属するDMグループのID
  senderId: number;                        // 送信者のユーザーID
  replyToId: number | null;                // 返信先メッセージのID、またはnull
  content: string;                         // メッセージテキスト
  encryptedContent: string | null;         // 暗号化されたコンテンツ（暗号化が有効な場合）
  isEncrypted: boolean;                    // メッセージがエンドツーエンド暗号化されているか

  // システムメッセージ（例: 「ユーザーがグループに参加しました」）
  systemType: "MEMBER_JOINED"              // メンバーが追加された
             | "MEMBER_LEFT"              // メンバーが退出した
             | null;                       // システムメッセージではない
  systemMeta: object | null;               // システムメッセージの追加メタデータ
  systemActorId: number | null;            // システムイベントを発生させたユーザー

  // 添付ファイル
  attachmentUrls: string[];                // 添付ファイル/画像のURL
  attachmentTypes: string[];               // MIMEタイプまたはタイプ識別子（例: "image", "video"）
  attachmentAlts: string[];                // 各添付ファイルの代替テキスト
  attachmentSpoilerFlags: boolean[];       // 各添付ファイルがスポイラーかどうか
  attachmentR18Flags: boolean[];           // 各添付ファイルがR18/NSFWかどうか

  // 状態
  isDeleted: boolean;                      // メッセージが削除されたか（トゥームストーン）
  editedAt: string | null;                 // ISO 8601形式の最終編集日時、またはnull
  createdAt: string;                       // ISO 8601形式の作成日時
  updatedAt: string;                       // ISO 8601形式の最終更新日時

  // 埋め込みオブジェクト
  sender: {
    id: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
    officialMark: string[];
  };
  replyTo: DmMessage | null;              // 返信先メッセージ（ネスト）
  reactions: Array<{
    emoji: string;
    userId: number;
  }>;

  // アンケート（メッセージにアンケートが含まれる場合）
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

## DM Group（DMグループ）

Represents a DM group conversation (1-on-1 or group chat).

```ts
interface DMGroup {
  id: string;                              // 一意のグループ識別子（CUIDまたはUUID）
  name: string | null;                     // グループ名（1対1のDMではnull）
  isGroup: boolean;                        // グループチャットならtrue、1対1ならfalse
  ownerId: number | null;                  // グループオーナーのユーザーID（1対1ではnull）
  iconUrl: string | null;                  // カスタムグループアイコンURL

  // メンバー
  members: Array<{
    userId: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
    officialMark: string[];
    role: string;                          // "owner" | "member"
    joinedAt: string;
  }>;

  // 最新メッセージプレビュー
  lastMessage: {
    id: number;
    content: string;
    senderId: number;
    senderUsername: string;
    createdAt: string;
    isDeleted: boolean;
  } | null;

  // 既読状態
  unreadCount: number;                     // 閲覧者の未読メッセージ数
  lastReadMessageId: number | null;        // 閲覧者が最後に読んだメッセージID

  // DMリクエスト状態（相互フォローでない場合）
  isRequest: boolean;                      // 保留中のDMリクエストかどうか
  requestStatus: "pending"                 // 承認待ち
               | "accepted"               // リクエスト承認済み
               | "declined"               // リクエスト拒否済み
               | null;                    // リクエストではない

  // 設定
  isMuted: boolean;                        // 閲覧者がこのグループをミュートしているか
  isPinned: boolean;                       // 閲覧者がこのグループをピン留めしているか

  // アクティブ通話状態
  activeCall: {
    callId: string;
    startedAt: string;
    participants: Array<{
      userId: number;
      username: string;
    }>;
  } | null;

  // タイムスタンプ
  createdAt: string;
  updatedAt: string;
}
```

---

## Draw Room（絵チャルーム）

Represents a collaborative drawing room.

```ts
interface DrawRoom {
  id: string;                              // 一意のルーム識別子
  name: string;                            // ルーム名
  ownerId: number;                         // ルーム作成者のユーザーID
  ownerUsername: string;                    // ルーム作成者のユーザー名
  maxParticipants: number;                 // 最大同時参加者数

  // 現在の参加者
  participants: Array<{
    userId: number;
    username: string;
    displayName: string;
    avatarUrl: string | null;
  }>;
  participantCount: number;                // 現在の参加者数

  // レイヤーデータ
  layers: DrawLayer[];                     // ルーム内の全レイヤー

  // 設定
  isPublic: boolean;                       // ルームが公開リストに載っているか
  allowAnonymous: boolean;                 // 未認証ユーザーが参加できるか

  // タイムスタンプ
  createdAt: string;
  updatedAt: string;
}
```

---

## Draw Layer（絵チャレイヤー）

Represents a single layer within a draw room canvas.

```ts
interface DrawLayer {
  id: string;                              // 一意のレイヤー識別子
  name: string;                            // レイヤー名（例: "Layer 1", "Background"）
  order: number;                           // Z順序（0 = 最背面）
  visible: boolean;                        // レイヤーが表示されているか
  opacity: number;                         // レイヤーの不透明度（0.0 - 1.0）
  dataUrl: string;                         // 完全な画像データ（data URI形式）
                                           // "data:image/png;base64,iVBOR..."
  lockedBy: number | null;                 // このレイヤーをロックしたユーザーID、またはnull
}
```

---

## Board（掲示板）

Represents a topic board (community board / forum-like feature).

```ts
interface Board {
  id: number;                              // 一意の掲示板識別子
  title: string;                           // 掲示板タイトル
  slug: string;                            // URLフレンドリーなスラッグ（全掲示板URLで使用）
  description: string | null;              // 掲示板の説明
  minimumAge: number;                      // アクセスに必要な最低年齢（13-99）
  threadCount: number;                     // スレッドの合計数
  replyCount: number;                      // 全スレッドのリプライ合計数
  lastPostAt: string | null;              // ISO 8601形式の最新アクティビティ日時
  creator: {
    id: number;                            // 作成者のユーザーID
  };

  // タイムスタンプ
  createdAt: string;
  updatedAt: string;
}
```

---

## Thread（スレッド）

Represents a thread within a board.

```ts
interface Thread {
  id: number;                              // 一意のスレッド識別子
  boardId: number;                         // 親掲示板のID
  title: string;                           // スレッドタイトル
  content: string;                         // スレッド本文
  author: UserCompact;                     // スレッド作成者（コンパクトユーザーオブジェクト）

  // メディア
  mediaUrls: string[];
  mediaAlts: string[];
  mediaWidths: number[];
  mediaHeights: number[];

  // エンゲージメント
  repliesCount: number;
  viewsCount: number;
  likesCount: number;
  liked: boolean;                          // 閲覧者がこのスレッドをいいねしたか

  // 状態
  isPinned: boolean;                       // スレッドがピン留めされているか
  isLocked: boolean;                       // リプライが無効化されているか
  isArchived: boolean;                     // スレッドがアーカイブされているか

  // タイムスタンプ
  createdAt: string;
  updatedAt: string;
  lastActivityAt: string;                  // 最新リプライのタイムスタンプ
}
```

---

## Reply（掲示板スレッドリプライ）

Represents a reply within a board thread. Not to be confused with post replies.

```ts
interface Reply {
  id: number;                              // 一意のリプライ識別子
  threadId: number;                        // 親スレッドのID
  content: string;                         // リプライテキスト
  author: UserCompact;                     // リプライ投稿者
  parentReplyId: number | null;            // 返信先リプライのID（ネストされたリプライ）

  // メディア
  mediaUrls: string[];
  mediaAlts: string[];

  // エンゲージメント
  likesCount: number;
  liked: boolean;

  // 状態
  isDeleted: boolean;
  editedAt: string | null;

  // タイムスタンプ
  createdAt: string;
  updatedAt: string;
}
```

---

## Session（セッション）

Represents an active login session on a device.

```ts
interface Session {
  id: string;                              // 一意のセッション識別子（例: "sess_abc123"）
  deviceId: string;                        // デバイスのUUID
  deviceName: string;                      // 人間が読めるデバイス名（例: "Chrome on Windows"）
  clientType: "web" | "ios" | "android";   // プラットフォームタイプ
  isCurrent: boolean;                      // このセッションがリクエスト元かどうか
  createdAt: string;                       // ISO 8601形式のセッション作成日時
  lastUsedAt: string;                      // ISO 8601形式の最終アクティビティ日時
  expiresAt: string;                       // ISO 8601形式のセッション有効期限
}
```

---

## Account（マルチアカウント）

Represents a stored account entry for multi-account support. Up to 5 accounts can be stored per device.

```ts
interface Account {
  userId: string;                          // ユーザーID（CUID形式）
  username: string;                        // ユーザー名
  displayName: string;                     // 表示名
  avatarUrl: string | null;                // アバターURL
  sessionId: string;                       // このアカウントのセッションID
  accessToken: string;                     // 現在のJWTアクセストークン
  refreshToken: string;                    // 現在のリフレッシュトークン
  deviceId: string;                        // このセッションで使用するデバイスID
  isActive: boolean;                       // 現在アクティブなアカウントかどうか
  unreadNotifications: number;             // 通知のバッジカウント
  unreadMessages: number;                  // DMのバッジカウント
  lastSwitchedAt: string;                  // ISO 8601形式のこのアカウントへの最終切替日時
}
```

---

## ページネーション

Most list endpoints use cursor-based pagination.

### カーソルベースページネーション

```ts
interface CursorPagination {
  hasNext: boolean;                        // さらに結果が存在するか
  nextCursor: string | null;               // 次のページ取得時に?cursor=で渡すカーソル
  total: number;                           // 合計数（常に含まれるとは限らない）
}
```

### ページベースページネーション

Some older endpoints use traditional page-based pagination.

```ts
interface PagePagination {
  page: number;                            // 現在のページ番号（1始まり）
  limit: number;                           // 1ページあたりのアイテム数
  total: number;                           // アイテムの合計数
  pages: number;                           // ページの合計数
}
```

---

## Story（ストーリー）

Represents a temporary story (disappearing post, similar to Instagram Stories).

```ts
interface Story {
  id: number;                              // 一意のストーリー識別子
  author: UserCompact;                     // ストーリー投稿者
  mediaUrl: string;                        // ストーリーメディアURL（画像または動画）
  mediaType: "image" | "video";            // メディアの種類
  caption: string | null;                  // 任意のキャプションテキスト
  viewsCount: number;                      // 閲覧数

  // 閲覧者の状態
  viewed: boolean;                         // 認証ユーザーがこのストーリーを閲覧済みか

  // 管理者フラグ
  adminForceR18: boolean;
  adminForceHidden: boolean;

  // タイムスタンプ
  createdAt: string;                       // ISO 8601
  expiresAt: string;                       // ISO 8601（通常、作成から24時間後）
}
```

---

## Circle（サークル）

Represents a user-defined circle for restricting post visibility and reply permissions.

```ts
interface Circle {
  id: number;                              // 一意のサークル識別子
  name: string;                            // サークル名
  ownerId: number;                         // サークル作成者のユーザーID
  memberCount: number;                     // サークルのメンバー数
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

## Report（通報）

Represents a user-filed report against content or a user.

```ts
interface Report {
  id: number;
  reporterId: number;                      // 通報を提出したユーザー
  targetType: "post" | "user" | "story" | "dm_message";
  targetId: number | string;               // 通報されたコンテンツのID
  reason: string;                          // 通報理由カテゴリ
  details: string;                         // 通報者が提供した追加詳細
  status: "pending" | "reviewed" | "resolved" | "dismissed";
  adminNotes: string | null;               // 管理者レビューからのメモ
  createdAt: string;
  updatedAt: string;
}
```

---

## Survey（投稿の投票）

Represents a poll attached to a post.

```ts
interface Survey {
  postId: number;                          // このアンケートが添付された投稿
  isActive: boolean;                       // 投票がまだ受付中かどうか
  totalVotes: number;
  satisfactionScore: number;               // 算出された満足度スコア（0-100）
  options: Array<{
    text: string;                          // 選択肢の表示テキスト
    votes: number;                         // 投票数
    percentage: number;                    // 投票割合（0-100）
  }>;
}
```

---

## お問い合わせフォーム送信

```ts
interface ContactSubmission {
  name: string;                            // 送信者の名前
  email: string;                           // 送信者のメールアドレス
  category: "general"                      // 一般的なお問い合わせ
            | "legal"                      // 法的な問題
            | "privacy"                    // プライバシーに関する懸念
            | "rights"                     // 権利/知的財産の問題
            | "safety"                     // 安全性に関する懸念
            | "bug"                        // バグ報告
            | "business";                  // ビジネスに関するお問い合わせ
  subject: string;                         // 件名
  message: string;                         // メッセージ本文
  attachments: string[];                   // アップロードされたファイルのURL
}
```
