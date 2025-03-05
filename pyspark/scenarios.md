## Basic Scenarios

### 1. Generate the expected output data from the given data frames as shown below

```sh
Input DF1:
---+----+
| id|name|
+---+----+
|  1|   A|
|  2|   B|
|  3|   C|
|  4|   D|
+---+----+

Input DF2:
+---+-----+
| id|name1|
+---+-----+
|  1|    A|
|  2|    B|
|  4|    X|
|  5|    F|
+---+-----+

Expected Output:
+---+-------------+
| id|      comment|
+---+-------------+
|  3|New in Source|
|  4|   Mismatched|
|  5|New in Target|
+---+-------------+

```

**_Observation:_**

- If records are matching across the data frames, then dont consider it.
- If records has matching id and names mismatch then mark it as 'Mismatched'
- If id exists in DF1 and not present in DF2 then mark it as 'New in Source'
- If id exists in DF2 and not present in DF1 then mark it as 'New in Target'

**_Solution:_**

```sh
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
spark = SparkSession.builder.getOrCreate()

source_rdd = spark.sparkContext.parallelize([
    (1, "A"),
    (2, "B"),
    (3, "C"),
    (4, "D")
],1)

target_rdd = spark.sparkContext.parallelize([
    (1, "A"),
    (2, "B"),
    (4, "X"),
    (5, "F")
],2)

# Convert RDDs to DataFrames using toDF()
df1 = source_rdd.toDF(["id", "name"])
df2 = target_rdd.toDF(["id", "name1"])

# Show the DataFrames
print("DF1:")
df1.show()
print("DF2:")
df2.show()

print("Combine both data frames using full join.")

fulljoindf = df1.join(df2, ["id"], "full")
fulljoindf.show()


print("New column as 'comment' and whose value will be 'matched' or 'mismatched' based on 'name' and 'name1' data.")
matchdf = fulljoindf.withColumn("comment",expr("""
    case
    when name = name1 then 'matched'
    when name is null then 'New in Target'
    when name1 is null then 'New in Source'
    else 'Mismatched'
    end
"""))
matchdf.show()

print("Remove the matched records by filtering.")
filterdf = matchdf.filter("comment != 'matched'")
filterdf.show()

print("Drop unwanted columns to get the expected output.")
dropdf = filterdf.drop("name","name1")
dropdf.show()
```

**_Output:_**

```sh
DF1:
+---+----+
| id|name|
+---+----+
|  1|   A|
|  2|   B|
|  3|   C|
|  4|   D|
+---+----+

DF2:
+---+-----+
| id|name1|
+---+-----+
|  1|    A|
|  2|    B|
|  4|    X|
|  5|    F|
+---+-----+

Combine both data frames using full join.
+---+----+-----+
| id|name|name1|
+---+----+-----+
|  1|   A|    A|
|  2|   B|    B|
|  3|   C| NULL|
|  4|   D|    X|
|  5|NULL|    F|
+---+----+-----+

New column as 'comment' and whose value will be 'matched' or 'mismatched' based on 'name' and 'name1' data.
+---+----+-----+-------------+
| id|name|name1|      comment|
+---+----+-----+-------------+
|  1|   A|    A|      matched|
|  2|   B|    B|      matched|
|  3|   C| NULL|New in Source|
|  4|   D|    X|   Mismatched|
|  5|NULL|    F|New in Target|
+---+----+-----+-------------+

Remove the matched records by filtering.
+---+----+-----+-------------+
| id|name|name1|      comment|
+---+----+-----+-------------+
|  3|   C| NULL|New in Source|
|  4|   D|    X|   Mismatched|
|  5|NULL|    F|New in Target|
+---+----+-----+-------------+

Drop unwanted columns to get the expected output.
+---+-------------+
| id|      comment|
+---+-------------+
|  3|New in Source|
|  4|   Mismatched|
|  5|New in Target|
+---+-------------+
```

### 2. Generate the expected output data from the given data frame as shown below

```sh
Input DF:
+-----+------+
|child|parent|
+-----+------+
|    A|    AA|
|    B|    BB|
|    C|    CC|
|   AA|   AAA|
|   BB|   BBB|
|   CC|   CCC|
+-----+------+

Expected Output:
+-----+------+-----------+
|child|parent|grandparent|
+-----+------+-----------+
|    A|    AA|        AAA|
|    B|    BB|        BBB|
|    C|    CC|        CCC|
+-----+------+-----------+
```

**_Observation:_**

- Create two data frames from one Input
- Join two data frames using inner join based on parent from one df and child from other df
- Drop the unnecessary columns

**_Solution:_**

