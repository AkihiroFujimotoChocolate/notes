# 仕様書・テスト仕様書の作成ガイド

サービス、API、ライブラリ、ツールの仕様を、実装担当者とテスト担当者が同じ意味に解釈できる形で記述するための実践的なガイド。Markdownを用い、人間にもコーディングエージェントにも読みやすい資料を目指す。

対象の契約とチームの規約に合わせて章構成・設計・試験方法を選ぶ。完成例は架空のメモ登録APIの仕様書・テスト仕様書であり、実装・試験の実績を示すものではない。

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

個別の記述例には、完成例とは独立した契約・構成も含む。対象に合う図表や列を選ぶ。

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
