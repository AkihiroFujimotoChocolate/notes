# Copilot Studio × Fabric Data Agent 運用におけるレスポンス/コスト（CU）最適化メモ

更新日: 2026-01-29

本資料は、**「レスポンスとCU消費の大きさが気になっている」**という課題の改善を目的に、現状構成の整理と、実施優先度の高い対策案（特に **Copilot Studio側をルーティング専用化**して二重オーケストレーションを減らす案）をまとめたものです。

---

# 1. 現状の構成（運用中）

## Fabric側
## LakeHouse
- 3つのテーブル
- ファイルはなし
## FabricDataAgent
- 上記LakeHouseを「データの追加」で追加
- LakeHouseが3つのテーブルのうち、一つのチェックボックスをオン
- 残りの2つのテーブルを基に一つのViewをつくり、そのViewのチェックボックスをオン

参考:
- Fabric データ エージェントの概要（プレビュー）  
  https://learn.microsoft.com/ja-jp/fabric/data-science/concept-data-agent
- Fabric データ エージェントの作成（プレビュー）  
  https://learn.microsoft.com/en-us/fabric/data-science/how-to-create-data-agent

## CopilotStudio側
- 上記FabcirDataAgentを外部エージェントとして接続
- Teamsに公開

参考:
- Copilot Studio から Microsoft Fabric Data Agent に接続（プレビュー）  
  https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/add-agent-fabric-data-agent

## Teams側
- 上記CopilotStudioエージェントとユーザが個別チャットで会話

---

# 2. 発生している問題
いい面もあったが、**レスポンスとCU消費の大きさが気になっている**

---

# 3. なぜ遅い＆CUが増えやすいのか（要因整理）

「遅延」「CU消費」の要因は、大きく **(1)トークン由来**、**(2)クエリ実行由来**、**(3)二重オーケストレーション由来**に分解して考えると整理しやすいです。

## 3.1 LLM側（入出力トークン）のコスト
- 会話が長い／指示が長い／出力が長い → トークン増 → **CU増**＆**待ち時間増**

参考:
- Fabric Data Agent の課金（トークン→CU秒の換算、入力/出力のレート）  
  https://learn.microsoft.com/ja-jp/fabric/fundamentals/data-agent-consumption
- Fabric ブログ: 容量消費/課金の考え方（Copilot/AIのレート等の背景）  
  https://blog.fabric.microsoft.com/en-us/blog/understanding-operations-agent-capacity-consumption-usage-reporting-and-billing/

## 3.2 クエリ実行側（SQL等）のコスト
- Fabric Data Agentがクエリを生成して実行する場合、**クエリ実行は対応するクエリエンジン側に“別途”課金**され得る
- View/JOIN/スキャンが重いと、レイテンシもコストも増える

参考:
- 「クエリ実行はクエリ エンジン項目に個別に課金される」注意点  
  https://learn.microsoft.com/ja-jp/fabric/fundamentals/data-agent-consumption

## 3.3（重要）Copilot Studio と Data Agent による「二重オーケストレーション」
現在の構成は、Copilot Studio が（Topic/ツール/外部エージェント呼び出し等を）**オーケストレーション**し、さらに Fabric Data Agent 側でも（データ探索・クエリ生成/実行を）**オーケストレーション**するため、次のような「ムダ」が発生しやすいです。

- Copilot Studio側で **余計な生成（要約/補足）**が入り、Data Agentへの入力が増える（＝トークン増）
- Data Agent呼び出し後に、Copilot Studioが **さらに応答生成**してしまう（二重生成）
- エージェント間の委譲で **会話履歴がデフォルトで引き継がれる**ため、履歴肥大化でトークンが増える

参考:
- 生成オーケストレーションをOFFにしてクラシック化する手順（Copilot Studio）  
  https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/advanced-generative-actions
- マルチエージェントで「会話履歴がデフォルトで引き継がれる」旨（Copilot Studio）  
  https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/multi-agent-patterns
- 生成オーケストレーションのFAQ（委譲時に履歴を渡すか等を設定できる旨）  
  https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-orchestration

---

# 4. 解決策（いまの構成を活かしつつ効きやすい順）

## 4.0（最優先候補）Copilot Studio を「ルーティング専用」にして二重オーケストレーションを抑える
**目的:** Copilot Studio側での余計な生成・長い履歴引き継ぎを減らし、Data Agent側に渡すトークンと往復を削減する。  
→ 期待効果は **レスポンス短縮**と **Data Agent CUの削減**（入力/出力トークンが減るため）です。  
※ただし、Data Agentが実行するSQLが重い場合の“根本”は別途対策が必要です（4.2〜4.4）。

