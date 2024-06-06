# テーブルの集合演算・結合

## 集合演算の注意点
- 演算対象となるレコードの列数が同じであること。例えばShohin1テーブルの2つのカラム，Shohin2テーブルの3つのカラムを結合することはできない。
- 演算対象となるレコードの列のデータ型が一致していること
- SELECT文はどのようなものでもいいが，ORDER BY句は最後に1つだけ付けられる


## UNION(和集合)
`UNION` : 2つのテーブルの和集合を求める集合演算子  
`UNION ALL` : 2つのテーブルの和集合を求める集合演算子。重複行を削除しない。
```sql
SELECT 列名1, 列名2, ...
FROM テーブル1
UNION
SELECT 列名1, 列名2, ...
FROM テーブル2;
```

Shohin, Shohin2テーブルの和集合を求めて，shohin_idとshohin_meiを表示する例
```sql
SELECT shohin_id, shohin_mei
FROM Shohin
UNION
SELECT shohin_id, shohin_mei
FROM Shohin2;
```

## INTERSECT(共通部分)
`INTERSECT` : 2つのテーブルの共通部分を求める集合演算子
`INTERSECT ALL`  : 2つのテーブルの共通部分を求める集合演算子。重複行削除しない。

Shohin, Shohin2テーブルの共通部分を求めて，shohin_idとshohin_meiを表示する例
```sql
SELECT shohin_id, shohin_mei
FROM Shohin
INTERSECT
SELECT shohin_id, shohin_mei
FROM Shohin2;
```

## EXCEPT(差集合)
`EXCEPT` : 2つのテーブルの差集合を求める演算子。`SELECT文A EXCEPT SELECT文B`を実行すると，A-Bの差集合が計算される。
```sql
SELECT shohin_id, shohin_mei
FROM Shohin
EXCEPT
SELECT shohin_id, shohin_mei
FROM Shohin2;
```

## 内部結合(INNER JOIN)
集合演算は，行に対する演算であったのに対して，結合は列に対する演算を行う。

`INNER JOIN` : 共通のキー(結合キー)を持つ行のみを結合する。結合キーが合致しないデータを持つレコードは結果から除外される。INNER JOINはFROM句に記述する。このため，結合後にWHERE句で抽出を行うときは，ONの後に記述する。

```sql
SELECT 列名1, 列名2, ...
FROM テーブル名1 AS tableA 
INNER JOIN テーブル名2 AS tableB 
ON tableA.結合キー = tableB.結合キー;
```

TenpoShohinテーブルとShohinテーブルを，shohin_idを結合キーとして内部結合する例
```sql
SELECT TS.tenpo_id, TS.tenpo_mei, TS.shohin_id, S.shohin_mei, S.hanbai_tanka
FROM TenpoShohin as TS
INNER JOIN Shohin AS S
On S.shohin_id = TS.shohin_id;
```

Shohinテーブル
```
 shohin_id |   shohin_mei   | shohin_bunrui | hanbai_tanka | shiire_tanka | tourokubi  
-----------+----------------+---------------+--------------+--------------+------------
 0001      | Tシャツ        | 衣服          |         1000 |          500 | 2009-09-20
 0002      | 穴あけパンチ   | 事務用品      |          500 |          320 | 2009-09-11
 0003      | カッターシャツ | 衣服          |         4000 |         2800 | 
 0004      | 包丁           | キッチン用品  |         3000 |         2800 | 2009-09-20
 0005      | 圧力なべ       | キッチン用品  |         6800 |         5000 | 2009-01-15
 0006      | フォーク       | キッチン用品  |          500 |              | 2009-09-20
 0007      | おろしがね     | キッチン用品  |          880 |          790 | 2008-04-28
 0008      | ボールペン     | 事務用品      |          100 |              | 2009-11-11
```

