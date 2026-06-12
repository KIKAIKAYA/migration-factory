# 実例: JPetStore 6 現状調査

[survey-unit](../../survey-unit/) を [flowsmith](https://github.com/KIKAIKAYA/flowsmith) で
実行した、そのままの出力。対象は [mybatis/jpetstore-6](https://github.com/mybatis/jpetstore-6)
（Stripes + Spring + MyBatis + JSP / Apache-2.0 / 160 ファイル・15,007 行）。

- 実行: Claude Code（Sonnet 4.6）、コールドスタート（事前文脈なし）、全 15 ステップ完走 約 35 分
- 中断・再開 2 回（実負荷でのチェックポイント再開の実証）
- レビューループの実動: `impact_matrix` と `survey_report` が round 1 不合格 → 差し戻し → round 2 合格
  （経緯は [reviews/](reviews/) と [.flowsmith/run.jsonl](.flowsmith/run.jsonl) に証跡）

## 読み方

| 場所 | 内容 |
|------|------|
| [report/00-summary.md](report/00-summary.md) | 意思決定者向け報告（技術用語なし・3 ページ） |
| [report/01-technical.md](report/01-technical.md) | 技術者向け報告（数値はすべて data/ から引用） |
| [data/](data/) | 調査データ本体（CSV 17 本。スキーマは [契約](../../survey-unit/contracts/data-dictionary.md)） |
| [flows/](flows/) | 処理フロー詳細（Mermaid 図付き） |
| [reviews/](reviews/) | checker のレビュー表（全ラウンド分） |
| [.flowsmith/run.jsonl](.flowsmith/run.jsonl) | 実行ログ（再開・差し戻し・ゲート不合格の記録） |

## 数字でみる出力

- 棚卸し 160 ファイル → 分類 160 行（1:1、設定ファイルの取りこぼしゼロ）
- 依存関係 143 行 — うち間接依存: config 15 / annotation 14 / dispatch 12 / sql 12 / taglib 3
  （AST だけでは見えない、設定・注釈が作る配線）
- 証拠区分: fact 762 / inferred 43 / 確認事項 30 件（[13-confirm-items.csv](data/13-confirm-items.csv)）

## 主な発見（報告書より）

1. Stripes 1.6.0 の開発停止が他のすべての移行作業のブロッカー（risk: high）
2. 採番キャッシュ重複リスク — 本番 DB 移行前の優先対処が必要（risk: high）
3. 確認事項 30 件を顧客提示可能な一覧に集約

> 注: 調査対象のソースコードはこのリポジトリに含めていません（上記リンクから取得可能）。
> ここにあるのは survey-unit が生成した調査成果物のみです。
