# 仕様書・テスト仕様書の作成ガイド

サービス、API、ライブラリ、ツールの仕様を、実装担当者とテスト担当者が同じ意味に解釈できる形で記述するための実践的なガイド。Markdownを用い、人間にもコーディングエージェントにも読みやすい資料を目指す。

ここで示す章構成・設計・試験方法は用途に応じて選択する推奨例である。既存のチーム規約や外部契約に適応して利用する。サンプルは架空のメモ登録APIであり、実在するサービスの仕様ではない。

## ガイド

- [基本原則・文書構成](principles.md)
- [外部仕様書](external-specifications.md)
- [内部仕様書](internal-specifications.md)
- [テスト仕様書](test-specifications.md)
- [仕様・テスト対応表](specification-test-mapping.md)
- [Markdownの記述方法](markdown-style.md)
- [作成・レビュー・改訂](workflow-and-review.md)

## 個別の記述例

- [外部仕様の表・文章](writing-examples/external-specifications.md)
- [内部仕様の表・文章](writing-examples/internal-specifications.md)
- [テスト仕様の表・文章](writing-examples/test-specifications.md)
- [図の記述例（Mermaid）](writing-examples/diagrams.md)

各記述例は対象に合う形式を選ぶための資料である。すべての図表や列を一つの仕様書に設けることを求めるものではない。完成例とは独立した例も含む。

## テンプレートと完成例

| 文書 | テンプレート | サンプル |
|---|---|---|
| 外部仕様書 | [ひな形](templates/external-specification.md) | [記述例](examples/external-specification.md) |
| 内部仕様書 | [ひな形](templates/internal-specification.md) | [記述例](examples/internal-specification.md) |
| 仕様書 | [ひな形](templates/specification.md) | [記述例](examples/specification.md) |
| 外部テスト仕様書 | [ひな形](templates/external-test-specification.md) | [記述例](examples/external-test-specification.md) |
| 内部テスト仕様書 | [ひな形](templates/internal-test-specification.md) | [記述例](examples/internal-test-specification.md) |
| テスト仕様書 | [ひな形](templates/test-specification.md) | [記述例](examples/test-specification.md) |
| 仕様・テスト対応表 | [ひな形](templates/specification-test-mapping.md) | [記述例](examples/specification-test-mapping.md) |

「仕様書」「テスト仕様書」のひな形と記述例は、外部と内部を分けずに一つの文書に記載する場合に使用できる。外部と内部を別文書にする例も同じ契約を扱っている。どちらか一方を正本として採用する。対応表のサンプルには両方の構成への参照を示す。

## 利用手順

1. [基本原則](principles.md)で文書の役割と構成を選ぶ。
2. 対象に適したテンプレートをコピーし、実際の契約を記載する。
3. [記述方法](markdown-style.md)と各文書のガイドを参考にレビューする。
4. 仕様からテスト仕様・対応表を作成し、実装と照合する。

## 適用上の留意点

図表の簡潔化、内部仕様の実装環境表、テスト仕様の共通見出し、版別境界の設計は実務上の推奨例として掲載する。特定の言語、DIコンテナ、テストフレームワークの採用を要求するものではない。
