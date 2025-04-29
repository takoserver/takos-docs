# Foundation api

takosサーバーではfoundation apiを試用して相互に通信します。
takosサーバーはこれらのapiを試用して、メッセージをリアルタイムで共有します。

API は、各サーバー間の HTTPS リクエストを使用して実装されます。これらの HTTPS
リクエストは、TLS トランスポート層での公開キー署名と、HTTP 層での HTTP
認証ヘッダー内の公開キー署名を使用して強力に認証されます。

## GET API 一覧

以下は takos サーバーの GET 系 API 一覧です。

### `GET` /_takos/v1/version

サーバーの実装名とバージョンを取得します。

- レート制限: なし
- 認証: なし

#### レスポンス

```json
{
  "version": "1.0.0",
  "name": "takos"
}
```

---

### `GET` /_takos/v1/key/server

サーバーの公開鍵を取得します。

- レート制限: なし
- 認証: なし

#### クエリ

| パラメータ | 説明           |
| ---------- | -------------- |
| `expire`   | キーの有効期限 |

#### レスポンス

```json
{
  "key": "PUBLIC_KEY"
}
```

---

### `GET` /_takos/v1/key/server/{serverName}

他のサーバーの公開鍵を取得します。

- レート制限: なし
- 認証: なし

#### クエリ

| パラメータ | 説明           |
| ---------- | -------------- |
| `expire`   | キーの有効期限 |

#### レスポンス

```json
{
  "key": "PUBLIC_KEY"
}
```

---

### `GET` /_takos/v1/user/{userId}/{key}

ユーザーのデータ（icon, nickName, description, verify）を取得します。

- レート制限: なし
- 認証: なし

#### パスパラメータ

| パラメータ | 説明       |
| ---------- | ---------- |
| `userId`   | ユーザーID |
| `key`      | 取得項目 (icon, nickName, description, verify) |

#### レスポンス

```json
{
  "data": /* 取得したデータ */
}
```

### `GET` /_takos/v1/group/{groupId}/{key}

- `key` : icon | name | description | defaultChannelId

#### レスポンス

```json
{ "data": /* 取得したデータ */ }
```

#### レスポンス

```json
{ "hash": "CURRENT_HASH" }
```


### `GET` /_takos/v1/crypto/key/{kind}

ユーザーの暗号鍵（identityKey, roomKey）を取得するエンドポイントです。

#### クエリパラメーター

| パラメーター   | 型     | 説明                                                      |
| -------------- | ------ | --------------------------------------------------------- |
| `userId`       | string | ユーザーID。serverKey以外必須。                           |
| `hash`         | string | `identityKey` と `roomKey` の場合に必須となるハッシュ値。 |
| `targetUserId` | string | `roomKey` の場合に必要な相手のユーザーID。                |

#### レスポンス

```json
{
  "key": "data",
  "signature"?: "signature"
}
```

---

### `GET` /_takos/v1/group/search

グループを検索するエンドポイントです。

#### クエリパラメーター

| パラメーター | 型     | 説明           |
| ------------ | ------ | -------------- |
| `query`      | string | キーワード     |
| `limit`      | number | 取得する最大数 |

#### レスポンス

```json
{
  "groups": string[]
}
```

---

### `GET` /_takos/v1/server/{item}

サーバーのデータを取得します。

- レート制限: なし
- 認証: なし

items

- name - サーバー名
- description - サーバーの説明
- icon - サーバーアイコン
- version - サーバーバージョン

---

### `GET` /_takos/v1/message/:messageId

メッセージのデータを取得します。

- レート制限: なし
- 認証: なし

#### パスパラメータ

| パラメータ  | 説明         |
| ----------- | ------------ |
| `messageId` | メッセージID |

#### レスポンス

```json
{
  "message": message.message,
  "signature": message.sign,
  "timestamp": message.timestamp,
  "userName": message.userName
}
```

---

## サーバー実装

GET系APIについては「GET API 一覧」を参照してください。

### `POST` /_takos/v1/event

イベントを送信します。

レート制限: あり 認証: あり

#### リクエスト

`body`

| パラメータ | 型       | 説明           |
| ---------- | -------- | -------------- |
| `event`    | `string` | イベント名     |
| `eventId`  | `string` | イベントID     |
| `payload`  | `object` | イベントデータ |

`payload` は、イベントデータを含むオブジェクトです。

#### レスポンス

レスポンスの内容はありません。 成功かどうかは、HTTP
ステータスコードで判断します。

## 認証

サーバーによって行われるすべてのHTTPリクエストは、公開鍵署名を使用して認証されます。リクエストボディの署名は、`Authorization`ヘッダーに含まれます。

### `Authorization` ヘッダー

#### `Authorization` ヘッダー例
以下のように HTTP リクエストヘッダーとして送信します。
```
POST /_takos/v1/event HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: X-Takos-Signature signature="<署名>", Expires="<有効期限>", origin="<ドメイン>"

{ /* リクエストボディ */ }
```

Authorization ヘッダーの形式は、HTTP Message Signature (IETF draft) を参考にしています。

