# 定数とEnum

Karotterプラットフォーム全体で使用されるすべてのEnum、定数、マジック文字列、設定値の完全なリファレンスです。

---

## 公式マークカラー

管理者によってユーザーに割り当てることができる認証バッジカラーです。ユーザーは同時に複数のマークを持つことができます。

| コード | Hex | RGB | 日本語ラベル | 説明 |
|------|-----|-----|----------------|-------------|
| `BLUE` | `#1d9bf0` | `rgb(29, 155, 240)` | 本人確認 | 本人確認済みの個人 |
| `YELLOW` | `#f8c500` | `rgb(248, 197, 0)` | 団体・企業 | 団体または企業 |
| `ORANGE` | `#ff7a00` | `rgb(255, 122, 0)` | オレンジ | 特別な指定 |
| `PURPLE` | `#8b5cf6` | `rgb(139, 92, 246)` | 運営 | プラットフォームスタッフ / 管理者 |
| `GRAY` | `#9ca3af` | `rgb(156, 163, 175)` | 政府・公的機関 | 政府 / 公的機関 |
| `BLACK` | `#111827` | `rgb(17, 24, 39)` | ブラック | 特別な指定 |
| `RED` | `#ef4444` | `rgb(239, 68, 68)` | レッド | 特別な指定 |
| `GREEN` | `#22c55e` | `rgb(34, 197, 94)` | グリーン | 特別な指定 |

**型:** `string[]` (配列 -- ユーザーは複数のマークを保持可能)

```ts
const OFFICIAL_MARK_COLORS = ["BLUE", "YELLOW", "ORANGE", "PURPLE", "GRAY", "BLACK", "RED", "GREEN"] as const;
type OfficialMarkColor = typeof OFFICIAL_MARK_COLORS[number];
```

---

## 性別

ユーザーの性別選択。登録時に必須です。

| 値 | 説明 |
|-------|-------------|
| `MALE` | 男性 |
| `FEMALE` | 女性 |
| `OTHER` | その他 / 回答しない |

```ts
type Gender = "MALE" | "FEMALE" | "OTHER";
```

---

## Post（投稿） Visibility

投稿の表示可能なユーザーを制御します。

| 値 | 説明 |
|-------|-------------|
| `PUBLIC` | 全員に表示、タイムラインと検索に表示される |
| `CIRCLE` | 指定したサークルのメンバーのみに表示 |

```ts
type PostVisibility = "PUBLIC" | "CIRCLE";
```

---

## リプライ制限

投稿にリプライ可能なユーザーを制御します。

| 値 | 説明 |
|-------|-------------|
| `EVERYONE` | 誰でもリプライ可能 |
| `FOLLOWING` | 投稿者がフォローしているユーザーのみリプライ可能 |
| `MENTIONED` | 投稿内でメンションされたユーザーのみリプライ可能 |
| `CIRCLE` | 指定したサークルのメンバーのみリプライ可能（`replyCircleId` が必要） |

```ts
type ReplyRestriction = "EVERYONE" | "FOLLOWING" | "MENTIONED" | "CIRCLE";
```

---

## オンラインステータス

ユーザーの現在のオンライ��ステータス。プライバシ���設定に基づいて他のユーザーに表示されます。

| 値 | 説明 |
|-------|-------------|
| `ONLINE` | 現在プラットフォーム上でアクティブ |
| `OFFLINE` | 現在アクティブではない |
| `DND` | 取り込み中 -- オンラインだが連絡を受けたくない状態 |

```ts
type OnlineStatus = "ONLINE" | "OFFLINE" | "DND";
```

---

## オンラインステータス Visibility

ユーザーのオンラインステータスを閲覧可能なユーザーを制御します。

| 値 | 説明 |
|-------|-------------|
| `PUBLIC` | 全員がオンラインステータスを閲覧可能 |
| `PRIVATE` | 本人のみステータスを閲覧可能 |

```ts
type OnlineStatusVisibility = "PUBLIC" | "PRIVATE";
```

---

## 誕生日の公開範囲

ユーザーの誕生日を閲覧可能なユーザーを制御します。

