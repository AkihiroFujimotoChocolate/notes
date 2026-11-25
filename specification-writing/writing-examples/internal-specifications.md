# 内部仕様の記述例

各表は独立した架空の例を含む。[完成サンプル](../examples/internal-specification.md)の仕様を変更するものではない。

## 1. 用語・実装環境

### 1.1 用語

| 用語 | 意味 |
|---|---|
| `Note` | 保存するメモのデータ |
| `INoteStore` | 保存インターフェース |

### 1.2 実装環境

| 項目 | 名称 | バージョン |
|---|---|---|
| 言語 | C# | 12 |
| ターゲット フレームワーク | .NET | 8.0 |
| プロジェクトSDK | Microsoft.NET.Sdk.Web | .NET 8対応 |

## 2. 構造・責務

### 2.1 責務分担

| 構成要素 | 責務 |
|---|---|
| HTTP受付 | 入力検証とHTTP応答への変換 |
| `NoteRegistration` | ID生成、保存要求、登録結果の生成 |
| 保存実装 | メモの保存 |

### 2.2 依存方向

| 構成要素 | 利用する境界・実装 |
|---|---|
| HTTP受付 | `NoteRegistration` |
| `NoteRegistration` | `INoteStore` |
| 構成処理 | `NoteRegistration`、選択した `INoteStore` 実装 |

責務の詳細は責務分担で示す。依存の選択規則を表現し、将来の全実装を列挙しない。

## 3. 内部インターフェース

### 3.1 一覧

| 境界 | 呼出し元 | 操作 | 実装・生成 |
|---|---|---|---|
| `INoteStore` | `NoteRegistration` | `SaveAsync` | DIで解決した保存実装 |
| `NoteRegistration` | HTTP受付 | `RegisterAsync` | 要求スコープで解決 |

メソッド宣言：`Task<bool> SaveAsync(Note note, CancellationToken cancellationToken)`

### 3.2 引数

| 引数 | 型 | 意味 |
|---|---|---|
| `note` | Note | 保存対象 |
| `cancellationToken` | CancellationToken | 呼出し元から渡す取消トークン |

### 3.3 プロパティ

| プロパティ | 型 | 意味 |
|---|---|---|
| `Id` | Guid | メモの識別子 |
| `Title` | string | タイトル |
| `Content` | string | 本文 |

### 3.4 入力の組立て

| `Note`のプロパティ | 型 | 渡す値 |
|---|---|---|
| `Id` | Guid | 登録処理で生成したUUID |
| `Title` | string | 登録処理の引数 `title` |
| `Content` | string | 登録処理の引数 `content` |

**内部で受け渡す情報**

| 項目 | 型 | 取得元・用途 |
|---|---|---|
| `RequestId` | string | HTTP要求の識別子。診断記録で使用 |

### 3.5 結果

| 境界 | 結果 | 意味 |
|---|---|---|
| `INoteStore.SaveAsync` | `true` | 保存成功 |
| `INoteStore.SaveAsync` | `false` | 保存失敗 |

**登録結果**

| `RegistrationResult`の項目 | 型 | 意味 |
|---|---|---|
| `Saved` | bool | 保存結果 |
| `Id` | Guid? | 成功時は保存したId、失敗時はnull |

### 3.6 取消・例外

| 条件 | 境界の動作 |
|---|---|
| 保存境界から取消例外を受けた場合 | 取消例外を呼出し元へ伝播 |

結果が生じる条件を定めていない場合も、その結果を受け取った側の処理を規定できる。

## 4. 構成・差替え

### 4.1 依存解決

| 項目 | 条件 |
|---|---|
| 構成時点 | アプリケーション起動時 |
| 実装選択 | 設定で選択した契約版のキーを使用 |

### 4.2 実装選択

| キー | 入力組立ての実装 | 保存処理の実装 |
|---|---|---|
| v1 | `V1NoteMapper` | `V1NoteStore` |
| v2 | `V2NoteMapper` | `V2NoteStore` |

同じ版の組合せを使用することを構成側の契約として定める。版固有の詳細は各版に置く。

### 4.3 DI登録

| 登録対象 | 生存期間 |
|---|---|
| `NoteRegistration` | Scoped |
| `INoteStore` の実装 | Scoped |
| バックグラウンド処理 | Singleton |

### 4.4 スコープ

| 処理 | スコープ | 依存解決と処理範囲 |
|---|---|---|
| HTTP受付 | HTTP要求スコープ | 要求の受付から応答まで |
| バックグラウンド処理 | 取得した1件ごとのスコープ | 一件の処理開始時に解決し、終了時に破棄 |

### 4.5 差替え

| 対象 | 差替えの契約 |
|---|---|
| 保存実装 | `INoteStore` の登録を変更する |
| 通信方式 | 保存実装内でSDK型と共通型を変換する |

## 5. 版別契約型

| 項目 | v1の契約型 | v2の契約型 |
|---|---|---|
| `RequestId` | string? | string |
| `Content` | string | string |

v1の通信契約が`RequestId`の省略またはnullを許し、v2が非nullの値を要求する例。内部処理で採番済みのIDを使う場合は別の内部型にできる。通信上の必須条件はserializer等の設定も含めて成立させる。

[図の記述例](diagrams.md)
