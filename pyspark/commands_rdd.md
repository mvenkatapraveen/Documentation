# Resilient Distributed Dataset (RDD) Basics

## 1. Create RDD list from a list

Convert list of integers to RDD list of integers

```sh
print("List of integers:")
l= [1,2,3,4,5]
print(l)

print("RDD List of integers")
rddl = sc.parallelize(l)
print(rddl.collect())
```

**_Output:_**

```sh
List of integers:
[1, 2, 3, 4, 5]
RDD List of integers
[1, 2, 3, 4, 5]
```

Convert list of strings to RDD list of strings

```sh
print("List of Strings:")
lstr = ["mvk", "mvp","mvnl"]
print(lstr)

print("RDD List of Strings:")
rddl = sc.parallelize(lstr)
print(rddl.collect())
```

**_Output:_**
```sh
List of Strings:
['mvk', 'mvp', 'mvnl']
RDD List of Strings:
['mvk', 'mvp', 'mvnl']
```

## 2. Perform operation on each element of list

Convert list of integers to RDD list of integers and then perform operation on each element

```sh
print("List of integers:")
l= [1,2,3,4,5]
print(l)

print("RDD List of integers")
rddl = sc.parallelize(l)
print(rddl.collect())

print("Modified Data:")
finaldf = rddl.map(lambda x : x*10)
print(finaldf.collect())
```

**_Output:_**
```sh
List of integers:
[1, 2, 3, 4, 5]
RDD List of integers
[1, 2, 3, 4, 5]
Modified Data:
[10, 20, 30, 40, 50]
```

## 3. Filter some elements of list

Convert list of integers to RDD list of integers and apply filters

```sh
print("List of integers:")
l= [1,2,3,4,5]
print(l)

print("RDD List of integers")
rddl = sc.parallelize(l)
print(rddl.collect())

print("Filtered Data:")
finaldf = rddl.filter(lambda x : x>2)
print(finaldf.collect())
```

**_Output_**
```sh
List of integers:
[1, 2, 3, 4, 5]
RDD List of integers
[1, 2, 3, 4, 5]
Filtered Data:
[3, 4, 5]
```

## 4. Flatten the elements of list

convert list of integers to RDD list of integers and flatten the data

```sh
print("Raw Data:")
l = ['A~B','C~D','E~F']
print(l)

print("RDD Data")
rddl = sc.parallelize(l)
print(rddl.collect())

print("Flattened Data:")
finaldf = rddl.flatMap(lambda x : x.split("~"))
print(finaldf.collect())
```

**_Output:_**
```sh
Raw Data:
['A~B', 'C~D', 'E~F']
RDD Data
['A~B', 'C~D', 'E~F']
Flattened Data:
['A', 'B', 'C', 'D', 'E', 'F']
```

## 5. Column Filter

Input file (dt.txt) :

```sh
00000000,06-26-2011,200,Exercise,GymnasticsPro,cash
00000001,05-26-2011,300,Exercise,Weightlifting,credit
00000002,06-01-2011,100,Exercise,GymnasticsPro,cash
00000003,06-05-2011,100,Gymnastics,Rings,credit
00000004,12-17-2011,300,Team Sports,Field,paytm
00000005,02-14-2011,200,Gymnastics,,cash
```

Applying filter on specific column value

```sh
data = sc.textFile("dt.txt")
print("Raw Data:")
data.foreach(print)


mapsplit = data.map(lambda x: x.split(','))
print()
print("Map Split Data")
mapsplit.foreach(print)

columns = namedtuple('columns', ['tno', 'tdate', 'amount', 'category', 'product', 'mode'])

mappedcolumns = mapsplit.map(lambda x: columns(x[0],x[1],x[2],x[3],x[4],x[5]))

filtereddata = mappedcolumns.filter(lambda x: 'Gymnastics' in x.product)
print()
print("Filtered Data based on column")
filtereddata.foreach(print)
```

**_Output_**
```sh
Raw Data:
00000000,06-26-2011,200,Exercise,GymnasticsPro,cash
00000001,05-26-2011,300,Exercise,Weightlifting,credit
00000002,06-01-2011,100,Exercise,GymnasticsPro,cash
00000003,06-05-2011,100,Gymnastics,Rings,credit
00000004,12-17-2011,300,Team Sports,Field,paytm
00000005,02-14-2011,200,Gymnastics,,cash

Map Split Data
['00000000', '06-26-2011', '200', 'Exercise', 'GymnasticsPro', 'cash']
['00000001', '05-26-2011', '300', 'Exercise', 'Weightlifting', 'credit']
['00000002', '06-01-2011', '100', 'Exercise', 'GymnasticsPro', 'cash']
['00000003', '06-05-2011', '100', 'Gymnastics', 'Rings', 'credit']
['00000004', '12-17-2011', '300', 'Team Sports', 'Field', 'paytm']
['00000005', '02-14-2011', '200', 'Gymnastics', '', 'cash']

Filtered Data based on column
columns(tno='00000000', tdate='06-26-2011', amount='200', category='Exercise', product='GymnasticsPro', mode='cash')
columns(tno='00000002', tdate='06-01-2011', amount='100', category='Exercise', product='GymnasticsPro', mode='cash')
```
