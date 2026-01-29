# 1. 現状の構成（運用中）

## Fabric側
- Lakehouse（テーブル3つ／ファイルなし）
- Fabric Data Agent
  - Lakehouseをデータソースとして追加
  - テーブル1つ＋（残り2テーブルから作った）Viewを選択

参考:
- Fabric Data Agent 概要（Concepts）
  - https://learn.microsoft.com/ja-jp/fabric/data-science/concept-data-agent

## Copilot Studio側
- Fabric Data Agent を「外部エージェント」として接続
- Teamsに公開（ユーザはTeamsの個別チャットで利用）

参考:
- Copilot Studio から Fabric Data Agent（外部エージェント）へ接続（Preview）
  - https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/add-agent-fabric-data-agent

---

# 2. 発生している問題
- 体験として良い面はある
- ただし **レスポンスが遅い** / **CU消費が大きい** のが気になる

---

# 3. なぜ遅い＆CUが増えやすいのか（ざっくり）
この構成だと、1回の質問で「二重のコスト」になりがち。

## 3.1 LLM側（入出力トークン）のコスト
- 会話が長い／指示が長い／出力が長い → トークン増 → CU増＆待ち時間増

参考:
- Data Agent の課金（トークン→CU秒の換算、入力/出力のレート）
  - https://learn.microsoft.com/ja-jp/fabric/fundamentals/data-agent-consumption

## 3.2 クエリ実行側（SQL等）のコスト
- Data Agentがクエリを生成して実行する場合、
  **クエリ実行は対応するクエリエンジン側に“別途”課金**され得る
- View/JOIN/スキャンが重いと、レイテンシもコストも増える

参考:
- 「クエリ実行はクエリ エンジン項目に個別に課金される」注意点
  - https://learn.microsoft.com/ja-jp/fabric/fundamentals/data-agent-consumption

---

# 4. 解決策（いまの構成を活かしつつ効きやすい順）

## 4.1 Copilot Studio → Data Agent に渡す「トークン」を減らす（即効性が出やすい）
- Copilot Studio側の指示文を短くする（長い定型文・冗長な要件を削る）
- 出力を短く縛る（例：「結論→根拠2点→数値→参照元」）
- よくある質問は“固定回答”に寄せる（後述のSemantic model/Verified answers が効く）

参考:
- Data Agent課金（トークン依存なので、短文化が効く理由の根拠）
  - https://learn.microsoft.com/ja-jp/fabric/fundamentals/data-agent-consumption

## 4.2 Lakehouse（SQL analytics endpoint）のクエリ待ちを減らす（JOIN/Viewが重い時に効く）
- SQL analytics endpoint は同期/遅延や設計要因で体感が変わる
- View/JOINが重いなら、元データ設計や運用（最適化/更新頻度）も見直し対象

参考:
- Lakehouse の SQL analytics endpoint パフォーマンス考慮事項
  - https://learn.microsoft.com/ja-jp/fabric/data-warehouse/sql-analytics-endpoint-performance

## 4.3 Lakehouse → Warehouse寄りにする（対話でT-SQLが主戦場なら有利な場合）
- 構造化データ中心で、T-SQLで安定して回したいなら Warehouse を検討
- 「3テーブル＋View」のように定義が固いなら移行も現実的

参考:
- Lakehouse と Warehouse の意思決定ガイド
  - https://learn.microsoft.com/ja-jp/fabric/fundamentals/decision-guide-lakehouse-warehouse
- Warehouse のパフォーマンス ガイドライン
  - https://learn.microsoft.com/ja-jp/fabric/data-warehouse/guidelines-warehouse-performance

## 4.4 Semantic model 経由に変える（“余計な探索”を抑えて軽くなることがある）
- Data Agentは Power BI semantic model をデータソースにできる
- 「Prep for AI」で列・指示・Verified answers を整備すると、
  迷走クエリ/余計な探索が減ってコストが安定しやすい

参考:
- Data Agent × semantic model（Prep for AI の扱いを含む）
  - https://learn.microsoft.com/en-us/fabric/data-science/data-agent-semantic-model
- Prep data for AI（Verified answers の設定/管理）
  - https://learn.microsoft.com/en-us/power-bi/create-reports/copilot-prepare-data-ai

---

# 5. Teams UI ＋ Fabric で「他のやり方」はある？

## 5.1 ルートA：Copilot Studioを挟まず、Microsoft 365 Copilot（Teams）から Data Agent を直接使う
- Data Agent を発行して Agent Store 経由で Teams から直接操作する構成
- 多段のオーケストレーションが減れば、無駄なトークンや呼び出しが減る可能性

参考:
- Microsoft 365 Copilot から Data Agent を利用（Agent Store への発行含む）
  - https://learn.microsoft.com/ja-jp/fabric/data-science/data-agent-microsoft-365-copilot

## 5.2 ルートB：Copilot Studioは使うが、Data Agentではなく「REST APIアクション」で定型APIを叩く
- Copilot Studio に OpenAPI 仕様の REST API ツールを追加
- “自由にSQL生成して探索”を減らし、API側で定型化＋キャッシュしやすくする
  → レイテンシ/コストが読みやすい運用になりやすい

参考:
- Copilot Studio：REST API（OpenAPI）でツールを追加
  - https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/agent-extend-action-rest-api

## 5.3 ルートC：Teams内は Power BI アプリ（タブ）中心で完結させる
- 「会話で回答」より「可視化で自己解決」が多いなら強い
- チャットは補助にして、普段はレポートで回す設計もアリ

参考:
- Teams の Power BI アプリ
  - https://learn.microsoft.com/ja-jp/power-bi/collaborate-share/service-microsoft-teams-app

---

# 6. 当面のおすすめ（実務的な優先順位）
1. **Copilot Studio側の指示と出力制約を短くする**（トークン削減で即効性）
2. **View/JOINが重いならSQL analytics endpointの観点で見直す**（Lakehouse側の待ち削減）
3. 中期で
   - **Warehouse化** or **Semantic model＋Prep for AI（Verified answers含む）**
   - もしくは **REST APIアクション＋定型クエリ＋キャッシュ**で“自由探索”を減らす
