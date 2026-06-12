# flow_trace レビュー — Round 1

検査日: 2026-06-12
検査対象:
- survey-out/data/11-flows.csv
- survey-out/flows/FLOW-001.md
- survey-out/flows/FLOW-002.md
- survey-out/data/04-dependencies.csv（突き合わせ用）

---

## レビュー表

| 観点 | 重大度 | 判定 | 根拠（引用） | 修正指示 |
|------|--------|------|-------------|---------|
| F01 | high | pass | FLOW-001.md ステップ表 #1〜#19 が入口（`index.html:32 <a href="actions/Catalog.action">`）から StripesFilter（`web.xml:40-63`）→ CatalogActionBean（`CatalogActionBean.java:143-146`）→ CatalogService（`CatalogService.java:59-61`）→ ProductMapper（`ProductMapper.java:30`）→ ProductMapper.xml（`ProductMapper.xml:36-44`）→ PRODUCT テーブルまで 20 ステップ断絶なく追跡されている。FLOW-002.md も Cart.jsp（`Cart.jsp:86`）から LINEITEM テーブル（`LineItemMapper.xml:37`）まで 23 ステップで完結。最低 1 本（FLOW-001）は入口→DB まで完全に追跡済み。 | - |
| F02 | high | pass | 全ステップに具体的な evidence が付与されている。例: FLOW-001 ステップ#2「`web.xml:40-63`」、#3「`CatalogActionBean.java:143-146`」、#7「`ProductMapper.xml:36-44`」、#10「`CategoryMapper.xml:26-33`」、#18「`ItemMapper.xml:47-68`」。FLOW-002 でも #10「`OrderService.java:59-77`」、#12「`SequenceMapper.xml:26-29`」、#16「`OrderMapper.xml:93-101`」等、各ホップにファイルパスと行番号が記載されている。04-dependencies.csv と突き合わせると、CatalogActionBean→CatalogService の呼出（`CatalogActionBean.java:155,156,168,169,180`）、OrderService→各 Mapper 呼出（`OrderService.java:67,71,72,75,122,128`）がいずれもフロー文書の evidence と一致している。 | - |
| F03 | mid | pass | 11-flows.csv の binding 列で FLOW-001 が `config-driven`、FLOW-002 が `mixed` と区別されている。FLOW-001.md ステップ表の「呼出種別」列では StripesFilter 解決・@DefaultHandler・MyBatis マッパー XML バインドを `config-driven`、CatalogActionBean→CatalogService・CatalogService→各 Mapper の Java 直接呼出を `direct` と明示的に区別している（ステップ#4「`config-driven（Stripesイベント名）`」vs ステップ#5「`direct`」）。FLOW-002.md ステップ#3 では「`config-driven（Stripesイベント名）+ direct（セッション参照）`」と複合パターンを 1 行で区別記載している。また Mermaid 図中でも各矢印に `[config-driven: Stripesイベント名]` と `[direct]` のラベルが付与されている。 | - |

---

## 総評

3 観点すべて pass。入口→DB まで断絶のない追跡（F01）、各ホップの行番号付き evidence（F02）、呼出種別の明示的区別（F03）はいずれも満たされている。
04-dependencies.csv との突き合わせでも、フロー文書の evidence と依存関係データに矛盾は確認されなかった。
