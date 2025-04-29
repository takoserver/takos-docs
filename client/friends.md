# Friends API
認証: 必要 (Authorization ヘッダーにトークンを設定)

## エンドポイント一覧
| メソッド | パス                      | 説明                         |
| -------- | ------------------------- | ---------------------------- |
| POST     | /api/v2/friends/request   | フレンド申請を送信           |
| GET      | /api/v2/friends/requests  | 受信した申請一覧を取得       |
| POST     | /api/v2/friends/accept    | フレンド申請を承認           |
| GET      | /api/v2/friends/list      | フレンド一覧を取得           |

## リクエスト／レスポンス定義

### 共通ヘッダー
- cookie sessionid:"token"

---

## POST /api/v2/friends/request
説明: フレンド申請を送信します。

リクエストボディ (application/json):
```json
{
  "userName": "target@example.com"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "Request sent" }
  ```
- エラー (400/401)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## GET /api/v2/friends/requests
説明: 受信したフレンド申請の一覧を取得します。

クエリパラメータ:
- limit?: number (取得件数、デフォルト10)
- befor?: ISO 日時文字列 (指定した日時より古い申請を取得)

レスポンス (200):
```json
{
  "requests": [
    {
      "id": "request-id",
      "type": "friend",
      "sender": "alice@example.com",
      "receiver": "you@example.com",
      "local": true,
      "timestamp": "2024-06-01T12:00:00.000Z"
    },
    ...
  ]
}
```

---

## POST /api/v2/friends/accept
説明: フレンド申請を承認します。

リクエストボディ:
```json
{
  "id": "request-id"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "Request accepted" }
  ```
- エラー (400/401)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## GET /api/v2/friends/list
説明: フレンド一覧を取得します。

クエリパラメータ:
- befor?: ISO 日時文字列 (指定した日時より古いフレンドを取得)

レスポンス (200):
```json
[
  "bob@example.com",
  "charlie@example.com",
  ...
]
```
