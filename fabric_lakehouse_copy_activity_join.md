# Fabric（Lakehouse）で「Copy Activity＋JOINクエリ」から新しいテーブルを作る手順（初心者向け）

この手順は、**同一 Lakehouse 内の複数テーブルを JOIN して、その結果を Lakehouse の新しいテーブルとして作成**するやり方です。  
Copy Activity の **Source を「T‑SQL Query（Preview）」**にすると、**Lakehouse の SQL analytics endpoint 経由で SELECT（JOIN含む）を実行して結果を読み取り**、その結果を Destination に書き込めます（公式: [Copy アクティビティで Lakehouse を構成する方法](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity)）。 ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))

---

## 0. 事前確認（ここだけ先にやると詰まりにくい）

### 0-1. JOIN対象テーブルが「Tables」にあるか確認
Lakehouse の左ペインで、JOINしたいテーブルが **Tables** 配下に見えていることを確認します。

### 0-2. SQL analytics endpoint から見えるか確認（重要）
T‑SQL Query モードは **SQL analytics endpoint 経由**です（公式: [Lakehouse SQL analytics endpoint](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-sql-analytics-endpoint)）。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-sql-analytics-endpoint?utm_source=chatgpt.com))  
もし「Sparkで作った外部Deltaテーブル」などが endpoint から見えない場合があります（その場合ショートカット等が必要、公式に注意あり）。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-sql-analytics-endpoint?utm_source=chatgpt.com))

---

## 1. パイプラインを作る

1. Fabric の対象ワークスペースを開く  
2. 左メニューで **Data Factory** に切り替え  
3. **New item** → **Data pipeline** を作成  
4. キャンバスに **Copy data（Copy activity）** を追加  
   - Copy activity の概念/使い方（公式: [How to copy data using copy activity](https://learn.microsoft.com/en-us/fabric/data-factory/copy-data-activity)） ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-factory/copy-data-activity?utm_source=chatgpt.com))

---

## 2. Copy activity の Source を設定（JOINクエリを書く）

Copy activity をクリック → **Source** タブで設定します。

### 2-1. Connection / Lakehouse
- **Connection**：対象 Lakehouse の接続を選択（無ければ作成）  
  - ※動的に Lakehouse を切り替えたい場合、**Lakehouse object ID** をパラメータで渡せると公式に書かれています（公式: [Lakehouse コネクタ（Copy activity）](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity)）。 ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))
- **Root folder**：**Tables** を選択

### 2-2. 「Use query」＝ T‑SQL Query（Preview）
- **Use query**：**T‑SQL Query (Preview)** を選択  
  - これは **SQL analytics endpoint 経由でカスタムSQLを読みに行く**モードです（公式: [Lakehouse connector – T‑SQL Query (Preview)](https://learn.microsoft.com/en-us/fabric/data-factory/connector-lakehouse-copy-activity)）。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-factory/connector-lakehouse-copy-activity))

### 2-3. JOIN クエリを貼る（例）
```sql
SELECT
  s.order_id,
  s.order_date,
  s.customer_id,
  c.customer_name,
  p.product_name,
  s.amount
FROM dbo.sales s
JOIN dbo.customers c ON s.customer_id = c.customer_id
JOIN dbo.products  p ON s.product_id  = p.product_id;
```

> ヒント：スキーマ名を省略すると `dbo` が既定になる旨が公式にあります（Source/ Destination とも同様の注意が記載）。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-factory/connector-lakehouse-copy-activity))

---

## 3. Copy activity の Destination を設定（新しいテーブルを作る）

Copy activity → **Destination** タブで設定します。

### 3-1. Connection / Root folder
- **Connection**：同じ Lakehouse を選択
- **Root folder**：**Tables** を選択

