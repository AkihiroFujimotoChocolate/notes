# 図の記述例

Mermaidを使った独立した記述例。対象仕様に存在する契約・構造を図にする。図表と説明文の扱いは[Markdownの記述方法](../markdown-style.md#21-図表と説明文)に従う。

## 1. 外部仕様の概念フロー

### 1.1 同期処理

```mermaid
flowchart TD
    A[要求受信] --> B{入力条件成立}
    B -->|いいえ| C[HTTP 400]
    B -->|はい| D[保存要求]
    D --> E{保存成功}
    E -->|はい| F[HTTP 201]
    E -->|いいえ| G[HTTP 503]
```

### 1.2 非同期処理

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

応答後に後続処理へ渡す契約の例。順序が未規定なら図でも順序を確定しない。

## 2. 内部仕様の処理フロー

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

## 4. 複数の図を配置する場合の見出し

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
