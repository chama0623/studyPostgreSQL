# 述語・CASE式

## 述語
`述語` : 関数の一種で，戻り値が真理値(True，FALSE，UNKNOWN)になるもののこと

### LIKE述語
`LIKE述語` : 文字列の部分一致検索を行うときに用いる述語。0文字以上の任意の文字列を意味する`%`と，任意の1文字を表す`_`を用いて，文字列の部分一致検索を行う。

前方一致検索 : SampleLikeテーブルのstrcolカラムから，dddで始める文字列を検索する例
```sql
SELECT *
FROM SampleLike
WHERE strcol LIKE 'ddd%';
```

後方一致検索 : SampleLikeテーブルのstrcolカラムから，dddで終わる文字列を検索する例
```sql
SELECT *
FROM SampleLike
WHERE strcol LIKE '%ddd';
```

中間一致検索 : SampleLikeテーブルのstrcolカラムから，dddを含む文字列を検索する例
```sql
SELECT *
FROM SampleLike
WHERE strcol LIKE '%ddd%';
```

### BETWEEN述語
`BETWEEN述語` : 範囲検索を行うときに用いる述語。BETWEEN述語では範囲を`BETWEEN a and b`で記述する。この範囲はa以上b以下で，閾値を含む。

Shohinテーブルから，hanbai_tankaが100円以上1000円以下の商品のshohin_meiとhanbai_tankaを取得する例
```sql
SELECT shohin_mei, hanbai_tanka
FROM Shohin
WHERE hanbai_tanka BETWEEN 100 AND 1000;
```

### IS NULL述語
`IS NULL`述語 : NULLであるか判定するときに用いる述語。  
`IS NOT NULL`述語 : NULLでないか判定するときに用いる述語。

Shoshinテーブルから，shiire_tankaがNULLのレコードを取得する例
```sql
SELECT shohin_mei, shiire_tanka
FROM Shohin
WHERE shiire_tanka IS NULL;
```

### IN述語
`IN述語` : 特定の集合に含まれることを判定を行うときに用いる述語。
`NOT IN述語` : 特定の集合に含まれないことを判定を行うときに用いる述語。

Shohinテーブルから，shiire_tankaが320円，500円，5000円のレコードを取得する例
```sql
SELECT shohin_mei, shiire_tanka
FROM Shohin
WHERE shiire_tanka IN (320, 500, 5000);
```

IN述語の引数にはサブクエリを与えることができる。  
大阪店(tenpo_id = '000C')に置いてある商品のhanbai_tankaを取得する例
```sql
SELECT shohin_mei, hanbai_tanka
FROM Shohin
WHERE shohin_id IN (SELECT shohin_id 
                    FROM TenpoShohin
                    WHERE tenpo_id = '000C');
```

### EXISTS述語
`EXISTS述語` : ある条件に合致するレコードの存在有無を調べるときに用いる述語。サブクエリのSELECT句は何でもよいが，慣習上`*`にしている。
`NOT EXISTS述語` : ある条件に合致するレコードの存在有無を調べるときに用いる述語。EXISTS述語は，レコードが存在するときにTRUEを返すのに対して，NOT EXISTS述語は存在しないときにTRUEを返す。

先の大阪店(tenpo_id = '000C')に置いてある商品のhanbai_tankaを取得する例をEXISTS述語で記述すると次のようになる。
```sql
SELECT shohin_mei, hanbai_tanka
FROM Shohin AS S
WHERE EXISTS(SELECT *
            FROM TenpoShohin AS TS
            WHERE TS.tenpo_id = '000C'
            AND S.shohin_id = TS.shohin_id);
```

EXISTS述語で記述したクエリの動作の手順は次のとおりである。
1. メインクエリから1つのレコードが選択される。(相関サブクエリによるもの)
2. 選択されたレコードに対してEXISTS述語の引数で与えられ得た相関サブクエリが実行される。すなわち，tenpo_idが'000C'で，かつ選択されたレコードと同じshohin_idを持つレコードが返される。
3. EXISTS述語は，相関サブクエリの実行結果で得られたレコードが1つ以上あればTRUE，それ以外のときFALSEを返す。
4. メインクエリのWHERE句が実行される。TRUEのときはそのレコードのshohin_mei，hanbai_tankaが表示される。
5. 1に戻って，メインクエリから別のレコードを1つ選択し2~5の手順を実行する。

### CASE式
`CASE式` : 場合分けを記述する場合に用いる関数。CASE式には単純CASE式と検索CASE式の二種類があるが，ここでは検索CASE式を扱う。

CASE式は次に示す手順で実行される。
1. 評価式1が評価される。TRUEのとき式1が戻される。
2. 評価式2が評価される。TRUEのとき式2が戻される。  
...  
3. どの評価式もFALSEであれば，ELSEの評価式を実行する。

```sql
CASE WHEN 評価式1 THEN 式1
     WHEN 評価式2 THEN 式2
     ...
     ELSE 式
END
```

shohin_bunruiごとにA，B，Cのラベルをつけて「A:衣服」のように表示する例
```sql
SELECT shohin_mei,
        CASE WHEN shohin_bunrui = '衣服'
            THEN 'A:' || shohin_bunrui
            WHEN shohin_bunrui = '事務用品'
            THEN 'B:' || shohin_bunrui
            WHEN shohin_bunrui = 'キッチン用品'
            THEN 'C:' || shohin_bunrui
            ELSE NULL
        END AS abc_shohin_bunrui
FROM Shohin;
```
