# 図の記述例

Mermaidを使った独立した記述例。対象仕様に存在する契約・構造を図にする。図表と説明文の扱いは[Markdownの記述方法](../markdown-style.md#21-図表と説明文)に従う。

## 1. 外部仕様の概念フロー

外部から観測できる分岐、要求と応答、同期・非同期の関係を表す。内部クラスや詳細な実装手順を図に持ち込まない。

```mermaid
flowchart TD
    A[要求受信] --> B{入力条件成立}
    B -->|いいえ| C[HTTP 400]
    B -->|はい| D[保存要求]
    D --> E{保存成功}
    E -->|はい| F[HTTP 201]
    E -->|いいえ| G[HTTP 503]
```

非同期の受付と処理を区別する例：

```mermaid
sequenceDiagram
    participant C as 呼出し元
    participant S as 受付サービス
    participant W as 後続処理
    C->>S: 登録要求
    S-->>C: HTTP 202
    S->>W: 受付済みデータ
    W->>W: 保存
```

この図は応答後に後続処理へ渡す契約を採用した例。応答と後続処理の順序が未規定の場合はこの順序を要求しない。

## 2. 内部仕様の処理フロー

構成要素間の処理順序と受渡しを示す。詳細な型や引数は内部インターフェースの表に置く。

```mermaid
flowchart TD
    A[HTTP受付: 入力検証] --> B{検証成功}
    B -->|いいえ| C[HTTP受付: 400応答]
    B -->|はい| D[NoteRegistration: ID生成]
    D --> E[INoteStore: 保存]
    E --> F[NoteRegistration: 結果生成]
    F --> G[HTTP受付: 応答変換]
```

## 3. クラス図

採用済みの構成要素、必要な操作、依存方向を示す。構成全体を実装するための全クラスや補助メソッドを列挙する必要はない。

**登録処理**

```mermaid
classDiagram
    class NoteRegistration {
        +RegisterAsync(title, content, cancellationToken)
    }
    class INoteStore {
        <<interface>>
        +SaveAsync(note, cancellationToken)
    }
    class StorageAdapter
    NoteRegistration --> INoteStore : uses
    StorageAdapter ..|> INoteStore : implements
```

**版別構成**

```mermaid
classDiagram
    class INoteRequest {
        <<interface>>
    }
    class V1NoteRequest
    class V2NoteRequest
    V1NoteRequest ..|> INoteRequest
    V2NoteRequest ..|> INoteRequest
```

内部仕様の全体構造を複数図で示す場合の見出し例：

```markdown
### 3.1 全体構造

#### 3.1.1 処理フロー

［Mermaidフロー図］

#### 3.1.2 クラス図

**登録処理**

［Mermaidクラス図］

**版別構成**

［Mermaidクラス図］
```

節タイトルが図題を兼ねる場合は図題を省略する。同種の図が複数ある場合は「登録処理」「版別構成」など、対象が分かる簡潔な図題にする。図の意味や依存方向が本文と一致することを確認する。
