1. Clone TPCH Repo with ```git clone https://github.com/electrum/tpch-dbgen.git``` 
2. ```cd tpch-dbgen```
3. Replace default Makefile with Makefile.suite ```cp makefile.suite makefile```
4. Edit new makefile by fill following settings:
  ```
  CC=gcc
  DATABASE=ORACLE
  MACHINE=LINUX
  WORKLOAD=TPCH
  ```
6. Compile dbgen by running ```make makefile```
7. Now, you can generate the database, for example, by  ```./dbgen -s 1```
8. Open psql and create a new database ```CREATE DATABASE tpch;```
9. Connect to the created database ```\c tpch```
10. Create a schema for testing ```tpch=# create schema tpch1g;```
    ```tpch=# set search_path to tpch1g;```
12. Create empty tables; simply copy and paste the following table definition statements into the PostgreSQL prompt. We will import the data later into these tables.
```
CREATE TABLE IF NOT EXISTS "nation" (
  "n_nationkey"  INT,
  "n_name"       CHAR(25),
  "n_regionkey"  INT,
  "n_comment"    VARCHAR(152),
  "n_dummy"      VARCHAR(10),
  PRIMARY KEY ("n_nationkey"));

CREATE TABLE IF NOT EXISTS "region" (
  "r_regionkey"  INT,
  "r_name"       CHAR(25),
  "r_comment"    VARCHAR(152),
  "r_dummy"      VARCHAR(10),
  PRIMARY KEY ("r_regionkey"));

CREATE TABLE IF NOT EXISTS "supplier" (
  "s_suppkey"     INT,
  "s_name"        CHAR(25),
  "s_address"     VARCHAR(40),
  "s_nationkey"   INT,
  "s_phone"       CHAR(15),
  "s_acctbal"     DECIMAL(15,2),
  "s_comment"     VARCHAR(101),
  "s_dummy"       VARCHAR(10),
  PRIMARY KEY ("s_suppkey"));

CREATE TABLE IF NOT EXISTS "customer" (
  "c_custkey"     INT,
  "c_name"        VARCHAR(25),
  "c_address"     VARCHAR(40),
  "c_nationkey"   INT,
  "c_phone"       CHAR(15),
  "c_acctbal"     DECIMAL(15,2),
  "c_mktsegment"  CHAR(10),
  "c_comment"     VARCHAR(117),
  "c_dummy"       VARCHAR(10),
  PRIMARY KEY ("c_custkey"));

CREATE TABLE IF NOT EXISTS "part" (
  "p_partkey"     INT,
  "p_name"        VARCHAR(55),
  "p_mfgr"        CHAR(25),
  "p_brand"       CHAR(10),
  "p_type"        VARCHAR(25),
  "p_size"        INT,
  "p_container"   CHAR(10),
  "p_retailprice" DECIMAL(15,2) ,
  "p_comment"     VARCHAR(23) ,
  "p_dummy"       VARCHAR(10),
  PRIMARY KEY ("p_partkey"));

CREATE TABLE IF NOT EXISTS "partsupp" (
  "ps_partkey"     INT,
  "ps_suppkey"     INT,
  "ps_availqty"    INT,
  "ps_supplycost"  DECIMAL(15,2),
  "ps_comment"     VARCHAR(199),
  "ps_dummy"       VARCHAR(10),
  PRIMARY KEY ("ps_partkey","ps_suppkey");

CREATE TABLE IF NOT EXISTS "orders" (
  "o_orderkey"       INT,
  "o_custkey"        INT,
  "o_orderstatus"    CHAR(1),
  "o_totalprice"     DECIMAL(15,2),
  "o_orderdate"      DATE,
  "o_orderpriority"  CHAR(15),
  "o_clerk"          CHAR(15),
  "o_shippriority"   INT,
  "o_comment"        VARCHAR(79),
  "o_dummy"          VARCHAR(10),
  PRIMARY KEY ("o_orderkey"));

CREATE TABLE IF NOT EXISTS "lineitem"(
  "l_orderkey"          INT,
  "l_partkey"           INT,
  "l_suppkey"           INT,
  "l_linenumber"        INT,
  "l_quantity"          DECIMAL(15,2),
  "l_extendedprice"     DECIMAL(15,2),
  "l_discount"          DECIMAL(15,2),
  "l_tax"               DECIMAL(15,2),
  "l_returnflag"        CHAR(1),
  "l_linestatus"        CHAR(1),
  "l_shipdate"          DATE,
  "l_commitdate"        DATE,
  "l_receiptdate"       DATE,
  "l_shipinstruct"      CHAR(25),
  "l_shipmode"          CHAR(10),
  "l_comment"           VARCHAR(44),
  "l_dummy"             VARCHAR(10));
```
12. Load the data into tables by:
```
\copy "region"     from 'PATH_TO_tpch-dbgen_REPO/region.tbl'        DELIMITER '|' CSV;
\copy "nation"     from 'PATH_TO_tpch-dbgen_REPO/nation.tbl'        DELIMITER '|' CSV;
\copy "customer"   from 'PATH_TO_tpch-dbgen_REPO/customer.tbl'    DELIMITER '|' CSV;
\copy "supplier"   from 'PATH_TO_tpch-dbgen_REPO/supplier.tbl'    DELIMITER '|' CSV;
\copy "part"       from 'PATH_TO_tpch-dbgen_REPO/part.tbl'            DELIMITER '|' CSV;
\copy "partsupp"   from 'PATH_TO_tpch-dbgen_REPO/partsupp.tbl'    DELIMITER '|' CSV;
\copy "orders"     from 'PATH_TO_tpch-dbgen_REPO/orders.tbl'        DELIMITER '|' CSV;
\copy "lineitem"   from 'PATH_TO_tpch-dbgen_REPO/lineitem.tbl'    DELIMITER '|' CSV;
COMMIT
```



