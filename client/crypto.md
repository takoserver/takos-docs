# takos web の鍵

## 概要
takos web クライアントで利用する鍵について、その主な用途と構造をまとめています。

## 鍵一覧
| 鍵名            | 主な用途                                  |
|-----------------|-------------------------------------------|
| shareKey        | 他セッションへの accountKey 共有          |
| shareSignKey    | 共有鍵で暗号化されたデータの署名          |
| migrateKey      | 新規デバイスへの鍵移行                    |
| migrateSignKey  | 移行データの署名検証                      |
| deviceKey       | デバイス固有データの暗号化                |

## shareKey
```ts
interface shareKey {
  keyType: "shareKeyPublic" | "shareKeyPrivate";
  key: string; // Base64 encoded binary key
  algorithm: "ML-KEM-768";
  timestamp: number;
  sessionUuid: string;
}
```

## shareSignKey
```ts
interface shareSignKey {
  keyType: "shareSignKeyPublic" | "shareSignKeyPrivate";
  key: string; // Base64 encoded binary key
  algorithm: "ML-DSA-65";
  timestamp: number;
  sessionUuid: string;
}
```

## migrateKey
```ts
interface migrateKey {
  keyType: "migrateKeyPublic" | "migrateKeyPrivate";
  key: string; // Base64 encoded binary key
  timestamp?: number;
  algorithm: "ML-KEM-768";
}
```

## migrateSignKey
```ts
interface migrateSignKey {
  keyType: "migrateSignKeyPublic" | "migrateSignKeyPrivate";
  key: string; // Base64 encoded binary key
  timestamp?: number;
  algorithm: "ML-DSA-65";
}
```

## deviceKey
```ts
interface deviceKey {
  keyType: "deviceKey";
  key: string; // Base64 encoded binary key
}
```