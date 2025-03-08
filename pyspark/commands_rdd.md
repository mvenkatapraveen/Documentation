# Resilient Distributed Dataset (RDD) Basics

## 1. Create RDD list from a list

Convert list of integers to RDD list of integers

```sh
#print("=====================List of Integers=====================")
l= [1,2,3,4,5]
print(l)

#print("=====================RDD List of Integers=====================")
rddl = sc.parallelize(l)
print(rddl.collect())

```

Convert list of strings to RDD list of strings

```sh
#print("=====================List of Strings=====================")
lstr = ["vk", "vp","vnl"]
print(lstr)

#print("=====================RDD List of Strings=====================")
rddl = sc.parallelize(lstr)
print(rddl.collect())
```

## 2. Perform operation on each element of list

Convert list of integers to RDD list of integers and then perform operation on each element

```sh
#print("=====================List of Integers=====================")
l= [1,2,3,4,5]
print(l)

#print("=====================RDD List of Integers=====================")
rddl = sc.parallelize(l)
print(rddl.collect())

#print("=====================Modified List=====================")
finalrdd = rddl.map(lambda x: x+10)
print(finalrdd.collect())
```

## 3. Filter some elements of list

Convert list of integers to RDD list of integers and apply filters

```sh
#print("=====================List of Integers=====================")
l= [1,2,3,4,5]
print(l)

#print("=====================RDD List of Integers=====================")
rddl = sc.parallelize(l)
print(rddl.collect())

#print("=====================Filtered List=====================")
finalrdd = rddl.filter(lambda x: x>2)
print(finalrdd.collect())
```


## 4. Flatten the elements of list

convert list of integers to RDD list of integers and flatten the data

```sh
#print("=====================Raw Data=====================")
l = ['A~B','C~D','E~F']
print(l)

#print("=====================RDD Data=====================")
rddl = sc.parallelize(l)
print(rddl.collect())

#print("=====================Flattened Data=====================")
flatdata = rddl.flatMap(lambda x: x.split('~'))
print(flatdata.collect())
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
data = sc.textFile("../dt.txt")
print("==========Raw Data==============")
print(data.collect())
data.foreach(print)


mapsplit = data.map(lambda x: x.split(','))
print("==========Map Split Data==============")
mapsplit.foreach(print)

columns = namedtuple('columns', ['tno', 'tdate', 'amount', 'category', 'product', 'mode'])

mappedcolumns = mapsplit.map(lambda x: columns(x[0],x[1],x[2],x[3],x[4],x[5]))

filtereddata = mappedcolumns.filter(lambda x: 'Gymnastics' in x.product)
print("==============Filtered Data================")
filtereddata.foreach(print)
```
