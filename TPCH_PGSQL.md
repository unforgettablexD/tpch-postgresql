1. Clone TPCH Repo with ```git clone https://github.com/electrum/tpch-dbgen.git``` 
2. ```cd tpch-dbgen```
3. Replace default Makefile with Makefile.suite ```cp makefile.suite makefile```
4. Edit new makefile by fill following settings:
  ```
  CC=gcc
  DATABASE=POSTGRESQL
  MACHINE=LINUX
  WORKLOAD=TPCH
  ```
5. Modify tpcd.h by adding following macros:
```
#ifdef POSTGRESQL
#define GEN_QUERY_PLAN  "EXPLAIN"
#define START_TRAN      "BEGIN TRANSACTION"
#define END_TRAN        "COMMIT;"
#define SET_OUTPUT      ""
#define SET_ROWCOUNT    "LIMIT %d\n"
#define SET_DBASE       ""
#endif
```
6. Compile dbgen by running ```make -j```
7. Now, you can generate the database, for example, by  ```./dbgen -vf -s 1```
8. We need to process generated tbl files by removing the "|" at the ends of each line. You can use this script:
```
mkdir tbl
for i in `ls *.tbl`
do
 name="tbl/$i"
 echo $name
 `touch $name`
 `chmod 777 $name`
 sed 's/|$//' $i >> $name;
done
```
9. ```chmod +x script.sh && ./script.sh```
10. Open psql and create a new database ```CREATE DATABASE tpch;```
11. Connect to the created database ```\c tpch```
12. Create the tables by running the sql commands in the ```dss.ddl``` file
```
-- Sccsid:     @(#)dss.ddl	2.1.8.1
CREATE TABLE NATION  ( N_NATIONKEY  INTEGER NOT NULL,
                            N_NAME       CHAR(25) NOT NULL,
                            N_REGIONKEY  INTEGER NOT NULL,
                            N_COMMENT    VARCHAR(152));

CREATE TABLE REGION  ( R_REGIONKEY  INTEGER NOT NULL,
                            R_NAME       CHAR(25) NOT NULL,
                            R_COMMENT    VARCHAR(152));

CREATE TABLE PART  ( P_PARTKEY     INTEGER NOT NULL,
                          P_NAME        VARCHAR(55) NOT NULL,
                          P_MFGR        CHAR(25) NOT NULL,
                          P_BRAND       CHAR(10) NOT NULL,
                          P_TYPE        VARCHAR(25) NOT NULL,
                          P_SIZE        INTEGER NOT NULL,
                          P_CONTAINER   CHAR(10) NOT NULL,
                          P_RETAILPRICE DECIMAL(15,2) NOT NULL,
                          P_COMMENT     VARCHAR(23) NOT NULL );

CREATE TABLE SUPPLIER ( S_SUPPKEY     INTEGER NOT NULL,
                             S_NAME        CHAR(25) NOT NULL,
                             S_ADDRESS     VARCHAR(40) NOT NULL,
                             S_NATIONKEY   INTEGER NOT NULL,
                             S_PHONE       CHAR(15) NOT NULL,
                             S_ACCTBAL     DECIMAL(15,2) NOT NULL,
                             S_COMMENT     VARCHAR(101) NOT NULL);

CREATE TABLE PARTSUPP ( PS_PARTKEY     INTEGER NOT NULL,
                             PS_SUPPKEY     INTEGER NOT NULL,
                             PS_AVAILQTY    INTEGER NOT NULL,
                             PS_SUPPLYCOST  DECIMAL(15,2)  NOT NULL,
                             PS_COMMENT     VARCHAR(199) NOT NULL );

CREATE TABLE CUSTOMER ( C_CUSTKEY     INTEGER NOT NULL,
                             C_NAME        VARCHAR(25) NOT NULL,
                             C_ADDRESS     VARCHAR(40) NOT NULL,
                             C_NATIONKEY   INTEGER NOT NULL,
                             C_PHONE       CHAR(15) NOT NULL,
                             C_ACCTBAL     DECIMAL(15,2)   NOT NULL,
                             C_MKTSEGMENT  CHAR(10) NOT NULL,
                             C_COMMENT     VARCHAR(117) NOT NULL);

CREATE TABLE ORDERS  ( O_ORDERKEY       INTEGER NOT NULL,
                           O_CUSTKEY        INTEGER NOT NULL,
                           O_ORDERSTATUS    CHAR(1) NOT NULL,
                           O_TOTALPRICE     DECIMAL(15,2) NOT NULL,
                           O_ORDERDATE      DATE NOT NULL,
                           O_ORDERPRIORITY  CHAR(15) NOT NULL,  
                           O_CLERK          CHAR(15) NOT NULL, 
                           O_SHIPPRIORITY   INTEGER NOT NULL,
                           O_COMMENT        VARCHAR(79) NOT NULL);

CREATE TABLE LINEITEM ( L_ORDERKEY    INTEGER NOT NULL,
                             L_PARTKEY     INTEGER NOT NULL,
                             L_SUPPKEY     INTEGER NOT NULL,
                             L_LINENUMBER  INTEGER NOT NULL,
                             L_QUANTITY    DECIMAL(15,2) NOT NULL,
                             L_EXTENDEDPRICE  DECIMAL(15,2) NOT NULL,
                             L_DISCOUNT    DECIMAL(15,2) NOT NULL,
                             L_TAX         DECIMAL(15,2) NOT NULL,
                             L_RETURNFLAG  CHAR(1) NOT NULL,
                             L_LINESTATUS  CHAR(1) NOT NULL,
                             L_SHIPDATE    DATE NOT NULL,
                             L_COMMITDATE  DATE NOT NULL,
                             L_RECEIPTDATE DATE NOT NULL,
                             L_SHIPINSTRUCT CHAR(25) NOT NULL,
                             L_SHIPMODE     CHAR(10) NOT NULL,
                             L_COMMENT      VARCHAR(44) NOT NULL);
```
13.Load the data into tables by:
```
copy region from 'PATH_TO_tpch-dbgen_REPO/tbl/region.tbl' with delimiter as '|' NULL '';
copy nation from 'PATH_TO_tpch-dbgen_REPO/tbl/nation.tbl' with delimiter as '|' NULL '';
copy partsupp from 'PATH_TO_tpch-dbgen_REPO/tbl/partsupp.tbl' with delimiter as '|' NULL '';
copy customer from 'PATH_TO_tpch-dbgen_REPO/tbl/customer.tbl' with delimiter as '|' NULL '';
copy lineitem from 'PATH_TO_tpch-dbgen_REPO/tbl/lineitem.tbl' with delimiter as '|' NULL '';
copy orders from 'PATH_TO_tpch-dbgen_REPO/tbl/orders.tbl' with delimiter as '|' NULL '';
copy part from 'PATH_TO_tpch-dbgen_REPO/tbl/part.tbl' with delimiter as '|' NULL '';
copy supplier from 'PATH_TO_tpch-dbgen_REPO/tbl/supplier.tbl' with delimiter as '|' NULL '';
```
14. Add primary Keys and foreign Keys by running:
```
ALTER TABLE REGION ADD PRIMARY KEY (R_REGIONKEY);
ALTER TABLE NATION ADD PRIMARY KEY (N_NATIONKEY);
ALTER TABLE PART ADD PRIMARY KEY (P_PARTKEY);
ALTER TABLE SUPPLIER ADD PRIMARY KEY (S_SUPPKEY);
ALTER TABLE PARTSUPP ADD PRIMARY KEY (PS_PARTKEY,PS_SUPPKEY);
ALTER TABLE CUSTOMER ADD PRIMARY KEY (C_CUSTKEY);
ALTER TABLE LINEITEM ADD PRIMARY KEY (L_ORDERKEY,L_LINENUMBER);
ALTER TABLE ORDERS ADD PRIMARY KEY (O_ORDERKEY);

ALTER TABLE NATION ADD FOREIGN KEY (N_REGIONKEY) references REGION;
ALTER TABLE SUPPLIER ADD FOREIGN KEY (S_NATIONKEY) references NATION;
ALTER TABLE CUSTOMER ADD FOREIGN KEY (C_NATIONKEY) references NATION;
ALTER TABLE PARTSUPP ADD FOREIGN KEY (PS_SUPPKEY) references SUPPLIER;
ALTER TABLE PARTSUPP ADD FOREIGN KEY (PS_PARTKEY) references PART;
ALTER TABLE ORDERS ADD FOREIGN KEY (O_CUSTKEY) references CUSTOMER;
ALTER TABLE LINEITEM ADD FOREIGN KEY (L_ORDERKEY)  references ORDERS;
ALTER TABLE LINEITEM ADD FOREIGN KEY (L_PARTKEY,L_SUPPKEY) references PARTSUPP;
COMMIT
```


