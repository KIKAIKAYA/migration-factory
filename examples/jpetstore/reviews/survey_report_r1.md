# 検査レビュー表 — 調査報告書の生成（survey_report / round 1）

> 検査日: 2026-06-12  
> 検査者: checker  
> 対象: survey-out/report/00-summary.md, 01-technical.md, data/13-confirm-items.csv

---

| 観点 | 重大度 | 判定 | 根拠（引用） | 修正指示 |
|------|--------|------|------------|---------|
| R01 | high | **fail** | 00-summary.md 行73「**IBM Liberty や WildFly 26** を本番で使用しているか」に製品名（ツール名）が明記されている。同行に「**Jakarta** 標準への移行時の動作確認」という技術フレームワーク名も登場している。また行79「**Jakarta** 標準への移行時」も同様。検査仕様では「Stripes, MyBatis, Spring, HSQLDB, javax, jakarta, Tomcat, Maven 等の固有技術名称」を技術用語として不可とする。IBM Liberty・WildFly 26 はツール名に該当する（Tomcat と同類の APサーバー製品名）。Jakarta は技術名称に該当する。なお行31「画面処理に関わるすべてのプログラム部品（4 つの処理クラス）」は説明的表現であり問題なし。 | 00-summary.md 行73の「IBM Liberty や WildFly 26 を本番で使用しているか（これらはすでにサポート終了しており追加対応が必要）」を「（例）現在使用している特定のアプリケーションサーバーがすでにサポート終了しているかどうか」等の一般的表現に置き換える。同行内の「Jakarta EE 互換性」「Jakarta 標準」を「最新のサーバー規格への対応」等の非技術表現に置き換える。 |
| R02 | high | **fail** | 00-summary.md 行75「技術検証で確認（12 件）」・行81「運用担当に確認（4 件）」の数値が 13-confirm-items.csv の実データと一致しない。実際の ask_to 別件数は「技術検証: 14 件、運用: 3 件、ベンダー: 0 件」（PowerShell で csv を集計して確認）。同じ不一致が 01-technical.md 行275-278 にも存在する。なお合計 30 件（00-summary.md 行65）・顧客 13 件（行67）は正確。 | 00-summary.md 行75の「技術検証で確認（12 件）」を「技術検証で確認（14 件）」に修正。行81「運用担当に確認（4 件）」を「運用担当に確認（3 件）」に修正。「ベンダー」項目がある場合は削除する（0件のため）。01-technical.md 行275-278 も同様に修正する。 |
| R03 | high | **fail** | 13-confirm-items.csv（30件）の件数と各 CSV の needs_confirm=yes 行の集計が一致しない。集計結果: 03-libraries.csv=0件、08-runtime-impact.csv=5件、09-db.csv=6件、10-platform.csv=5件、12-impact-matrix.csv=14件、00-asset-check.csv=5件、合計=35件。30件との差（5件）は重複統合によるものだが、統合後の対応関係を確認すると以下の問題がある。(1) 12-impact-matrix.csv 行2「全 JSP・APサーバーデプロイ / Tomcat 9（APサーバー runtime）/ replace / needs_confirm=yes」が 13-confirm-items.csv の row 13（本番APサーバーの最終決定）に統合されているが、当該 row の origin に Tomcat 9 行への参照が明示されておらず、`12-impact-matrix.csv:APサーバー選択行` のみが記載されている。(2) 04-unresolved.csv 由来の4件（rows 27-30）が含まれており、「全 CSV」の範囲が指定 CSV（03,08,09,10,12,00）を超えている（検査範囲外の追加分を含む）。 | (1) 13-confirm-items.csv row 13 の origin に `12-impact-matrix.csv:Tomcat9行（needs_confirm=yes）` を追記して統合根拠を明示する。(2) 04-unresolved.csv は検査指定対象外 CSV のため、その4件の origin 欄に「追加調査: 04-unresolved.csv 由来」と区別できる注記を入れるか、あるいは01-technical.md の集計説明に「指定CSVに加えて 04-unresolved.csv からも集約した」旨を明記する。 |

---

## 判定サマリー

| 観点 | 判定 |
|------|------|
| R01 | fail |
| R02 | fail |
| R03 | fail |

---

## 補足メモ

### R01 詳細: 00-summary.md の技術用語一覧

| 行番号 | 問題箇所 | 指摘分類 |
|--------|---------|---------|
| 行73 | `IBM Liberty や WildFly 26` | ツール名（APサーバー製品名） |
| 行73 | `Jakarta EE 互換性` | 技術フレームワーク名 |
| 行79 | `Jakarta 標準への移行時の動作確認` | 技術フレームワーク名 |

なお以下は許容範囲と判断した：
- 「Java」: 検査仕様で明示的に「一般的な言葉として許容」
- 「07-framework.csv より」等のCSVファイル名参照: 出所明記として許容
- 「4 つの処理クラス」「16 本の画面ファイル」: クラス名・ファイル名ではなく数量表現

### R02 詳細: ask_to 別件数の実データ対照表

| ask_to | 報告書記載 | 実データ（CSV集計） | 一致 |
|--------|-----------|------------------|------|
| 顧客 | 13件 | 13件 | ✓ |
| 技術検証 | 12件 | **14件** | ✗ |
| 運用 | 4件 | **3件** | ✗ |
| ベンダー | 1件 | **0件** | ✗ |
| **合計** | **30件** | **30件** | ✓ |

※ 上記不一致は 00-summary.md 行75/81 および 01-technical.md 行275-278 の両方に存在する。

### R03 詳細: needs_confirm=yes 件数の突き合わせ

| CSV | needs_confirm=yes 件数 | 13-confirm-items.csv での扱い |
|-----|----------------------|----------------------------|
| 03-libraries.csv | 0件 | 対象なし（全行 needs_confirm=no） |
| 08-runtime-impact.csv | 5件 | rows 6-10 として全件集約済み ✓ |
| 09-db.csv | 6件 | rows 15/17/18/24/25/26 に集約済み（一部 12-impact-matrix と統合）✓ |
| 10-platform.csv | 5件 | rows 8/13/19/22/23 に集約済み（一部統合）✓ |
| 12-impact-matrix.csv | 14件 | rows 11-14/17-21/その他統合で概ね集約、ただし row 2 の Tomcat 9 行の origin 記載不備あり △ |
| 00-asset-check.csv | 5件 | rows 1-5/20/21 として集約済み（一部統合）✓ |
| **合計（指定CSV）** | **35件** | 重複統合5件込みで30件に圧縮（合理的） |
| 04-unresolved.csv | （検査対象外） | rows 27-30 として追加集約（検査対象外CSVからの追加） |
