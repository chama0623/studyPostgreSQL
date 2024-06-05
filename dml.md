# INSERT・UPDATE・DELETE文

ここでは，SELECT文以外の3つのDML文であるINSERT，UPDATE，DELETE文について説明する。

## INSERT文
(列1, 列2, ...)と(値1, 値2, ...)の対応を一致させなければいけないことに注意する。なお列リストは省略することができる。省略した場合は値リストで指定された値を，テーブル定義のカラム順で格納する。  
また，NULLを挿入するときは，シングルクオーテーションで囲まずに`NULL`と記述する。
```sql
INSERT INTO テーブル名 (列1, 列2, ...) VALUES (値1, 値2, ...); 
```

ShohinInsテーブルにデータを追加する例
```sql
INSERT INTO ShohinIns (shohin_id, shohin_mei, shohin_bunrui, hanbai_tanka, shiire_tanka, torokubi) VALUES ('0001', 'Tシャツ', '衣服', 1000, 500, '2009-09-29');
```