| 値 | 説明 |
|-------|-------------|
| `PUBLIC` | 全員がプロフィール上の誕生日を閲覧可能 |
| `PRIVATE` | 本人のみ自分の誕生日を閲覧可能 |

```ts
type BirthdayVisibility = "PUBLIC" | "PRIVATE";
```

---

## DMリクエストポリシー

事前の相互フォローなしにユーザーにダイレクトメッセージを送信可能なユーザーを制御します。

| 値 | 説明 |
|-------|-------------|
| `EVERYONE` | 誰でもDMを送信可能（フォローしていない場合はリクエストとして表示される場合あり） |
| `FOLLOWING` | ユーザーがフォローしている人のみ直接DM可能 |
| `CIRCLE` | ユーザーのサークルのメンバーのみ直接DM可能 |

```ts
type DmRequestPolicy = "EVERYONE" | "FOLLOWING" | "CIRCLE";
```

---

## 通知タイプ

システム内のすべての通知タイプです。

| 値 | 説明 | 主なトリガー |
|-------|-------------|-----------------|
| `FOLLOW` | フォローされた | `POST /api/follow/:userId` |
| `FOLLOW_REQUEST` | フォローリクエストを受けた | 非公開アカウントへのフォロー |
| `LIKE` | 投稿がいいねされた | `POST /api/posts/:id/like` |
| `REPLY` | 投稿にリプライされた | `parentId` 付きの投稿作成 |
| `MENTION` | 投稿内でメンションされた | 投稿内容に `@username` |
| `REKAROT` | 投稿がリカロットされた | `POST /api/posts/:id/rekarot` |
| `REACTION` | 投稿にリアクションされた | `POST /api/posts/:id/react` |
| `DM` | 新しいダイレクトメッセージを受信 | DMグループ内の新メッセージ |
| `QUOTE` | 投稿が引用された | `quotedPostId` 付きの投稿作成 |
| `REPORT_UPDATE` | 報告の更新 | 管理者による報告のレビュー |
| `SYSTEM` | システム通知 | プラットフォームのお知らせ、メンテナンス |

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

## スペース（ラジオ）ロール

ライブ音声ルーム（ラジオ/スペース）内のロールです。

| 値 | 説明 |
|-------|-------------|
| `HOST` | ルーム作成者。スピーカーの管理やルームの終了が可能 |
| `SPEAKER` | 全参加者に音声を配信可能 |
| `LISTENER` | 聴取のみ可能。スピーカーになるにはリクエストが必要 |

```ts
type SpaceRole = "HOST" | "SPEAKER" | "LISTENER";
```

---

## スペースモード

ラジオルームの公開範囲/アクセスモードです。

| 値 | 説明 |
|-------|-------------|
| `public` | 誰でもルームに参加可能 |
| `followers` | ホストのフォロワーのみ参加可能 |
| `invite` | 明示的に招待されたユーザーのみ参加可能 |

```ts
type SpaceMode = "public" | "followers" | "invite";
```

---

## スピーカーポリシー

ラジオルームでスピーカーになれるユーザーを制御します。

| 値 | 説明 |
|-------|-------------|
| `following` | ホストがフォローしているユーザーのみスピーカーリクエスト可能 |
| `everyone` | 全参加者がスピーカーリクエスト可能 |
| `invited` | スピーカーとして明示的に招待されたユーザーのみ発言可能 |

```ts
type SpeakerPolicy = "following" | "everyone" | "invited";
```

---

## DMシステムメッセージタイプ

DMグループ内のシステム生成メッセージのタイプです。

| 値 | 説明 |
|-------|-------------|
| `MEMBER_JOINED` | ユーザーがグループに追加された |
| `MEMBER_LEFT` | ユーザーがグループを退出した |

```ts
type DmSystemMessageType = "MEMBER_JOINED" | "MEMBER_LEFT";
```

---

## お問い合わせフォームカテゴリ

お問い合わせ/サポートフォームのカテゴリです。

| 値 | 説明 |
|-------|-------------|
| `general` | 一般的なお問い合わせ |
| `legal` | 法的な事項 |
| `privacy` | プライバシーに関する懸念またはデータリクエスト |
| `rights` | 知的財産 / 権利に関する問題 |
| `safety` | 安全に関する懸念（嫌がらせ、脅迫） |
| `bug` | バグ報告 |
| `business` | ビジネスに関するお問い合わせ / 提携 |

