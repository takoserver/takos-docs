# groupの仕様

> 注: `GroupData` 型はサーバー間の内部通信専用です。外部クライアントには `icon`, `name`, `description` のみ公開されます。

groupにはこのような概念があります。

- channel
- category
- member
- role

## channel

channelはgroup内のチャンネルです。 すべてのメッセージはchannelに送信されます。
channelはcategoryに属したり属さなかったりします。
デフォルトの権限では、channelやcategoryにおいて送信する権限がありません。

次のような形式で表すことができます。

channel

```ts
{
    name: string;
    id: string;
    categoryId?: string;               // optionalに変更、名称をcategoryIdに統一
    type: "text" | "voice";
    permissions: {                      // 複数形に統一
        roleId: string;
        permissions: string[];          // 複数形に統一
    }[];
}
```

## category

categoryはchannelをまとめるためのものです。
categoryにも権限を設定することができ、channelに権限を継承させることができます。

次のような形式で表すことができます。

category

```ts
{
    name: string;
    id: string;
    permissions: {                      // 複数形に統一
        roleId: string;
        permissions: string[];
    }[];
}
```

## member

memberはgroupに参加しているユーザーです。
memberにはroleが設定されており、roleIdsによって複数のロールが紐付けられます。

次のような形式で表すことができます。

member

```ts
{
    userId: string;
    roleIds: string[]; // グループ内で保有するロールのID一覧
}
```

## role

roleはmemberに参加しているユーザーに権限を与えるためのものです。
roleにはpermissionsが設定されており、permissionsによって権限が与えられます。

次のような形式で表すことができます。

role

```ts
{
    name: string;
    id: string;
    permissions: string[];            // そのロール固有の権限一覧
    color: string;
}
```

## userのロール

userのroleは次のように設定されます。

```ts
{
    userId: string,
    roleIds: string[]
}
```

## GroupData 型

```ts
interface GroupData {
  id: string;                     // グループID
  type: "privateGroup" | "publicGroup";     // 種別
  owner: string;                  // オーナーのユーザーID
  members: {
    userId: string;
    roleIds: string[];
  }[];
  roles: {
    id: string;
    name: string;
    color: string;
    permissions: string[];           // ロールに紐づく権限一覧
  }[];
  categories: {
    id: string;
    name: string;
    permissions: {
      roleId: string;
      permissions: string[];
    }[];
  }[];
  channels: {
    id: string;
    name: string;
    type: "text" | "voice";
    categoryId?: string;            // オプション: 親カテゴリーID
    permissions: {
      roleId: string;
      permissions: string[];
    }[];
  }[];
}
```

## GroupMetadata

```ts
interface GroupMetadata {
  icon: string;
  name: string;
  description: string;
  defaultChannelId?: string;
}
```

## API: グループデータ取得

### サーバー間通信 (内部用)
GET /_takos/v1/group/{groupId}
> グループ全体データ (GroupData) を取得します。

#### レスポンス 200
```json
{ "data": "/* full GroupData JSON */" }
```

### 外部クライアント向け (メタデータ取得)
GET /_takos/v1/group/{key}/{groupId}
> icon, name, description, defaultChannelId のみ取得 (GroupMetadata)

#### レスポンス 200
```json
{ "data": /* GroupMetadata */ }
```

### orderプロパティは廃止: 配列の順序はJSON配列の順番で決定されます。

## groupへの参加

グループへの参加操作は、グループ種別によって使用するイベントが異なります。

### プライベートグループ(privateGroup)

- t.group.invite.send    : 招待を送信
- t.group.invite.accept  : 招待を承認して参加
- t.group.invite.cancel  : 招待を取り消し
- t.friend.group.invite  : 招待通知 (招待を受けたことを通知)
- t.group.leave          : グループから退出
- t.group.kick           : 強制退出
- t.group.ban            : BAN

### パブリックグループ(publicGroup)

