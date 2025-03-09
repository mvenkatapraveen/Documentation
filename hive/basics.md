## HIVE

HIVE has its own personal space(kind of home directory)

- /user/hive/warehouse

**Every table in HIVE is mapped to a directory in HDFS.**

> Note: By default if location is not provided, then during table creation, it will create sub directories in /user/hive/warehouse folder. Else it will create in specified location.

![Hive Directory](./assets/images/HiveDirectory.png)
<br>

- Default table creation syntax, creates a managed table.

```sh
create table test(id int, name string)
```

- If we don’t specify delimiter, then default delimiter is blank (it means no separation between column data)

- So it’s always better to use a delimiter while creating table.

```sh
create table test(id int, name string) row format delimited fields terminated by ‘,’ ;
```

- We can create a table mapped to a directory at specific location:

![Hive Directory mapping with a table](./assets/images/HiveDirectoryMapping.png)
<br>

```sh
create table test(id int, name string) row format delimited fields terminated by ‘,’ location ‘/user/cloudera/testdir’;
```

### 1. Types of tables

- 2 Types

#### 1.1. Managed Table

By default managed table is created

**Generally used for intermittent or temporary tables**

If we drop the managed table, the data in the backend HDFS directory is also deleted.

Ex:
In hive terminal

```sh
create table test(id int, name string) row format delimited fields terminated by ‘,’ location ‘/user/cloudera/testdir’;
```

#### 1.2 External Table

** Generally used for tables where data is to be preserved**

If we drop the external table, the data in the backend HDFS directory will not be impacted.

Ex:
In hive terminal

```sh
create external table test(id int, name string) row format delimited fields terminated by ‘,’ location ‘/user/cloudera/testdir’;
```

### 2. Loading Data to HIVE tables

Data can be loaded from a file on edge node using hive shell.

- To load data from a file on Edge Node

```sh
hive
load data local inpath ‘/home/cloudera/mvp/alldata’ into table test;
```

- Data can be loaded from edge node to hive table directly

```sh
hadoop fs -put /home/cloudera/testdata /user/cloudera/testdir
```


### 3. Partitions

- This is mainly used for performance tuning.
- Queries run faster with partitioned data.

**Data can be loaded into the partitioned table in 3 ways.**

>  Data is already partitioned and have separate files for each partition then use static load

**_Ex:_**

- INDTxns, USATxns, UKTxns files on edge node.
- Load individual txn data to partitioned table.
- Problem in static load Each Partition has to be loaded separately.

> Data is present in a single file  use static insert

**_Ex:_**

- AllTxns on edge node
- Create a temp table
- Load AllTxn data to temple
- Insert the partitioned data to partitioned table from temp table

> Data is present in a single file  use dynamic load

**_Ex:_**

- AllTxns on edge node
- Create a temp table
- Load AllTxn data to temple
- Insert the partitioned data to partitioned table from temp table without static filter(hard coded value)