```ts
type ContactCategory = "general" | "legal" | "privacy" | "rights" | "safety" | "bug" | "business";
```

---

## 管理フラグ

### ユーザー管理フラグ

管理者がユーザーアカウントに設定できるブールフラグです。

| フラグ | 説明 |
|------|-------------|
| `adminForceHidden` | ユーザーのプロフィールと全投稿を公開ビューから非表示にする |
| `adminForceParody` | ユーザー自身の設定に関わらず「パロディアカウント」ラベルを表示する |
| `adminForceBot` | ユーザー自身の設定に関わらず「Botアカウント」ラベルを表示する |

### Post（投稿）管理フラグ

管理者が個別の投稿に設定できるブールフラグです。

| フラグ | 説明 |
|------|-------------|
| `adminForceR18` | 投稿を強制的にR18/NSFWコンテンツとして扱う |
| `adminForceHidden` | 投稿を全タイムラインと検索結果から非表示にする |

### Story（ストーリー）管理フラグ

管理者がストーリーに設定できるブールフラグです。

| フラグ | 説明 |
|------|-------------|
| `adminForceR18` | ストーリーを強制的にR18/NSFWコンテンツとして扱う |
| `adminForceHidden` | ストーリーを全ユーザーから非表示にする |

---

## プロフィール非表示理由

ユーザーのプロフィールが閲覧できない理由です。

| 理由 | 説明 |
|--------|-------------|
| `BANNED` | ユーザーがプラットフォームからBANされている |
| `SUSPENDED` | ユーザーのアカウントが一時的に凍結されている |
| `DEACTIVATED` | ユーザーが自分のアカウントを無効化した |
| `PRIVATE` | ユーザーのアカウントが非公開で、あなたはフォローしていない |
| `BLOCKED` | ユーザーにブロックされている |
| `BLOCKED_BY_YOU` | あなたがユーザーをブロックしている |
| `ADMIN_HIDDEN` | 管理者がユーザーのプロフィールを非表示にした（`adminForceHidden`） |
| `NOT_FOUND` | ユーザーが存在しない |
| `MINOR_RESTRICTED` | 未成年アカウントからプロフィールが非表示になっている（`hideProfileFromMinors`） |

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

## 通話設定デフォルト

音声/ビデオ通話のデフォルト設定値です。

| 設定 | デフォルト値 | 説明 |
|---------|---------------|-------------|
| 音声コーデック | `opus` | 優先音声コーデック |
| 映像コーデック | `VP8` | 優先映像コーデック |
| 最大ビットレート（音声） | `64000` bps | 音声の最大ビットレート |
| 最大ビットレート（映像） | `1500000` bps | 映像の最大ビットレート |
| ICEトランスポートポリシー | `all` | 利用可能な全ICE候補を使用 |
| バンドルポリシー | `max-bundle` | 全メディアを単一トランスポートにバンドル |
| RTCPマルチプレックスポリシー | `require` | RTCPマルチプレキシングを必須にする |

### STUN/TURNサーバー設定

```ts
const ICE_SERVERS = [
  { urls: "stun:stun.l.google.com:19302" },
  { urls: "stun:stun1.l.google.com:19302" },
];
```

---

## クライアントタイプ

`x-client-type` ヘッダーで送信されるプラットフォーム識別子です。

| 値 | 説明 |
|-------|-------------|
| `web` | Webブラウザクライアント |
| `ios` | iOSネイティブアプリ |
| `android` | Androidネイティブアプリ |

```ts
type ClientType = "web" | "ios" | "android";
```

---

## localStorageキー

Karotter Webクライアントで使用される全ての `localStorage` キーです。全てのキーには `karotter:` プレフィックスが付きます。

