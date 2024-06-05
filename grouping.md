# テーブルの集約

## 集約関数

- `COUNT` : レコード数をカウントする。`SELECT(*)`はNULLを含むレコード，`SELECT(カラム)`はNULLを含まないレコードの数をカウントする。
- `SUM` : 数値列のデータの総和を求める。カラムを指定して総和を計算するときに，NULLが含まれる場合はそれを除外して計算する。
- `AVG` : 数値列のデータの平均を求める。NULLのセルは，分子/分母どちらの計算からも除外される。
- `MAX` : 任意の列のデータの最大値を求める。数値以外に日付などほとんどのデータ型に適用できる。
- `MIN` : 任意の列のデータの最小値を求める。数値以外に日付などほとんどのデータ型に適用できる。

shohinテーブルの全ての行数と，shiire_tankaの行数をカウントする例
```sql
SELECT COUNT(*), COUNT(shiire_tanka) -- -> 8, 6
FROM shohin;
```

shiire_tankaの総和を計算する例
```sql
SELECT SUM(shiire_tanka)
FROM shohin;
```

重複値を除外して集約関数を用いる場合には，集約関数の中に`DISTINCT`キーワードを用いることができる。  
shohin_bunruiから重複値を除外して，レコード数をカウントする例
```sql
SELECT COUNT(DISTINCT shohin_bunrui)
FROM shohin;
```

## GROUP BY句
GROUP BY句は，テーブルをいくつかのグループに分類して，集約を行いたいような場合に用いる。表示順は指定がない限り，ランダムな順番で表示される。なお集約キーにNULLが含まれている場合は，NULLを1つのグループとして扱う。
```sql
SELECT 列名1, ..., 列名2
FROM テーブル名
GROUP BY 集約キーA, ... 集約キーB;
```

shohin_bunruiごとに行数をカウントする例
```sql
SELECT shohin_bunrui, COUNT(*)
FROM shohin
GROUP BY shohin_bunrui;
```

shohin_bunruiが衣服であるレコードを，shiire_tankaごとにカウントする例
```sql
SELECT shiire_tanka, COUNT(*)
FROM shohin
WHERE shohin_bunrui = '衣服'
GROUP BY shiire_tanka;
```

## HAVING句
HAVING句は，集約したデータに対して条件を付けて絞り込みたいときに用いる。WHERE句とHAVING句の違いは次の通りである。集約キーに対する絞り込みはどちらの句にも記述できるが，WHERE句に書いた方が実行速度が高速であり，一貫性のあるクエリを記述できる。
- WHERE句 : GROUP BYにより集約を行う`前`に条件を付けて絞り込む
- HAVING句 : GROUP BYにより集約を行った`後`に条件を付けて絞り込む

```sql
SELECT 列名1, ... ,列名2
FROM テーブル名
GROUP BY 集約キーA, ..., 集約キーB
HAVING グループに対する条件;
```

shohin_bunruiで集約したグループから，カウントが2のshohin_bunruiを取得する例
```sql
SELECT shohin_bunrui, COUNT(*)
FROM shohin
GROUP BY shohin_bunrui
HAVING COUNT(*) = 2;
```

## ORDER BY句
ORDER BY句は，表示順をソートする句である。ORDER BY句を指定しない場合はランダムな順番で表示される。またORDER BY句では昇順(ASC)/降順(DESC)を指定できる。順番が指定されていない場合のデフォルトはASCで処理される。また，ソートキーがNULLになっているレコードは先頭または末尾にまとめて表示される。
```sql
SELECT 列名1, ..., 列名2
FROM テーブル名
ORDER BY ソートキー1 [ASC|DESC], ..., ソートキー2 [ASC|DESC]; 
```

hanbai_tankaの昇順ですべてのカラムを取得する例
```sql
SELECT *
FROM shohin
ORDER BY hanbai_tanka ASC;
```