TenpoShohinテーブル
```
 tenpo_id | tenpo_mei | shohin_id | suryo 
----------+-----------+-----------+-------
 000A     | 東京      | 0001      |    30
 000A     | 東京      | 0002      |    50
 000A     | 東京      | 0003      |    15
 000B     | 名古屋    | 0002      |    30
 000B     | 名古屋    | 0003      |   120
 000B     | 名古屋    | 0004      |    20
 000B     | 名古屋    | 0006      |    10
 000B     | 名古屋    | 0007      |    40
 000C     | 大阪      | 0003      |    20
 000C     | 大阪      | 0004      |    50
 000C     | 大阪      | 0006      |    90
 000C     | 大阪      | 0007      |    70
 000D     | 福岡      | 0001      |   100
```

内部結合の結果
```
 tenpo_id | tenpo_mei | shohin_id |   shohin_mei   | hanbai_tanka 
----------+-----------+-----------+----------------+--------------
 000A     | 東京      | 0001      | Tシャツ        |         1000
 000A     | 東京      | 0002      | 穴あけパンチ   |          500
 000A     | 東京      | 0003      | カッターシャツ |         4000
 000B     | 名古屋    | 0002      | 穴あけパンチ   |          500
 000B     | 名古屋    | 0003      | カッターシャツ |         4000
 000B     | 名古屋    | 0004      | 包丁           |         3000
 000B     | 名古屋    | 0006      | フォーク       |          500
 000B     | 名古屋    | 0007      | おろしがね     |          880
 000C     | 大阪      | 0003      | カッターシャツ |         4000
 000C     | 大阪      | 0004      | 包丁           |         3000
 000C     | 大阪      | 0006      | フォーク       |          500
 000C     | 大阪      | 0007      | おろしがね     |          880
 000D     | 福岡      | 0001      | Tシャツ        |         1000
```

## 外部結合(OUTER JOIN)
`OUTER JOIN` : 片方のテーブルを基準(マスタ)として，テーブルの結合を行う。マスタテーブルの情報はすべて出力される。また，結合キーが見当たらない場合には，マスタテーブルではない方のテーブルから得られるデータはNULLとして結合される。

`LEFT OUTER JOIN` : FROM句で指定したテーブルをマスタテーブルとする。

`RIGHT OUTER JOIN` : OUTER JOINの後で指定したテーブルをマスタテーブルとする。

TenpoShohinテーブルとShohinテーブルを，shohin_idを結合キーとして右外部結合する例(Shohinテーブルが基準になる)
```sql
SELECT TS.tenpo_id, TS.tenpo_mei, TS.shohin_id, S.shohin_mei, S.hanbai_tanka
FROM TenpoShohin as TS
RIGHT OUTER JOIN Shohin AS S
On S.shohin_id = TS.shohin_id;
```

```
 tenpo_id | tenpo_mei | shohin_id |   shohin_mei   | hanbai_tanka 
----------+-----------+-----------+----------------+--------------
 000A     | 東京      | 0001      | Tシャツ        |         1000
 000A     | 東京      | 0002      | 穴あけパンチ   |          500
 000A     | 東京      | 0003      | カッターシャツ |         4000
 000B     | 名古屋    | 0002      | 穴あけパンチ   |          500
 000B     | 名古屋    | 0003      | カッターシャツ |         4000
 000B     | 名古屋    | 0004      | 包丁           |         3000
 000B     | 名古屋    | 0006      | フォーク       |          500
 000B     | 名古屋    | 0007      | おろしがね     |          880
 000C     | 大阪      | 0003      | カッターシャツ |         4000
 000C     | 大阪      | 0004      | 包丁           |         3000
 000C     | 大阪      | 0006      | フォーク       |          500
 000C     | 大阪      | 0007      | おろしがね     |          880
 000D     | 福岡      | 0001      | Tシャツ        |         1000
          |           |           | ボールペン     |          100
          |           |           | 圧力なべ       |         6800
```

## 直積結合(CROSS JOIN)
`CROSS JOIN` : 2つのテーブルの直積を行う結合

```sql
SELECT TS.tenpo_id, TS.tenpo_mei, TS.shohin_id, S.shohin_mei
FROM TenpoSHohin AS TS
CROSS JOIN Shohin AS S;
```