### 実装の考え方（要点）
1) **Copilot Studioの生成オーケストレーションをOFF（クラシック化）**し、Topicで明示的に転送する  
- 生成オーケストレーションをOFFにする手順:  
  https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/advanced-generative-actions

2) 生成オーケストレーションをONのまま運用する場合は、**「親の自動応答」を抑止**する  
- トリガー「AI で生成された応答が送信される時」で `ContinueResponse=false` を設定し、親の追加応答（要約・補足）を送らない  
  https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/authoring-triggers

3) **会話履歴の引き継ぎを最小化**する（渡すのは “直近の質問 + 必須パラメータ” を基本方針に）  
- マルチエージェントでは会話履歴がデフォルトで引き継がれるため、渡す文脈を設計することが推奨される  
  https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/multi-agent-patterns
- 委譲時に「履歴を渡すか」「どのタスクを完了させるか」を設定できる旨（FAQ）  
  https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-orchestration

4) 外部エージェント（Fabric Data Agent）へは **Topicで明示的に転送**する  
- 外部エージェント（Microsoft Fabric）接続手順:  
  https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/add-agent-fabric-data-agent  
- 他エージェントを追加してモジュール化する考え方（Copilot Studio）:  
  https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/authoring-add-other-agents

### 期待効果と限界（取引先向けの説明観点）
- 本対策は、**Copilot Studio側の“余計な生成/往復”を減らす**ためのもの。  
- Data Agent側が行う **クエリ実行の重さ**（JOIN/スキャン等）には直接効かない場合があるため、後続のデータ側最適化（4.2〜4.4）と組み合わせる。

---

## 4.1 Copilot Studio → Data Agent に渡す「トークン」を減らす（即効性が出やすい）
- Copilot Studio側の指示文を短くする（長い定型文・冗長な要件を削る）
- 出力を短く縛る（例：「結論→根拠2点→数値→参照元」）
- よくある質問は“固定回答”に寄せる（後述のSemantic model/Verified answers が効く）

参考:
- Data Agent課金（トークン依存なので、短文化が効く理由の根拠）  
  https://learn.microsoft.com/ja-jp/fabric/fundamentals/data-agent-consumption

## 4.2 Lakehouse（SQL analytics endpoint）のクエリ待ちを減らす（JOIN/Viewが重い時に効く）
- SQL analytics endpoint は同期/遅延や設計要因で体感が変わる
- View/JOINが重いなら、元データ設計や運用（最適化/更新頻度）も見直し対象

参考:
- Lakehouse の SQL 分析エンドポイント パフォーマンス考慮事項  
  https://learn.microsoft.com/ja-jp/fabric/data-warehouse/sql-analytics-endpoint-performance
- SQL 分析エンドポイントの概要  
  https://learn.microsoft.com/ja-jp/fabric/data-engineering/lakehouse-sql-analytics-endpoint

## 4.3 Lakehouse → Warehouse寄りにする（対話でT-SQLが主戦場なら有利な場合）
- 構造化データ中心で、T-SQLで安定して回したいなら Warehouse を検討
- 「3テーブル＋View」のように定義が固いなら移行も現実的

参考:
- Lakehouse と Warehouse の意思決定ガイド  
  https://learn.microsoft.com/ja-jp/fabric/fundamentals/decision-guide-lakehouse-warehouse

## 4.4 Semantic model 経由に変える（“余計な探索”を抑えて軽くなることがある）
- Data Agentは Power BI semantic model をデータソースにできる
- 「Prep for AI」で列・指示・Verified answers を整備すると、迷走クエリ/余計な探索が減ってコストが安定しやすい

参考:
- Fabric Data Agent でセマンティック モデルをデータ ソースとして使う（Prep for AI/Verified answers含む）  
  https://learn.microsoft.com/ja-jp/fabric/data-science/data-agent-semantic-model
- Power BI: Prep data for AI（Verified answersの管理）  
  https://learn.microsoft.com/ja-jp/power-bi/create-reports/copilot-prepare-data-ai-verified-answers
- Power BI: Prep data for AI（全体）  
  https://learn.microsoft.com/ja-jp/power-bi/create-reports/copilot-prepare-data-ai

---

# 5. Teams UI ＋ Fabric で「他のやり方」はある？

