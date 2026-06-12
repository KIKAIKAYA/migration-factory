# 技術調査報告書 — JPetStore

> 調査日: 2026-06-12  
> 対象: mybatis/jpetstore-6（ローカルクローン）  
> 本報告書の数値はすべて `survey-out/data/` の CSV から引用する。出所は各項目に明記する。

---

## 1. 資産棚卸し

**出所: 01-stats.csv, 01-inventory.csv, 00-asset-check.csv**

| 言語 / 種別 | ファイル数 | 総行数 |
|------------|-----------|-------|
| Java       | 43        | 5,381 |
| XML        | 22        | 3,129 |
| other      | 64        | 3,320 |
| JSP        | 20        | 1,472 |
| shell      | 2         | 551   |
| SQL        | 3         | 550   |
| CSS        | 2         | 363   |
| HTML       | 2         | 221   |
| properties | 2         | 20    |

- **業務ソース (本番)**: Java 24 クラス（domain 9 / mapper 7 / service 3 / web/actions 5）
- **テストコード**: Java 18 クラス
- **画面**: JSP 16 本 + 静的 HTML 2 本 + CSS 1 本
- **DB SQL**: スキーマ DDL + 初期データ + データロード SQL（各 1 本）+ MyBatis Mapper XML 7 本
- **資産欠落**: 業務バッチクラス・シェル未発見（00-asset-check.csv:batch-shell=partial）、業務設計書なし（00-asset-check.csv:document=partial）
- **ライブラリ実体**: WEB-INF/lib なし。Maven 依存管理で取得（00-asset-check.csv:library=missing）

---

## 2. レイヤー分類

**出所: 02-layer-map.csv**

