# layer_map レビュー Round 1

検査日: 2026-06-12
検査対象:
- `survey-out/data/02-layer-map.csv`（161件）
- `survey-out/data/01-inventory.csv`（161件）

---

## レビュー表

| 観点 | 重大度 | 判定 | 根拠（引用） | 修正指示 |
|------|--------|------|-------------|---------|
| S01 | high | pass | inventory.csv データ行 161件（行2〜162）、layer-map.csv データ行 161件（行2〜162）。全パスが1対1で一致。資産種別カバレッジ: source（Java・JSP・HTML）、config（XML・properties）、sql（3件）、shell（mvnw・mvnw.cmd）、build（pom.xml）、deploy（Dockerfile・docker-compose・GitHub Actions 8件）、document（LICENSE等）をすべて収録。棚卸しとの件数完全一致を確認。 | - |
| S02 | high | pass | evidence 列（第5列）が空の行はゼロ。confidence=inferred の行（format.xml 行6、domain/*.java 行29〜37、mapper/*.java 行38〜44、service/*.java 行45〜47、web/actions/*.java 行49〜52）はいずれも「パッケージ配置・参照元クラス・フレームワーク登録」などの具体的根拠を記載。例: 行38「src/main/java/org/mybatis/jpetstore/mapper/ パッケージ配置。MyBatis Mapper インターフェース。DB操作定義」。confidence=fact の行はファイルパス・行数・設定ファイル参照先など直接確認可能な根拠を記載。 | - |
| S03 | mid | pass | layer-map 全161行に domain=unknown / layer=unknown / asset_type=unknown の行は存在しない。したがって「unknown 行が needs_confirm=yes になっているか」の検査対象はゼロ件であり、強引な unknown 隠蔽（unknown を別値に上書きして needs_confirm=no にする）も確認されない。confidence=inferred の行は全件 needs_confirm=no だが、これはパッケージ構造・フレームワーク設定からの合理的推論であり、不当な強引分類には該当しない。 | - |

---

## 総評

全3観点 pass。layer-map は棚卸し161件を完全収録し、全行に evidence を備え、unknown 行の残留も強引な分類も認められない。品質上の問題は検出されなかった。
