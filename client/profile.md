# Profile API
認証: 必要 (Authorization ヘッダーにトークンを設定)

## エンドポイント一覧
| メソッド | パス                        | 説明                     |
| -------- | --------------------------- | ------------------------ |
| POST     | /api/v2/profile/icon        | アイコン画像の更新       |
| POST     | /api/v2/profile/nickName    | ニックネームの更新       |
| POST     | /api/v2/profile/description | プロフィール説明の更新   |
| POST     | /api/v2/profile/password    | パスワードの変更         |

### 共通ヘッダー
- cookie sessionid:"token"

---
　
## POST /api/v2/profile/icon
説明: プロフィールアイコン画像を更新します。

リクエストボディ (application/json):
```json
{
  "icon": "<base64エンコード画像データ>"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (401)
  ```json
  { "message": "Unauthorized" }
  ```

---

## POST /api/v2/profile/nickName
説明: ニックネームを更新します。

リクエストボディ:
```json
{
  "nickName": "新しいニックネーム"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (401)
  ```json
  { "message": "Unauthorized" }
  ```

---

## POST /api/v2/profile/description
説明: プロフィール説明を更新します。

リクエストボディ:
```json
{
  "description": "新しい説明文"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (401)
  ```json
  { "message": "Unauthorized" }
  ```

---

## POST /api/v2/profile/password
説明: パスワードを変更します。

リクエストボディ:
```json
{
  "password": "新しいパスワード",
  "oldPassword": "現在のパスワード"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (401)
  ```json
  { "message": "Unauthorized" }
  ```
