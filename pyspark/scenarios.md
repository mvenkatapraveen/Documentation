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

### 4. Generate the expected output data from the given data frame as shown below

```sh
Input DF:
+-----+----+------+
|empid|name|salary|
+-----+----+------+
|    1|   a| 10000|
|    2|   b|  5000|
|    3|   c| 15000|
|    4|   d| 25000|
|    5|   e| 50000|
|    6|   f|  7000|
+-----+----+------+

Expected Output:
+-----+----+------+-----------+
|empid|name|salary|Designation|
+-----+----+------+-----------+
|    1|   a| 10000|   Employee|
|    2|   b|  5000|   Employee|
|    3|   c| 15000|    Manager|
|    4|   d| 25000|    Manager|
|    5|   e| 50000|    Manager|
|    6|   f|  7000|   Employee|
+-----+----+------+-----------+
```

**_Observation:_**

- If salary greater than 10000 then mark the designation as Manager else Employee

**_Solution:_**

```sh
from pyspark import SparkConf, SparkContext
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
conf = SparkConf().setAppName("pyspark").setMaster("local[*]").set("spark.driver.host","localhost").set("spark.default.parallelism", "1")
sc = SparkContext(conf=conf)
spark = SparkSession.builder.getOrCreate()

data = [
    ("1", "a", "10000"),
    ("2", "b", "5000"),
    ("3", "c", "15000"),
    ("4", "d", "25000"),
    ("5", "e", "50000"),
    ("6", "f", "7000")
]
myschema = ["empid","name","salary"]
df = spark.createDataFrame(data,schema=myschema)
print("Input DF:")
df.show()

finaldf = df.withColumn("Designation", expr("""

    case
    when salary <= 10000 then 'Employee'
    else 'Manager'
    end

"""))
print("Output:")
finaldf.show()
```

**_Output:_**

```sh
Input DF:
+-----+----+------+
|empid|name|salary|
+-----+----+------+
|    1|   a| 10000|
|    2|   b|  5000|
|    3|   c| 15000|
|    4|   d| 25000|
|    5|   e| 50000|
|    6|   f|  7000|
+-----+----+------+

Output:
+-----+----+------+-----------+
|empid|name|salary|Designation|
+-----+----+------+-----------+
|    1|   a| 10000|   Employee|
|    2|   b|  5000|   Employee|
|    3|   c| 15000|    Manager|
|    4|   d| 25000|    Manager|
|    5|   e| 50000|    Manager|
|    6|   f|  7000|   Employee|
+-----+----+------+-----------+
```

### 5. Generate the expected output data from the given data frame as shown below

```sh
Input DF:
+---+----+-----------+------+
| id|name|       dept|salary|
+---+----+-----------+------+
|  1|Jhon|    Testing|  5000|
|  2| Tim|Development|  6000|
|  3|Jhon|Development|  5000|
|  4| Sky| Prodcution|  8000|
+---+----+-----------+------+


Expected Output:
+---+----+-----------+------+
| id|name|       dept|salary|
+---+----+-----------+------+
|  1|Jhon|    Testing|  5000|
|  2| Tim|Development|  6000|
|  4| Sky| Prodcution|  8000|
+---+----+-----------+------+
```

**_Observation:_**

- Drop the duplicates based on name and salary if matched.

**_Solution:_**

```sh
from pyspark import SparkConf, SparkContext
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
conf = SparkConf().setAppName("pyspark").setMaster("local[*]").set("spark.driver.host","localhost").set("spark.default.parallelism", "1")
sc = SparkContext(conf=conf)
spark = SparkSession.builder.getOrCreate()


data = [(1, "Jhon", "Testing", 5000),
        (2, "Tim", "Development", 6000),
        (3, "Jhon", "Development", 5000),
        (4, "Sky", "Prodcution", 8000)]
df = spark.createDataFrame(data, ["id", "name", "dept", "salary"])
print("Input DF:")
df.show()

print("Dropping records based on matching names and salary")
dropdupdf = df.drop_duplicates(["name", "salary"])
dropdupdf.show()

print("Ordering the data")
ordereddf = dropdupdf.orderBy("id")
ordereddf.show()
```

**_Output:_**

```sh
Input DF:
+---+----+-----------+------+
| id|name|       dept|salary|
+---+----+-----------+------+
|  1|Jhon|    Testing|  5000|
|  2| Tim|Development|  6000|
|  3|Jhon|Development|  5000|
|  4| Sky| Prodcution|  8000|
+---+----+-----------+------+

Dropping records based on matching names and salary
+---+----+-----------+------+
| id|name|       dept|salary|
+---+----+-----------+------+
|  1|Jhon|    Testing|  5000|
|  4| Sky| Prodcution|  8000|
|  2| Tim|Development|  6000|
+---+----+-----------+------+

Ordering the data
+---+----+-----------+------+
| id|name|       dept|salary|
+---+----+-----------+------+
|  1|Jhon|    Testing|  5000|
|  2| Tim|Development|  6000|
|  4| Sky| Prodcution|  8000|
+---+----+-----------+------+
```

### 6. Generate the expected output data from the given data frame as shown below

```sh
Input DF1:
+---+
|col|
+---+
|  1|
|  2|
|  3|
+---+

Input DF2:
+---+
|col|
+---+
|  1|
|  2|
|  3|
|  4|
|  5|
+---+

Expected Output:
+---+
|col|
+---+
|  1|
|  2|
|  4|
|  5|
+---+
```

**_Observation:_**

- drop the max id matching record

**_Solution:_**

```sh
from pyspark import SparkConf, SparkContext
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
conf = SparkConf().setAppName("pyspark").setMaster("local[*]").set("spark.driver.host","localhost").set("spark.default.parallelism", "1")
sc = SparkContext(conf=conf)
spark = SparkSession.builder.getOrCreate()

# Create df1
df1 = spark.createDataFrame([("1",), ("2",), ("3",)], ["col"])

# Show df1
print("Input DF1:")
df1.show()

# Create df2
df2 = spark.createDataFrame([("1",), ("2",), ("3",), ("4",), ("5",)], ["col"])

# Show df2
print("Input DF2:")
df2.show()

print("Finding max id")
maxdf = df1.agg(max("col").alias("col"))
maxdf.show()

print("Anti Join")
joindf = df2.join(maxdf , ["col"] , "left_anti")

joindf.show()
```

**_Output:_**

```sh
Input DF1:
+---+
|col|
+---+
|  1|
|  2|
|  3|
+---+

Input DF2:
+---+
|col|
+---+
|  1|
|  2|
|  3|
|  4|
|  5|
+---+

Finding max id
+---+
|col|
+---+
|  3|
+---+

Anti Join
+---+
|col|
+---+
|  1|
|  2|
|  4|
|  5|
+---+
```