- t.group.join.request   : 参加リクエストを送信
- t.group.join.accept    : リクエストを承認して参加
- t.friend.group.accept  : 承認通知
- t.group.join.remove    : リクエストを拒否
- t.group.join.cancel    : リクエストをキャンセル
- t.group.leave          : グループから退出
- t.group.kick           : 強制退出
- t.group.ban            : BAN

## groupからの退出

メンバーの退出は

### t.group.leave

groupから退出するためのイベントです。

このeventを利用した場合、t.group.sync.member.removeは利用したユーザーのサーバーには送信されません。

### t.group.kick & t.group.ban

groupから強制的に退出させるためのイベントです。

## groupの同期（新方式）

差分同期ではなく、グループ全体データを文字列化してハッシュ化したものを比較し、必要に応じて全データを取得します。

### 同期プロトコルバージョン
各同期イベントには、`protocolVersion` (string) と `timestamp` (ISO 8601 UTC) フィールドを必須で含めます。現行バージョンは `"1.0"` です。

- GET /_takos/v1/group/{groupId}/hash  
  現在のグループデータのハッシュを取得します。  
  レスポンス例  
  ```json
  { "hash": "CURRENT_HASH" }
  ```

- Event: t.group.sync  
  ペイロード  
  | フィールド        | 型      | 説明                            |
  |-------------------|---------|---------------------------------|
  | protocolVersion   | string  | 同期プロトコルバージョン (`"1.0"`) |
  | timestamp         | string  | 送信時刻 (ISO 8601 UTC)          |
  | userId            | string  | 要求元ユーザーID                |
  | groupId           | string  | グループID                      |
  | clientHash        | string  | クライアント側の現行ハッシュ    |

  サーバー（または相手）は `clientHash` と自サイドのハッシュを比較し、異なる場合は以下のイベントで全データを送ります。

- Event: t.group.sync.data  
  ペイロード  
  | フィールド        | 型      | 説明                 |
  |-------------------|---------|----------------------|
  | protocolVersion   | string  | 同期プロトコルバージョン (`"1.0"`) |
  | timestamp         | string  | 送信時刻 (ISO 8601 UTC)          |
  | groupId           | string  | グループID           |
  | data              | object  | フルグループデータ     |

## Diff同期フォーマット

JSON Patch (RFC 6902) を用いて差分同期を行います。

```ts
// JSONPatch 型定義
interface JSONPatch {
  op: "add" | "remove" | "replace" | "move" | "copy" | "test";
  path: string;          // JSON Pointer
  value?: any;           // add, replace, test 時に必須
  from?: string;         // move, copy 時に必須
}

interface DiffSyncPayload {
  protocolVersion: string;  // 同期プロトコルバージョン (`"1.0"`)
  timestamp: string;        // 送信時刻 (ISO 8601 UTC)
  userId: string;           // 要求元ユーザーID
  groupId: string;
  patch: JSONPatch[];       // 差分配列 (RFC 6902 準拠)
}
```

> 注意: t.group.sync.data はフルデータ同期を行い、t.group.sync.diff は上記の DiffSyncPayload で差分同期を行います。

## ハッシュ用 Canonical JSON フォーマット

ハッシュ生成時は以下のルールで JSON を文字列化します。

1. 全てのオブジェクトキーをアルファベット順にソート  
2. インデントや余分な空白を除去し、最小化  
3. 文字列はダブルクォート `"` で統一  
4. 配列は元の順序を維持  
5. Unicode は UTF‑8 でエンコード  

TypeScript 例:

```ts
// filepath: c:\Users\shout\Desktop\takos\takos-docs\group.md
function canonicalStringify(obj: any): string {
  if (Array.isArray(obj)) {
    return '[' + obj.map(canonicalStringify).join(',') + ']';
  } else if (obj !== null && typeof obj === 'object') {
    const keys = Object.keys(obj).sort();
    return '{' + keys
      .map(k => JSON.stringify(k) + ':' + canonicalStringify(obj[k]))
      .join(',') + '}';
  } else {
    return JSON.stringify(obj);
  }
}
```
