# migration-factory

レガシーシステムの**現状調査・マイグレーション評価**を自動化するワークフロー・Skill 群。
レガシー移行の「工場」——調査（分析ユニット）から始まり、将来は再構築まで。

「2025年の崖」を越えるための最初の一歩——「このシステム、中身どうなってるの？」に答える作業を、AI エージェントに任せます。

## なにができるか

> 📊 **実例**: [examples/jpetstore/](examples/jpetstore/) — JPetStore 6 を実際に調査した出力一式
> （CSV 17 本 + 二層報告書 + 実行ログ。コールドスタート 35 分・全 15 ステップ完走）

### survey-unit（推奨）— flowsmith 駆動の調査ワークフロー

[survey-unit/](survey-unit/) は、現状調査の全 15 工程（資産棚卸し → 分類 → 依存抽出 →
影響マトリクス → 検収 → 二層報告書）を 1 つの YAML で宣言したワークフロー定義。
[flowsmith](https://github.com/KIKAIKAYA/flowsmith) エンジンで実行し、
**全成果物が列定義済みの CSV データ**として出る（報告書はその派生ビュー）。
中断しても続きから再開できる。

### 単体 Skill（個別利用向け）

| Skill | 役割 |
|-------|------|
| `legacy-survey` | **調査ワークフロー全体のオーケストレーター**。下記 Skill を順に実行し、最終報告書まで出す |
| `tech-inventory` | 技術スタックの棚卸し（言語/フレームワーク/ビルド/バージョン/EOL チェック） |
| `structure-map` | 構造と依存関係の可視化（モジュール構成・エントリポイント・レイヤー分析） |
| `db-migration-assess` | データ層の調査と移行評価（Oracle → PostgreSQL 互換性チェックポイント） |
| `survey-report` | As-Is 調査報告書の生成（日本語・経営層/開発者の両向け） |

## 使い方

```bash
# Claude Code のプラグインとしてインストール
/plugin install <このリポジトリ>

# または skills/ を手動で ~/.claude/skills/ にコピー

# 調査対象のリポジトリで実行
claude
> /legacy-survey
```

個別の Skill だけ使うこともできます（例：DB 移行評価だけ欲しい → `/db-migration-assess`）。

## 設計方針

- **モデル非依存**：Skill は Markdown の手順書。特定モデルにロックインしない
- **手順はすべて実務由来**：レガシーマイグレーション現場での調査手法を一般化・再構成したもの。特定の組織・案件の情報は一切含まない
- **報告書ファースト**：調査の終着点は「人が読んで意思決定できる日本語の報告書」。中間生成物もすべて Markdown で残す

## 方法論

調査の考え方（なぜこの順番か、何を見るか）は [docs/methodology.md](docs/methodology.md) を参照。

## ステータス

🚧 **v0.1 — 骨格構築中。** Skill の手順は随時、実務知見を反映して深化させます。

## License

MIT © kikaikaya