```sh
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
spark = SparkSession.builder.getOrCreate()

input_rdd = spark.sparkContext.parallelize([
    ("A", "AA"),
    ("B", "BB"),
    ("C", "CC"),
    ("AA", "AAA"),
    ("BB", "BBB"),
    ("CC", "CCC")
],1)


# Convert RDDs to DataFrames using toDF()
df1 = input_rdd.toDF(["child", "parent"])

df2 = input_rdd.toDF(["child1", "parent1"])
# Show the DataFrames
print("DF1:")
df1.show()
print("DF2:(Copy of DF1) ")
df2.show()

print("Combine both dataframes using inner join")
innerjoindf = df1.join(df2, df1["parent"]==df2["child1"], "inner")
innerjoindf.show()

print("Drop the columns that are not required")
dropdf = innerjoindf.drop("child1")
dropdf.show()

print("Rename the column")
finaldf = dropdf.withColumnRenamed("parent1", "grandparent")
finaldf.show()
```

**_Output:_**

```sh
DF1:
+-----+------+
|child|parent|
+-----+------+
|    A|    AA|
|    B|    BB|
|    C|    CC|
|   AA|   AAA|
|   BB|   BBB|
|   CC|   CCC|
+-----+------+

DF2:(Copy of DF1)
+------+-------+
|child1|parent1|
+------+-------+
|     A|     AA|
|     B|     BB|
|     C|     CC|
|    AA|    AAA|
|    BB|    BBB|
|    CC|    CCC|
+------+-------+

Combine both dataframes using inner join
+-----+------+------+-------+
|child|parent|child1|parent1|
+-----+------+------+-------+
|    A|    AA|    AA|    AAA|
|    B|    BB|    BB|    BBB|
|    C|    CC|    CC|    CCC|
+-----+------+------+-------+

Drop the columns that are not required
+-----+------+-------+
|child|parent|parent1|
+-----+------+-------+
|    A|    AA|    AAA|
|    B|    BB|    BBB|
|    C|    CC|    CCC|
+-----+------+-------+

Rename the column
+-----+------+-----------+
|child|parent|grandparent|
+-----+------+-----------+
|    A|    AA|        AAA|
|    B|    BB|        BBB|
|    C|    CC|        CCC|
+-----+------+-----------+
```

### 3. Generate the expected output data from the given data frame as shown below

```sh
Input DF1:
+---+-----+
| id| name|
+---+-----+
|  1|Henry|
|  2|Smith|
|  3| Hall|
+---+-----+

Input DF2:
+---+------+
| id|salary|
+---+------+
|  1|   100|
|  2|   500|
|  4|  1000|
+---+------+

Expected Output:
+---+-----+------+
| id| name|salary|
+---+-----+------+
|  1|Henry|   100|
|  2|Smith|   500|
|  3| Hall|     0|
+---+-----+------+
```

**_Observation:_**

- Join two data frames using left join because expected output has ids only from DF1
- Replace null with 0

**_Solution:_**

```sh
from pyspark import SparkConf, SparkContext
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
conf = SparkConf().setAppName("pyspark").setMaster("local[*]").set("spark.driver.host","localhost").set("spark.default.parallelism", "1")
sc = SparkContext(conf=conf)
spark = SparkSession.builder.getOrCreate()


print("===========INPUT DATA=============")
data1 = [
    (1, "Henry"),
    (2, "Smith"),
    (3, "Hall")
]
columns1 = ["id", "name"]
rdd1 = sc.parallelize(data1,1)
df1 = rdd1.toDF(columns1)
print("DF1:")
df1.show()


data2 = [
    (1, 100),
    (2, 500),
    (4, 1000)
]
columns2 = ["id", "salary"]
rdd2 = sc.parallelize(data2,1)
df2 = rdd2.toDF(columns2)
print("DF2:")
df2.show()


print("Join Data:")
joindf = df1.join(df2, ["id"], "left")
joindf.show()

print("Order By")
ordereddf = joindf.orderBy("id")
ordereddf.show()

print("Final Output:")
finaldf = ordereddf.withColumn("salary", expr("""
    case
    when salary is null then 0
    else salary
    end
"""))
finaldf.show()

```

**_Output:_**

```sh
DF1:
+---+-----+
| id| name|
+---+-----+
|  1|Henry|
|  2|Smith|
|  3| Hall|
+---+-----+

DF2:
+---+------+
| id|salary|
+---+------+
|  1|   100|
|  2|   500|
|  4|  1000|
+---+------+

Join Data:
+---+-----+------+
| id| name|salary|
+---+-----+------+
|  1|Henry|   100|
|  3| Hall|  NULL|
|  2|Smith|   500|
+---+-----+------+

Order By
+---+-----+------+
| id| name|salary|
+---+-----+------+
|  1|Henry|   100|
|  2|Smith|   500|
|  3| Hall|  NULL|
+---+-----+------+

Final Output:
+---+-----+------+
| id| name|salary|
+---+-----+------+
|  1|Henry|   100|
|  2|Smith|   500|
|  3| Hall|     0|
+---+-----+------+
```
