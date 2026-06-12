# レビュー: survey_report round 2
> 検査日: 2026-06-12
> 検査対象: 00-summary.md / 01-technical.md / 13-confirm-items.csv

---

## R01 [high] 00-summary.md に技術用語が含まれないか

### 判定: PASS

**許容: 「Java」のみ**

00-summary.md 全文を精読した結果、以下を確認した。

| 禁止用語 | 記載有無 |
|---------|---------|
| Stripes | なし |
| MyBatis | なし |
| Spring / Spring MVC | なし |
| HSQLDB | なし |
| javax / jakarta | なし |
| Tomcat | なし |
| Maven | なし |
| IBM Liberty | なし |
| WildFly | なし |
| Jakarta EE | なし |

許容された「Java」は「Java の標準仕様の世代交代」「Java の標準仕様」という形で使用されており問題なし。

根拠 CSV への言及はすべて「03-libraries.csv」「07-framework.csv」等のファイル名のみで、製品名は記載されていない。

**指摘事項: なし**

---

## R02 [high] 数値が CSV から引用でき出所が明記されているか

### 判定: PASS

#### 00-summary.md の ask_to 別件数 vs 13-confirm-items.csv の実カウント

| ask_to | 報告書記載 | CSV 実カウント | 一致 |
|--------|-----------|---------------|------|
| 顧客 | 13 件 | 13 件 | OK |
| 技術検証 | 14 件 | 14 件 | OK |
| 運用 | 3 件 | 3 件 | OK |
| 合計 | 30 件 | 30 件 | OK |

#### 顧客 13 件の内訳（13-confirm-items.csv より）
行番号2,3,4,5,6,7,9,10,11,13,14,20,21（ask_to=顧客）= 13件

#### 技術検証 14 件の内訳（13-confirm-items.csv より）
行番号8,12,15,16,17,18,24,25,26,27,28,29,30,31（ask_to=技術検証）= 14件

#### 運用 3 件の内訳（13-confirm-items.csv より）
行番号19,22,23（ask_to=運用）= 3件

#### 出所明記の確認
- 00-summary.md の確認事項セクションに「全 30 件の詳細は `data/13-confirm-items.csv` にある」と明記されている
- 01-technical.md の §13 に「全 CSV の `needs_confirm=yes` 行から集約した確認事項は 30 件。詳細は `data/13-confirm-items.csv` を参照」と明記されている
- ask_to 別件数の出所として「13-confirm-items.csv」が明記されている

**指摘事項: なし**

---

## R03 [high] 13-confirm-items.csv が全 CSV の needs_confirm=yes 行を漏れなく集約しているか

### 判定: PASS

#### 各上流 CSV の needs_confirm=yes 行数

| 上流 CSV | needs_confirm=yes 行数 | 主な該当行 |
|---------|----------------------|-----------|
| 00-asset-check.csv | 5 件 | batch-shell / library / db / deploy-env / document |
| 03-libraries.csv | 1 件 | spring-batch-infrastructure |
| 08-runtime-impact.csv | 6 件 | language-version(×2) / server-api(liberty/wildfly) / library-compat(spring-batch) / encoding(pom.xml mybatis-parent) |
| 09-db.csv | 6 件 | ORDERSTATUS(timestamp) / insertOrderStatus / encoding(HSQLDB) / encoding(SQL) / AccountMapper_cache / SequenceMapper_cache |
| 10-platform.csv | 5 件 | mvnw(MVNW_REPOURL) / maven-wrapper.properties / pom.xml(deploy) / support.yaml(Liberty) / encoding(pom.xml mybatis-parent) |
| 12-impact-matrix.csv | 14 件 | JSTL localization / Tomcat9 / APサーバー選択 / Liberty・WildFly / HSQLDB置換 / ORDERSTATUS.timestamp / SEQUENCEテーブル名 / SequenceMapper採番キャッシュ / AccountMapperキャッシュ / Java17 / Mavenリポジトリ / mybatis-parent:52 / バッチ処理 / 業務設計書 |
| 04-unresolved.csv | 4 件 | OrderActionBean#newOrderForm / Order#initOrder / beans.xml / net.sourceforge.stripes |
| **合計** | **41 件** | |

#### 集約後: 13-confirm-items.csv = 30 件

上流 41 件 → 13-confirm-items.csv 30 件への差分 11 件は、複数 CSV の同一テーマを 1 行に統合（origin 列で複数出所を `/` 区切りで明記）していることで説明できる。

主な統合例:
- HSQLDB 置換（12-impact-matrix.csv）と本番DB置換要否（00-asset-check.csv:db + 09-db.csv）→ 1 件に統合
- SequenceMapper 採番キャッシュ（12-impact-matrix.csv + 09-db.csv）→ 1 件に統合
- バッチ処理の有無（12-impact-matrix.csv + 00-asset-check.csv:batch-shell）→ 1 件に統合
- 業務設計書（12-impact-matrix.csv + 00-asset-check.csv:document）→ 1 件に統合
- APサーバー確定（12-impact-matrix.csv:Tomcat9 + 12-impact-matrix.csv:APサーバー選択 + 00-asset-check.csv:deploy-env）→ 1 件に統合
- Maven リポジトリ URL（12-impact-matrix.csv + 10-platform.csv）→ 1 件に統合
- mvnw 環境制限（10-platform.csv:mvnw + 10-platform.csv:maven-wrapper.properties）→ 1 件に統合

#### origin 列の出所明記確認

13-confirm-items.csv 全 30 行について origin 列を確認した結果、すべての行で上流 CSV 名（ファイル名 + 行キーまたは列名）が明記されており、トレーサビリティが確保されている。

04-unresolved.csv 由来の 4 件については origin 列に「04-unresolved.csv:（ソース名）行（needs_confirm=yes）（追加調査: 04-unresolved.csv由来）」と明記されており確認できた（行28〜31）。

**指摘事項: なし**

---

## まとめ

| ID | severity | verdict | 詳細 |
|----|---------|---------|------|
| R01 | high | **pass** | 00-summary.md から禁止技術用語をすべて除去済み。「Java」のみ残存（許容範囲内） |
| R02 | high | **pass** | ask_to 別件数（顧客13/技術検証14/運用3）が 13-confirm-items.csv の実カウントと完全一致。出所明記あり |
| R03 | high | **pass** | 上流 41 件を 30 件に統合集約。重複統合は origin 列の複数出所明記により追跡可能。漏れなし |

round 1 で指摘された R01/R02/R03 の全観点が適切に修正・対応されていることを確認した。
