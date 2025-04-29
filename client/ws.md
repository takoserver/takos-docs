# WebSocket API

## 共通ヘッダー
- 認証: sessionid をクエリまたは Cookie で送信
- プロトコル: JSON メッセージ

---

リアルタイム通信のための WebSocket API です。

## 接続

エンドポイント: `wss://<your-domain>/ws`

接続時には、認証のために `sessionid` をクエリパラメータまたは Cookie で送信する必要があります。

例 (クエリパラメータ): `wss://<your-domain>/ws?sessionid=<セッションID>`

例 (Cookie): `Cookie: sessionid=<セッションID>`

認証に失敗した場合、接続はすぐに閉じられます。
　
## サーバー -> クライアント メッセージ

サーバーは以下のタイプのメッセージを JSON 形式で送信します。

```json
{
  "type": "<メッセージタイプ>",
  "data": "<メッセージ内容 (多くの場合 JSON 文字列)>"
}
```

### メッセージタイプ

| タイプ           | data の内容 (JSON 文字列)                                                                                                | 説明                                                                 |
| :--------------- | :----------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------- |
| `message`        | `{ users: string[], data: any }`                                                                                         | 指定されたユーザー (`users`) に汎用メッセージ (`data`) を送信します。        |
| `migrateRequest` | `{ migrateid: string, migrateKey: string }`                                                                              | 他のデバイスからのデータ移行リクエストを受信したことを通知します。         |
| `migrateAccept`  | `{ migrateid: string, migrateSignKey: string }`                                                                          | データ移行リクエストが承認されたことを通知します。                     |
| `migrateData`    | `{ migrateid: string, data: string, sign: string }`                                                                      | 移行データ (`data`) と署名 (`sign`) を送信します。                     |
| `callRequest`    | `{ caller: string, callee: string, callid: string, sessionid: string, sdp: RTCSessionDescriptionInit }`                    | 通話リクエストを受信したことを通知します。                             |
| `callAccept`     | `{ caller: string, callee: string, callid: string, sessionid: string, sdp: RTCSessionDescriptionInit }`                    | 通話リクエストが承認されたことを通知します。                           |

## クライアント -> サーバー メッセージ

クライアントは以下のメッセージを JSON 形式で送信できます。

```json
{
  "type": "<メッセージタイプ>",
  "data": "<メッセージ内容>"
}
```

### メッセージタイプ

| タイプ       | data の内容 | 説明                                                                                                                               |
| :----------- | :---------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| `subGroup`   | `string`    | 特定のサブグループに参加します。現在、この機能がどのように利用されているかはコードからは明確ではありませんが、サーバー側で関連付けられます。 |

**注意:** 上記以外のメッセージタイプをクライアントから送信しても、現在の実装では無視されます。
