# karotter - Karotter Python クライアント

[Karotter](https://karotter.com) の非公式 Python クライアントライブラリ。投稿、DM、フォロー、検索、通知、スペース、絵チャ、掲示板、管理パネルなど全エンドポイントに対応。

> **注意:** これは非公式ライブラリです。独自調査をもとに作成しており、**最新のAPIとは異なる場合があります。** 都度更新しますが、常に最新とは限りません。利用は自己責任で。

## インストール

```bash
pip install -e .
# または
pip install httpx python-socketio[asyncio_client] aiofiles
```

## 使い方

### ログインして投稿

```python
import asyncio
from karotter import KarotterClient

async def main():
    async with KarotterClient(username="ユーザー名", password="パスワード") as client:
        await client.login()

        # 投稿
        post = await client.create_post("Karotterに投稿!")
        print(f"投稿ID: {post.id}")

        # タイムライン取得
        posts, pagination = await client.get_timeline(mode="latest", limit=10)
        for p in posts:
            print(f"@{p.author.username}: {p.content[:50]}")

asyncio.run(main())
```

### トークン認証

```python
async with KarotterClient(token="eyJ...") as client:
    me = await client.get_me()
    print(f"@{me.username} (ID: {me.id}) でログイン中")
```

### DM

```python
async with KarotterClient(username="user", password="pass") as client:
    await client.login()

    # DMグループ一覧
    groups = await client.get_groups()
    for g in groups:
        print(f"グループ {g.id}: {g.name or 'DM'} (未読: {g.unread_count})")

    # メッセージ送信
    msg = await client.send_message(487, "やっほー!")
    print(f"送信: メッセージID {msg.id}")

    # 新しいDMを開始
    group = await client.start_dm(user_id=12345)
    await client.send_message(group.id, "はじめまして!")
```

### いいね・リカロット・リアクション

```python
# いいね / 取消
await client.like(post_id=119340)
await client.unlike(post_id=119340)

# リカロット
await client.rekarot(post_id=119340)

# リアクション (最大32文字)
await client.react(post_id=119340, emoji="fire")

# ブックマーク
await client.bookmark(post_id=119340)

# 投票
await client.vote_poll(post_id=119340, option_id=1)
```

### フォロー / ブロック / ミュート

```python
await client.follow(user_id=100)
await client.unfollow(user_id=100)
await client.block(user_id=200)
await client.mute(user_id=300)

blocked = await client.get_blocked()
muted = await client.get_muted()
```

### 検索

```python
posts, _ = await client.search_posts("karotter", sort="latest")
users = await client.search_users("claude")
trends = await client.get_trending_topics(limit=5)
```

### リアルタイム (Socket.IO)

```python
from karotter import KarotterRealtime

rt = KarotterRealtime(token="eyJ...")

@rt.on_dm_message
async def on_dm(data):
    print("新着DM:", data)

@rt.on_notification
async def on_notif(data):
    print("通知:", data)

await rt.connect()
await rt.join_dm(group_id=487)
await rt.start_typing(group_id=487)
await rt.wait()
```

### 絵チャ

```python
from karotter import DrawLayer

room = await client.create_room("お絵かき部屋")
print(f"ルームID: {room.id}")

# レイヤー更新 (base64 PNGデータ)
layer = DrawLayer(id="layer1", name="背景", order=0, visible=True, opacity=1.0,
                  data_url="data:image/png;base64,iVBOR...")
await client.update_layers(room.id, [layer])
```

### エラーハンドリング

```python
from karotter import KarotterError, RateLimitError, NotFoundError

try:
    post = await client.get_post(999999)
except NotFoundError:
    print("投稿が見つかりません")
except RateLimitError as e:
    print(f"レート制限。{e.retry_after}秒後にリトライ")
except KarotterError as e:
    print(f"APIエラー: {e.message} (HTTP {e.status_code})")
```

## モデル

APIレスポンスは型付きdataclassに変換されます:

- `Post` — 投稿 (全フィールド)
- `User` — ユーザー (/auth/me の完全版)
- `Profile` — プロフィール (フォロー関係フラグ付き)
- `Author` — 簡易ユーザー情報 (投稿等に埋め込み)
- `DmMessage` / `DMGroup` — DMモデル
- `Notification` — 通知 (アクター、投稿付き)
- `DrawRoom` / `DrawLayer` — 絵チャモデル
- `Space` — スペースモデル
- `Circle` / `UserList` / `Story` — ソーシャル機能
- `Hashtag` / `Trend` — 検索/トレンド
- `Poll` / `PollOption` / `Reaction` / `ReactionSummary`
- `PostAnalytics` — 投稿アナリティクス
- `Session` / `ApiKey` / `Pagination`

各モデルには `from_dict()` クラスメソッドがあります。

## APIカバレッジ

| モジュール | メソッド | 説明 |
|-----------|---------|------|
| Auth | login, logout, refresh, me, sessions, email, password reset | 認証 |
| Posts | create, edit, delete, like, rekarot, bookmark, react, poll, timeline, analytics | 投稿 |
| Users | get profile, posts, followers, following, edit profile, settings, avatar, header | ユーザー |
| DM | groups, messages, send, edit, delete, react, poll, call, group management | DM |
| Follow | follow, unfollow, block, mute, remove follower, pending requests | フォロー |
| Search | posts, users, hashtags, trending topics | 検索 |
| Notifications | list, unread count, mark read, push register/unregister | 通知 |
| Social | circles, lists, stories, link preview | ソーシャル |
| Radio | create space, join/leave, messages, speaker management | スペース |
| Draw | rooms, layers, join, invite rotation | 絵チャ |
| Boards | boards, threads, replies | 掲示板 |
| Admin | users, posts, stories, bans, stats, settings (管理者権限必要) | 管理 |

## ライセンス

MIT
