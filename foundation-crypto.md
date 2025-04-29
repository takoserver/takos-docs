# Takos Protocol - メッセージ暗号化

## 概要
このドキュメントでは、Takos Protocolにおいて利用される各種鍵の種類や役割、メッセージの暗号化・署名方法を整理しています。

## 鍵システムの概要

Takos Protocolでは、以下の目的のために複数種類の鍵を使用します：
- **認証** - メッセージや鍵の送信元を確認
- **暗号化** - データの機密性を確保
- **鍵管理** - 安全な鍵の共有と更新

### 鍵の種類と役割

| 鍵の種類 | アルゴリズム | 主な役割 | 備考 |
|---------|------------|---------|------|
| **masterKey** | ML-DSA-87 | 鍵の信頼の根幹となる鍵 | 最重要鍵 |
| **identityKey** | ML-DSA-65 | メッセージやroomKeyのメタ情報の署名 | セッション毎に生成 |
| **accountKey** | ML-KEM-768 | roomKeyの暗号化と送信 | セッション毎に生成 |
| **roomKey** | AES-256-GCM | メッセージの暗号化 | ルーム毎に生成 |
| **serverKey** | ML-DSA-65 | サーバー間通信の署名 | |

*注意: 鍵のバイナリ長は使用される暗号アルゴリズムによって固定されますが、JSON文字列としての長さは変動する可能性があります。*

## 鍵の詳細構造

各鍵は特定のJSON構造を持ち、文字列として保存・送信されます。

### masterKey

システム全体の信頼の根幹となる最重要の鍵です。

```ts
interface masterKey {
  keyType: "masterKeyPublic" | "masterKeyPrivate";
  key: string; // Base64 encoded binary key
}
```

### identityKey

ユーザーの身元を証明し、メッセージや他の鍵の署名に使用します。

```ts
interface identityKey {
  keyType: "identityKeyPublic" | "identityKeyPrivate";
  key: string; // Base64 encoded binary key
  algorithm: "ML-DSA-65";
  timestamp: number;
  sessionUuid: string;
}
```

### accountKey

roomKeyを暗号化して送信するための鍵。

```ts
interface accountKey {
  keyType: "accountKeyPublic" | "accountKeyPrivate";
  key: string; // Base64 encoded binary key
  algorithm: "ML-KEM-768";
  timestamp: number;
}
```

### roomKey

メッセージを暗号化するための鍵。

```ts
interface roomKey {
  keyType: "roomKey";
  key: string; // Base64 encoded binary key
  algorithm: "AES-256-GCM"; // AES-256-GCM
  timestamp: number;
  sessionUuid: string;
}
```

### serverKey

サーバー側で署名に使用する鍵。

```ts
interface serverKey {
  keyType: "serverKeyPublic" | "serverKeyPrivate";
  key: string; // Base64 encoded binary key
  timestamp: number;
  algorithm: "ML-DSA-65"; // 署名アルゴリズム
}
```

## 暗号化と署名

### 暗号化データの形式

暗号化されたデータは共通の形式で表現され、鍵やメッセージなど様々なデータに適用されます。

```ts
export interface EncryptedData {
  keyType: string;              // 暗号化に使用された鍵の種類
  keyHash: string;              // 暗号化に使用された公開鍵のハッシュ値
  encryptedData: string;        // AES-GCMで暗号化されたデータ（Base64）
  iv: string;                   // AES-GCMの初期化ベクトル（Base64）
  algorithm?: string;           // 暗号化アルゴリズム (例: "AES-GCM")
  cipherText?: string;          // ML-KEM使用時の暗号文（Base64）
}
```

### 署名の形式

デジタル署名はデータの真正性と完全性を保証します。

```ts
export interface Sign {
  keyHash: string;    // 署名に使用された公開鍵のハッシュ値
  signature: string;  // 署名データ（Base64）
  keyType: string;    // 署名に使用された鍵の種類
  algorithm?: string; // 署名アルゴリズム (例: "ML-DSA-87", "ML-DSA-65")
}
```

## メッセージ交換

### メッセージの基本構造

Takosプロトコルでは、暗号化されたメッセージと非暗号化メッセージの2種類があります。

isLargeフラグは、メッセージが256KBを超える場合にtrueとなります。大きなメッセージは、サーバー側で分割されて送信されることがあります。
256kbを超えるメッセージは、通常送信することが不可能なため、サムネイルを経由して送信されます。

