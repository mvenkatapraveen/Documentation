## Interview Scenarios

### 1. Generate the expected output data from the given data frame as shown below

```sh
Input DF:
+--------+---------+--------+------+-------------------+------+
|workerid|firstname|lastname|salary|        joiningdate|depart|
+--------+---------+--------+------+-------------------+------+
|     001|   Monika|   Arora|100000|2014-02-20 09:00:00|    HR|
|     002| Niharika|   Verma|300000|2014-06-11 09:00:00| Admin|
|     003|   Vishal| Singhal|300000|2014-02-20 09:00:00|    HR|
|     004|  Amitabh|   Singh|500000|2014-02-20 09:00:00| Admin|
|     005|    Vivek|   Bhati|500000|2014-06-11 09:00:00| Admin|
+--------+---------+--------+------+-------------------+------+

Expected Output:
+--------+---------+--------+------+-------------------+------+
|workerid|firstname|lastname|salary|        joiningdate|depart|
+--------+---------+--------+------+-------------------+------+
|     002| Niharika|   Verma|300000|2014-06-11 09:00:00| Admin|
|     003|   Vishal| Singhal|300000|2014-02-20 09:00:00|    HR|
|     004|  Amitabh|   Singh|500000|2014-02-20 09:00:00| Admin|
|     005|    Vivek|   Bhati|500000|2014-06-11 09:00:00| Admin|
+--------+---------+--------+------+-------------------+------+
```

**_Observation:_**

- Find the records with matching salaries

**_Solution:_**

```sh
from pyspark.sql import SparkSession
from pyspark.sql.functions import *


spark = SparkSession.builder.getOrCreate()

input_rdd = spark.sparkContext.parallelize([
    ("001", "Monika", "Arora", 100000, "2014-02-20 09:00:00", "HR"),
    ("002", "Niharika", "Verma", 300000, "2014-06-11 09:00:00", "Admin"),
    ("003", "Vishal", "Singhal", 300000, "201      4-02-20 09:00:00","HR"),
    ("004", "Amitabh", "Singh", 500000, "2014-02-20 09:00:00", "Admin"),
    ("005", "Vivek", "Bhati", 500000, "2014-06-11 09:00:00", "Admin")
],1)


# Convert RDDs to DataFrames using toDF()
df1 = input_rdd.toDF(["workerid", "firstname", "lastname", "salary", "joiningdate", "depart"])
print("Input DF:")
df1.show()

print("find counts for each unique salary value")
salgroupdf = df1.groupby("salary").agg(count("*").alias("count"))
salgroupdf.show()

print("Join Input with salary counts using inner join")
joindf = df1.join(salgroupdf,["salary"], "inner")
joindf.show()

print("Filter salaries that are repeated")
filterdf = joindf.filter("count>1")
filterdf.show()

print("Drop the column")
dropdf = filterdf.drop("count")
dropdf.show()

print("Select the columns in order")
finaldf = dropdf.select("workerid", "firstname", "lastname", "salary", "joiningdate", "depart")
finaldf.show()

```

**_Output:_**

```sh
Input DF:
+--------+---------+--------+------+--------------------+------+
|workerid|firstname|lastname|salary|         joiningdate|depart|
+--------+---------+--------+------+--------------------+------+
|     001|   Monika|   Arora|100000| 2014-02-20 09:00:00|    HR|
|     002| Niharika|   Verma|300000| 2014-06-11 09:00:00| Admin|
|     003|   Vishal| Singhal|300000|201      4-02-20 ...|    HR|
|     004|  Amitabh|   Singh|500000| 2014-02-20 09:00:00| Admin|
|     005|    Vivek|   Bhati|500000| 2014-06-11 09:00:00| Admin|
+--------+---------+--------+------+--------------------+------+

find counts for each unique salary value
+------+-----+
|salary|count|
+------+-----+
|300000|    2|
|100000|    1|
|500000|    2|
+------+-----+

Join Input with salary counts using inner join
+------+--------+---------+--------+--------------------+------+-----+
|salary|workerid|firstname|lastname|         joiningdate|depart|count|
+------+--------+---------+--------+--------------------+------+-----+
|300000|     002| Niharika|   Verma| 2014-06-11 09:00:00| Admin|    2|
|300000|     003|   Vishal| Singhal|201      4-02-20 ...|    HR|    2|
|100000|     001|   Monika|   Arora| 2014-02-20 09:00:00|    HR|    1|
|500000|     004|  Amitabh|   Singh| 2014-02-20 09:00:00| Admin|    2|
|500000|     005|    Vivek|   Bhati| 2014-06-11 09:00:00| Admin|    2|
+------+--------+---------+--------+--------------------+------+-----+

Filter salaries that are repeated
+------+--------+---------+--------+--------------------+------+-----+
|salary|workerid|firstname|lastname|         joiningdate|depart|count|
+------+--------+---------+--------+--------------------+------+-----+
|300000|     002| Niharika|   Verma| 2014-06-11 09:00:00| Admin|    2|
|300000|     003|   Vishal| Singhal|201      4-02-20 ...|    HR|    2|
|500000|     004|  Amitabh|   Singh| 2014-02-20 09:00:00| Admin|    2|
|500000|     005|    Vivek|   Bhati| 2014-06-11 09:00:00| Admin|    2|
+------+--------+---------+--------+--------------------+------+-----+

Drop the column
+------+--------+---------+--------+--------------------+------+
|salary|workerid|firstname|lastname|         joiningdate|depart|
+------+--------+---------+--------+--------------------+------+
|300000|     002| Niharika|   Verma| 2014-06-11 09:00:00| Admin|
|300000|     003|   Vishal| Singhal|201      4-02-20 ...|    HR|
|500000|     004|  Amitabh|   Singh| 2014-02-20 09:00:00| Admin|
|500000|     005|    Vivek|   Bhati| 2014-06-11 09:00:00| Admin|
+------+--------+---------+--------+--------------------+------+

Select the columns in order
+--------+---------+--------+------+--------------------+------+
|workerid|firstname|lastname|salary|         joiningdate|depart|
+--------+---------+--------+------+--------------------+------+
|     002| Niharika|   Verma|300000| 2014-06-11 09:00:00| Admin|
|     003|   Vishal| Singhal|300000|201      4-02-20 ...|    HR|
|     004|  Amitabh|   Singh|500000| 2014-02-20 09:00:00| Admin|
|     005|    Vivek|   Bhati|500000| 2014-06-11 09:00:00| Admin|
+--------+---------+--------+------+--------------------+------+
```


### 2. Generate the expected output data from the given data frame as shown below

```sh
Input DF:
+-------+----------+----------+
|orderid|statusdate|    status|
+-------+----------+----------+
|      1|     1-Jan|   Ordered|
|      1|     2-Jan|dispatched|
|      1|     3-Jan|dispatched|
|      1|     4-Jan|   Shipped|
|      1|     5-Jan|   Shipped|
|      1|     6-Jan| Delivered|
|      2|     1-Jan|   Ordered|
|      2|     2-Jan|dispatched|
|      2|     3-Jan|   shipped|
+-------+----------+----------+

Expected Output:
+-------+----------+----------+
|orderid|statusdate|    status|
+-------+----------+----------+
|      1|     2-Jan|dispatched|
|      1|     3-Jan|dispatched|
|      2|     2-Jan|dispatched|
+-------+----------+----------+
```

**_Observation:_**
