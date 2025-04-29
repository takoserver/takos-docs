# Messages API
認証: 必要 (Authorization ヘッダーにトークンを設定)

## エンドポイント一覧
| メソッド | パス                           | 説明                                 |
| -------- | ------------------------------ | ------------------------------------ |
| POST     | /api/v2/messages/send          | メッセージ送信                       |
| GET      | /api/v2/messages/friend/:roomId| フレンド間メッセージ取得             |
| GET      | /api/v2/messages/group/:roomId/:channelId | グループチャンネルのメッセージ取得   |
| POST     | /api/v2/messages/delete        | メッセージ削除                       |
| POST     | /api/v2/messages/sync          | メッセージIDの同期                   |

---
　
## POST /api/v2/messages/send
説明: メッセージを送信します。

リクエストボディ (application/json):
```json
{
  "roomId": "m{bob}@example.com",
  "message": "{\"text\":\"こんにちは\"}",
  "sign": "署名文字列(省略可)",
  "type": "friend", // または "group"
  "channelId": "channel-id(グループ時必須)",
  "isEncrypted": false
}
```

レスポンス:
- 成功 (200)
  ```json
  { "messageId": "..." }
  ```
- エラー (400/401/403)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## GET /api/v2/messages/friend/:roomId
説明: フレンド間のメッセージ履歴を取得します。

クエリパラメータ:
- limit?: number (取得件数、デフォルト50)
- before?: ISO 日時文字列 (指定した日時より前のメッセージ)

レスポンス (200):
```json
{
  "messages": [
    {
      "messageid": "...",
      "timestamp": "2024-06-01T12:00:00.000Z",
      "userName": "alice@example.com"
    },
    ...
  ]
}
```

---

## GET /api/v2/messages/group/:roomId/:channelId
説明: グループチャンネルのメッセージ履歴を取得します。

クエリパラメータ:
- limit?: number (取得件数、デフォルト50)
- before?: ISO 日時文字列 (指定した日時より前のメッセージ)

レスポンス (200):
```json
{
  "messages": [
    {
      "messageid": "...",
      "timestamp": "2024-06-01T12:00:00.000Z",
      "userName": "bob@example.com"
    },
    ...
  ]
}
```

---

## POST /api/v2/messages/delete
説明: 自分のメッセージを削除します。

リクエストボディ:
```json
{
  "messageId": "..."
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "Deleted" }
  ```
- エラー (401/404)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/messages/sync
説明: 各ルームの未取得メッセージID一覧を取得します。

リクエストボディ:
```json
[
  { "roomId": "m{bob}@example.com", "time": "2024-06-01T00:00:00.000Z" },
  { "roomId": "g{group1}@example.com", "time": "2024-06-01T00:00:00.000Z" }
]
```

レスポンス (200):
```json
[
  {
    "id": "m{bob}@example.com",
    "messages": ["msgid1", "msgid2", ...]
  },
  {
    "id": "g{group1}@example.com",
    "messages": ["msgid3", ...]
  }
]
```