```ts
// ユーザー識別子（必要に応じて詳細な型定義に変更可能）
export type UserIdentifier = string;

// 返信情報
export interface ReplyInfo {
  id: string; // 返信対象のメッセージID
}

// 非暗号化メッセージの構造
export interface NotEncryptMessage {
  encrypted: false;
  value: {
    type: "text" | "image" | "video" | "audio" | "file" | "thumbnail";
    content: string; // 各タイプに応じたJSON文字列（下記参照）
    reply?: ReplyInfo; // 返信情報
    mention?: string[]; // メンション対象のユーザーIDリスト
  };
  channel: string; // メッセージが属するチャンネル
  /**
   * isLargeがtrueの場合、またはサムネイルの場合に元のコンテンツへの参照として利用。
   */
  original?: string;
  timestamp: number; // メッセージ送信時のタイムスタンプ
  isLarge: boolean; // コンテンツサイズが大きいかどうかのフラグ
  roomid: string; // メッセージが属するルームID
}

// 暗号化メッセージの構造
export interface EncryptedMessage {
  encrypted: true;
  value: string; // 暗号化されたデータ（EncryptedDataのJSON文字列）
  channel: string;
  /**
   * サムネイルメッセージや大容量メッセージの場合、元のコンテンツへの参照として指定。
   */
  original?: string;
  timestamp: number;
  isLarge: boolean;
  roomid: string;
}

// メッセージ全体の型（非暗号化・暗号化のどちらか）
export type Message = NotEncryptMessage | EncryptedMessage;

// 各コンテンツタイプの詳細な形式

// 1. textタイプのコンテンツ
export interface TextContent {
  text: string;                 // テキスト本文
  format?: "plain" | "markdown"; // テキスト形式（未指定の場合はplainを想定）
  thumbnailText?: string;       // テキストサムネイル用
}

// 2. imageタイプのコンテンツ
export interface ImageContent {
  uri: string;                  // 画像のURI（Base64データURIまたはURL）
  metadata: {
    filename: string;           // ファイル名
    mimeType: string;           // MIMEタイプ（例: image/jpeg, image/png）
  };
  thumbnailUri?: string;        // サムネイルURI
  thumbnailMimeType?: string;   // サムネイルMIMEタイプ
}

// 3. videoタイプのコンテンツ
export interface VideoContent {
  uri: string;                  // 動画のURI（Base64データURIまたはURL）
  metadata: {
    filename: string;           // ファイル名
    mimeType: string;           // MIMEタイプ（例: video/mp4）
  };
  thumbnailUri?: string;        // サムネイルURI
  thumbnailMimeType?: string;   // サムネイルMIMEタイプ
}

// 4. audioタイプのコンテンツ
export interface AudioContent {
  uri: string;                  // 音声のURI（Base64データURIまたはURL）
  metadata: {
    filename: string;           // ファイル名
    mimeType: string;           // MIMEタイプ（例: audio/mp3, audio/wav）
  };
  thumbnailText?: string;       // ファイル名など
}

// 5. fileタイプのコンテンツ
export interface FileContent {
  uri: string;                  // ファイルのURI（Base64データURIまたはURL）
  metadata: {
    filename: string;           // ファイル名
    mimeType: string;           // MIMEタイプ
  };
  thumbnailText?: string;       // ファイル名など
}

// メッセージのコンテンツタイプ
export type MessageContent = TextContent 
  | ImageContent 
  | VideoContent 
  | AudioContent 
  | FileContent;

```

### メンション機能

テキストメッセージには特定のユーザーへのメンション機能があります。`mention` フィールドにユーザーIDの配列として格納されます。

## 鍵の管理と運用

### roomKeyの共有プロセス

1. 送信者は受信者の**accountKey**でroomKeyを暗号化
2. 暗号化されたroomKeyを受信者に送信
3. 受信者は自身のaccountKeyでroomKeyを復号
4. 復号したroomKeyでメッセージを復号

### 鍵の更新ポリシー

セキュリティ強化のため、以下の鍵は定期的に更新されます：
- **identityKey**
- **accountKey**
- **roomKey**

### 新規デバイスの追加手順

1. 既存デバイスが新デバイスの**migrateKey**公開鍵で、移行対象の鍵（masterKey, deviceKeyなど）を暗号化します。
2. 既存デバイスが自身の**migrateSignKey**秘密鍵で、暗号化されたデータに署名します。
3. 新デバイスは、既存デバイスのmigrateSignKey公開鍵で署名を検証します。
4. 検証後、新デバイスは自身のmigrateKey秘密鍵で鍵データを復号します。

### トークルームにおける鍵の規則

- 同じsessionUUIDを持つroomKeyとidentityKeyは、そのセッション内でのみ有効です。
- サーバー側のタイムスタンプとメッセージのタイムスタンプの誤差が**1分以上**ある場合、メッセージは無効と見なされる可能性があります。