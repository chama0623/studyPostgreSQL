# ビュー・サブクエリ

## ビュー
- ビュー : 頻繁に使用するSELECT文を保存するために用いる。ビューが実行されると，保存されたSELECT文が実行され，一時的に仮想的なテーブルに結果が保存される。データそのものは保持していないことに注意する。

Q. なぜビューを使用するのか?
- ビューを用いることで，SELECT文のみを保持して実記憶容量を削減するため
- ビューによりアクセス制限をかけることができるため，セキュリティ対策が行えるため
- ビューを保存して使いまわすことで，メンテナンス性，提供するデータの一貫性を確保するため

### ビューの作成
ここではSELECT文からビューを作成しているが，SELECT文にビューを指定して新しいビューを作成することも可能である(多段ビュー)。ただし，ビューを重ねるとパフォーマンスが低下することに注意する。
```sql
CREATE VIEW ビュー名 (ビューの列名1, ビューの列名2, ...)
AS
SELECT文
```

Shohinテーブルからshohin_bunruiごとにカウントを行うSELECT文を実行するビューの作成例
```sql
CREATE VIEW ShohinSum(shohin_bunrui, cnt_shohin)
AS
SELECT shohin_bunrui, COUNT(*)
FROM Shohin
GROUP BY shohin_bunrui;
```

### ビューの一覧の確認
```sql
\dv
```

### ビューの定義の確認
```sql
d+ ビュー名
```

### ビューの使用
```sql
SELECT 列名1, 列名2, ...
FROM ビュー名;
```

### ビューの削除
```sql
DROP VIEW ビュー名;
```

### ビューに対する制限事項
1. ビュー定義にORDER BY句は使用できない。これはテーブルと同様にビューに行の順序がないためである。
2. ビューを指定した更新(INSERT, UPDATE, DELETE)は次の条件を満たしているときのみ実行できる。ビューが更新されると連動してテーブルも更新される。
    - SELECT句にDISTINCTが含まれていない
    - FROM句に含まれるテーブルが1つだけである
    - GROUP BY句を使用していない
    - HAVING句を使用していない

## サブクエリ
- サブクエリ : 使い捨てのビューで，ビューの定義をそのままSELECT文のFROM句に持ち込んだもの。サブクエリはSELECT文の実行直後に削除される。

Shohinテーブルのshohin_bunruiごとにカウントを行い，その結果を表示するサブクエリの例
```sql
SELECT shohin_bunrui, cnt_shohin
FROM (SELECT shohin_bunrui, COUNT(*) AS cnt_shohin
    FROM Shohin
    GROUP BY shohin_bunrui) AS ShohinSum;
```

## スカラ・サブクエリ
- スカラ・サブクエリ : 結果が必ず1行1列となるようなサブクエリのこと。スカラ・サブクエリは定数や列名が書けるすべての場所に記述できる。注意点としてスカラ・サブクエリの実行結果は必ず1行1列になる必要がある。

hanbai_tankaがhanbai_tankaの平均よりも大きくなるshohin_id，shohin_mei，hanbai_tankaを表示するスカラ・サブクエリの例
```sql
SELECT shohin_id, shohin_mei, hanbai_tanka
FROM Shohin
WHERE hanbai_tanka > (SELECT AVG(hanbai_tanka) FROM Shohin);
```

shohin_id，shohin_mei，hanbai_tanka，hanbai_tankaの平均を表示するスカラ・サブクエリの例
```sql
SELECT shohin_id, shohin_mei, hanbai_tanka, (SELECT AVG(hanbai_tanka) FROM Shohin)
FROM Shohin;
```

shohin_bunruiごとに集約して，hanbai_tankaの平均よりもshohin_bunruiごとの平均が高いhanbai_bunruiを表示する例
```sql
SELECT shohin_bunrui, AVG(hanbai_tanka)
FROM Shohin
GROUP BY shohin_bunrui
HAVING AVG(hanbai_tanka) > (SELECT AVG(hanbai_tanka) FROM Shohin); 
```

## 相関サブクエリ
- 相関サブクエリ : 集約によりグループ分けしたグループ内での比較を行うときに用いるサブクエリ

先の例において，shohin_bunruiごとに集計して，そのグループの平均よりも各商品のhanbai_tankaが高いものだけを取得するような場合を考える。この処理は，shohin_bunruiごとの平均を求める必要があるため，スカラ・サブクエリが使えない。そこで登場するのが相関サブクエリである。

先の問題を解決する相関サブクエリ
```sql
SELECT shohin_bunrui, shohin_mei, hanbai_tanka
FROM Shohin as S1
WHERE hanbai_tanka > (SELECT AVG(hanbai_tanka) 
                    FROM Shohin as S2 
                    WHERE S1.shohin_bunrui = S2.shohin_bunrui
                    GROUP BY shohin_bunrui);
```

相関サブクエリが実行される順番  
先の例の相関サブクリエは，具体的に次の手順で実行される。なお，Shohinテーブルの内容は次の通りである。  

Shohinテーブル
```
 shohin_bunrui |   shohin_mei   | hanbai_tanka 
---------------+----------------+--------------
 衣服          | Tシャツ        |         1000
 事務用品      | 穴あけパンチ   |          500
 衣服          | カッターシャツ |         4000
 キッチン用品  | 包丁           |         3000
 キッチン用品  | 圧力なべ       |         6800
 キッチン用品  | フォーク       |          500
 キッチン用品  | おろしがね     |          880
 事務用品      | ボールペン     |          100
```

1. メインクエリ(外側のクエリ)が参照され，shohin_meiがTシャツのレコードが選ばれる。
```
 shohin_bunrui |   shohin_mei   | hanbai_tanka 
---------------+----------------+--------------
 衣服          | Tシャツ        |         1000
```

2. `S1.shohin_bunrui`に1で選ばれたレコードのshohin_bunruiが代入されて，サブクエリの処理が行われる。これによりshohin_bunruiが`衣服`のレコードのhanbai_tankaの平均が計算される。(WHERE句でshohin_bunruiが`衣服`だけに絞り込んでいるので，GROUP BY句は冗長であり不要)
```sql
SELECT AVG(hanbai_tanka) 
FROM Shohin as S2 
WHERE '衣服' = S2.shohin_bunrui
GROUP BY shohin_bunrui
```

3. メインクエリのWHERE句で，サブクエリで計算されたshohin_bunruiが`衣服`の商品のhanbai_tankaの平均と，1で選択したレコードのhanbai_tankaが比較される。1で選択したレコードのhanbai_tankaの方が大きいときは結果に表示される。
4. メインクエリが再度参照され，別のレコードに対して1～4の処理が行われる。