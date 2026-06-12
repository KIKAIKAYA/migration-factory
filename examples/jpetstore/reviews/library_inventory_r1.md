# library_inventory レビュー — Round 1

- 検査日: 2026-06-12
- 検査対象:
  - `survey-out/data/03-libraries.csv`
  - `target-repo/pom.xml`
- 検査者: checker (flowsmith)

---

## レビュー表

| 観点 | 重大度 | 判定 | 根拠（引用） | 修正指示 |
|------|--------|------|-------------|----------|
| L01 | high | **pass** | pom.xml 宣言依存 23件（`<dependencies>` 全行）＋ `.mvn/extensions.xml` 掲載の maven-profiler 1件 = 計 24件。CSV データ行（ヘッダ除く）も 24行で全件一致。欠落・重複なし。 | なし |
| L02 | high | **pass** | CSV 全 24行の `needs_confirm` カラムがすべて `no`、`confidence` がすべて `fact`。各バージョンは pom.xml の明示値またはプロパティ変数（`${spring.version}=6.2.19` 等）の解決済み値を使用しており、推測埋めは存在しない。 | なし |
| L03 | mid | **pass** | APサーバ関連・Servlet/JSP ランタイム系の各行に移行リスクが具体的に記載されている。主な確認箇所: (1) `spring-web` risk=high「5.3.x は EOL 間近（Spring 5 サポート終了 2024年末）で移行リスク高い」(2) `stripes` risk=high「開発停止状態で後継なし。javax.servlet API 依存のため Jakarta EE 移行に対応不可。移行リスク最高。」(3) `jakarta.servlet-api` risk=mid「Stripes 1.6.0 が javax.servlet 依存のため移行できない」(4) `jakarta.servlet.jsp-api` risk=low「Jakarta EE 10 の 3.x へ移行するとパッケージが変わる」(5) `taglibs-standard-spec/impl` risk=mid「Jakarta EE 移行時は jakarta.servlet.jsp.jstl への置換が必要」 | なし |

---

## 追記所見（参考）

- `spring-web` が意図的に `5.3.39`（Spring 5 系）で固定されており、他の Spring モジュールが `6.2.19` であることから、バージョン混在が移行作業の最大ボトルネックである点が CSV の note/risk で正確に表現されている。
- `spring-batch-infrastructure` は Liberty APサーバの OSGi 回避策という特殊な用途であり、pom.xml コメントと CSV の note が一致している。
- Java ランタイムバージョン（`java.version=17`）は CSV には記載対象外だが、pom.xml の `<properties>` に明示されており、必要に応じて別途 runtime_inventory で捕捉すべき項目。
