# SELECT文

ここでは，DML文の1つであるSELECT文について説明する。
SELECT文はSELECT句とFROM句の2つの句から構成される。SELECT句では取得する列を指定する。FROM句ではレコードを取得するテーブルを指定する。

## すべてのデータを取得する

カラムの表示される順番は，テーブル定義で記述した順番になる。
```sql
SELECT * 
FROM テーブル名;
```

## 列を指定してデータを取得する
カラムが表示される順番は，SELECT文で指定した列の順番になる。
```sql
SELECT 列名, 列名, ... , 列名 
FROM テーブル名;
```

## 列に別名をつける
別名が英数字のときは，特にシングルクオーテーションで囲む必要はない。
別名が日本語のときは，ダブルクオーテーションで囲むこと。
```sql
SELECT 列名1 AS 別名1,
列名2 AS 別名2,
列名3 AS 別名3
FROM テーブル名;
```

## 定数を出力する
数値定数は623，文字列定数は'abc'，日付定数は'2024-06-04'で記述する。次に，定数を出力する例を示す。
```sql
SELECT shohin_mei AS "商品", 23 AS num, '2024-06-04' AS TODAY FROM shohin;
```

## 重複する行を取り除く
```sql
SELECT DISTINCT shohin_bunrui 
FROM shohin;
```

## WHERE句
```sql
SELECT 列名, ... 列名
FROM テーブル名
WHERE 条件式;
```

shohin_bunruiが衣類の商品について，shohin_meiとshohin_bunruiを取得する例
```sql
SELECT shohin_mei, shohin_bunrui
FROM shohin
WHERE shohin_bunrui = '衣服';
```

## 算術演算子
SELECT文では足し算，引き算，掛け算，割り算の算術演算子を行うことができる。次に算術演算子の例を示す。
注意点として，NULLに対して算術演算子を行った場合，その結果はNULLになる。
```sql
SELECT hanbai_tanka, hanbai_tanka * 2
FROM shohin;
```

## 比較演算子
WHERE句で条件判定を行うときは次に示す比較演算子が使用できる。注意点として比較演算子でNULLのセルが比較されたとき，真理値はUNKNOWNになる。SQLは2値論理ではなく3値論理であり，UNKNOWNはTRUEでもFALSEでもない。
```sql
= -- 等しい
<> -- 等しくない
>= -- 以上
> -- より大きい
<= -- 以下
< -- より小さい
```


## NULL判定・非NULL判定
shohin_tankaがNULLのレコードを取得する。
```sql
SELECT shohin_mei, shohin_tanka
FROM shohin
WHERE shohin_tanka IS NULL;
```

shohin_tankaがNULLではないレコードを取得する。
```sql
SELECT shohin_mei, shohin_tanka
FROM shohin
WHERE shohin_tanka IS NOT NULL;
```

## 論理演算子
WHERE句で，複数の条件に対して絞り込みを行うような場合には論理演算子が使用できる。注意点として，UNKNOWNとTRUEまたはFALSEの論理演算の結果がどうなるのか，77Pの表で確認した方が良い。
```sql
NOT 条件
条件1 AND 条件2
条件1 OR 条件2
```