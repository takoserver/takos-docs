# Keys API
認証: 必要 (Authorization ヘッダーにトークンを設定)

## エンドポイント一覧

| メソッド | パス                         | 説明                                 |
|----------|------------------------------|--------------------------------------|
| GET      | /api/v2/keys/deviceKey       | デバイスキー取得                     |
| POST     | /api/v2/keys/deviceKey       | デバイスキー生成                     |
| POST     | /api/v2/keys/accountKey      | アカウントキー登録                   |
| GET      | /api/v2/keys/accountKey      | アカウントキー取得                   |
| POST     | /api/v2/keys/accountKey/notify | アカウントキー通知削除             |
| POST     | /api/v2/keys/identityKey     | アイデンティティキー登録             |
| POST     | /api/v2/keys/shareSignKey    | 署名共有鍵登録                       |
| GET      | /api/v2/keys/shareSignKey    | 署名共有鍵取得                       |
| POST     | /api/v2/keys/roomKey         | ルーム鍵登録                         |
　
---

## GET /api/v2/keys/deviceKey
説明: 現在のセッションのデバイスキーを取得します。

レスポンス:
```json
{
  "deviceKey": "<デバイスキー文字列>"
}
```

---

## POST /api/v2/keys/deviceKey
説明: 新しいデバイスキーを生成し、セッションに保存します。

レスポンス:
```json
{
  "deviceKey": "<新しいデバイスキー文字列>"
}
```

---

## POST /api/v2/keys/accountKey
説明: アカウントキーと暗号化済みアカウントキー群を登録します。

リクエストボディ:
```json
{
  "accountKey": "<アカウントキー文字列>",
  "accountKeySign": "<署名文字列>",
  "encryptedAccountKeys": [
    ["<sessionUUID>", "<暗号化済みアカウントキー>"],
    ...
  ],
  "shareDataSign": "<共有データ署名>"
}
```

レスポンス:
```json
{ "message": "success" }
```

---

## GET /api/v2/keys/accountKey
説明: 指定したハッシュのアカウントキーを取得します。

クエリパラメータ:
- hash: string (アカウントキーのハッシュ)

レスポンス:
```json
{
  "accountKey": "<暗号化済みアカウントキー>",
  "shareDataSign": "<共有データ署名>"
}
```

---

## POST /api/v2/keys/accountKey/notify
説明: アカウントキーの通知を削除します。

リクエストボディ:
```json
{
  "hash": "<アカウントキーのハッシュ>"
}
```

レスポンス:
```json
{ "message": "success" }
```

---

## POST /api/v2/keys/identityKey
説明: アイデンティティキーを登録します。

リクエストボディ:
```json
{
  "identityKey": "<アイデンティティキー文字列>",
  "identityKeySign": "<署名文字列>"
}
```

レスポンス:
```json
{ "message": "success" }
```

---

## POST /api/v2/keys/shareSignKey
説明: 署名共有鍵を登録します。

リクエストボディ:
```json
{
  "shareSignKey": "<署名共有鍵文字列>",
  "shareSignKeySign": "<署名文字列>"
}
```

レスポンス:
```json
{ "message": "success" }
```

---

## GET /api/v2/keys/shareSignKey
説明: 指定したハッシュの署名共有鍵を取得します。

クエリパラメータ:
- hash: string (署名共有鍵のハッシュ)

レスポンス:
```json
{
  "shareSignKey": "<署名共有鍵文字列>",
  "sign": "<署名文字列>"
}
```

---

## POST /api/v2/keys/roomKey
説明: ルーム鍵を登録します。

リクエストボディ:
```json
{
  "roomId": "<ルームID>",
  "encryptedRoomKeys": [
    ["<userName>", "<暗号化済みルームキー>"],
    ...
  ],
  "hash": "<ルームキーのハッシュ>",
  "metaData": "<メタデータ文字列>",
  "sign": "<署名文字列>",
  "type": "friend" | "group"
}
```

レスポンス:
```json
{ "message": "success" }
```
