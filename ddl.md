# DDL文

## データベースの作成
```sql
CREATE DATABASE データベース名;
```

shoopデータベースを作成する例
```sql
CREATE DATABASE shop; 
```

## データベース一覧の確認
セミコロンを付けないことに注意する。
```sql
\l
```

## データベースへの接続
セミコロンを付けないことに注意する。
```sql
\c データベース名
```

## データベースの削除
削除しようとしているデータベースに接続しているときは削除できないことに注意。
```sql
DROP DATABASE データベース名;
```

## テーブルの作成
```sql
CREATE TABLE テーブル名
(列名1 データ型 制約,
列名2 データ型 制約,
...

列名3 データ型 制約
テーブル制約1, テーブル制約2, ..., テーブル制約3);
```

Shohinテーブルを作成する例
```sql
CREATE TABLE Shohin
(shohin_id CHAR(4) NOT NULL,
shohin_mei VARCHAR(100) NOT NULL,
shohin_bunrui VARCHAR(32) NOT NULL,
hanbai_tanka INTEGER,
shiire_tanka INTEGER,
tourokubi DATE,
PRIMARY KEY(shohin_id)
);
```

項目の命名規則
- 使用できる文字は半角英数字とアンダーバー
- 1文字目は半角アルファベット
- 他の項目と名前を重複させない

### データ型
- `INTEGER` : 数値型で整数を格納する
- `CHAR` : 固定長文字列。引数で最大長を指定する。格納された文字が最大長に満たないときは半角スペースで埋められる。
- `VARCHAR` : 可変長文字列。引数で最大長を指定する。格納された文字が最大長に満たない場合でも，そのまま文字列が格納される。
- `DATA` : 日付と時間を格納する。例として2003年04月12日 04時05分06秒を入力する場合には`'2003-04-12 04:05:06'`と入れる。

### 制約
postgreSQL 16の制約 : https://www.postgresql.jp/docs/9.4/ddl-constraints.html


#### NOT NULL制約
カラムがNULLにならないことを指定する。制約に`NOT NULL`と記述する。
#### UNIQUE制約
カラムあるいはカラムを含むグループで，レコードが一意な値をとることを指定する。制約に`UNIQUE`と記述する。

#### PRIMARY KEY
NOT NULL制約とUNIQUE制約の組み合わせ。内部的にはPRIMARY KEYのカラム，またはカラムのグループからBTreeインデックスが自動的に生成される。テーブル制約に`PRIMARY KEY (カラム名)`で記述する。

#### CHECK制約
カラムに格納される値が論理式を満たすように指定する。例えば，hanbai_tankaは必ず正の数でなければならない場合にはhanbai_tankaカラムの制約に`CHECK(hanbai_tanka > 0)`と記述する。
また，複数のカラムが関わるCHECK制約を加える場合には，テーブル制約にCHECK制約を記述する。例として，hanbai_tankaは必ずshiire_tankaよりも大きな値をとらなければいけない
場合には，テーブル制約に`CHECK(hanbai_tanka > shiire_tanka)`と記述する。

#### 外部キー制約
外部キー制約のドキュメント : https://www.postgresql.jp/document/16/html/ddl-constraints.html#DDL-CONSTRAINTS-FK

カラムに格納される値が，他のテーブルの値を参照するときに外部キー制約を記述する。例としてshohin_bunruiカラムが，productsテーブルのproduct_bunruiカラムを参照する場合には，shohin_bunruiカラムの制約に`REFERENCES products(product_bunrui)`と記述する。  なお，参照する列は省略できる。参照する列を省略した場合には指定したテーブルの主キーが外部キーに設定される。  
また外部キー制約はテーブル制約にまとめて記述することもできる。t1テーブルにおいてカラムa，cがother_tableテーブルのカラムc1，c2を参照する例を次に示す。
```sql
CREATE TABLE t1 (
  a INTEGER PRIMARY KEY,
  b INTEGER,
  c INTEGER,
  FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
);
```

さらに外部キー制約では，参照するキーが更新・削除された場合の対応を記述することができる。外部キー制約は次に示すようなものがある。ここで被参照列，参照列とは次のような意味である。
```sql
CREATE TABLE child (
    id INT PRIMARY KEY,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES parent(id)
    /* 被参照列はparentテーブルのカラムid
       参照列はchildテーブルのparent_id
    */
);
```
- `CASCADE` : 被参照列が更新されると，参照列も自動で更新される。被参照列が削除された場合は，参照先の行も削除される。
- `RESTRICT` : 被参照列を変更・削除しようとしたときに，その行が参照されていれば，変更・削除を拒否し，変更操作に失敗する。RESTRICTではトランザクション中に制約の検査が後回しにできない。
- `NO ACTION` : デフォルトの設定で，RESTRICTと同様の機能をもつ。ただし，NO ACTIONではトランザクション中に制約の検査を後回しにできる。
- `SET NULL` : 被参照列が削除されたとき，参照列にはNULLを格納する。
- `SET DEFAULT` : 被参照列が削除されたとき，参照列にはDEFALUT値を格納する。

product_noに対して，UPDATE文はCASCADE，DELETE文はRESTRICTで実行し，order_idに対してDELETE文をCASCADEで実行するテーブル定義の例
```sql
CREATE TABLE order_items (
    product_no integer REFERENCES products ON UPDATE CASCADE ON DELETE RESTRICT,
    order_id integer REFERENCES orders ON DELETE CASCADE,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```

#### DEFAULT制約
カラムに格納される値がINSERT文で指定されていないときに，代入する値を記述する。例えばhanbai_tankaのデフォルトとして0を代入する場合には，hanbai_tankaカラムの制約に`DEFAULT 0`と記述する。



## テーブルの一覧の確認
```sql
\dt;
```

```
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | shohin | table | postgres
(1 row)
```

## テーブル定義の確認
```sql
\d テーブル名;
```

```
                          Table "public.shohin"
    Column     |          Type          | Collation | Nullable | Default 
---------------+------------------------+-----------+----------+---------
 shohin_id     | character(4)           |           | not null | 
 shohin_mei    | character varying(100) |           | not null | 
 shohin_bunrui | character varying(32)  |           | not null | 
 hanbai_tanka  | integer                |           |          | 
 shiire_tanka  | integer                |           |          | 
 tourokubi     | date                   |           |          | 
Indexes:
    "shohin_pkey" PRIMARY KEY, btree (shohin_id)
```

## テーブルの削除
```sql
DROP TABLE テーブル名
```

Shohinテーブルを削除する
```
DROP TABLE Shohin;
```

## テーブル定義の変更
### カラムの追加
テーブルにカラムを追加するときは，次に示すALTER TABLE文を用いる。
```sql
ALTER TABLE テーブル名 ADD COLUMN カラム名 データ型;
```

Shohinテーブルに，shohin_mei_kanaを追加する例
```sql
ALTER TABLE Shohin ADD COLUMN shohin_mei_kana VRACHAR(100);
```

### カラムの削除
テーブルからカラムを削除するときは，次に示すALTER TABLE文を用いる。
```aql
ALTER TABLE テーブル名 DROP COLUMN カラム名
```

### カラムの変更
カラム名を変更するときは，次に示すALTER TABLE文を用いる。
```sql
ALTER TABLE 変更前カラム名 RENAME TO 変更後カラム名;
```