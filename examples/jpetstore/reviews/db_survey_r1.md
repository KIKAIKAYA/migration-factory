# db_survey レビュー — round 1

検査日: 2026-06-12
検査者: checker（自動）
対象成果物:
- `survey-out/data/09-db.csv`
- `survey-out/data/09-crud.csv`
- `target-repo/src/main/resources/org/mybatis/jpetstore/mapper/*.xml`（7ファイル）

---

## レビュー表

| 観点 | 重大度 | 判定 | 根拠（引用） | 修正指示 |
|------|--------|------|-------------|---------|
| B01 | high | **pass** | 実XMLを全件照合。AccountMapper(8件), CategoryMapper(2件), ItemMapper(4件), LineItemMapper(2件), OrderMapper(4件), ProductMapper(3件), SequenceMapper(2件)、合計25 SQL IDがすべて09-db.csvのobject_name列に収録されている。09-crud.csvでもMapper/Serviceレイヤー双方の CRUD 操作が記録されており網羅性に欠落なし。 | なし |
| B02 | high | **partial-pass** | カンマ結合（旧式JOIN）・lower()関数・MyBatis `<bind>` OGNL変換・ORDERSTATUS.timestamp列名予約語リスク・SEQUENCEテーブル名予約語リスク・jdbcType=NUMERIC型不整合については specific 列に根拠付きで記載されている。ただし以下1点が specific 列に昇格されておらず note 列にとどまっている。**未昇格事項**: `OrderMapper.xml:105` の `insertOrderStatus` で `LINENUM` カラムへのバインドに `#{orderId,jdbcType=NUMERIC}` を誤用（LINENUM と ORDERID の両カラムに同じ orderId パラメータを使用）。これはデータ不整合を引き起こしうる論理バグであり、specific 列に `"LINENUM列へのorderId二重バインド（論理バグ）"` として明示すべき移行・テスト必須リスクである。 | 09-db.csvの `insertOrderStatus` 行のspecific列を `"jdbcType=NUMERIC/TIMESTAMP/VARCHAR指定; LINENUM列へorderId二重バインド（論理バグ：LINENUMに常にORDERIDと同値がセットされる）"` に修正し、risk列を `mid` → `high` に引き上げることを推奨。needs_confirmは現状`yes`のまま維持可。 |
| B03 | mid | **fail** | 09-db.csv 全行を精査したが、ストアドプロシージャ・トリガ・ビュー・DBファンクション等のDB内ロジックに言及するレコードが存在しない。スキーマSQLファイル（`jpetstore-hsqldb-schema.sql`）にはCREATE TABLE/INDEXのみが記録されており、DB内ロジックの「調査実施・非存在の確認」を示す明示的な記述が成果物にない。 | 09-db.csvに1行追加すること。推奨フォーマット: `source=src/main/resources/database/jpetstore-hsqldb-schema.sql, object_kind=survey, object_name=stored_proc_trigger_view, specific=none, risk=none, evidence=jpetstore-hsqldb-schema.sql全体, confidence=fact, needs_confirm=no, note="ストアドプロシージャ・トリガ・ビュー・DBファンクションは存在しない。スキーマはCREATE TABLE/INDEX/FK制約のみで構成される"` |

---

## 補足メモ

- B02のLINENUM二重バインド問題: `OrderMapper.xml:105` を実際に確認すると `INSERT INTO ORDERSTATUS (ORDERID, LINENUM, TIMESTAMP, STATUS) VALUES (#{orderId,jdbcType=NUMERIC}, #{orderId,jdbcType=NUMERIC}, ...)` となっており、LINENUMに独立したlineNumber等のパラメータではなくorderIdが渡されている。これはjpetstore実装上のknown quirk（注文に対して1ステータスのみ存在する運用でLINENUMを常に注文IDと同値にするという設計）である可能性もあるが、移行時のデータモデル整合性確認が必要なため specific列への明記が望ましい。
- B01・B03の評価は09-crud.csvも参照。SUPPLIER, BANNERDATAテーブルへのMapper経由直接INSERT/UPDATEが存在しない点（初期データ投入SQLを除く）も正しく記録されている。
