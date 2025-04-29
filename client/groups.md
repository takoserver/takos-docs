# Groups API
認証: 必要 (Authorization ヘッダーにトークンを設定)

## エンドポイント一覧

### グループ管理
| メソッド | パス                      | 説明                         |
| -------- | ------------------------- | ---------------------------- |
| POST     | /api/v2/groups/create     | 新しいグループを作成         |
| POST     | /api/v2/groups/settings   | グループ設定を変更           |
| POST     | /api/v2/groups/leave      | グループから退出             |
| POST     | /api/v2/groups/accept     | グループ招待を承認           |
| POST     | /api/v2/groups/name       | グループ名を変更             |
| POST     | /api/v2/groups/icon       | グループアイコンを変更         |
| POST     | /api/v2/groups/description| グループ説明を変更           |

### 招待・参加管理
| メソッド | パス                      | 説明                         |
| -------- | ------------------------- | ---------------------------- |
| POST     | /api/v2/groups/invite/invite | ユーザーをグループに招待     |
| POST     | /api/v2/groups/join/join    | パブリックグループに参加     |
| POST     | /api/v2/groups/join/request | パブリックグループへの参加申請 |
| POST     | /api/v2/groups/join/accept  | 参加申請を承認             |
| POST     | /api/v2/groups/join/remove  | 参加申請を拒否/削除        |
| POST     | /api/v2/groups/join/cancel  | 参加申請を取り消し         |

### メンバー管理
| メソッド | パス                      | 説明                         |
| -------- | ------------------------- | ---------------------------- |
| POST     | /api/v2/groups/user/kick    | メンバーをグループから追放   |
| POST     | /api/v2/groups/user/ban     | ユーザーをグループからBAN    |
| POST     | /api/v2/groups/user/unban   | ユーザーのBANを解除        |
| POST     | /api/v2/groups/user/owner   | グループオーナーを変更       |

### チャンネル・カテゴリ管理
| メソッド | パス                      | 説明                         |
| -------- | ------------------------- | ---------------------------- |
| POST     | /api/v2/groups/channel/add    | チャンネルを追加/更新      |
| POST     | /api/v2/groups/channel/delete | チャンネルを削除           |
| POST     | /api/v2/groups/channel/default| デフォルトチャンネルを設定   |
| POST     | /api/v2/groups/channel/order  | チャンネル/カテゴリの順序を設定 |
| POST     | /api/v2/groups/category/add   | カテゴリを追加/更新        |
| POST     | /api/v2/groups/category/delete| カテゴリを削除           |

### ロール管理
| メソッド | パス                      | 説明                         |
| -------- | ------------------------- | ---------------------------- |
| POST     | /api/v2/groups/role/add     | ロールを追加/更新          |
| POST     | /api/v2/groups/role/delete  | ロールを削除               |
| POST     | /api/v2/groups/role/user/role | ユーザーにロールを割り当て   |

## リクエスト／レスポンス定義

### 共通ヘッダー
- cookie sessionid:"token"

---

## POST /api/v2/groups/create
説明: 新しいグループを作成します。

リクエストボディ (application/json):
```json
{
  "name": "グループ名",
  "icon": "Base64エンコードされたアイコン画像データ",
  "groupid": "任意のグループID (省略可能)",
  "isPublic": true // true: public, false: private
}
```

