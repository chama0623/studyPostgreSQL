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

### デフォルト値の設定
デフォルト値を指定する場合には次の2つの方法がある。
1. 明示的なデフォルト値の設定
```sql
INSERT INTO ShohinIns (shohin_id, shohin_mei, shohin_bunrui, hanbai_tanka, shiire_tanka, torokubi) VALUES ('0001', 'Tシャツ', '衣服', DEFAULT, 500, '2009-09-29');
```

2. 暗黙的なデフォルト値の設定(列リストにも値リストにも記述しない)
```sql
INSERT INTO ShohinIns (shohin_id, shohin_mei, shohin_bunrui, shiire_tanka, torokubi) VALUES ('0001', 'Tシャツ', '衣服', 500, '2009-09-29');
```

### 他のテーブルから値をコピーする
```sql
INSERT INTO コピー先テーブル (列1, 列2, ...)
SELECT (列1, 列2, ...)
FROM コピー元テーブル;
```

## DELETE文
テーブルの中身を空にする。
```sql
DELETE FROM テーブル名;
```

条件に合致するレコードを削除する。
```sql
DELETE FROM テーブル名
WHERE 条件; 
```

## UPDATE文
```sql
UPDATE テーブル名
SET 列名 = 式;
WHERE 条件;
```