# SNS API
認証: 必要 (Authorization ヘッダーにトークンを設定)

## エンドポイント一覧
| メソッド | パス                               | 説明                     |
| -------- | ---------------------------------- | ------------------------ |
| POST     | /api/v2/sns/posts                  | 新規投稿を作成           |
| GET      | /api/v2/sns/posts/:userId/:postId  | 投稿を取得               |
| DELETE   | /api/v2/sns/posts/:postId          | 投稿を削除               |
| POST     | /api/v2/sns/posts/:userId/:postId/like | 投稿にいいねする         |
| DELETE   | /api/v2/sns/posts/:userId/:postId/like | 投稿のいいねを解除する   |
| POST     | /api/v2/sns/stories                | ストーリーを投稿         |
| GET      | /api/v2/sns/timeline               | タイムラインを取得       |

## リクエスト／レスポンス定義

### 共通ヘッダー
- cookie sessionid:"token"
　
---

## POST /api/v2/sns/posts
説明: 新規投稿を作成します。

リクエストボディ (application/json):
```json
{
  "text": "投稿内容 (1-256文字)",
  "media": [
    "data:image/jpeg;base64,...", // Base64エンコードされた画像データ (JPEG, PNG, WEBP, 各15MBまで, 最大4枚)
    // ...
  ]
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "Success" } // 仮の成功レスポンス。実際の実装に合わせる必要あり
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## GET /api/v2/sns/posts/:userId/:postId
説明: 指定された投稿を取得します。

パスパラメータ:
- userId: string (投稿者のユーザーID)
- postId: string (投稿のID)

レスポンス (200):
```json
{
  // MessageModel に基づく投稿オブジェクト
  "id": "https://<domain>/u/<userId>/<postId>",
  "type": "Note",
  "username": "<userId>",
  "actor": "https://<domain>/u/<userId>",
  "body": "投稿内容",
  "attachment": [
    "https://<domain>/media/<uuid>",
    // ...
  ],
  "createdAt": "2024-06-01T12:00:00.000Z",
  "updatedAt": "2024-06-01T12:00:00.000Z",
  "isRemote": false,
  "originalId": null,
  "url": null
  // ... その他 MessageModel のフィールド
}
```
- エラー (400/404)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## DELETE /api/v2/sns/posts/:postId
説明: 自分の投稿を削除します。

パスパラメータ:
- postId: string (削除する投稿のID)

レスポンス:
- 成功 (200)
  ```json
  { "message": "Success" } // 仮の成功レスポンス。実際の実装に合わせる必要あり
  ```
- エラー (400/401/404)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/sns/posts/:userId/:postId/like
説明: 指定された投稿にいいねします。

パスパラメータ:
- userId: string (投稿者のユーザーID)
- postId: string (いいねする投稿のID)

レスポンス:
- 成功 (200)
  ```json
  { "message": "Success" } // 仮の成功レスポンス。実際の実装に合わせる必要あり
  ```
- エラー (400/401/404/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## DELETE /api/v2/sns/posts/:userId/:postId/like
説明: 指定された投稿のいいねを解除します。

パスパラメータ:
- userId: string (投稿者のユーザーID)
- postId: string (いいねを解除する投稿のID)

レスポンス:
- 成功 (200)
  ```json
  { "message": "Success" } // 仮の成功レスポンス。実際の実装に合わせる必要あり
  ```
- エラー (400/401/404/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/sns/stories
説明: 新しいストーリーを投稿します。ストーリーは24時間後に自動的に削除されます。

リクエストボディ (application/json):
```json
{
  "media": "data:image/jpeg;base64,..." // Base64エンコードされた画像データ (JPEG, PNG, WEBP, 15MBまで)
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "Success" } // 仮の成功レスポンス。実際の実装に合わせる必要あり
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## GET /api/v2/sns/timeline
説明: フォローしているユーザーの投稿とストーリーを組み合わせたタイムラインを取得します。

レスポンス (200):
```json
{
  "posts": [
    {
      "id": "投稿ID (URL形式)",
      "content": "投稿内容",
      "createdAt": "投稿日時 (ISO 8601)",
      "media": ["メディアURL", ...],
      "author": {
        "userName": "ユーザー名@ドメイン",
        "domain": "ドメイン"
      },
      "stats": {
        "likes": 10, // いいね数
        "hasLiked": true // 自分がいいねしたか
      },
      "isRemote": false, // リモート投稿か
      "originalId": null, // リモート投稿のオリジナルID
      "url": null // リモート投稿のオリジナルURL
    },
    // ... 他の投稿
  ],
  "stories": [
    {
      "id": "ストーリーID",
      "mediaUrl": "メディアURL",
      "mediaType": "Image", // 現在は Image のみ
      "createdAt": "投稿日時 (ISO 8601)",
      "expiresAt": "有効期限 (ISO 8601)",
      "author": {
        "userName": "ユーザー名@ドメイン",
        "displayName": "表示名",
        "avatar": "アバターURL or null",
        "domain": "ドメイン"
      },
      "viewed": false, // 自分が閲覧済みか
      "isRemote": false // リモートストーリーか
    },
    // ... 他のストーリー
  ]
}
```
- エラー (401)
  ```json
  { "message": "Unauthorized" }
  ```