### 3-2. 出力先テーブル名を指定（新規作成）
- **Table**：既存テーブルを選ぶか、**New** を選んで新規テーブルを作れます（公式: [Lakehouse copy activity – Destination](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity)）。 ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))  
- テーブル名の制約（`/` `\` を含まない、末尾ドット不可、前後スペース不可 など）も公式に注意があります。 ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))

### 3-3. Table action（まずは Overwrite が楽）
- **Overwrite（上書き）**：毎回作り直す（初心者の最初の動作確認におすすめ） ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))  
- **Append（追加）**：追記する ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))  
- **Upsert（Preview）**：更新＋挿入。ただし **パーティションテーブルではUpsert非対応**、Upsert中はパーティション有効化もできない（公式に明記）。 ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))

### 3-4. V-Order（既定ON）
Destination 側には **V-Order の適用（既定ON）**があり、オフにすると追加の最適化をしない、という説明が公式にあります。  
- V-Order の説明（公式: [Delta Lake テーブルの最適化と V-Order](https://learn.microsoft.com/ja-jp/fabric/data-engineering/delta-optimization-and-v-order)） ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-engineering/delta-optimization-and-v-order?utm_source=chatgpt.com))

---

## 4. Mapping（基本は自動でOK）
**Mapping** タブは、まずは既定の自動マッピングでOKです。  
型が怪しい（例: 数値が文字列扱い等）ときだけ、列マッピング/型変換を調整します（型マッピングの一覧も公式ページに載っています）。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-factory/connector-lakehouse-copy-activity))

---

## 5. 実行して結果を確認

1. パイプラインを **Save**  
2. **Run**（または Debug）で実行  
3. 実行ログは Monitor で確認（Data Factory の基本導線は公式トップから辿れます: [Data Factory in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/)）。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-factory/?utm_source=chatgpt.com))  
4. Lakehouse の **Tables** に、出力テーブルができているか確認

---

## 6. （任意）パイプラインパラメータを使ってクエリを可変にする

パイプラインには **Parameters** があり、`pipeline().parameters.xxx` を式で参照できます（公式: [Parameters for Data Factory in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/parameters)）。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-factory/parameters?utm_source=chatgpt.com))  
式言語自体は `@concat()` や `@pipeline().parameters.xxx` など（Microsoft公式の式言語解説: [ADF の式言語（参照構文が同じ）](https://learn.microsoft.com/ja-jp/azure/data-factory/how-to-expression-language-functions)）。 ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/azure/data-factory/how-to-expression-language-functions?utm_source=chatgpt.com))

### 6-1. 例：日付で絞る（from_date / to_date）
1. パイプラインのプロパティで Parameters を追加  
   - `from_date`（string 例: `2026-01-01`）
   - `to_date`（string 例: `2026-02-01`）

2. Source の **T‑SQL Query** 欄を **動的式**にして、例えばこんな形にします：

```text
@concat(
'SELECT s.order_id, s.order_date, s.customer_id, c.customer_name, p.product_name, s.amount
 FROM dbo.sales s
 JOIN dbo.customers c ON s.customer_id = c.customer_id
 JOIN dbo.products  p ON s.product_id  = p.product_id
 WHERE s.order_date >= ''', pipeline().parameters.from_date, '''
   AND s.order_date <  ''', pipeline().parameters.to_date,  ''''
)
```

> ※ UI のどの入力欄が “Add dynamic content” 対応かは項目ごとに差があります。少なくとも **Lakehouse 接続は object ID をパラメータで指定できる**ことが公式に書かれています。 ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))

---

## 7. よくある詰まりポイント

- **「T‑SQL Query（Preview）」が動かない / テーブルが見えない**  
  → SQL analytics endpoint に見えていない可能性。特に **外部Deltaは見えない**注意が公式にあります。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-sql-analytics-endpoint?utm_source=chatgpt.com))
- **Upsert を選んだらエラー**  
  → **パーティションテーブルはUpsert非対応**、Upsert中はパーティション有効化も不可（公式に明記）。 ([learn.microsoft.com](https://learn.microsoft.com/ja-jp/fabric/data-factory/connector-lakehouse-copy-activity))
- **Preview 機能の注意**  
  → T‑SQL Query モードは Preview で、かつ **workspace-level private link をサポートしない**旨が公式にあります。 ([learn.microsoft.com](https://learn.microsoft.com/en-us/fabric/data-factory/connector-lakehouse-copy-activity))
