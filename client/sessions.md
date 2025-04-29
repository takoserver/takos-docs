# Sessions API

認証: 必要（Cookie に `sessionid` を設定、一部エンドポイントを除く）

## エンドポイント一覧
| メソッド | パス                           | 説明                                        | 認証     |
| -------- | ------------------------------ | ------------------------------------------- | -------- |
| POST     | /api/v2/sessions/register/temp | 仮登録（メールアドレス確認コード送信）      | 不要     |
| POST     | /api/v2/sessions/register/check| メールアドレス確認コード検証                | 不要     |
| POST     | /api/v2/sessions/register      | 本登録（ユーザー名、パスワード設定）        | 不要     |
| POST     | /api/v2/sessions/login         | ログインしてセッションを開始                | 不要     |
| POST     | /api/v2/sessions/logout        | ログアウトしてセッションを終了              | 必要     |
| GET      | /api/v2/sessions/status        | 現在のログイン／セットアップ状況を取得      | 必要(省略可)|
| GET      | /api/v2/sessions/list          | 自デバイスのセッション一覧を取得            | 必要     |
| POST     | /api/v2/sessions/delete/:uuid  | 指定セッション（デバイス）を削除          | 必要     |
| POST     | /api/v2/sessions/setUp         | 初回暗号化キー・プロフィール情報を登録      | 必要     |
| POST     | /api/v2/sessions/reset         | 暗号化キーをリセット                        | 必要     |
| POST     | /api/v2/sessions/encrypt/request | 暗号化キー移行リクエスト送信              | 必要     |
| POST     | /api/v2/sessions/encrypt/accept  | 暗号化キー移行リクエスト承認              | 必要     |
| POST     | /api/v2/sessions/encrypt/send    | 暗号化キー移行データ送信                  | 必要     |
| POST     | /api/v2/sessions/encrypt/success | 暗号化キー移行完了通知                    | 必要     |
　
---

## POST /api/v2/sessions/register/temp
説明: メールアドレスを受け取り、確認コードを送信して仮登録を行います。
認証: 不要
リクエスト (application/json):
```json
{
  "email": "user@example.com"
}
```
レスポンス:
- 200 OK
  ```json
  { "token": "<仮登録トークン>" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "This email is already in use" }
  ```

---

## POST /api/v2/sessions/register/check
説明: 仮登録トークンとメールで送信された確認コードを検証します。
認証: 不要
リクエスト (application/json):
```json
{
  "token": "<仮登録トークン>",
  "checkCode": "12345678"
}
```
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "Invalid token or checkCode" }
  ```
- 400 Bad Request (試行回数超過)
  ```json
  { "status": "error", "message": "You have tried too many times" }
  ```

---

## POST /api/v2/sessions/register
説明: 検証済みの仮登録トークン、ユーザー名、パスワードを受け取り、本登録を完了します。
認証: 不要
リクエスト (application/json):
```json
{
  "token": "<仮登録トークン>",
  "userName": "newuser",
  "password": "securepassword"
}
```
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "<エラー内容>" }
  ```

---

## POST /api/v2/sessions/login
リクエスト (application/json):
```json
{
  "userName": "alice",
  "password": "hunter2",
  "sessionUUID": "xxxxxxxx-xxxx-7xxx-xxxx-xxxxxxxxxxxx" // クライアントで生成したUUID v7
}
```
レスポンス:
- 200 OK
  ```json
  { "sessionid": "<セッションID>" } // Cookieにもセットされる
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "Invalid username or password" }
  ```

---

