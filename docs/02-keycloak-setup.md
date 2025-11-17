# Keycloak ハンズオン - セットアップ編

## このドキュメントで行うこと

1. ✅ Keycloakの起動
2. ✅ 管理画面へのログイン
3. ✅ Realmの作成
4. ✅ ユーザーの作成
5. ✅ クライアントの登録
6. ✅ 動作確認

所要時間: **約15分**

---

## ステップ1: Keycloakの起動

### 1-1. Docker Composeファイルの確認

プロジェクトルートに `docker-compose.yml` があることを確認してください。

```bash
ls -la docker-compose.yml
```

### 1-2. Keycloakの起動

以下のコマンドでKeycloakを起動します：

```bash
docker-compose up -d
```

**コマンドの説明:**
- `up`: サービスを起動
- `-d`: バックグラウンドで実行（デタッチモード）

**期待される出力:**
```
Creating network "oidc-demo_oidc-network" with driver "bridge"
Creating oidc-keycloak ... done
```

### 1-3. 起動確認

Keycloakが正常に起動しているか確認します：

```bash
docker-compose ps
```

**期待される出力:**
```
     Name                   Command               State           Ports
--------------------------------------------------------------------------------
oidc-keycloak   /opt/keycloak/bin/kc.sh ...   Up      0.0.0.0:8080->8080/tcp
```

### 1-4. ログの確認（オプション）

起動ログを見たい場合：

```bash
docker-compose logs -f keycloak
```

**起動完了のサイン:**
```
INFO  [io.quarkus] (main) Keycloak 23.0.0 on JVM (powered by Quarkus ...) started in 15.234s.
INFO  [io.quarkus] (main) Listening on: http://0.0.0.0:8080
```

`Ctrl + C` でログ表示を終了できます。

---

## ステップ2: 管理画面へのログイン

### 2-1. ブラウザで管理画面を開く

ブラウザで以下のURLにアクセスします：

```
http://localhost:8080/admin
```

### 2-2. ログイン

**ログイン情報:**
- Username: `admin`
- Password: `admin`

> ⚠️ **注意**: これは開発環境用の設定です。本番環境では強力なパスワードを設定してください。

### 2-3. 管理画面の確認

ログインに成功すると、Keycloak Administration Consoleが表示されます。

**画面の構成:**
```
┌─────────────────────────────────────────────────────────┐
│ Keycloak Administration Console                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  左サイドバー:                                           │
│  ├─ Realm Settings  (Realmの設定)                      │
│  ├─ Clients         (クライアント管理)                   │
│  ├─ Client scopes   (スコープ管理)                      │
│  ├─ Roles           (ロール管理)                        │
│  ├─ Users           (ユーザー管理)                       │
│  ├─ Groups          (グループ管理)                      │
│  ├─ Sessions        (セッション管理)                     │
│  └─ Events          (イベントログ)                       │
│                                                         │
│  上部:                                                  │
│  ├─ Realm選択ドロップダウン (現在: master)              │
│  └─ ユーザーメニュー (admin)                            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## ステップ3: 新しいRealmの作成

現在は `master` Realmにいますが、これは管理用です。アプリケーション用の新しいRealmを作成しましょう。

### 3-1. Realm作成画面を開く

1. 左上の **Realmドロップダウン**（"master"と表示されている部分）をクリック
2. **"Create Realm"** ボタンをクリック

### 3-2. Realmの設定

**入力項目:**

| 項目 | 値 | 説明 |
|------|-----|------|
| Realm name | `myrealm` | Realmの識別名（英数字のみ） |
| Enabled | ✅ ON | Realmを有効化 |

### 3-3. Realmを作成

**"Create"** ボタンをクリックします。

**成功メッセージ:**
```
Realm created
```

### 3-4. 確認

左上のRealmドロップダウンが `myrealm` に変わっていることを確認してください。

---

## ステップ4: ユーザーの作成

テスト用のユーザーを作成しましょう。

### 4-1. ユーザー管理画面を開く

1. 左サイドバーの **"Users"** をクリック
2. **"Create new user"** ボタンをクリック

### 4-2. ユーザー情報の入力

**入力項目:**

| 項目 | 値 | 説明 |
|------|-----|------|
| Username | `testuser` | ログインに使用するユーザー名 |
| Email | `testuser@example.com` | メールアドレス |
| Email verified | ✅ ON | メール確認済みにする |
| First name | `太郎` | 名前 |
| Last name | `テスト` | 苗字 |

### 4-3. ユーザーを作成

**"Create"** ボタンをクリックします。

### 4-4. パスワードの設定

ユーザーが作成されると、詳細画面が表示されます。

1. 上部のタブから **"Credentials"** タブをクリック
2. **"Set password"** ボタンをクリック
3. パスワード設定画面が表示されます：

| 項目 | 値 |
|------|-----|
| Password | `password` |
| Password confirmation | `password` |
| Temporary | ❌ OFF（一時的なパスワードにしない） |

4. **"Save"** ボタンをクリック
5. 確認ダイアログが表示されたら **"Save password"** をクリック

**成功メッセージ:**
```
The password has been set.
```

---

## ステップ5: クライアントの登録

あなたのWebアプリケーション（クライアント）をKeycloakに登録します。

### 5-1. クライアント作成画面を開く

1. 左サイドバーの **"Clients"** をクリック
2. **"Create client"** ボタンをクリック

### 5-2. 一般設定（General Settings）

**Step 1/3: General Settings**

| 項目 | 値 | 説明 |
|------|-----|------|
| Client type | `OpenID Connect` | プロトコルを選択 |
| Client ID | `my-web-app` | クライアントの識別子 |
| Name | `My Web Application` | わかりやすい名前 |
| Description | `OIDCハンズオン用Webアプリ` | 説明（任意） |

**"Next"** をクリック

### 5-3. 機能設定（Capability config）

**Step 2/3: Capability config**

| 項目 | 値 | 説明 |
|------|-----|------|
| Client authentication | ✅ ON | 機密クライアント（サーバーサイドアプリ） |
| Authorization | ❌ OFF | 今回は不要 |
| Authentication flow | | |
| └ Standard flow | ✅ ON | Authorization Code Flow を有効化 |
| └ Direct access grants | ✅ ON | テスト用（本番では慎重に） |

**"Next"** をクリック

### 5-4. ログイン設定（Login settings）

**Step 3/3: Login settings**

| 項目 | 値 | 説明 |
|------|-----|------|
| Root URL | `http://localhost:3000` | アプリのベースURL |
| Home URL | `http://localhost:3000` | ホームページURL |
| Valid redirect URIs | `http://localhost:3000/callback` | 認証後のリダイレクト先（改行で複数可） |
| | `http://localhost:3000/*` | |
| Valid post logout redirect URIs | `http://localhost:3000` | ログアウト後のリダイレクト先 |
| Web origins | `http://localhost:3000` | CORS許可するオリジン |