| キー | タイプ | 説明 |
|-----|------|-------------|
| `karotter:token` | `string` | 現在のJWTアクセストークン |
| `karotter:refreshToken` | `string` | 現在のリフレッシュトークン |
| `karotter:csrfToken` | `string` | 現在のCSRFトークン |
| `karotter:deviceId` | `string` | UUID v4デバイス識別子（一度生成され永続化） |
| `karotter:sessionId` | `string` | 現在のセッションID |
| `karotter:userId` | `string` | 現在のユーザーのID |
| `karotter:username` | `string` | 現在のユーザーのユーザー名 |
| `karotter:accounts` | `string` (JSON) | マルチアカウント対応の `Account` オブジェクトのシリアライズ配列 |
| `karotter:active-account-id` | `string` | 現在アクティブなアカウントのID |
| `karotter:activeAccountIndex` | `string` (number) | アカウント配列内のアクティブアカウントのインデックス |
| `karotter:theme` | `string` | UIテーマ: `"light"`, `"dark"`, または `"system"` |
| `karotter:accentColor` | `string` | アクセントカラーの16進コード |
| `karotter:fontSize` | `string` (number) | フォントサイズの設定 |
| `karotter:language` | `string` | UI言語コード（例: `"ja"`, `"en"`） |
| `karotter:sidebarCollapsed` | `string` (boolean) | サイドバーが折りたたまれているかどうか |
| `karotter:notificationPermission` | `string` | ブラウザ通知の許可状態 |
| `karotter:lastNotificationCheck` | `string` (ISO date) | 最後の通知ポーリングのタイムスタンプ |
| `karotter:drafts` | `string` (JSON) | 保存された投稿の下書き |
| `karotter:recentSearches` | `string` (JSON) | 最近の検索クエリの配列 |
| `karotter:recentEmojis` | `string` (JSON) | 最近使用した絵文字の配列 |
| `karotter:mutedKeywords` | `string` (JSON) | ミュートキーワードのローカルキャッシュ |
| `karotter:timelineMode` | `string` | タイムライン表示モード: `"latest"`, `"recommend"` |
| `karotter:autoplayVideos` | `string` (boolean) | タイムラインで動画を自動再生するかどうか |
| `karotter:reduceMotion` | `string` (boolean) | UIアニメーションを軽減するかどうか |
| `karotter:contentWarningExpanded` | `string` (boolean) | ネタバレ/CWコンテンツをデフォルトで展開するかどうか |
| `karotter:showR18Content` | `string` (boolean) | R18コンテンツ設定のローカルキャッシュ |
| `karotter:betaExperimentVariant` | `string` | 割り当てられたA/Bテストバリアント（A-E） |
| `karotter:onboardingCompleted` | `string` (boolean) | オンボーディングフローが完了したかどうか |
| `karotter:cookieConsent` | `string` (boolean) | ユーザーがCookieバナーに同意したかどうか |
| `karotter:dmEncryptionKey` | `string` | E2E暗号化DM用のクライアント側暗号化キー |
| `karotter:callSettings` | `string` (JSON) | 保存された通話の音声/映像設定 |
| `karotter:drawToolSettings` | `string` (JSON) | 最後に使用したお絵かきツールの設定（色、太さ、ツール） |

---

## ネイティブ設定キー

ネイティブ（iOS/Android）アプリで使用されるキーです。プラットフォーム固有のセキュアストレージ（Keychain / SharedPreferences）に保存されます。

| キー | タイプ | 説明 |
|-----|------|-------------|
| `karotter_native_accessToken` | `string` | JWTアクセストークン（セキュアに保存） |
| `karotter_native_refreshToken` | `string` | リフレッシュトークン（セキュアに保存） |
| `karotter_native_deviceId` | `string` | デバイスUUID |
| `karotter_native_active_account_id` | `string` | 現在アクティブなアカウントのID |
| `karotter_native_sessionId` | `string` | 現在のセッションID |
| `karotter_native_userId` | `string` | 現在のユーザーID |
| `karotter_native_biometricEnabled` | `boolean` | 生体認証ロック解除が有効かどうか |
| `karotter_native_pushToken` | `string` | Firebase/APNsプッシュ通知トークン |
| `karotter_native_pushEnabled` | `boolean` | プッシュ通知が有効かどうか |
| `karotter_native_accounts` | `string` (JSON) | マルチアカウントデータ（暗号化済み） |
| `karotter_native_lastSyncTimestamp` | `string` (ISO date) | 最終データ同期のタイムスタンプ |
| `karotter_native_offlineCache` | `boolean` | オフラインキャッシュが有効かどうか |
| `karotter_native_dataUsage` | `string` | データ使用モード: `"wifi_only"`, `"always"`, `"never"` |
| `karotter_native_videoQuality` | `string` | 優先動画品質: `"auto"`, `"low"`, `"medium"`, `"high"` |
| `karotter_native_autoDownloadMedia` | `boolean` | メディア添付ファイルを自動ダウンロードするかどうか |
| `karotter_native_hapticFeedback` | `boolean` | 触覚フィードバックが有効かどうか |
| `karotter_native_appBadgeCount` | `number` | 現在のアプリバッジ数 |

