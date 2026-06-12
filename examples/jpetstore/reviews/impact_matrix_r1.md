# 影響マトリクス 検査レビュー（round 1）

**対象**: `survey-out/data/12-impact-matrix.csv`
**検査日**: 2026-06-12
**検査観点**: M01 / M02 / M03

---

## レビュー表

| 観点 | 重大度 | 判定 | 根拠（引用） | 修正指示 |
|------|--------|------|-------------|---------|
| M01 | high | **fail** | 下記 6 行のbasis/evidence が、検査対象5上流CSV（07/03/08/10/05）に存在しない `09-db.csv` または `00-asset-check.csv` のみを根拠とする。追跡不可。<br>**行13** (ORDERSTATUS.timestamp): `basis="09-db.csv:ORDERSTATUS行"` のみ。07/03/08/10/05 のいずれにも ORDERSTATUS.timestamp に言及する行は存在しない。<br>**行14** (SEQUENCE予約語): `basis="09-db.csv:SEQUENCE行; 09-crud.csv:SequenceMapper行"` のみ。同上。<br>**行15** (MyBatis L2 Cache SequenceMapper): `basis="09-db.csv:SequenceMapper_cache行(risk=mid)"` のみ。08-runtime-impact.csv に SequenceMapper キャッシュへの言及なし（12-impact-matrix の記述自体は03-libraries.csv:mybatis行のrisk=lowと矛盾する可能性もあり）。<br>**行16** (MyBatis L2 Cache AccountMapper): `basis="09-db.csv:AccountMapper_cache行"` のみ。同上。<br>**行31** (旧式JOIN構文): `basis="09-db.csv:getAccountByUsername行・getItem行・getOrder行"` のみ。上流5CSVに旧式JOIN構文への言及なし。<br>**行32** (業務設計書不在): `basis="00-asset-check.csv:document行"` のみ。00-asset-check.csv は上流指定5CSVに含まれない。 | 行13/14/15/16/31/32 の各行について、07/03/08/10/05 のいずれかに実在する行を basis として追記するか、対応する上流 CSV（09-db.csv 等）を上流リストに追加してトレーサビリティを確立すること。上流リスト追加が正当な場合は次ラウンドで再確認する。 |
| M02 | high | **pass** | risk=high の全9行（行2/3/4/8/9/12/15）について policy 列に対応方針案が記述されている。<br>行2 (Stripes, risk=high): `policy="Stripes を Spring MVC（+ Thymeleaf または JSP 継続）または Jakarta Faces 等の Jakarta EE 対応フレームワークへ全面置換する。…"` — 方針案あり。<br>行3 (spring-web, risk=high): `policy="Stripes 置換と同時に spring-web を 6.x へアップグレードする。…EOL のため脆弱性対応が受けられないリスクを PM へ周知する。"` — 方針案あり。<br>行4 (javax→jakarta 名前空間, risk=high): `policy="javax.servlet.* を jakarta.servlet.* へ一括置換する…"` — 方針案あり。<br>行8 (jakarta.servlet-api 4.0.4, risk=high): `policy="Stripes 置換後に jakarta.servlet-api を 5.0+ へアップグレードし…"` — 方針案あり。<br>行9 (Tomcat 9, risk=high): `policy="Stripes 置換・jakarta 名前空間移行完了後に Tomcat 10+ へ切り替える。…"` — 方針案あり。<br>行12 (HSQLDB, risk=high): `policy="移行先 DB を決定し、DataSource Bean を外部 DB 接続設定に差し替える。…"` — 方針案あり。<br>行15 (MyBatis L2 Cache SequenceMapper, risk=high): `policy="SequenceMapper.xml の <cache/> を削除してキャッシュを無効化する。…"` — 方針案あり。<br>（注: 行13/14/16 は risk=mid のため M02 対象外。） | - |
| M03 | mid | **pass** | action=none の全4行（行17/18/22/25）について note 列に理由・根拠が記述されている。<br>行17 (MyBatis, action=none): `note="置換影響は大きいが現時点での置換必要性はない。DB 移行後も MyBatis を継続使用することを前提に SQL の方言差異確認を行う。"` — risk=low の根拠（`03-libraries.csv:mybatis行(risk=low)`）も basis に明記。<br>行18 (Spring Framework, action=none): `note="spring-context/spring-jdbc は 6.x 対応済みのため対応不要。spring-web のみ 5.3.x 固定という状態を解消すること。"` — risk=low の根拠（`03-libraries.csv:spring-context/spring-jdbc行(risk=low)`）も basis に明記。<br>行22 (IE5/Mac CSS, action=none): `note="モダンブラウザでは voice-family ハックは無視されるため機能影響なし。コードクリーンアップの一環として対処。"` — 機能上の問題がない理由を明記。<br>行25 (JAVA_HOME, action=none): `note="設定済み環境では問題なし。新規セットアップ時の手順整備として対処。"` — 問題が顕在化しない条件を明記。 | - |

---

## 補足メモ

- **M01 fail の影響範囲**: 行13/14/15/16/31/32 の 6 行（全 31 行中）がトレーサビリティ未確立。行13/14/15/16 は DB 移行に関わる重要な指摘（予約語衝突・採番重複リスク）を含むため、次ラウンドで 09-db.csv を上流リストに追加するか、08-runtime-impact.csv など既存上流CSVに対応エントリを追加して追跡可能にすることを推奨する。
- **行15 の risk=high 昇格の根拠**: `09-db.csv:SequenceMapper_cache行(risk=mid)` は mid だが 12-impact-matrix では high に昇格されている。昇格理由は note 欄に「採番重複は注文データの整合性を破壊する可能性があり高リスクと判断」と説明されているが、上流CSV追跡が確立された後に正式確認が必要。
