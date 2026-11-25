# メモ登録API v1 内部仕様書

## 1. 対象・適用範囲

メモ登録API v1の内部構造を定める。HTTP受付、登録、保存を分離し、保存実装を差し替えられる構成とする。

## 2. 用語・前提

### 2.1 用語

| 用語 | 意味 |
|---|---|
| `Note` | 保存対象のメモ |
| `INoteStore` | 保存インターフェース |

### 2.2 実装環境

| 項目 | 名称 | バージョン |
|---|---|---|
| 言語 | C# | 12 |
| ターゲット フレームワーク | .NET | 8.0 |
| プロジェクトSDK | Microsoft.NET.Sdk.Web | .NET 8対応 |

## 3. 構造・責務

### 3.1 全体構造

#### 3.1.1 処理フロー

```mermaid
flowchart TD
    A[HTTP受付: 入力検証] --> B{検証成功}
    B -->|いいえ| C[HTTP受付: エラー応答]
    B -->|はい| D[NoteRegistration: ID発行]
    D --> E[INoteStore: 保存]
    E --> F[NoteRegistration: 登録結果]
    F --> G[HTTP受付: 応答変換]
```

#### 3.1.2 クラス図

```mermaid
classDiagram
    class NoteRegistration {
        +RegisterAsync(title, content, cancellationToken)
    }
    class INoteStore {
        <<interface>>
        +SaveAsync(note, cancellationToken)
    }
    class NoteStoreImplementation
    NoteRegistration --> INoteStore
    NoteStoreImplementation ..|> INoteStore
```

`NoteStoreImplementation`は選択される保存実装を表す図上の名称。

### 3.2 責務分担

| 構成要素 | 責務 |
|---|---|
| HTTP受付 | JSON検証、登録呼出し、結果のHTTP変換 |
| `NoteRegistration` | UUID発行、`Note`生成、保存呼出し、登録結果の生成 |
| `INoteStore`の実装 | `Note`の保存と保存結果の返却 |

### 3.3 依存方向

| 構成要素 | 利用する境界・実装 |
|---|---|
| HTTP受付 | `NoteRegistration` |
| `NoteRegistration` | `INoteStore` |
| 構成処理 | `NoteRegistration`、選択した保存実装 |

## 4. 内部インターフェース

### 4.1 一覧

| 境界 | 呼出し元 | 操作 | 実装・生成 |
|---|---|---|---|
| `NoteRegistration` | HTTP受付 | `RegisterAsync` | DIで解決 |
| `INoteStore` | `NoteRegistration` | `SaveAsync` | DIで解決した保存実装 |

**宣言**

| 型・メソッド | 宣言 |
|---|---|
| `Note` | `record Note(Guid Id, string Title, string Content)` |
| `INoteStore.SaveAsync` | `Task<bool> SaveAsync(Note note, CancellationToken cancellationToken)` |
| `NoteRegistration.RegisterAsync` | `Task<RegistrationResult> RegisterAsync(string title, string content, CancellationToken cancellationToken)` |
| `RegistrationResult` | `record RegistrationResult(bool Saved, Guid? Id)` |

### 4.2 入力

#### 4.2.1 登録処理

| 引数 | 型 | 意味 |
|---|---|---|
| `title` | string | 検証済みのタイトル |
| `content` | string | 検証済みの本文 |
| `cancellationToken` | CancellationToken | HTTP受付から渡す取消トークン |

#### 4.2.2 保存境界

| `Note`のプロパティ | 型 | 渡す値 |
|---|---|---|
| `Id` | Guid | 登録処理が発行したUUID |
| `Title` | string | 登録処理の引数`title` |
| `Content` | string | 登録処理の引数`content` |

保存境界の引数`cancellationToken`には、登録処理が受け取った取消トークンを渡す。

### 4.3 結果

| `INoteStore.SaveAsync`の結果 | `RegistrationResult.Saved` | `RegistrationResult.Id` |
|---|---|---|
| `true`（保存成功） | `true` | 保存した`Note.Id` |
| `false`（保存失敗） | `false` | `null` |

HTTP受付は登録結果を[外部仕様3.4](external-specification.md#34-応答)の保存成功または保存失敗の応答へ変換する。

### 4.4 取消・タイムアウト・例外

取消トークンによる取消例外は呼出し元へ伝播する。その他の例外分類・処理期限は未規定。

## 5. 構成・差替え

### 5.1 依存解決

**DI登録**

| 登録対象 | 生存期間 |
|---|---|
| `NoteRegistration` | Scoped |
| `INoteStore`の実装 | Scoped |

**スコープ**

| 処理 | スコープ | 依存解決と処理範囲 |
|---|---|---|
| HTTP受付 | HTTP要求スコープ | 要求ごとに登録処理と保存実装を解決する |

DI登録はアプリケーション起動時に行う。

### 5.2 差替え

| 対象 | 差替えの契約 |
|---|---|
| 保存実装 | `INoteStore`のDI登録を変更する |

`NoteRegistration`は`INoteStore`をコンストラクター引数に取る。

## 6. 参照資料

| 資料 | 参照範囲 |
|---|---|
| [外部仕様書](external-specification.md) | メモ登録API v1の外部契約 |