origin: 送信サーバーのサーバー名 expires: 有効期限 sign: 署名

### レスポンス認証

応答は TLS サーバー証明書によって認証されます。

## Events

### t.friend.request.send

友達リクエストを送信します。

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `friendId` | `string` | 友達ID     |

### t.friend.request.cancel

友達リクエストをキャンセルします。

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `friendId` | `string` | 友達ID     |

### t.friend.request.accept

友達リクエストを受け入れます。

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `friendId` | `string` | 友達ID     |

### t.friend.remove

友達を削除します。

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `friendId` | `string` | 友達ID     |

### t.message.send

メッセージを送信します。

`payload`

| パラメータ  | 型                               | 説明         |
| ----------- | -------------------------------- | ------------ |
| `userId`    | `string`                         | ユーザーID   |
| `messageId` | `string`                         | メッセージID |
| `roomId`    | `string`                         | ルームID     |
| `roomType`  | `"friend" | "privateGroup" | "publicGroup"` | ルームタイプ |
| `channelId` | `string`                         | チャンネルID |

### t.group.invite.send

グループに招待します。 privateのみ

`payload`

| パラメータ     | 型       | 説明           |
| -------------- | -------- | -------------- |
| `userId`       | `string` | ユーザーID     |
| `groupId`      | `string` | グループID     |
| `inviteUserId` | `string` | 招待ユーザーID |

### t.group.invite.accept

グループ招待を受け入れます。 privateのみ

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `groupId`  | `string` | グループID |

### t.group.invite.cancel

グループ招待を削除します。 privateのみ

`payload`

| パラメータ     | 型       | 説明           |
| -------------- | -------- | -------------- |
| `groupId`      | `string` | グループID     |
| `userId`       | `string` | ユーザーID     |
| `inviteUserId` | `string` | 招待ユーザーID |

### t.friend.group.invite

友達にグループに招待されたことを通知します。 privateのみ

`payload`

| パラメータ     | 型       | 説明           |
| -------------- | -------- | -------------- |
| `userId`       | `string` | ユーザーID     |
| `groupId`      | `string` | グループID     |
| `inviteUserId` | `string` | 招待ユーザーID |

### t.group.leave

グループを退出します。

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `groupId`  | `string` | グループID |

### t.group.channel.add

グループチャンネルを作成/上書きします。

`payload`

| パラメータ    | 型                   | 説明         |
| ------------- | -------------------- | ------------ |
| `userId`      | `string`             | ユーザーID   |
| `groupId`     | `string`             | グループID   |
| `channelId`   | `string`             | チャンネルID |
| `channelName` | `string`             | チャンネル名 |
| `categoryId`  | `string or undefined` | カテゴリー   |
| `permissions` | `object[]`           | 権限         |

`object`

| パラメータ   | 型         | 説明        |
| ------------ | ---------- | ----------- |
| `roleId`     | `string`   | ロールID    |
| `permissions`| `string[]` | 権限        |

### t.group.channel.remove

グループチャンネルを削除します。

`payload`

| パラメータ  | 型       | 説明         |
| ----------- | -------- | ------------ |
| `userId`    | `string` | ユーザーID   |
| `groupId`   | `string` | グループID   |
| `channelId` | `string` | チャンネルID |

### t.group.category.add

グループカテゴリーを作成/上書きします。

`payload`

| パラメータ     | 型         | 説明         |
| -------------- | ---------- | ------------ |
| `userId`       | `string`   | ユーザーID   |
| `groupId`      | `string`   | グループID   |
| `categoryId`   | `string`   | カテゴリーID |
| `categoryName` | `string`   | カテゴリー名 |
| `permissions`  | `object[]` | 権限         |

`object`

| パラメータ   | 型         | 説明        |
| ------------ | ---------- | ----------- |
| `roleId`     | `string`   | ロールID    |
| `permissions`| `string[]` | 権限        |

### t.group.category.remove

グループカテゴリーを削除します。

`payload`

| パラメータ   | 型       | 説明         |
| ------------ | -------- | ------------ |
| `userId`     | `string` | ユーザーID   |
| `groupId`    | `string` | グループID   |
| `categoryId` | `string` | カテゴリーID |

### t.group.channel.order
チャンネルとカテゴリーの順番を変更します。
`payload`
| パラメータ    | 型         | 説明         |
| ------------- | ---------- | ------------ |
| `userId`      | `string`   | ユーザーID   |
| `groupId`     | `string`   | グループID   |
| `channelId`   | `string[]` | チャンネルID |
| `categoryId`  | `string[]` | カテゴリーID |

### t.group.role.add

グループロールを作成/上書きします。

`payload`

| パラメータ   | 型         | 説明         |
| ------------ | ---------- | ------------ |
| `userId`     | `string`   | ユーザーID   |
| `groupId`    | `string`   | グループID   |
| `roleId`     | `string`   | ロールID     |
| `roleName`   | `string`   | ロール名     |
| `permissions` | `string[]` | 権限         |
| `color`      | `string`   | カラーコード |

### t.group.role.remove