---

## HTTPステータスコード

APIが返す一般的なステータスコードとその意味です。

| ステータス | 意味 |
|--------|---------|
| `200` | 成功 |
| `201` | 作成済み（新しいリソース） |
| `400` | 不正なリクエスト（バリデーションエラー、無効なパラメータ） |
| `401` | 未認証（トークン期限切れまたは欠落） -- 自動リフレッシュをトリガー |
| `403` | 禁止（CSRF無効、BAN済み、権限不足） |
| `404` | 見つからない（リソースまたはルートが存在しない） |
| `409` | 競合（リソースの重複、例: ユーザー名/メールアドレスが使用済み） |
| `429` | リクエスト過多（レート制限） |
| `500` | 内部サーバーエラー |

---

## レート制限

既知のレート制限ウィンドウです。

| エンドポイント / アクション | 制限 | ウィンドウ |
|-------------------|-------|--------|
| ログイン試行 | 5 | 15分あたり |
| 登録 | 3 | 1時間あたり |
| メール認証の再送信 | 1 | 10分あたり（600秒クールダウン） |
| 投稿作成 | 30 | 1時間あたり |
| DMメッセージ | 60 | 1分あたり |
| リアクション | 30 | 1分あたり |
| フォロー/フォロー解除 | 30 | 1時間あたり |
| 一般API | 300 | 1分あたり |

---

## メディア制約

| 制約 | 値 |
|------------|-------|
| 1投稿あたりの最大画像数 | 4 |
| 1投稿あたりの最大動画数 | 1 |
| 画像と動画の混在は不可 | N/A |
| 最大画像ファイルサイズ | 10 MB |
| 最大動画ファイルサイズ | 100 MB |
| 対応画像フォーマット | JPEG, PNG, GIF, WebP |
| 対応動画フォーマット | MP4, WebM |
| お絵かきキャンバスサイズ | 2560 x 2560 px |
| お絵かきレイヤーペイロード上限 | 約5 MB（全レイヤー合計） |
| リアクション絵文字の最大長 | 32文字 |
| ユーザー名の最大長 | 15文字 |
| ユーザー名パターン | `/^[a-zA-Z0-9_]{1,15}$/` |
| パスワード長 | 8-72文字 |
| 1デバイスあたりの最大アカウント数 | 5 |
| 投稿のメディアフィールド名 | `media` |
| DMのメディアフィールド名 | `attachments` |

---

## APIベースURL

| 優先度 | ドメイン | 備考 |
|----------|--------|-------|
| プライマリ | `https://api.karotter.com/api` | メインAPIエンドポイント |
| フェイルオーバー1 | `https://api.karotter.jp/api` | 日本ドメインフェイルオーバー |
| フェイルオーバー2 | `https://api.karotter.net/api` | .netドメインフェイルオーバー |
| フェイルオーバー3 | `https://apikarotter.karon.jp/api` | レガシードメインフェイルオーバー |

### CDN / メディアURL

APIレスポンス内のメディアURLは相対パスです。完全なURLは以下のように構築します:

```
Full URL = "https://karotter.com" + mediaUrl
```

---

## Cloudflare Turnstile

登録時のBot対策に使用されます。

| プロパティ | 値 |
|----------|-------|
| サイトキー | `0x4AAAAAACujb-w-3YVWR1zA` |
| 必須エンドポイント | `POST /api/auth/register` |
| フィールド名 | `turnstileToken` |