| ドメイン     | 主なレイヤー | 代表資産 |
|-------------|------------|---------|
| online-ui   | app        | web/actions/*.java, WEB-INF/jsp/**/*.jsp |
| online-logic| app        | service/*.java, mapper/*.java |
| cross       | app        | domain/*.java（両ドメインから参照） |
| cross       | framework  | WEB-INF/applicationContext.xml, web.xml, beans.xml |
| cross       | build/infra | pom.xml, Dockerfile, .github/workflows/* |

---

## 3. ライブラリ

**出所: 03-libraries.csv**

### リスク高 (risk=high)

| ライブラリ | バージョン | リスク内容 |
|-----------|----------|-----------|
| stripes   | 1.6.0    | 開発停止（2015年頃）。javax.servlet 依存のため Jakarta EE 移行不可。後継なし |
| spring-web | 5.3.39  | Spring 5 EOL（2024年末）。spring-context 6.2.x と混在。pom.xml コメント「Keep spring-web at 5.3.39 until jakarta upgrade occurs」で意図的に固定 |

### リスク中 (risk=mid)

| ライブラリ | バージョン | リスク内容 |
|-----------|----------|-----------|
| taglibs-standard-spec | 1.2.5 | JSTL 1.2 / javax.servlet.jsp 依存。Jakarta EE 移行時に Jakarta JSTL 3.x への置換が必要 |
| taglibs-standard-impl | 1.2.5 | 上記と同様 |
| hsqldb | 2.7.4 | デモ/開発用インメモリ DB。本番 DB 置換が前提 |
| spring-batch-infrastructure | 5.2.6 | Liberty APサーバーの OSGi 問題回避目的。Liberty 不使用の場合は不要（needs_confirm） |

### リスク低・なし（テストスコープ含む）

- mybatis 3.5.19、mybatis-spring 3.0.5、spring-context 6.2.19、spring-jdbc 6.2.19 — アクティブ開発中・問題なし
- JUnit5、Mockito、AssertJ、Selenide、htmlunit3-driver — テストスコープのみ

---

## 4. 依存関係

**出所: 04-dependencies.csv**

- 依存行数合計: 145 行
- Stripes 関連依存: 20 行（implements=1 / annotation=9 / config=2 / library=7 / taglib=1）
- Spring 関連依存: 13 行（annotation=5 / library=8）
- MyBatis 関連依存: 17 行（Mapper XML config=7 / library=7 / pom=1 / mybatis-spring=2）
- Service→Mapper 呼び出し: 27 行

### 未解決依存 (04-unresolved.csv)

| 未解決箇所 | 理由 | needs_confirm |
|-----------|------|---------------|
| OrderActionBean#newOrderForm — セッション属性キー | Stripes 実装依存で静的追跡不可 | yes |
| Order#initOrder — Cart→LineItem 変換ロジック | Order.java 後半未読 | yes |
| beans.xml — Tomcat 上の CDI 有効化有無 | ソースから判断不可 | yes |
| Stripes 1.6.0 — Jakarta EE 互換性 | Stripes メンテ状況不明 | yes |

---

## 5. 画面資産

**出所: 05-frontend.csv**

- 画面ページ: 16 本の JSP + 2 本の静的 HTML
- 共通部品: IncludeTop.jsp（全 JSP からインクルード）/ IncludeBottom.jsp
- すべての JSP が Stripes TLD を利用（IncludeTop.jsp 経由）
- **特記事項**:
  - IncludeTop.jsp: ページ宣言 charset=UTF-8 と `<meta>` charset=windows-1252 の不一致（05-frontend.csv:IncludeTop.jsp行、risk=mid）
  - Main.jsp: imgmap 内の `<area href>` に Catalog.action への相対 URL がハードコード（コンテキストルート変更時に破損リスク、risk=mid）
  - Product.jsp: `<jsp:useBean class="...CatalogActionBean">` による直接インスタンス化（Stripes 競合リスク、risk=mid）
  - jpetstore.css: IE5/Mac 向け voice-family CSS ハックが複数箇所に残存（risk=mid）

---

## 6. バックエンド処理

**出所: 06-backend.csv**

| クラスロール | 件数 | 主な依存フレームワーク |
|------------|------|-------------------|
| entry（入口処理）| 4 | Stripes（全 ActionBean） |
| logic（業務処理）| 3 | Spring（@Service/@Transactional） |
| db-access（DB 操作）| 7 | MyBatis（全 Mapper） |
| data（ドメインモデル）| 9 | なし（一部 Stripes @Validate） |
| fw-extension | 1 | AbstractActionBean（Stripes 基底） |

- **共通部品（common=yes）**: CatalogService（3 ActionBean から参照）、ItemMapper（CatalogService + OrderService から参照）
- **DB アクセスあり**: AccountService / CatalogService / OrderService / 全 7 Mapper

---

## 7. フレームワーク・共通基盤

**出所: 07-framework.csv**

| フレームワーク | origin | 利用箇所数 | 置換影響 |
|-------------|--------|-----------|---------|
| Stripes     | oss    | 20        | **high** — Web レイヤー全体の書き直し必要 |
| Spring Framework | oss | 13 | **high** — DI・トランザクション管理の中核（ただし置換不要） |
| MyBatis     | oss    | 17        | **high** — 全 DB 操作経路（ただし置換不要） |
| mybatis-spring | oss | 2        | mid |
| JSTL        | oss    | 2         | low |
| HSQLDB      | oss    | 4         | low（本番 DB 置換前提） |
| AbstractActionBean | in-house | 4 | mid（Stripes 置換に伴い再実装が必要） |

---

## 8. 実行環境影響

**出所: 08-runtime-impact.csv**

| area | 箇所 | impact | risk |
|------|------|--------|------|
| namespace-migration | AccountActionBean:22, OrderActionBean:22, CartActionBean:20（import javax.servlet.http.*） | Jakarta EE 9+ コンテナでコンパイル・デプロイ不可 | **high** |
| library-compat | spring-web 5.3.39 固定 | Spring 5 EOL（2024年末）。spring-context 6.x と混在 | **high** |
| library-compat | stripes 1.6.0 | 開発停止・後継なし。javax.servlet 依存のため Jakarta EE 移行不可 | **high** |
| server-api | web.xml（StripesFilter + Tomcat9 profile activeByDefault） | Tomcat 10+（EE 9+）では動作不可 | **high** |
| library-compat | jakarta.servlet-api 4.0.4（Servlet 4.0 / javax名前空間） | Servlet 5.0+ への移行時に全 API 参照コードの修正が必要 | **high** |
| namespace-migration | web.xml:24-27（Servlet 3.0 スキーマ） | Jakarta EE 移行時に xmlns 変更が必要 | mid |
| library-compat | taglibs-standard-spec/impl 1.2.5 | Jakarta EE 移行時に置換必要 | mid |
| language-version | pom.xml（Java 17） | 目標バージョン未定（needs_confirm） | low |
| deprecated-api | Order.java:21（java.util.Date） | コンパイル不可にならないが技術的負債 | low |

---

## 9. DB・SQL

**出所: 09-db.csv, 09-crud.csv**

### テーブル一覧（スキーマ DDL から）

SUPPLIER / SIGNON / ACCOUNT / PROFILE / BANNERDATA / ORDERS / ORDERSTATUS / LINEITEM / CATEGORY / PRODUCT / ITEM / INVENTORY / SEQUENCE — 合計 13 テーブル

### CRUD マトリクス（主要）

| テーブル | 主なアクセス元 | 操作 |
|---------|------------|------|
| ACCOUNT / SIGNON / PROFILE | AccountMapper | CRU |
| ORDERS / ORDERSTATUS | OrderMapper | CR |
| LINEITEM | LineItemMapper | CR |
| ITEM | ItemMapper | R |
| INVENTORY | ItemMapper | RU（注文時に在庫減算） |
| SEQUENCE | SequenceMapper | RU（注文番号採番） |
| CATEGORY / PRODUCT | CategoryMapper / ProductMapper | R のみ |

### SQL 品質上の注意点（09-db.csv より）

- **旧式 JOIN 構文**: AccountMapper.xml:26/52、ItemMapper.xml:26/47、OrderMapper.xml:26/59 の合計 6 クエリでカンマ区切り JOIN（ANSI SQL92 以前）を使用。動作はするが移行時の DB 互換性確認を推奨（risk=low）
- **ORDERSTATUS.timestamp 列名**: PostgreSQL/MySQL では TIMESTAMP が予約語。移行先 DB によってはクォートまたはリネームが必要（risk=mid、needs_confirm）
- **SEQUENCE テーブル名**: PostgreSQL では予約済みオブジェクト名。テーブル名変更またはクォートが必要（09-db.csv:SEQUENCE行）
- **MyBatis `<cache/>` 全 Mapper で有効**: SequenceMapper（採番テーブル）の L2 キャッシュは並行処理時に採番重複リスクあり（09-db.csv:SequenceMapper_cache行、risk=mid、needs_confirm）

---

## 10. 環境・プラットフォーム

**出所: 10-platform.csv**

- **Java バージョン**: pom.xml:62-63 で Java 17 指定。CI は Java 21/25/26/27-ea でテスト
- **ビルド**: Maven Wrapper（mvnw/mvnw.cmd）。JAVA_HOME 未設定時はビルドエラー（risk=high）
- **パッケージング**: war 形式。サーブレットコンテナへのデプロイ前提（スタンドアロン非対応）
- **デプロイ先**: pom.xml に 8 種の APサーバープロファイル（Tomcat9/TomEE8/WildFly26/Liberty-ee8/Jetty12/GlassFish5/Payara5/Resin4）。本番使用サーバーは未確定（needs_confirm）
- **コンテナ**: Dockerfile — `FROM openjdk:25`（タグ固定なし、risk=mid）
- **DB 接続**: applicationContext.xml:31-34 で HSQLDB インメモリ DB を使用。起動ごとにデータリセット（永続化不可、risk=high）
- **環境変数**: JAVA_HOME（必須）、MAVEN_OPTS、各種 CI シークレット（SONAR_TOKEN 等）
- **javax 名前空間直接参照**: web.xml / beans.xml / ActionBean 3 クラスの合計 5 箇所（Jakarta EE 9+ 移行時にすべて修正が必要）

---

## 11. 処理フロー

**出所: 11-flows.csv**

| flow_id | 業務名 | kind | binding |
|---------|-------|------|---------|
| FLOW-001 | 商品カタログ参照 | online | config-driven |
| FLOW-002 | 商品注文 | online | mixed |

**FLOW-001 経路**: index.html → CatalogActionBean#viewMain → CatalogService（CategoryMapper/ProductMapper/ItemMapper） → JSP 画面

**FLOW-002 経路**: Cart.jsp → OrderActionBean#newOrderForm → セッション検証 → NewOrderForm.jsp → OrderActionBean#newOrder → (配送先) ShippingForm.jsp → ConfirmOrder.jsp → OrderService#insertOrder [@Transactional] → SequenceMapper + ItemMapper（在庫減算）+ OrderMapper + LineItemMapper → ViewOrder.jsp

**FLOW-002 の @Transactional 管理テーブル**: SEQUENCE / INVENTORY / ORDERS / ORDERSTATUS / LINEITEM（5 テーブルを 1 トランザクションで更新）

---

## 12. 影響マトリクス

**出所: 12-impact-matrix.csv**

### risk=high の影響（移行ブロッカー）

| 影響元 | 依存先 | action | 影響内容 |
|-------|-------|--------|---------|
| Webレイヤー全体 | Stripes 1.6.0 | replace | Web レイヤー全クラス・全 JSP の書き直し必要。Jakarta 移行の最大ボトルネック |
| Webレイヤー全体 | spring-web 5.3.x | upgrade | Stripes 置換後に spring-web 6.x へ統一。EOL リスク |
| web.xml / ActionBean 3 クラス | javax → jakarta 名前空間 | migrate | Stripes 置換後に一括置換。4 箇所（AccountActionBean:22 / OrderActionBean:22 / CartActionBean:20 / web.xml:24-27） |
| 全 ActionBean / Service / JSP | jakarta.servlet-api 4.0.4 | migrate | Stripes 置換後に Servlet 5.0+ へアップグレード |
| 全 JSP / APサーバー | Tomcat 9 | replace | Stripes 置換・jakarta 移行後に Tomcat 10+ へ切り替え |
| applicationContext.xml / 全業務フロー | HSQLDB（本番 DB 置換） | replace | DataSource Bean を外部 DB 接続に差し替え。SQL 方言差異の確認必要 |
| SequenceMapper（採番） | MyBatis L2 Cache | migrate | SequenceMapper.xml の `<cache/>` 削除。採番重複リスク回避。本番 DB 移行前に優先対応 |

### risk=mid の影響

| 影響元 | 依存先 | action |
|-------|-------|--------|
| 全 JSP | JSTL 1.2 taglibs-standard | replace（Jakarta JSTL 3.x へ） |
| AccountMapper（認証） | MyBatis L2 Cache | migrate（クラスタ環境への移行前に分散キャッシュ設定） |
| Main.jsp | コンテキストルート依存 | confirm（imgmap ハードコード URL） |
| beans.xml | javax → jakarta 名前空間 | migrate（low） |
| Dockerfile | openjdk:25 タグ未固定 | upgrade（固定バージョンに変更） |

### 移行順序の前提関係

```
1. Stripes 1.6.0 置換（最高優先度・最大工数）
   ↓
2. javax → jakarta 名前空間一括移行
   ↓
3. spring-web 5.3.x → 6.x アップグレード
   ↓
4. JSTL 置換（taglibs-standard → Jakarta JSTL 3.x）
   ↓
5. Tomcat 9 → Tomcat 10+ 切り替え
   ↓（並行可）
A. HSQLDB → 本番 DB 置換（DataSource 差し替え + SQL 方言確認）
B. SequenceMapper <cache/> 削除（本番 DB 移行前に優先）
```

---

## 13. 確認事項（13-confirm-items.csv の要約）

**出所: 13-confirm-items.csv**

全 CSV の `needs_confirm=yes` 行から集約した確認事項は 30 件。詳細は `data/13-confirm-items.csv` を参照。

**ask_to 別件数**:
- 顧客: 13 件（本番環境・APサーバー種別・DB種別・業務設計書・バッチ処理有無 等）
- 技術検証: 14 件（採番キャッシュ・名前空間互換性・未解決依存の動作確認 等）
- 運用: 3 件（CI シークレット引き継ぎ・Maven リポジトリ設定 等）

---

## key findings（まとめ）

1. **移行最大ブロッカーは Stripes 1.6.0**: 2015年頃に開発停止。javax.servlet に完全依存しており Jakarta EE への移行が不可能。Web レイヤー全体（ActionBean 4 クラス + JSP 16 本の stripes タグ）の書き直しが必要。すべての移行作業がこれに依存する。（根拠: 07-framework.csv、03-libraries.csv、08-runtime-impact.csv）
2. **spring-web 5.3.x の EOL**: Spring 5 は 2024年末 EOL。spring-context 6.2.x と意図的に混在させている状態が技術的負債になっている。Stripes 置換なしには解消できない。（根拠: 03-libraries.csv:spring-web行）
3. **DB は永続化なし**: 現行は HSQLDB インメモリ DB で起動ごとにデータリセット。本番移行前に外部 DB への切り替えが必須。（根拠: 10-platform.csv:applicationContext.xml行）
4. **採番テーブルの L2 キャッシュに採番重複リスク**: SequenceMapper.xml の `<cache/>` により並行処理時に注文番号の重複が発生しうる。本番 DB 移行前の優先対応が必要。（根拠: 09-db.csv:SequenceMapper_cache行、12-impact-matrix.csv）
5. **確認事項 30 件**: 本番 APサーバー種別・移行先 DB 種別・業務設計書の存在・バッチ処理の有無が未確定。移行方針の決定前に顧客への確認が必要。（根拠: 13-confirm-items.csv）