グループロールを削除します。

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `groupId`  | `string` | グループID |
| `roleId`   | `string` | ロールID   |

### t.group.user.role

グループロールを割り当てます。 (上書き)

`payload`

| パラメータ     | 型         | 説明               |
| -------------- | ---------- | ------------------ |
| `userId`       | `string`   | ユーザーID         |
| `groupId`      | `string`   | グループID         |
| `roleIds`      | `string[]` | ロールID一覧       |
| `assignUserId` | `string`   | 割り当てユーザーID |

### t.group.join.request

グループに参加リクエストを送信します。 publicGroupのみ

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `groupId`  | `string` | グループID |

### t.group.join.accept

グループ参加リクエストを受け入れます。 publicGroupのみ

`payload`

| パラメータ      | 型       | 説明                 |
| --------------- | -------- | -------------------- |
| `userId`        | `string` | ユーザーID           |
| `groupId`       | `string` | グループID           |
| `requestUserId` | `string` | リクエストユーザーID |

### t.friend.group.accept

友達にグループ参加リクエストを受け入れたことを通知します。 publicGroupのみ

`payload`

| パラメータ      | 型       | 説明                 |
| --------------- | -------- | -------------------- |
| `userId`        | `string` | ユーザーID           |
| `groupId`       | `string` | グループID           |
| `requestUserId` | `string` | リクエストユーザーID |

### t.group.join.remove

グループ参加リクエストを削除します。 publicGroupのみ

`payload`

| パラメータ      | 型       | 説明                 |
| --------------- | -------- | -------------------- |
| `userId`        | `string` | ユーザーID           |
| `groupId`       | `string` | グループID           |
| `requestUserId` | `string` | リクエストユーザーID |

### t.group.join.cancel

グループ参加リクエストをキャンセルします。 publicGroupのみ

`payload`

| パラメータ | 型       | 説明       |
| ---------- | -------- | ---------- |
| `userId`   | `string` | ユーザーID |
| `groupId`  | `string` | グループID |

### t.group.kick

グループからユーザーをキックします。

`payload`

| パラメータ   | 型       | 説明             |
| ------------ | -------- | ---------------- |
| `userId`     | `string` | ユーザーID       |
| `groupId`    | `string` | グループID       |
| `kickUserId` | `string` | キックユーザーID |

### t.group.ban

グループからユーザーをBANします。

`payload`

| パラメータ  | 型       | 説明          |
| ----------- | -------- | ------------- |
| `userId`    | `string` | ユーザーID    |
| `groupId`   | `string` | グループID    |
| `banUserId` | `string` | BANユーザーID |

### t.group.unban

グループからユーザーのBANを解除します。

`payload`

| パラメータ  | 型       | 説明          |
| ----------- | -------- | ------------- |
| `userId`    | `string` | ユーザーID    |
| `groupId`   | `string` | グループID    |
| `banUserId` | `string` | BANユーザーID |

### t.group.channel.default

グループのデフォルトチャンネルを変更します。

`payload`

| パラメータ  | 型       | 説明         |
| ----------- | -------- | ------------ |
| `userId`    | `string` | ユーザーID   |
| `groupId`   | `string` | グループID   |
| `channelId` | `string` | チャンネルID |

### t.group.sync

クライアントが最新ハッシュを送信し、差異があれば全データを要求・送信するためのトリガーイベントです。  
#### payload
| フィールド        | 型      | 説明                            |
|-------------------|---------|---------------------------------|
| protocolVersion   | string  | 同期プロトコルバージョン (`"1.0"`) |
| timestamp         | string  | 送信時刻 (ISO 8601 UTC)          |
| userId            | string  | 要求元ユーザーID                |
| groupId           | string  | グループID                      |

イベントで全データを送ります。

### t.group.sync.diff

JSON Patch (RFC 6902) による差分同期イベントです。  
#### payload
| フィールド        | 型          | 説明                                   |
|-------------------|-------------|----------------------------------------|
| protocolVersion   | string      | 同期プロトコルバージョン (`"1.0"`)       |
| timestamp         | string      | 送信時刻 (ISO 8601 UTC)                |
| userId            | string      | 要求元ユーザーID                       |
| groupId           | string      | グループID                             |
| patch             | JSONPatch[] | 差分配列 (RFC 6902 準拠)               |

```ts
// JSONPatch の詳細は group.md を参照
```

## エラー処理とリトライポリシー

分散環境下でのグループ操作（leave, kick, ban, unban, invite, join など）では、相手サーバーがタイムアウトや拒否を返すケースを考慮します。

1. ローカル先行更新  
   - 操作対象をローカルDB上で先に適用し、ユーザーには即時反映を通知します。  
   - 例）`t.group.leave` では自サーバーのメンバーリストから即時削除。

2. リモートリクエスト送信  
   - 対象サーバーへ HTTPS リクエストを送信し、成功レスポンスを待ちます。

3. タイムアウト・エラー時の挙動  
  　自サーバーDBのみ更新し、以降の同期で調整。  

## データの取得

GET系APIについては「GET API 一覧」を参照してください。
````
