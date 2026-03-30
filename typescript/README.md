# Karotter TypeScript クライアント

[Karotter](https://karotter.com) の非公式 TypeScript クライアントライブラリ。全エンドポイント対応、完全型定義付き。

> **注意:** これは非公式ライブラリです。独自調査をもとに作成しており、**最新のAPIとは異なる場合があります。** 都度更新しますが、常に最新とは限りません。利用は自己責任で。

## インストール

```bash
npm install
npm run build
```

## 使い方

```ts
import { KarotterClient } from "./src/client";

// ユーザー名/パスワードでログイン
const client = new KarotterClient({ username: "ユーザー名", password: "パスワード" });
await client.login();

// またはトークンで直接認証
const client2 = new KarotterClient({ token: "your-jwt-token" });
```

### 投稿

```ts
// カロートを投稿
const post = await client.posts.create({ content: "TypeScriptから投稿!" });

// 画像付き投稿
import { readFileSync } from "fs";
const image = readFileSync("./image.png");
const post2 = await client.posts.create({
  content: "写真付きカロート",
  media: [image],
  mediaAlts: ["代替テキスト"],
});

// タイムライン取得
const { posts } = await client.posts.getTimeline({ mode: "latest", limit: 20 });

// いいね / リカロット / ブックマーク
await client.posts.like(post.id);
await client.posts.rekarot(post.id);
await client.posts.bookmark(post.id);

// リアクション (最大32文字)
await client.posts.react(post.id, "👍");

// 投票
await client.posts.votePoll(postId, optionId);
```

### ユーザー

```ts
// ユーザー取得
const user = await client.users.getUser("karon");

// 自分の情報
const me = await client.auth.me();

// プロフィール更新
await client.users.updateProfile({ displayName: "新しい名前", bio: "自己紹介" });

// フォロワー一覧
const { users } = await client.users.getFollowers("karon");
```

### DM

```ts
// グループ一覧
const groups = await client.dm.getGroups();

// メッセージ取得
const { messages } = await client.dm.getMessages(487, 50);

// メッセージ送信
await client.dm.sendMessage(487, { content: "こんにちは!" });

// 既読マーク
await client.dm.markAsRead(487);
```

### フォロー / ブロック / ミュート

```ts
await client.follow.follow(userId);
await client.follow.unfollow(userId);
await client.follow.block(userId);
await client.follow.mute(userId);
```

### 検索

```ts
// 統合検索
const result = await client.search.searchAll({ q: "カロッター" });

// トレンド
const { trends } = await client.search.getTrendingTopics();
```

### 通知

```ts
const notifications = await client.notifications.getNotifications(30);
const { count } = await client.notifications.getUnreadCount();
await client.notifications.markAllRead();
```

### ソーシャル (サークル / リスト / ストーリー)

```ts
// サークル
const circles = await client.social.getCircles();
await client.social.createCircle("仲良し");

// リスト
const lists = await client.social.getLists();
const { posts } = await client.social.getListPosts(listId);

// ストーリー
const stories = await client.social.getStories();
await client.social.viewStory(storyId);
```

### 絵チャ

```ts
const rooms = await client.draw.getRooms();
const room = await client.draw.getRoom("room-id");

// レイヤー更新 (2560x2560 PNG, base64, 全レイヤー一括送信必須)
await client.draw.updateLayers(room.id, [
  { id: "layer1", name: "背景", order: 0, visible: true, opacity: 1, dataUrl: "data:image/png;base64,..." },
]);
```

### スペース

```ts
const { spaces } = await client.radio.getActiveSpaces();
const space = await client.radio.createSpace({ title: "雑談スペース" });
await client.radio.joinSpace(space.id);
await client.radio.sendMessage(space.id, "よろしく!");
```

### リアルタイム (Socket.IO)

```ts
const rt = client.realtime;
rt.connect();

rt.on("dm:message", (msg) => {
  console.log(`${msg.sender.username}: ${msg.content}`);
});

rt.on("notification", (notif) => {
  console.log(`通知: ${notif.type}`);
});

// DMルームに参加
rt.joinDM(487);

// タイピング通知
rt.startTyping(487);

// 切断
rt.disconnect();
```

### エラーハンドリング

```ts
import { AuthenticationError, RateLimitError, NotFoundError } from "./src/errors";

try {
  await client.posts.getPost(999999);
} catch (e) {
  if (e instanceof NotFoundError) {
    console.log("投稿が見つかりません");
  } else if (e instanceof RateLimitError) {
    console.log(`レート制限。${e.retryAfter}秒後にリトライ`);
  } else if (e instanceof AuthenticationError) {
    console.log("認証エラー。再ログインが必要");
  }
}
```

## APIカバレッジ

| モジュール | メソッド数 | 説明 |
|-----------|----------|------|
| `auth` | 12 | ログイン、セッション、メール認証 |
| `posts` | 27 | 投稿CRUD、いいね、リカロット、タイムライン |
| `users` | 15 | ユーザー情報、プロフィール編集、設定 |
| `dm` | 17 | DMグループ、メッセージ、通話 |
| `follow` | 9 | フォロー、ブロック、ミュート |
| `search` | 6 | 統合検索、トレンド |
| `notifications` | 7 | 通知取得、既読、プッシュ |
| `social` | 16 | サークル、リスト、ストーリー |
| `radio` | 12 | スペース作成、参加、チャット |
| `draw` | 6 | 絵チャルーム、レイヤー操作 |
| `boards` | 9 | 掲示板、スレッド、返信 |
| `admin` | 50+ | 管理パネル全操作 |
| `realtime` | 15 | Socket.IO イベント |

## 要件

- Node.js >= 18 (native fetch 必須)
- TypeScript >= 5.0

## ライセンス

MIT