**"Save"** をクリック

### 5-5. Client Secretの確認

クライアントが作成されたら、Client Secretを確認します。

1. クライアント詳細画面で、上部のタブから **"Credentials"** タブをクリック
2. **"Client Secret"** の値が表示されています
3. この値を**コピーして保存**してください（後で使います）

```
例: dQw4w9WgXcQ4w9WgXcQ4w9WgXcQ4w9
```

> 💡 **重要**: Client Secretは絶対に外部に漏らさないでください！

---

## ステップ6: 設定の確認

### 6-1. OpenID Configuration の確認

Keycloakが正しく設定されているか、Discovery Endpointにアクセスして確認します。

ブラウザで以下のURLを開く：

```
http://localhost:8080/realms/myrealm/.well-known/openid-configuration
```

**期待される出力（JSON）:**
```json
{
  "issuer": "http://localhost:8080/realms/myrealm",
  "authorization_endpoint": "http://localhost:8080/realms/myrealm/protocol/openid-connect/auth",
  "token_endpoint": "http://localhost:8080/realms/myrealm/protocol/openid-connect/token",
  "userinfo_endpoint": "http://localhost:8080/realms/myrealm/protocol/openid-connect/userinfo",
  "end_session_endpoint": "http://localhost:8080/realms/myrealm/protocol/openid-connect/logout",
  ...
}
```

これらのエンドポイントを後でクライアントアプリから使用します。

### 6-2. 設定サマリー

以下の情報を確認してメモしておきましょう：

```
【Realm情報】
Realm名: myrealm
Issuer: http://localhost:8080/realms/myrealm

【ユーザー情報】
Username: testuser
Password: password

【クライアント情報】
Client ID: my-web-app
Client Secret: (あなたがコピーした値)

【エンドポイント】
Authorization: http://localhost:8080/realms/myrealm/protocol/openid-connect/auth
Token: http://localhost:8080/realms/myrealm/protocol/openid-connect/token
UserInfo: http://localhost:8080/realms/myrealm/protocol/openid-connect/userinfo
Logout: http://localhost:8080/realms/myrealm/protocol/openid-connect/logout
```

---

## ステップ7: 動作確認（アカウント画面）

ユーザーの視点から、実際にKeycloakのアカウント画面にアクセスしてみましょう。

### 7-1. アカウント画面にアクセス

ブラウザで以下のURLを開く：

```
http://localhost:8080/realms/myrealm/account
```

### 7-2. ログイン

先ほど作成したユーザー情報でログイン：

- Username: `testuser`
- Password: `password`

### 7-3. アカウント画面を確認

ログインに成功すると、ユーザーのアカウント管理画面が表示されます。

**表示される情報:**
- Personal info（個人情報）
- Account security（セキュリティ設定）
- Applications（連携アプリ）
- Resources（リソース）

これでKeycloakのセットアップは完了です！🎉

---

## トラブルシューティング

### Keycloakが起動しない

**症状:** `docker-compose up -d` でエラーが出る

**確認:**
```bash
# ポート8080が既に使われていないか確認
lsof -i :8080

# または
netstat -an | grep 8080
```

**解決策:**
他のサービスがポート8080を使用している場合、`docker-compose.yml` のポート番号を変更：
```yaml
ports:
  - "8081:8080"  # 8081に変更
```

### 管理画面にアクセスできない

**確認:**
```bash
# Keycloakのログを確認
docker-compose logs keycloak

# Keycloakが起動しているか確認
docker-compose ps
```

### パスワードを忘れた

**管理者パスワードをリセット:**
```bash
# コンテナを削除して再作成
docker-compose down
rm -rf keycloak-data
docker-compose up -d
```

---

## 次のステップ

✅ Keycloakのセットアップが完了しました！

次は実際にクライアントアプリケーションを作成して、Keycloakと連携させます。

**次のドキュメント: 03-client-app.md**
