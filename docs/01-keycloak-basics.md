# Keycloak ハンズオン - 基礎編

## 目次
1. [Keycloakとは](#keycloakとは)
2. [Keycloakの主要機能](#keycloakの主要機能)
3. [Keycloakの重要な概念](#keycloakの重要な概念)
4. [環境構築](#環境構築)
5. [ステップバイステップ設定](#ステップバイステップ設定)

---

## Keycloakとは

**Keycloak（キークローク）** は、Red Hatが開発したオープンソースの認証・認可サーバーです。

### なぜKeycloakを使うのか？

**自分で認証システムを作ると大変なこと：**
- パスワードの安全な保管（ハッシュ化、ソルト）
- セッション管理
- パスワードリセット機能
- 2要素認証（2FA）
- ソーシャルログイン（Google、Facebook等）
- セキュリティ対策（ブルートフォース攻撃対策など）

**Keycloakを使うと：**
✅ これら全てが最初から用意されている
✅ 設定だけでOIDC/OAuth 2.0に対応
✅ 管理画面でGUIで設定可能
✅ 本番環境でも使える高いセキュリティ

---

## Keycloakの主要機能

### 1. 認証機能
- ユーザー名/パスワード認証
- ソーシャルログイン（Google、GitHub、Facebook等）
- LDAP/Active Directory連携
- 2要素認証（TOTP、SMS、メール）
- パスワードレス認証（WebAuthn）

### 2. 認可機能
- ロールベースアクセス制御（RBAC）
- 属性ベースアクセス制御（ABAC）
- グループ管理
- 細かい権限設定

### 3. プロトコル対応
- OpenID Connect（OIDC）
- OAuth 2.0
- SAML 2.0

### 4. 管理機能
- 管理コンソール（Webベース）
- ユーザー管理
- クライアント管理
- テーマカスタマイズ
- 監査ログ

---

## Keycloakの重要な概念

Keycloakには独自の用語があります。理解しておきましょう：

```
┌─────────────────────────────────────────────────────────────┐
│                     Keycloak の階層構造                       │
└─────────────────────────────────────────────────────────────┘

    ┌──────────────────────────────────────────────┐
    │         Keycloak Server (1つ)                │
    │  すべてを管理する最上位層                      │
    └────────────────┬─────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
         ↓                       ↓
    ┌─────────┐            ┌─────────┐
    │ Realm 1 │            │ Realm 2 │
    │(領域1)  │            │(領域2)  │
    └────┬────┘            └─────────┘
         │
         │  Realm = テナント（独立した空間）
         │  ・ユーザー、クライアント、設定が分離
         │  ・例: "会社A用"、"会社B用"
         │
         ├──→ Users (ユーザー)
         │     ・山田太郎
         │     ・佐藤花子
         │     ・鈴木一郎
         │
         ├──→ Clients (クライアントアプリ)
         │     ・my-web-app
         │     ・mobile-app
         │     ・api-service
         │
         ├──→ Roles (ロール/役割)
         │     ・admin (管理者)
         │     ・user (一般ユーザー)
         │     ・editor (編集者)
         │
         ├──→ Groups (グループ)
         │     ・営業部
         │     ・開発部
         │     ・管理部
         │
         └──→ Identity Providers
               ・Google
               ・GitHub
               ・Facebook
```

### 用語の詳細説明

#### 1. Realm（レルム）
- **意味**: 独立した管理領域（テナント）
- **例**: 会社ごと、サービスごとに別のRealmを作成
- **特徴**:
  - 各Realmは完全に分離されている
  - ユーザー、クライアント、設定は共有されない
  - デフォルトで「master」Realmが存在（管理用）

**使い分けの例：**
```
master Realm          → Keycloak自体の管理用（触らない）
production Realm      → 本番環境用
development Realm     → 開発環境用
test Realm            → テスト環境用
```

#### 2. Client（クライアント）
- **意味**: あなたが作るアプリケーション
- **種類**:
  - **confidential**: サーバーサイドアプリ（秘密情報を安全に保管できる）
  - **public**: SPAやモバイルアプリ（秘密情報を保管できない）

```
Client の例：
┌─────────────────────────────┐
│ Client: my-web-app          │
│ Type: confidential          │
│ Client ID: my-web-app       │
│ Client Secret: xxx-secret   │
│ Root URL: http://localhost:3000│
│ Redirect URIs:              │
│   - http://localhost:3000/callback│
└─────────────────────────────┘
```

#### 3. User（ユーザー）
- **意味**: 実際にログインする人
- **属性**:
  - Username（ユーザー名）
  - Email（メールアドレス）
  - First Name / Last Name（名前）
  - カスタム属性（会社、部署など）

#### 4. Role（ロール）
- **意味**: 権限や役割を表す
- **種類**:
  - **Realm Role**: Realm全体で共通の役割
  - **Client Role**: 特定のClientに紐づく役割

```
ロールの例：
Realm Roles:
  - admin        (システム全体の管理者)
  - user         (一般ユーザー)

Client Roles (my-web-app):
  - editor       (記事編集権限)
  - viewer       (閲覧のみ)
```

#### 5. Group（グループ）
- **意味**: ユーザーをまとめる単位
- **使い方**: 部署やチームごとにグループを作り、まとめてロールを割り当て

```
グループの例：
/営業部
  ├─ /営業1課
  └─ /営業2課
/開発部
  ├─ /フロントエンド
  └─ /バックエンド
```

#### 6. Identity Provider（IdP）
- **意味**: 外部の認証サービス
- **例**: Google、GitHub、Facebook
- **使い方**: 「Googleでログイン」機能を簡単に実装できる

---

## Keycloakのデータフロー（内部構造）

Keycloakがどのようにデータを管理しているか見てみましょう：

```
┌──────────────────────────────────────────────────────────┐
│                  Keycloak Internal Architecture           │
└──────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────┐
    │       Management Console (Admin UI)     │
    │       http://localhost:8080/admin       │
    └───────────────────┬─────────────────────┘
                        │
                        ↓
    ┌─────────────────────────────────────────┐
    │         Keycloak Core Engine            │
    │  ・認証処理                               │
    │  ・トークン生成・検証                      │
    │  ・セッション管理                          │
    │  ・ポリシー実行                           │
    └───────────────────┬─────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ↓               ↓               ↓
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Database   │  │   Cache     │  │  Session    │
│  (H2/       │  │  (Infinispan│  │  Store      │
│  PostgreSQL)│  │  /Redis)    │  │             │
└─────────────┘  └─────────────┘  └─────────────┘
│               │               │
│ 保存データ:    │ キャッシュ:    │ セッション情報: │
│ ・Users       │ ・Realm設定   │ ・ログイン状態  │
│ ・Clients     │ ・User情報    │ ・アクティブ    │
│ ・Realms      │ ・Token検証   │  セッション    │
│ ・Roles       │  結果         │               │
│ ・Groups      │               │               │
└─────────────┘  └─────────────┘  └─────────────┘
```

### 主要エンドポイント

Keycloakが提供するエンドポイント：

```
1. 管理コンソール
   http://localhost:8080/admin
   → GUI で設定を行う

2. Realm のベースURL
   http://localhost:8080/realms/{realm-name}

3. OpenID Connect Discovery
   http://localhost:8080/realms/{realm-name}/.well-known/openid-configuration
   → OIDCの設定情報を取得

4. Authorization Endpoint (認可エンドポイント)
   http://localhost:8080/realms/{realm-name}/protocol/openid-connect/auth
   → ユーザー認証を開始

5. Token Endpoint (トークンエンドポイント)
   http://localhost:8080/realms/{realm-name}/protocol/openid-connect/token
   → トークンを取得

6. UserInfo Endpoint
   http://localhost:8080/realms/{realm-name}/protocol/openid-connect/userinfo
   → ユーザー情報を取得

7. Logout Endpoint
   http://localhost:8080/realms/{realm-name}/protocol/openid-connect/logout
   → ログアウト
```

---

## 環境構築

次のステップで実際にKeycloakを起動します！

### 必要なもの
- Docker
- Docker Compose

### これから行うこと
1. Docker Composeファイルを作成
2. Keycloakを起動
3. 管理画面にログイン
4. Realmを作成
5. ユーザーを作成
6. クライアントを登録

---

次の資料「02-keycloak-setup.md」で実際の構築手順を説明します！
