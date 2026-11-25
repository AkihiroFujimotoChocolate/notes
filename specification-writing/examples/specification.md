# メモ登録API v1 仕様書

## 1. 対象・適用範囲

対象サービス、版、提供する機能。

メモ登録API v1の外部契約と内部構造を定める。タイトルと本文を受け取り、メモを保存してIDを返す。HTTP受付、登録、保存を分離する。


## 2. 用語・前提

用語と実装環境。

### 2.1 用語

| 用語 | 意味 |
|---|---|
| メモ | id、title、contentからなる保存対象 |
| 呼出し元 | HTTP APIを利用するクライアント |

本文はUTF-8 JSON objectとする。

| 用語 | 意味 |
|---|---|
| Note | 保存対象のメモ |
| INoteStore | 保存境界 |
| 登録処理 | ID発行と保存を担当 |

### 2.2 実装環境

| 項目 | 名称 | バージョン |
|---|---|---|
| 言語 | C# | 12 |
| ターゲット フレームワーク | .NET | 8.0 |
| プロジェクトSDK | Microsoft.NET.Sdk.Web | .NET 8対応 |


## 3. 提供インターフェース

呼出しを受ける際の要求・検証・応答。

### 3.1 接続条件

| 項目 | 仕様 |
|---|---|
| メソッド・パス | POST `/notes` |
| Content-Type | application/json（UTF-8） |

### 3.2 要求

| 項目 | 型 | 条件 |
|---|---|---|
| `title` | string | 必須。空文字は不可 |
| `content` | string | 必須。空文字可 |

```json
{"title":"買い物","content":"牛乳"}
```

### 3.3 検証

titleまたはcontentの欠落・null・型不一致、titleの空文字、JSON構文不正の場合は入力エラーとする。titleの空白だけの文字列は受理する。追加項目は無視する。

### 3.4 応答

| 条件 | HTTP | JSON本文 |
|---|---:|---|
| 保存成功 | 201 | `id`（string、UUID） |
| 入力エラー | 400 | `error`（string、固定値 `"invalid_input"`） |
| 保存失敗 | 503 | `error`（string、固定値 `"storage_unavailable"`） |

```json
{"id":"550e8400-e29b-41d4-a716-446655440000"}
```


## 4. 利用インターフェース

他のサービスを呼び出す際の要求・応答・認証。

### 4.1 接続一覧

該当なし（外部サービスへの通信契約は定めない）。

### 4.2 要求

該当なし。

### 4.3 応答

該当なし。

### 4.4 認証

該当なし。


## 5. 処理規則

インターフェース間の対応と処理順序。

### 5.1 全体フロー

```mermaid
flowchart TD
    A[要求受信] --> B{入力検証}
    B -->|失敗| C[400]
    B -->|成功| D[ID発行・保存]
    D -->|成功| E[201]
    D -->|失敗| F[503]
```

### 5.2 対象の選別

入力検証を満たす要求を保存対象とする。入力エラーの場合は保存を行わない。

### 5.3 入出力の対応

検証成功後にUUIDを発行し、id、title、contentを保存する。title・contentはデコードした文字列をそのまま保存する。応答idは保存したidと一致する。

### 5.4 応答と後続処理

保存処理を1回行い、その結果に応じてHTTP応答を返す。保存失敗時に同じ要求を自動再試行しない。


## 6. エラー・タイムアウト

異常時の結果と処理期限。

### 6.1 受付時の異常

入力エラー時はHTTP 400を返す。

### 6.2 呼出し結果の扱い

保存処理の失敗時はHTTP 503を返す。

### 6.3 期限と復帰

処理期限・クライアント切断時の外部結果は未規定。


## 7. 設定

設定項目と動作。

### 7.1 設定項目と動作

該当なし。

### 7.2 設定と通信値の区別

該当なし。


## 8. 構造・責務

内部構造、責務分担、依存方向。

### 8.1 全体構造

#### 8.1.1 構成図

```mermaid
flowchart LR
    A[HTTP受付] --> B[NoteRegistration]
    B --> C[INoteStore]
    D[保存実装] -. 実装 .-> C
```

### 8.2 責務分担

| 構成要素 | 責務 |
|---|---|
| HTTP受付 | JSON検証、登録呼出し、結果のHTTP変換 |
| NoteRegistration | UUID発行、Note生成、保存呼出し、登録結果 |
| INoteStore実装 | Noteの保存と保存結果の返却 |

### 8.3 依存方向

| 構成要素 | 利用する境界・実装 |
|---|---|
| HTTP受付 | NoteRegistration |
| NoteRegistration | INoteStore |
| 構成処理 | NoteRegistration・保存実装 |


## 9. 内部インターフェース

構成要素間の契約。

### 9.1 一覧

| 型・メソッド | 宣言 |
|---|---|
| Note | `record Note(Guid Id, string Title, string Content)` |
| INoteStore.SaveAsync | `Task<bool> SaveAsync(Note note, CancellationToken cancellationToken)` |
| NoteRegistration.RegisterAsync | `Task<RegistrationResult> RegisterAsync(string title, string content, CancellationToken cancellationToken)` |
| RegistrationResult | `record RegistrationResult(bool Saved, Guid? Id)` |

### 9.2 入力

登録処理は検証済みのtitle・contentを受け取り、生成したUUIDと共にNoteを保存境界へ渡す。取消トークンを保存境界へ引き渡す。

### 9.3 結果

保存結果trueの場合はSaved=trueかつIdは保存したUUID。falseの場合はSaved=falseかつId=null。保存失敗の通知は保存境界のfalseを用いる。HTTP受付は結果を外部契約の201または503へ変換する。

### 9.4 取消・タイムアウト・例外

取消トークンによる取消例外は呼出し元へ伝播する。その他の例外分類・処理期限は未規定。


## 10. 構成・差替え

依存解決と実装の選択。

### 10.1 依存解決

起動時に標準DIで登録する。

| 登録対象 | 生存期間 |
|---|---|
| NoteRegistration | Scoped |
| INoteStoreの実装 | Scoped |

| 処理 | スコープ | 依存解決と処理範囲 |
|---|---|---|
| HTTP受付 | HTTP要求スコープ | 要求スコープで解決する |

### 10.2 差替え

保存実装を変更する際はINoteStoreの登録を変更する。NoteRegistrationはINoteStoreをコンストラクター引数に取る。


## 11. 参照資料

根拠資料と契約の適用版。

### 11.1 プロジェクト資料

本書の例は架空のサービス契約。

### 11.2 接続先の実装資料

該当なし。

### 11.3 外部公式ドキュメント

該当なし。
