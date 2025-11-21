# 認証認可の歴史と最新トレンド

## 目次
1. [認証認可の歴史](#認証認可の歴史)
2. [OAuthの誕生と進化](#oauthの誕生と進化)
3. [OpenID Connectの誕生](#openid-connectの誕生)
4. [最新トレンド](#最新トレンド)
5. [今後の展望](#今後の展望)

---

## 認証認可の歴史

### 1990年代〜2000年代初期: カオスの時代

#### 問題1: アカウント爆発

```
ユーザーの悪夢：

Twitter        → user1 / password123
Facebook       → user1 / password456
Google         → user1@gmail.com / password789
Amazon         → user1@gmail.com / amazonpass
LinkedIn       → user1_linkedin / pass999
...

結果：
❌ 100個以上のサービス = 100個のID/パスワード
❌ パスワード使い回し → セキュリティリスク大
❌ パスワード忘れる → リセット地獄
```

#### 問題2: パスワード共有問題

**実例: Twitter連携アプリの恐怖（2007年頃）**

```
【悪い例】初期のTwitter連携アプリ

ユーザー: 「Twitter連携アプリを使いたい」
         ↓
アプリ:   「Twitterのユーザー名とパスワードを入力してください」
         ↓
ユーザー: username: myaccount
         password: mypassword ← これをアプリに渡す！
         ↓
アプリ:   Twitterにログインして、ツイートを投稿...

【何が問題？】
❌ アプリに自分のパスワードを渡している
❌ アプリはユーザーのアカウントで何でもできる
❌ パスワードを変更しない限り、アクセスを取り消せない
❌ どのアプリに渡したか分からなくなる
```

**実際に起きた問題：**
- 悪意あるアプリがパスワードを保存
- ユーザーのアカウントが乗っ取られる
- スパムツイートが大量投稿される
- パスワードが漏洩しても気づけない

#### 問題3: シングルサインオン（SSO）の不在

```
企業内の状況（2000年代）:

社員がログインしなければならないシステム：
- 人事システム
- 経費精算システム
- メールシステム
- ファイルサーバー
- 顧客管理システム
- プロジェクト管理ツール
...

毎朝の儀式：
1. PC起動
2. 6つのシステムにそれぞれログイン（各5分）
3. 30分後、やっと仕事開始...

❌ 生産性の低下
❌ IT部門のサポートコスト増大
❌ セキュリティリスク（付箋にパスワードメモ）
```

---

## OAuthの誕生と進化

### OAuth 1.0の誕生（2007年）

#### 背景: Twitterの苦悩

2006年、Twitterは急成長していましたが、大きな問題を抱えていました。

```
【課題】
サードパーティアプリがTwitter APIを使いたい
    ↓
でも、ユーザーのパスワードを渡すのは危険
    ↓
でも、各アプリ専用のパスワードを作るのは面倒
    ↓
解決策は？
```

**2007年12月: OAuth 1.0誕生**

TwitterのBlaine Cook、GoogleのChris Messina、そして他のエンジニアたちが集まり、
新しい認可プロトコル「OAuth」を開発しました。

#### OAuth 1.0の仕組み

```
【解決した問題】

従来:
  ユーザー → アプリにパスワードを渡す → アプリが何でもできる

OAuth 1.0:
  ユーザー → Twitterで認可 → アプリに限定的なトークンを発行
                                → アプリは許可された操作だけできる

【メリット】
✅ パスワードを渡さなくて良い
✅ アプリの権限を制限できる（読み取りのみ、など）
✅ いつでもアクセスを取り消せる
✅ どのアプリが連携しているか確認できる
```

#### OAuth 1.0の問題点

しかし、OAuth 1.0には致命的な問題がありました：

```
【技術的な複雑さ】

1. 署名アルゴリズムが複雑
   - HMAC-SHA1署名が必須
   - リクエストごとに署名を計算
   - 実装が難しく、バグの温床

2. モバイルアプリに不向き
   - クライアントシークレットを安全に保管できない
   - ネイティブアプリでの実装が困難

3. JavaScriptアプリで使えない
   - SPAは存在すらしていなかった時代

例: OAuth 1.0の署名生成（複雑すぎる...）
--------------------------------------------
Base String = HTTP_METHOD + "&" +
              URL_encode(base_uri) + "&" +
              URL_encode(sorted_parameters)

Signature = HMAC-SHA1(Base String, signing_key)
```

### OAuth 2.0の登場（2012年）

#### なぜOAuth 2.0が必要だったのか？

2010年頃、Webの世界は大きく変化していました：

```
【2010年頃の状況】

1. モバイルの台頭
   - iPhone (2007), Android (2008) が普及
   - ネイティブアプリが増加

2. SPAの登場
   - JavaScript が高機能化
   - フロントエンドとバックエンドの分離

3. APIエコノミーの成長
   - RESTful APIが標準に
   - マイクロサービスアーキテクチャ

4. クラウドサービスの普及
   - AWS, Azure, GCP
   - SaaSの急増

→ OAuth 1.0では対応できない！
```

**2012年10月: OAuth 2.0 (RFC 6749) 公開**

#### OAuth 2.0の改善点

```
【主な変更点】

1. 署名を廃止、HTTPSに依存
   OAuth 1.0: 複雑な署名アルゴリズム
   OAuth 2.0: HTTPS通信で安全性を確保（シンプル！）

2. 複数のグラントタイプ（フロー）
   - Authorization Code Flow      (サーバーサイドアプリ)
   - Implicit Flow                (SPA - 現在は非推奨)
   - Resource Owner Password Flow (レガシーシステム移行用)
   - Client Credentials Flow      (サーバー間通信)

3. トークンの種類を明確化
   - Access Token  (短期間有効)
   - Refresh Token (長期間有効)

4. Bearer Token の導入
   Authorization: Bearer <access_token>
   → シンプルで使いやすい！

5. スコープの標準化
   scope=read,write
   → 細かい権限制御が可能
```

#### OAuth 2.0のフロー図（代表例）

```
【Authorization Code Flow】

1. アプリがユーザーをOAuthサーバーにリダイレクト
   GET /authorize?
     response_type=code&
     client_id=APP_ID&
     redirect_uri=https://app.example/callback&
     scope=read,write

2. ユーザーがログイン・同意

3. 認可コードがアプリにリダイレクトで返される
   https://app.example/callback?code=AUTH_CODE

4. アプリがトークンエンドポイントで認可コードを交換
   POST /token
   code=AUTH_CODE&
   client_id=APP_ID&
   client_secret=SECRET

5. アクセストークンを取得
   {
     "access_token": "...",
     "token_type": "Bearer",
     "expires_in": 3600
   }
```

### OAuth 2.0の問題: 認証には使えない

**重要な誤解:**

```
❌ 間違い: 「OAuthでログインできる」
⭕ 正解:   「OAuthは認可のためのもので、認証ではない」

【何が問題？】

OAuth 2.0で取得できるのは：
  "access_token": "xyz123..."

でも、これだけでは分からない：
  - このトークンは誰のもの？
  - ユーザーの名前は？
  - メールアドレスは？
  - いつログインした？

各社が独自実装で対応：
  - Facebook: /me エンドポイント
  - Google: /userinfo エンドポイント
  - Twitter: /account/verify_credentials

→ 標準がない！各社バラバラ！
```

---

## OpenID Connectの誕生

### 背景: 認証の標準化の必要性（2012年〜）

```
【2012年頃の混乱】

問題1: 各社独自のユーザー情報API
-------------------------------
Facebook  → GET /me
Google    → GET /oauth2/v1/userinfo
Twitter   → GET /account/verify_credentials
GitHub    → GET /user

→ アプリ開発者は各サービスごとに実装が必要...

問題2: セキュリティリスク
-------------------------
OAuth 2.0のaccess tokenだけでは認証に不十分
→ 各社が独自の方法で補完
→ セキュリティホールが生まれやすい

問題3: ID連携の複雑さ
----------------------
「Googleでログイン」を実装するのに
各社が車輪の再発明...
```

### OpenID Connect 1.0の誕生（2014年）

**2014年2月: OpenID Connect 1.0 仕様確定**

#### 開発者たちの野望

```
「OAuth 2.0の上に、標準的な認証レイヤーを作ろう！」

目標：
✅ OAuth 2.0をベースにする（車輪の再発明をしない）
✅ IDトークンという標準形式を定義
✅ ユーザー情報の取得方法を標準化
✅ 既存のOAuthサーバーを簡単にOIDCに対応可能に
```

#### OpenID Connectの革新

```
【1. IDトークンの導入】

JWT (JSON Web Token) 形式の IDトークン:
{
  "iss": "https://accounts.google.com",      // 発行者
  "sub": "123456789",                        // ユーザーID
  "aud": "client_id_here",                   // 対象アプリ
  "exp": 1516239022,                         // 有効期限
  "iat": 1516239022,                         // 発行時刻
  "name": "山田太郎",                         // 名前
  "email": "yamada@example.com",             // メール
  "email_verified": true                     // メール検証済み
}

これで一気に分かる：
✅ 誰が (sub)
✅ どこで (iss)
✅ いつ (iat)
✅ ログインしたか！

【2. 標準エンドポイント】

/.well-known/openid-configuration
→ すべての設定情報をここで取得できる！

/authorize  → 認証開始
/token      → トークン取得
/userinfo   → ユーザー情報取得

【3. スコープの標準化】

openid          → OIDC を使うという宣言（必須）
profile         → 名前、写真など
email           → メールアドレス
address         → 住所
phone           → 電話番号

【4. 署名と検証】

IDトークンはJWTで署名されている
→ 改ざんを検知できる
→ セキュアな認証が可能
```

### OIDCの採用状況

```
【主要サービスの対応】

2014年〜:
✅ Google      (いち早く対応)
✅ Microsoft   (Azure AD)
✅ Yahoo
✅ PayPal

2015年〜:
✅ Amazon Cognito
✅ Auth0
✅ Okta

2016年〜:
✅ Salesforce
✅ GitHub (後に対応)
✅ Apple (2019年、Sign in with Apple)

現在:
ほぼすべての主要IdPがOIDCをサポート
```

---

## 最新トレンド

### 1. パスワードレス認証の台頭（2018年〜）

#### なぜパスワードレスが必要？

```
【パスワードの問題点】

統計データ:
❌ 平均的なユーザーは100個以上のオンラインアカウントを持つ
❌ 51%の人が同じパスワードを複数サイトで使い回し
❌ データ漏洩の81%はパスワードの脆弱性が原因
❌ パスワードリセットのサポートコストは年間70ドル/ユーザー

有名な漏洩事件:
- 2013 Adobe:        1.5億アカウント
- 2014 eBay:         1.45億ユーザー
- 2016 Dropbox:      6800万アカウント
- 2019 Facebook:     5.3億ユーザー
- 2021 LinkedIn:     7億ユーザー

→ パスワードは根本的に危険！
```

#### WebAuthn / FIDO2（2019年〜）

**W3CとFIDO Allianceが標準化**

```
【WebAuthnの仕組み】

従来:
  パスワード入力 → サーバーに送信 → 検証

WebAuthn:
  1. 生体認証 or セキュリティキー
     (指紋、顔認証、YubiKeyなど)
  2. デバイス内で公開鍵暗号を使用
  3. 秘密鍵はデバイスから出ない
  4. チャレンジ・レスポンス認証

【メリット】
✅ フィッシング攻撃に強い（ドメイン紐付け）
✅ パスワード不要
✅ 生体認証で便利
✅ セキュリティキー対応

【採用例】
- GitHub (2019年〜)
- Microsoft (Windows Hello)
- Google (Titan Security Key)
- Apple (Touch ID, Face ID)
- Dropbox
- Facebook
```

**実装例:**

```javascript
// WebAuthn登録
const credential = await navigator.credentials.create({
  publicKey: {
    challenge: new Uint8Array([/* サーバーからの challenge */]),
    rp: { name: "Example Corp" },
    user: {
      id: new Uint8Array([/* user id */]),
      name: "user@example.com",
      displayName: "山田太郎"
    },
    pubKeyCredParams: [{ type: "public-key", alg: -7 }]
  }
});

// 認証時
const assertion = await navigator.credentials.get({
  publicKey: {
    challenge: new Uint8Array([/* challenge */]),
    allowCredentials: [{
      type: "public-key",
      id: credentialId
    }]
  }
});
```

### 2. マジックリンク認証（2015年〜）

```
【仕組み】

1. ユーザーがメールアドレスを入力
2. ワンタイムログインリンクをメール送信
3. ユーザーがリンクをクリック
4. 自動ログイン

【メリット】
✅ パスワード不要
✅ 実装が簡単
✅ メールアドレスの確認も同時にできる

【採用例】
- Slack
- Medium
- Notion (オプション)
- Substack
```

### 3. OTP (ワンタイムパスワード) 認証

```
【種類】

1. TOTP (Time-based OTP)
   - Google Authenticator
   - Authy
   - 30秒ごとに変わる6桁の数字

2. SMS OTP
   - 電話番号にコード送信
   - セキュリティ懸念あり（SIMスワップ攻撃）

3. Email OTP
   - メールにコード送信
```

### 4. ソーシャルログインの進化

```
【新しい波】

2019: Sign in with Apple
  - プライバシー重視
  - メールアドレスの隠蔽機能
  - iOSアプリでは必須化（他のソーシャルログインがある場合）

特徴:
✅ ランダムメールアドレス生成
   (例: abc123@privaterelay.appleid.com)
✅ いつでも連携解除可能
✅ アプリへの情報提供を最小限に
```

### 5. Zero Trust アーキテクチャ（2019年〜）

```
【従来のセキュリティモデル】

城壁モデル:
  - 外は危険、内は安全
  - 一度ログインすれば、社内システム全部アクセス可

問題:
❌ 内部犯行に弱い
❌ 一度侵入されたら終わり

【Zero Trust】

基本原則:
  「何も信頼するな。常に検証せよ」
  (Never Trust, Always Verify)

実装:
✅ すべてのリクエストを認証・認可
✅ マイクロセグメンテーション
✅ 最小権限の原則
✅ 継続的な検証

OIDCの役割:
  - トークンベース認証
  - きめ細かいスコープ制御
  - 短命なアクセストークン
  - 継続的な再認証
```

### 6. OAuth 2.1（策定中）

```
【背景】

OAuth 2.0の問題:
  - Implicit Flowは非推奨だが仕様に残っている
  - PKCE（ピクシー）はオプション
  - セキュリティベストプラクティスが分散

【OAuth 2.1の目標】

✅ セキュリティベストプラクティスを標準に
✅ 古い非推奨フローを削除
✅ PKCE を必須化
✅ Refresh Token Rotation を推奨
✅ より安全なデフォルト設定

【主な変更点】

1. Implicit Flow を削除
   → Authorization Code Flow + PKCE を使用

2. PKCE (Proof Key for Code Exchange) 必須化
   - すべてのクライアントでPKCEを使用
   - 認可コード横取り攻撃を防ぐ

3. Refresh Token Rotation
   - Refresh Token使用時に新しいTokenを発行
   - 古いTokenは無効化
   - セキュリティ向上

状況: ドラフト段階（2024年現在）
```

### 7. CIBA (Client Initiated Backchannel Authentication)

```
【シナリオ】

ATMでお金を引き出すとき:
  1. ATMにカードを入れる
  2. スマホに通知が来る
  3. スマホで承認
  4. ATMから現金が出る

【仕組み】

   Client          OP           User's Device
     │              │                │
     │─ Auth Req ──→│                │
     │              │─ Push通知 ───→│
     │              │                │
     │              │←── 承認 ───────│
     │←─ Token ─────│                │

【ユースケース】
- ATM
- スマートTV (YouTubeのデバイスログイン)
- IoTデバイス
- コールセンター

【採用例】
- YouTube (TVでログイン)
- Netflix
- 銀行ATM (一部)
```

### 8. DID (Decentralized Identifier) と Verifiable Credentials

```
【次世代の認証】

従来の問題:
  - 中央集権的なID管理
  - GoogleやFacebookに依存
  - プライバシー懸念

【Decentralized Identity】

ブロックチェーン技術を活用:
  ✅ 自己主権型アイデンティティ
  ✅ 中央管理者不要
  ✅ ユーザーが自分のIDを完全制御
  ✅ 選択的情報開示

仕組み:
  1. ユーザーがDIDを作成（例: did:example:123456）
  2. 資格情報をブロックチェーンに記録
  3. 検証可能な証明書を発行
  4. 必要な情報だけを選択的に開示

例:
  - 年齢確認: 「20歳以上である」だけを証明
    （生年月日は開示しない）

【進行状況】
- W3Cで標準化中
- Microsoft ION
- Hyperledger Indy
- Sovrin Network

実用化はまだ先...
```

### 9. Continuous Authentication（継続的認証）

```
【従来の認証】

ログイン時に1回認証 → セッション有効期間中は信頼

問題:
  - ログイン後にデバイス盗難
  - セッションハイジャック
  - なりすまし

【継続的認証】

常にユーザーを検証:
  ✅ 行動分析（タイピングパターン、マウス移動）
  ✅ 位置情報の変化
  ✅ デバイス特性
  ✅ ネットワーク環境

異常検知時:
  → 再認証を要求
  → セッション終了
  → アラート送信

【実装例】
- Google (異常なログイン検知)
- Microsoft (Azure AD Conditional Access)
- 銀行アプリ
```

### 10. Passkeys（2022年〜）

```
【Apple, Google, Microsoftの共同取り組み】

2022年5月発表: FIDO Alliance準拠のPasskeys

【特徴】

✅ パスワード不要
✅ デバイス間で同期（iCloud Keychain, Google Password Manager）
✅ 生体認証
✅ フィッシング攻撃に完全免疫
✅ クロスプラットフォーム

【仕組み】

登録:
  1. アプリ/サイトでPasskey作成
  2. 生体認証（Face ID, Touch IDなど）
  3. 公開鍵をサーバーに保存、秘密鍵をデバイスに保存
  4. iCloudなどで秘密鍵を同期（暗号化されている）

ログイン:
  1. サイトにアクセス
  2. Passkeyを選択
  3. 生体認証
  4. 完了！

【採用例】
- Apple (2022年 iOS 16, macOS Ventura)
- Google (2022年 Android, Chrome)
- Microsoft (2023年 Windows 11)
- GitHub (2023年)
- PayPal
- Shopify

2024年現在、急速に普及中！
```

---

## 今後の展望

### 短期（1〜2年）

```
1. Passkeysのさらなる普及
   - 主要Webサイトが順次対応
   - パスワードの段階的廃止

2. OAuth 2.1の標準化完了
   - セキュリティのさらなる向上

3. AI活用の認証
   - 異常検知の精度向上
   - ボット対策の高度化
```

### 中期（3〜5年）

```
1. パスワード認証の減少
   - Passkeys, WebAuthnが主流に
   - パスワードは例外的存在に

2. 生体認証の進化
   - より精度の高い認証
   - 複数の生体情報の組み合わせ

3. Zero Trustの標準化
   - すべてのシステムがZero Trust前提に
```

### 長期（5〜10年）

```
1. Decentralized Identityの実用化
   - ブロックチェーンベースのID管理
   - ユーザーが完全にIDを制御

2. 量子耐性暗号への移行
   - 量子コンピュータ対策
   - 現在の暗号方式の置き換え

3. AIによる完全自動化認証
   - 行動パターンだけで認証
   - ユーザーは意識せずに認証される
```

---

## まとめ: 認証の進化

```
【年表】

1990年代  ユーザー名/パスワードの時代
          ↓
2007年    OAuth 1.0 誕生
          ↓
2012年    OAuth 2.0 標準化
          ↓
2014年    OpenID Connect 1.0 標準化
          ↓
2019年    WebAuthn / FIDO2 標準化
          ↓
2022年    Passkeys 登場
          ↓
2024年    パスワードレスへの移行加速
          ↓
未来      Decentralized Identity？
```

### 学ぶべき技術（優先順位）

```
【今すぐ学ぶべき】
1. ⭐⭐⭐ OpenID Connect
2. ⭐⭐⭐ OAuth 2.0
3. ⭐⭐  WebAuthn / FIDO2
4. ⭐⭐  JWT
5. ⭐   Passkeys

【将来のために】
6. OAuth 2.1
7. CIBA
8. Decentralized Identity
```

---

次のステップ: 実際にKeycloakを使ってOIDCを体験してみましょう！

👉 **[01-keycloak-basics.md](01-keycloak-basics.md)**
