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