## POST /api/v2/sessions/logout
リクエスト: Cookie に `sessionid`
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "Invalid session" }
  ```

---

## GET /api/v2/sessions/status
リクエスト: Cookie に `sessionid`（省略可）
レスポンス:
```json
{
  "login": boolean, // ログイン状態か
  "setup": boolean, // 初期セットアップ（プロフ・暗号化キー登録）済みか
  "encrypted": boolean, // 現在のセッションが暗号化済みか
  "deviceKey": string | null, // デバイス固有キー（未セットアップ時のみ）
  "requests": [ // 受信したフレンドリクエスト等
    {
      "id": string,
      "type": string,
      "sender": string, // "user@domain"
      "query": any,
      "timestamp": string // ISO 8601
    }
  ],
  "friendInfo": [ // フレンドごとの最新メッセージID ["friendId", "messageId"]
    ["bob@example.com", "msg-uuid-1"],
    ["charlie@example.com", null]
  ],
  "groupInfo": [ // グループごとの最新メッセージID ["groupId", "messageId"]
    ["group1@example.com", "msg-uuid-2"]
  ],
  "updatedAccountKeys": [string] // 他デバイスから共有されたアカウントキーのハッシュリスト
}
```

---

## GET /api/v2/sessions/list
リクエスト: Cookie に `sessionid`
レスポンス:
- 200 OK
  ```json
  [
    {
      "uuid": "xxxxxxxx-xxxx-7xxx-xxxx-xxxxxxxxxxxx", // セッションUUID
      "encrypted": boolean, // 暗号化済みか
      "userAgent": string, // User Agent
      "shareKey": string, // 公開共有鍵
      "shareKeySign": string // 共有鍵の署名
    },
    ...
  ]
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "Invalid session" }
  ```

---

## POST /api/v2/sessions/delete/:uuid
パスパラメータ:
- `uuid`: 削除対象セッションのUUID
リクエスト: Cookie に `sessionid`
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "Invalid session" }
  ```

---

## POST /api/v2/sessions/setUp
初回暗号化セットアップ
リクエスト (application/json):
```json
{
  "masterKey": "<公開マスターキー>",
  "accountKey": "<公開アカウントキー>",
  "accountKeySign": "<アカウントキー署名>",
  "shareKey": "<公開共有キー>",
  "shareKeySign": "<共有キー署名>",
  "nickName": "Alice",
  "icon": "<base64エンコード画像>"
}
```
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "<エラー内容>" }
  ```

---

## POST /api/v2/sessions/reset
暗号化キー再登録（他デバイスは全て削除される）
リクエスト (application/json):
```json
{
  "masterKey": "<公開マスターキー>",
  "accountKey": "<公開アカウントキー>",
  "accountKeySign": "<アカウントキー署名>",
  "shareKey": "<公開共有キー>",
  "shareKeySign": "<共有キー署名>"
}
```
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "<エラー内容>" }
  ```

---

## POST /api/v2/sessions/encrypt/request
説明: 新しいデバイス（未暗号化セッション）から、既存の暗号化済みデバイスへキー移行をリクエストします。
リクエスト (application/json):
```json
{
  "migrateKey": "<移行用公開鍵>" // 新デバイスで生成
}
```
レスポンス:
- 200 OK
  ```json
  { "migrateid": "<移行ID>" } // このIDを使って他デバイスに通知される
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "Invalid session" } // セッションが無効 or 既に暗号化済み
  ```

---

## POST /api/v2/sessions/encrypt/accept
説明: 既存の暗号化済みデバイスが、新しいデバイスからのキー移行リクエストを承認します。
リクエスト (application/json):
```json
{
  "migrateid": "<移行ID>",
  "migrateSignKey": "<移行署名用公開鍵>" // 既存デバイスで生成
}
```
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "<エラー内容>" } // セッションが無効、migrateidが無効など
  ```

---

## POST /api/v2/sessions/encrypt/send
説明: 移行リクエストを承認した既存デバイスが、暗号化されたキーデータを新しいデバイスへ送信します。
リクエスト (application/json):
```json
{
  "migrateid": "<移行ID>",
  "data": "<暗号化されたキーデータ>", // migrateKeyで暗号化
  "sign": "<署名>" // migrateSignKeyで署名
}
```
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "<エラー内容>" } // セッションが無効、migrateidが無効、未承認など
  ```

---

## POST /api/v2/sessions/encrypt/success
説明: キーデータを受け取った新しいデバイスが、自身の共有鍵を登録し、移行完了を通知します。
リクエスト (application/json):
```json
{
  "shareKey": "<新デバイスの公開共有キー>",
  "shareKeySign": "<共有キー署名>" // マスターキーで署名
}
```
レスポンス:
- 200 OK
  ```json
  { "status": "success" }
  ```
- 400 Bad Request
  ```json
  { "status": "error", "message": "<エラー内容>" } // セッションが無効、署名が無効など
  ```
