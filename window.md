# ウィンドウ関数・GOUPING演算子

## ウィンドウ関数
`ウィンドウ関数(OLAP関数;OnLine Analytical Processing)` : データベースを使用してリアルタイムにデータ処理を行う関数のこと

ウィンドウ関数の構文は次のとおりである。ウィンドウ関数としては集約関数と，RANK，DENSE_RANK，ROW_NUMBERなどのウィンドウ専用関数が使える。
```sql
ウィンドウ関数 OVER ([PARTITION BY 列リスト] ORDER BY ソート用列リスト)
```

## ウィンドウ専用関数
`RANK関数` : レコードのランキングを算出するときに使用する関数
`PATITION BY` : 順位付けする対象の範囲を設定する。PARTITION BYで区切られた集合をウィンドウと呼ぶ。PARTITION BYは省略可能であり，省略した場合はテーブル全体を1つのウィンドウとして処理を行う。
`ORDER BY` : 順位付けの基準を設定する。

shohin_bunruiごとに，hanbai_tankaの昇順でランク付けした例
```sql
SELECT shohin_mei, shohin_bunrui, hanbai_tanka,
RANK () OVER(PARTITION BY shohin_bunrui ORDER BY hanbai_tanka ) AS ranking
FROM Shohin;
```

```
   shohin_mei   | shohin_bunrui | hanbai_tanka | ranking 
----------------+---------------+--------------+---------
 フォーク       | キッチン用品  |          500 |       1
 おろしがね     | キッチン用品  |          880 |       2
 包丁           | キッチン用品  |         3000 |       3
 圧力なべ       | キッチン用品  |         6800 |       4
 ボールペン     | 事務用品      |          100 |       1
 穴あけパンチ   | 事務用品      |          500 |       2
 Tシャツ        | 衣服          |         1000 |       1
 カッターシャツ | 衣服          |         4000 |       2
```

`RANK_DENSE関数` : 同順位のレコードが複数存在する場合に，その順位を飛ばさないランキングを算出する関数  
`ROW_NUMBER` : 同順位のレコードが複数存在する場合にも，一意な順位を与えるランキングを算出する関数

```sql
SELECT shohin_mei, shohin_bunrui, hanbai_tanka,
RANK () OVER(ORDER BY hanbai_tanka) AS ranking,
DENSE_RANK () OVER(ORDER BY hanbai_tanka) AS dense_ranking,
ROW_NUMBER () OVER(ORDER BY hanbai_tanka) AS row_numb
FROM Shohin;
```

## 累計・移動平均の計算
ウィンドウ関数を用いた累計の計算例
```sql
SELECT shohin_id, shohin_mei, hanbai_tanka,
SUM (hanbai_tanka) OVER (ORDER BY shohin_id) AS current_sum
FROM Shohin;
```

ウィンドウ関数を用いた移動平均の計算例  
集計行の指定方法  
`ROWS 数値 PRECEDING` : 現在の行から数値で指定した行数戻った行までの移動平均を求める。例として数値に2を指定したときは現在行，1つ前の行，2つ前の行の3行の移動平均が計算される。  
`ROWS 数値 FOLLOWING` : 現在の行から数値で指定した行数進んだ行までの移動平均を求める。  
`ROWS BETWEEN 数値1 PRECEDING AND 数値2 FOLLOWING` : 現在の行から数値1で指定した行数戻った行～数値2で指定した行数進んだ行の移動平均を求める。

```sql
SELECT shohin_id, shohin_mei, hanbai_tanka, 
AVG (hanbai_tanka) OVER (ORDER BY shohin_id ROWS 2 PRECEDING) AS moving_avg
FROM Shohin;
```

## GROUPING演算子
`GROUPING演算子` : グループごとに合計や小計を求めるときに使用する演算子。GROUPING演算子には`ROLLUP`，`CUBE`，`GROUPING SETS`の3種類がある。

`ROLLUP` : グルーピングに加えて合計行を求める演算子。合計行は超集合行と呼ばれ，超集合行の集約キーはデフォルトでNULLが使用される。

shohin_bunruiごとの合計と，ROLLUPで全体の合計を求める例
```sql
SELECT shohin_bunrui, SUM(hanbai_tanka) AS sum_tanka
FROM Shohin
GROUP BY ROLLUP(shohin_bunrui);
```

```
 shohin_bunrui | sum_tanka 
---------------+-----------
               |     16780
 キッチン用品  |     11180
 衣服          |      5000
 事務用品      |       600
```

shohin_bunruiごとにグルーピングしたhanbai_tankaの合計を，tourokubiごとのグループの小計と全体の合計で表示する例
```sql
SELECT shohin_bunrui, tourokubi, SUM(hanbai_tanka) AS sum_tanka
FROM Shohin
GROUP BY ROLLUP(shohin_bunrui, tourokubi);
```

超集合行の集約キーはNULLになってしまうので，元々のデータにより生じたNULLと区別がつかないという問題がある。そこで次のようなGROUPING関数でこれらを区別できる。値が1のときは超集合行によるNULL, 0のときはそれ以外で生じたNULLである。
```sql
SELECT GROUPING(shohin_bunrui) AS shohin_bunrui, 
GROUPING(tourokubi) AS tourokubi, 
SUM(hanbai_tanka) AS sum_tanka
FROM Shohin
GROUP BY ROLLUP(shohin_bunrui, tourokubi);
```

## CUBE

`CUBE` : GROUP BYで指定されたキーについて，すべての可能な組み合わせで集計を行うときに用いる。

```sql
SELECT CASE WHEN GROUPING(shohin_bunrui) = 1
            THEN '商品分類 合計'
            ELSE shohin_bunrui END AS shohin_bunrui,
       CASE WHEN GROUPING(tourokubi) = 1
            THEN '登録日 合計'
            ELSE CAST(tourokubi AS VARCHAR(16)) END AS tourokubi,
       SUM(hanbai_tanka) AS sum_tanka
FROM Shohin
GROUP BY CUBE(shohin_bunrui, tourokubi);
```

## GROUPING SETS

`GROUPING SETS` : ROLLUPやCUBEで求めた結果の一部分のみが欲しいときに用いる。

```sql
SELECT CASE WHEN GROUPING(shohin_bunrui) = 1
            THEN '商品分類 合計'
            ELSE shohin_bunrui END AS shohin_bunrui,
       CASE WHEN GROUPING(tourokubi) = 1
            THEN '登録日 合計'
            ELSE CAST(tourokubi AS VARCHAR(16)) END AS tourokubi,
       SUM(hanbai_tanka) AS sum_tanka
FROM Shohin
GROUP BY GROUPING SETS(shohin_bunrui, tourokubi);
```