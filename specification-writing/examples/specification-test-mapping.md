# メモ登録API v1 仕様・テスト対応表

## 1. 対象・参照資料

メモ登録API v1。外部と内部を別文書にする場合と一つの文書にする場合の対応を併記する。確認方法は計画例。

| 資料 | 参照範囲 |
|---|---|
| [外部仕様](external-specification.md)・[内部仕様](internal-specification.md) | 外部・内部契約 |
| [外部テスト](external-test-specification.md)・[内部テスト](internal-test-specification.md) | 外部・内部の試験 |
| [仕様書](specification.md)・[テスト仕様書](test-specification.md) | 外部と内部を一つの文書に記載する場合 |

ケースIDは[テスト仕様書3章](test-specification.md#3-テストケース)でも共通。

## 2. 外部仕様との対応

| 仕様書の条項（別文書／一つの文書） | 規定内容 | 対応する節・テストケース |
|---|---|---|
| 3.1／3.1 | POST `/notes`、JSON | [外部-01](external-test-specification.md#external-01)（一部自動：正常経路）。JSON object以外の本文は未対応 |
| 3.2／3.2 | title、contentの入力 | [外部-01](external-test-specification.md#external-01)・[外部-02](external-test-specification.md#external-02)（自動） |
| 3.3／3.3 | 入力検証、追加項目 | 外部-01-02・[外部-02](external-test-specification.md#external-02)（一部自動）。JSON object以外の本文は未対応 |
| 3.4／3.4 | HTTPステータス・結果本文 | [外部-01](external-test-specification.md#external-01)〜[外部-03](external-test-specification.md#external-03)（自動） |
| 4章／4章 | 利用インターフェース | 該当なし |
| 5.1〜5.2／同左 | 正常・異常の処理分岐 | 外部-01〜03（自動） |
| 5.3／5.3 | UUID・文字列保持・応答ID | [外部-01](external-test-specification.md#external-01)（一部自動）。登録ごとのIDの相違と実保存の永続化は未対応（別途確認予定） |
| 5.4／5.4 | 保存1回、失敗時の処理終了 | 外部-01・外部-03（自動） |
| 6.1〜6.2／同左 | 異常時の応答 | 外部-02・外部-03（自動） |
| 6.3／6.3 | 処理期限・切断 | 未規定 |
| 7章／7章 | 外部設定 | 該当なし |

## 3. 内部仕様との対応

| 仕様書の条項（別文書／一つの文書） | 規定内容 | 対応する節・テストケース |
|---|---|---|
| 2.2／2.2 | 実装環境 | プロジェクト設定・ビルド確認（未実施） |
| 3.1〜3.2／8.1〜8.2 | 責務・構造 | 本体レビュー：受付、登録、保存の分担 |
| 3.3／8.3 | 依存方向 | [内部-03](internal-test-specification.md#internal-03)（一部自動）、本体レビュー：具象保存への依存の有無 |
| 4.1／9.1 | 内部宣言 | 内部-01のコンパイル・直接呼出しと本体レビュー（宣言全体） |
| 4.2〜4.3／9.2〜9.3 | 入力と結果 | [内部-01](internal-test-specification.md#internal-01)（自動） |
| 4.4／9.4 | 取消の伝播 | [内部-02](internal-test-specification.md#internal-02)（自動） |
| 5.1／10.1 | Scoped登録 | 内部-03-01（自動） |
| 5.2／10.2 | 保存実装の差替え | 内部-03-02（部分確認）、本体レビュー：登録変更時の依存、外部テストの代役構成 |
