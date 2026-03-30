# Karotter Selfbot API リファレンス & ライブラリ

[Karotter](https://karotter.com) の非公式APIリファレンスとセルフBot用クライアントライブラリです。

## 注意事項

- これは**非公式プロジェクト**です。Karotter / KaronNetWork とは一切関係ありません。
- APIの情報は独自に調査・解析したものです。**最新のAPI仕様とは異なる可能性があります。** エンドポイントやリクエスト/レスポンスの形式は予告なく変更されることがあります。
- **このリポジトリは都度更新していきますが、常に最新とは限りません。** 実際のAPIの挙動と照らし合わせて確認してください。
- 利用は自己責任でお願いします。このライブラリの使用によるアカウントBANやデータ損失等について、作者は一切責任を負いません。
- Karotterの [API・Bot利用規約](https://karotter.com/legal/api-bot-terms) を守ってください。スパム、大量投稿、無差別DM、レート制限回避、無断データ収集、AI学習利用等は禁止されています。

## 中身

### APIリファレンス (`docs/`)

Karotterの全APIエンドポイントを機能ごとにまとめたドキュメントです。

| ドキュメント | 内容 |
|------------|------|
| [Authentication](docs/authentication.md) | ログイン、新規登録、CSRF、トークンリフレッシュ、セッション、マルチアカウント |
| [Users](docs/users.md) | プロフィール、設定、アバター/ヘッダー画像、アカウント管理 |
| [Posts](docs/posts.md) | 投稿の作成/編集/削除、いいね、リカロット、ブックマーク、リアクション、投票、アナリティクス |
| [Timeline](docs/timeline.md) | タイムライン、おすすめアルゴリズム、A/Bテストバリアント |
| [Search](docs/search.md) | 投稿・ユーザー・ハッシュタグ検索、トレンド、ディスカバー |
| [Follow](docs/follow.md) | フォロー/アンフォロー、ブロック、ミュート、フォローリクエスト |
| [DM](docs/dm.md) | ダイレクトメッセージ、グループチャット、添付ファイル、投票、通話 |
| [Notifications](docs/notifications.md) | 通知タイプ、プッシュ通知 (FCM)、未読数 |
| [Social](docs/social.md) | サークル、リスト、ストーリー、リンクプレビュー |
| [Radio](docs/radio.md) | スペース (音声通話)、WebRTC、スピーカー管理 |
| [Draw Chat](docs/draw.md) | 絵チャ (共同お絵かき)、レイヤー、ストローク |
| [Boards](docs/boards.md) | 掲示板、スレッド、リプライ、SSE |
| [Socket.IO](docs/socketio.md) | リアルタイムイベント (DM、通話、絵チャ、スペース、通知) |
| [Data Models](docs/models.md) | 全オブジェクトの型定義 |
| [Admin](docs/admin.md) | 管理パネル、モデレーション、アナリティクス |
| [Constants](docs/constants.md) | Enum、フラグ、localStorageキー一覧 |

### Python ライブラリ (`python/`)

`httpx` + `python-socketio` を使った非同期クライアント。

```bash
cd python
pip install -e .
```

```python
import asyncio
from karotter import KarotterClient

async def main():
    client = KarotterClient(username="ユーザー名", password="パスワード")
    await client.login()

    # 投稿
    post = await client.create_post(content="Pythonから投稿!")

    # タイムライン取得
    timeline = await client.get_timeline(mode="latest", page=1)

    # いいね
    await client.like(post.id)

    # DM送信
    await client.send_message(group_id=123, content="やっほー！")

    # 検索
    results = await client.search("karotter")

    await client.close()

asyncio.run(main())
```

### TypeScript ライブラリ (`typescript/`)

`fetch` + `socket.io-client` を使ったクライアント。

```bash
cd typescript
npm install
npm run build
```

```typescript
import { KarotterClient } from "./src/client";

const client = new KarotterClient({
  username: "ユーザー名",
  password: "パスワード",
});

await client.login();

// 投稿
const post = await client.posts.create({ content: "TypeScriptから投稿!" });

// タイムライン
const timeline = await client.posts.getTimeline({ mode: "latest" });

// いいね
await client.posts.like(post.id);

// DM
await client.dm.sendMessage(123, { content: "やっほー!" });

// 検索
const results = await client.search.search("karotter");
```

## ベースURL

```
https://api.karotter.com/api
```

フェイルオーバー: `api.karotter.jp`, `api.karotter.net`, `apikarotter.karon.jp`

## コントリビュート

新しいエンドポイントを見つけた場合や、仕様が変わっていた場合はPRお願いします。ネットワークログやJSバンドルの参照箇所など、根拠を添えてもらえると助かります。

## ライセンス

MIT
