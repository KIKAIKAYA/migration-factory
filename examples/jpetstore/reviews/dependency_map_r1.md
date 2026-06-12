# 依存関係抽出 チェックレビュー — dependency_map / round 1

検査日: 2026-06-12
検査対象:
- `survey-out/data/04-dependencies.csv` (144 データ行)
- `survey-out/data/04-unresolved.csv` (15 データ行)

---

## レビュー表

| 観点 | 重大度 | 判定 | 根拠（引用） | 修正指示 |
|------|--------|------|-------------|---------|
| D01 | high | **PASS** | 直接依存（call/extends/implements）と間接依存（annotation/config/taglib/library/dispatch/file）の両方が網羅されている。extends: 行2-5、implements: 行6-7、call: 行17-66、annotation: 行8-16/35-39、config: 行67-97、taglib: 行98-100、library: 行122-143、dispatch(セッション経由間接参照): 行33-34 | なし |
| D02 | high | **PASS** | 全 144 行に `evidence` 列が存在し、いずれも `src/main/java/.../ClassName.java:行番号` または `src/main/webapp/WEB-INF/web.xml:行番号`、`pom.xml:行番号` 等のファイルパス＋行番号形式で記録されている。例: `src/main/java/org/mybatis/jpetstore/web/actions/AccountActionBean.java:43`（行2）、`src/main/webapp/WEB-INF/applicationContext.xml:43`（行86）、`pom.xml:87`（行137）。`confidence` 列はすべて `fact`。 | なし |
| D03 | mid | **PASS** | 未解決依存が 04-unresolved.csv に 15 件記録されており、それぞれ `reason` 列に理由が明記されている。例: Stripes内部実装依存でセッションキー静的追跡不可（行2）、Order.initOrder省略読み込みによるCart→Orderデータフロー未確認（行3）、JSP未読による内部参照詳細未確認（行4-14）、beans.xmlのCDI動作環境依存（行15）、Stripes 1.6.0のJakarta EE互換未確認（行16）。`needs_confirm` 列で yes/no による確認優先度も付与されており、黙って捨てていない。 | なし |

---

## 総評

- 全 3 観点ともに合格（PASS）。
- 144 行の依存エントリはレイヤー横断的（Web/Service/Mapper/Config/JSP/pom.xml）に整理されており、dep_type の種別（call/extends/implements/annotation/config/taglib/library/dispatch/file/sql）の多様性も十分。
- 未解決 15 件はすべて「読み込み省略」「フレームワーク内部依存」「環境依存」という実態に即した理由が付されている。
- 軽微な留意点: `dispatch` 型の間接依存（行33-34、行101-104 等）は dep_type が複数名称（`dispatch` / `file`）に分散しているが、現時点の検査観点 D01 の判定には影響しない。次回ラウンドで dep_type の統一基準が追加される場合は再確認を推奨する。