レスポンス:
- 成功 (200)
  ```json
  { "groupId": "作成されたグループID@ドメイン" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/settings
説明: グループの設定を変更します。変更したい項目のみ含めます。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "name": "新しいグループ名 (任意)",
  "description": "新しい説明文 (任意)",
  "icon": "Base64エンコードされた新しいアイコン画像データ (任意)",
  "allowJoin": true // パブリックグループへの自由参加を許可するかどうか (任意)
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/leave
説明: 参加しているグループから退出します。オーナーは退出できません。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/accept
説明: プライベートグループへの招待を承認します。

リクエストボディ (application/json):
```json
{
  "groupId": "招待されたグループID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/name
説明: グループ名を変更します。グループ管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "name": "新しいグループ名"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/icon
説明: グループアイコンを変更します。グループ管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "icon": "Base64エンコードされた新しいアイコン画像データ"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/description
説明: グループの説明を変更します。グループ管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "description": "新しい説明文"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/invite/invite
説明: プライベートグループにユーザーを招待します。招待権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "userId": "招待するユーザーID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/join/join
説明: 参加が許可されているパブリックグループに参加します。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/join/request
説明: 参加申請が必要なパブリックグループに参加申請を送ります。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/join/accept
説明: パブリックグループへの参加申請を承認します。ユーザー管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "userId": "申請者のユーザーID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/join/remove
説明: パブリックグループへの参加申請を拒否または削除します。ユーザー管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "userId": "申請者のユーザーID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/join/cancel
説明: 送信したパブリックグループへの参加申請を取り消します。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/user/kick
説明: グループメンバーを追放します。オーナーは追放できません。ユーザー管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "userId": "追放するユーザーID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/user/ban
説明: ユーザーをグループからBANします。BANされたユーザーは参加できなくなります。オーナーはBANできません。ユーザー管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "userId": "BANするユーザーID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/user/unban
説明: ユーザーのグループBANを解除します。ユーザー管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "userId": "BAN解除するユーザーID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/user/owner
説明: グループのオーナーを変更します。現在のオーナーのみ実行可能です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "userId": "新しいオーナーのユーザーID@ドメイン"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/channel/add
説明: チャンネルを追加または更新します。チャンネル管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "name": "チャンネル名",
  "id": "チャンネルID (新規作成時はユニークなID, 更新時は既存ID)",
  "categoryId": "所属させるカテゴリID (任意)",
  "permissions": [ // チャンネル固有の権限設定 (任意)
    {
      "roleId": "ロールID",
      "permissions": ["権限文字列", ...] // 例: "READ_MESSAGE", "SEND_MESSAGE"
    },
    ...
  ]
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/channel/delete
説明: チャンネルを削除します。チャンネル管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "channelId": "削除するチャンネルID"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/channel/default
説明: グループのデフォルトチャンネルを設定します。チャンネル管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "channelId": "デフォルトに設定するチャンネルID"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/channel/order
説明: チャンネルリストのチャンネルとカテゴリの表示順序を設定します。チャンネル管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "order": [
    { "type": "category", "id": "カテゴリID1" },
    { "type": "channel", "id": "チャンネルID1" },
    { "type": "channel", "id": "チャンネルID2" },
    { "type": "category", "id": "カテゴリID2" },
    ...
  ]
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/category/add
説明: カテゴリを追加または更新します。チャンネル管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "name": "カテゴリ名",
  "id": "カテゴリID (新規作成時はユニークなID, 更新時は既存ID)",
  "permissions": [ // カテゴリ固有の権限設定 (任意)
    {
      "roleId": "ロールID",
      "permissions": ["権限文字列", ...]
    },
    ...
  ]
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/category/delete
説明: カテゴリを削除します。カテゴリ内のチャンネルはカテゴリ未所属になります。チャンネル管理権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "categoryId": "削除するカテゴリID"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/role/add
説明: ロールを追加または更新します。管理者権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "name": "ロール名",
  "id": "ロールID (新規作成時はユニークなID, 更新時は既存ID)",
  "color": "カラーコード (#FFFFFF)",
  "permissions": ["権限文字列", ...] // 例: "MANAGE_CHANNEL", "KICK_USER"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/role/delete
説明: ロールを削除します。`everyone` ロールは削除できません。管理者権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "roleId": "削除するロールID"
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```

---

## POST /api/v2/groups/role/user/role
説明: ユーザーにロールを割り当てます。管理者権限が必要です。

リクエストボディ (application/json):
```json
{
  "groupId": "対象のグループID@ドメイン",
  "userId": "対象のユーザーID@ドメイン",
  "roleId": ["割り当てるロールID1", "割り当てるロールID2", ...] // 割り当てるロールIDの配列
}
```

レスポンス:
- 成功 (200)
  ```json
  { "message": "success" }
  ```
- エラー (400/401/500)
  ```json
  { "message": "<エラー内容>" }
  ```