## 5.1 ルートA：Copilot Studioを挟まず、Microsoft 365 Copilot（Teams）から Data Agent を直接使う
- Data Agent を発行して Agent Store 経由で Teams から直接操作する構成
- 多段のオーケストレーションが減れば、無駄なトークンや呼び出しが減る可能性

参考:
- Microsoft 365 Copilot から Fabric Data Agent を利用（Agent Storeへの発行含む）  
  https://learn.microsoft.com/ja-jp/fabric/data-science/data-agent-microsoft-365-copilot

## 5.2 ルートB：Copilot Studioは使うが、Data Agentではなく「REST APIアクション」で定型APIを叩く
- Copilot Studio に OpenAPI 仕様の REST API ツールを追加
- “自由にSQL生成して探索”を減らし、API側で定型化＋キャッシュしやすくする  
  → レイテンシ/コストが読みやすい運用になりやすい

参考:
- Copilot Studio：REST API（OpenAPI）でツールを追加  
  https://learn.microsoft.com/ja-jp/microsoft-copilot-studio/agent-extend-action-rest-api

## 5.3 ルートC：Teams内は Power BI アプリ（タブ）中心で完結させる
- 「会話で回答」より「可視化で自己解決」が多いなら強い
- チャットは補助にして、普段はレポートで回す設計も選択肢

参考:
- Teams の Power BI アプリ  
  https://learn.microsoft.com/ja-jp/power-bi/collaborate-share/service-microsoft-teams-app

## 5.4（補足）Dify / OpenAI Agent Builder 等の「外部エージェントビルダー」を使う選択肢
**狙い:** Fabric Data Agent を使わずに（または使用頻度を下げて）、「トークン消費やSQL実行をより定型化し、コスト/レスポンスを読みやすくする」。

- Dify はワークフロー/エージェント構築のためのプラットフォームで、WebhookトリガーやAPIで外部システムと連携可能  
  - Webhook Trigger（Dify Docs）: https://docs.dify.ai/ja/use-dify/nodes/trigger/webhook-trigger  
  - API連携（Dify Docs）: https://docs.dify.ai/en/use-dify/publish/developing-with-apis  
  - 公式サイト: https://dify.ai/jp  
  - OSSリポジトリ（GitHub）: https://github.com/langgenius/dify

- OpenAI Agent Builder は、マルチステップのエージェントワークフローをGUIで組み、SDKコードとして書き出すことができる（Beta）  
  - 公式ドキュメント: https://platform.openai.com/docs/guides/agent-builder

**留意点（取引先向けの説明観点）**
- Data Agent のCU消費を抑えられる可能性はある一方、Teams上でのSSO/権限制御/監査など、Microsoftエコシステムの運用要件に合わせるには追加設計が必要となるケースがある。  
- “Fabricのクエリ実行コスト”自体は、外部ビルダーに替えても（Fabricを叩く限り）残るため、定型化・キャッシュ・事前集計の設計が重要。

---

# 6. 当面のおすすめ（実務的な優先順位）

1. **Copilot Studio側を「ルーティング専用化」し、二重オーケストレーションを抑える（4.0）**  
   - クラシック化／Topicで明示転送／親の自動応答抑止／履歴最小化  
2. **Copilot Studio→Data Agent に渡すトークン（履歴/指示/出力）を削減（4.1）**  
3. **データ側のボトルネック対応（4.2〜4.4）**  
   - View/JOIN負荷の見直し、Warehouse化検討、Semantic model＋Prep for AI（Verified answers）  
4. 中期で、要件に応じて以下を検討  
   - **Data Agent直利用（5.1）**（多段構成を減らしてレイテンシ/トークンを抑える）  
   - **REST APIアクション＋定型クエリ＋キャッシュ（5.2）**（コスト/レスポンスの安定性を優先）  
   - **外部エージェントビルダーの活用（5.4）**（ガバナンス要件とトレードオフ）

---

# 付録A. 効果検証（CU/レスポンス）を“確実に”確認するための計測観点
本対策は「効く可能性が高い」一方、実際の削減幅は質問パターンやデータ側負荷に依存します。  
そのため、以下の比較を推奨します。

- 比較A：現状（Copilot Studio + Data Agent） vs ルーティング専用化（4.0）
- 指標：平均レスポンス秒数、Data Agentのトークン量（可能なら）、クエリ実行時間（SQL側）

参考:
- Data Agent 消費モデル（トークン換算）  
  https://learn.microsoft.com/ja-jp/fabric/fundamentals/data-agent-consumption
