# Call API

## 共通ヘッダー
- Content-Type: application/json
- 認証: なし

---

## エンドポイント一覧
| メソッド | パス                                     | 説明                         |
| :------- | :--------------------------------------- | :--------------------------- |
| POST     | /api/v2/call/friends/audio/request      | 友だちに音声通話リクエストを送信 |
| POST     | /api/v2/call/friends/audio/accept       | 受信した音声通話リクエストを承諾 |
| POST     | /api/v2/call/friends/audio/reject       | 受信した音声通話リクエストを拒否 |
| POST     | /api/v2/call/friends/video/request      | 友だちにビデオ通話リクエストを送信 |
| POST     | /api/v2/call/friends/video/accept       | 受信したビデオ通話リクエストを承諾 |
| POST     | /api/v2/call/friends/video/reject       | 受信したビデオ通話リクエストを拒否 |
| POST     | /api/v2/call/group/audio/start          | グループ音声通話を開始         |
| POST     | /api/v2/call/group/audio/join           | グループ音声通話に参加         |
| POST     | /api/v2/call/group/video/start          | グループビデオ通話を開始       |
| POST     | /api/v2/call/group/video/join           | グループビデオ通話に参加       |

## リクエスト／レスポンス定義

### リクエストボディ
- friendId: string (友だちのID、メールアドレス)
- roomKeyHash: string (通話ルームのハッシュ)
- groupId: string (グループID)

### レスポンスボディ
- status: "ok" | "error"
- message?: string (エラー時のみ)
- token?: string (通話用トークン)
- groupId?: string (グループID)

## POST /api/v2/call/friends/audio/request

説明: 友だちに音声通話リクエストを送信します。

リクエストボディ:
```json
{
  "friendId": "friend@example.com",
  "roomKeyHash": "abcdef123456"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "status": "ok" }
  ```
- エラー (400)
  ```json
  { "status": "error", "message": "Invalid request" }
  ```

## POST /api/v2/call/friends/video/request

説明: 友だちにビデオ通話リクエストを送信します。

リクエストボディ:
```json
{
  "friendId": "friend@example.com",
  "roomKeyHash": "abcdef123456"
}
```

レスポンス:
```json
{ "status": "ok" }
```

## POST /api/v2/call/friends/audio/accept

説明: 受信した音声通話リクエストを承諾します。

リクエストボディ:
```json
{
  "friendId": "caller@example.com"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "status": "ok", "token": "<通話用トークン>" }
  ```

## POST /api/v2/call/friends/video/accept

説明: 受信したビデオ通話リクエストを承諾します。

リクエストボディ:
```json
{
  "friendId": "caller@example.com"
}
```

レスポンス:
```json
{ "status": "ok", "token": "<通話用トークン>" }
```

## POST /api/v2/call/friends/audio/reject

説明: 受信した音声通話リクエストを拒否します。

リクエストボディ:
```json
{
  "friendId": "caller@example.com"
}
```

レスポンス:
```json
{ "status": "ok" }
```

## POST /api/v2/call/friends/video/reject

説明: 受信したビデオ通話リクエストを拒否します。

リクエストボディ:
```json
{
  "friendId": "caller@example.com"
}
```

レスポンス:
```json
{ "status": "ok" }
```

## グループ通話

### POST /api/v2/call/group/audio/start

説明: グループ音声通話を開始します。  
リクエストボディ:
```json
{
  "roomKeyHash": "abcdef123456",
  "groupId": "group123"
}
```
レスポンス (200):
```json
{
  "status": "ok",
  "groupId": "group123",
  "token": "<通話用トークン>"
}
```

### POST /api/v2/call/group/audio/join

説明: 開始済みのグループ音声通話に参加します。  
リクエストボディ:
```json
{
  "groupId": "group123"
}
```
レスポンス (200):
```json
{
  "status": "ok",
  "token": "<参加用トークン>"
}
```

### POST /api/v2/call/group/video/start

説明: グループビデオ通話を開始します。  
リクエストボディ:
```json
{
  "roomKeyHash": "abcdef123456",
  "groupId": "group123"
}
```
レスポンス (200):
```json
{
  "status": "ok",
  "token": "<通話用トークン>"
}
```

### POST /api/v2/call/group/video/join

説明: 開始済みのグループビデオ通話に参加します。  
リクエストボディ:
```json
{
  "groupId": "group123"
}
```
レスポンス (200):
```json
{
  "status": "ok",
  "token": "<参加用トークン>"
}
```