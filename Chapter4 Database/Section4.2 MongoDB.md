# Python with MongoDB

- [Python with MongoDB](#python-with-mongodb)
  - [MongoDB intro](#mongodb-intro)
  - [xubuntu安装mongoDB client&server](#xubuntu%e5%ae%89%e8%a3%85mongodb-clientserver)
  - [mongoDB查询](#mongodb%e6%9f%a5%e8%af%a2)
  - [MongoDB高级](#mongodb%e9%ab%98%e7%ba%a7)
    - [aggregate, 聚合](#aggregate-%e8%81%9a%e5%90%88)
      - [`$group`](#group)
      - [`$match`](#match)
      - [`$project`](#project)
      - [`$sort`](#sort)
      - [`$limit`, `$skip`](#limit-skip)
      - [`$unwind`](#unwind)
  - [索引](#%e7%b4%a2%e5%bc%95)
  - [安全](#%e5%ae%89%e5%85%a8)
  - [复制(副本集)](#%e5%a4%8d%e5%88%b6%e5%89%af%e6%9c%ac%e9%9b%86)
  - [手动备份(mongodump) & 恢复(mongorestore)](#%e6%89%8b%e5%8a%a8%e5%a4%87%e4%bb%bdmongodump--%e6%81%a2%e5%a4%8dmongorestore)
  - [MongDB with python](#mongdb-with-python)
    - [MongoClient](#mongoclient)

## MongoDB intro

NoSQL，全名为Not Only SQL，指的是**非关系型**(非E-R)的数据库，内存级的数据读写，也支持物理存储；不需要维护复杂的关系(不需要建表，不需要关心表结构, 直接插入json数据)，只需要维护数据本身；

MongoDB 是一个基于**分布式、文件存储**的NoSQL数据库

![](res/nosql01.png)

用户第一次查找的时候，内存里面没有，就从物理的关系数据库将结果放到内存级NoSQL中；接下来的就直接在内存中查找，很快；

比如tmall网站的商品分类信息，长时间不变，而访问量大，可以放到NoSQL中；

Install MongoDB in windows:
- download [mongodb zip](https://www.mongodb.com/download-center/community)
- write `mongod.cfg` file
- setting password
- modify `mongod.cfg` with authorization
- `mongod --config mongod.cfg`
- client connect to mongodb server

```yaml
# mongod.cfg
# before authentication
systemLog:
  destination: file
  path: "MongoLog\\log.log"
  logAppend: false
storage:
  dbPath: "data"
net:
  bindIp: 127.0.0.1
  port: 27017
```

```bash
# setting password
mongo.exe

use admin
db.createUser({user:'grey', pwd:'xxxxxx', roles:[{role:'root',db:'admin'}]})
exit
```

```yaml
# mongod.cfg
# after modification
systemLog:
  destination: file
  path: "MongoLog\\log.log"
  logAppend: false
storage:
  dbPath: "data"
security:
  authorization: enabled
net:
  bindIp: 127.0.0.1
  port: 27017
```

## xubuntu安装mongoDB client&server

```bash
sudo apt install mongodb
mongo -version

# 开启服务端
# service mongodb start
sudo systemctl start mongodb

sudo vim /etc/mongodb.conf
# 将bind改为0.0.0.0

sudo systemctl restart mongodb
sudo systemctl enable mongodb
# 查看是否启动
ps ajx|grep mongo

# 使用mongodb客户端
# MySQL的服务器和客户端都是mysql
mongo
```

```bash
# output, 默认端口27017
connecting to: mongodb://127.0.0.1:27017
```

```bash
mongo

# 默认的数据库为test，如果你没有创建新的数据库，集合将存放在test数据库中
# current databse: db
> db
test
# current database information
> db.stats()
{
	"db" : "test",
	"collections" : 0,
	"views" : 0,
	"objects" : 0,
	"avgObjSize" : 0,
	"dataSize" : 0,
	"storageSize" : 0,
	"numExtents" : 0,
	"indexes" : 0,
	"indexSize" : 0,
	"fileSize" : 0,
	"ok" : 1
}
# delete current database
> db.dropDatabase()


> show databases;
admin  0.000GB
local  0.000GB
> show dbs;
admin  0.000GB
local  0.000GB

# switch database
use test;
```

一定要用GUI, 因为terminal中的mongo写json特别恶心；

```bash
# database operation
# 创建新数据库
> show dbs;
admin  0.000GB
local  0.000GB

# 直接use来创建
> use newDB;
switched to db newDB
> show dbs;
admin  0.000GB
local  0.000GB
> db
newDB
> db.createCollection('students')
{ "ok" : 1 }
# 这个时候才出现newDB
> show dbs
admin  0.000GB
local  0.000GB
newDB  0.000GB

> show collections
students
```


```bash
# collection operation
> db.students.drop()
true
> show collections
# 自动生成students collection
> db.students.insert({name:'grey',gender:true})
> show collections
students
> db.students.find()
{ "_id" : ObjectId("5acb50f2291a6cd3c230ae34"), "name" : "grey", "gender" : true }
# mongoDB的`objectID`不需要维护，会自动生成；objectID是一个12字节的十六进制数

> db.students.insert({name:'james',gender:true})
> db.students.find()
{ "_id" : ObjectId("5acb50f2291a6cd3c230ae34"), "name" : "grey", "gender" : true }
{ "_id" : ObjectId("5acb522c291a6cd3c230ae35"), "name" : "james", "gender" : true }

# mongoDB默认只修改一行
# 保持结构的修改
> db.students.update({},{$set:{name:'chris'}})
> db.students.find()
{ "_id" : ObjectId("5acb542c291a6cd3c230ae36"), "name" : "chris", "gender" : true }
{ "_id" : ObjectId("5acb5432291a6cd3c230ae37"), "name" : "james", "gender" : true }
# 不保持结构的修改
> db.students.update({},{name:'tom'})
> db.students.find()
{ "_id" : ObjectId("5acb542c291a6cd3c230ae36"), "name" : "tom" }
{ "_id" : ObjectId("5acb5432291a6cd3c230ae37"), "name" : "james", "gender" : true }

# 多行修改
> db.students.update({},{$set:{name:'moriaty'}},{multi:true})
> db.students.find()
{ "_id" : ObjectId("5acb542c291a6cd3c230ae36"), "name" : "moriaty" }
{ "_id" : ObjectId("5acb5432291a6cd3c230ae37"), "name" : "moriaty", "gender" : true }

# 清空collection
> db.students.remove({})

# 一般用法
> db.students.update({gender:true},{$set:{gender:false}},{multi:true})
> db.students.find()
{ "_id" : ObjectId("5acb5641291a6cd3c230ae38"), "name" : "grey", "gender" : false }
{ "_id" : ObjectId("5acb5647291a6cd3c230ae39"), "name" : "james", "gender" : false }
```

example: pymongo example

```py
import pymongo

client=pymongo.MongoClient('localhost:27017')
db=client['weixin']

# .....

if db['articles'].update({'title': data['title']}, {'$set': data}, True):
    # 根据data['title']来检查重复
    # 设置为True, 如果没有同样的data就插入，有同样的data就更新
    print('update success')
```

## mongoDB查询

mongoDB的`size`规定了collection的最大尺寸，超出的话，会从第一个开始覆盖；

```bash
db.createCollection('subjects',{'capped':true,size:5})
```

```bash
# 直接robo3t在insert document输入
{
    "title" : "python",
    "count" : 10
}

{
    "title" : "linux",
    "count" : 12
}

{
    "title" : "mongo",
    "count" : 2
}

{
    "title" : "redis",
    "count" : 1
}

{
    "title" : "python",
    "count" : 10
}
```

```bash
# 只查询一个
db.subjects.findOne()

db.subjects.find({count:{$gt:2}})

# 用js函数十分强大, 一般就这么干
db.students.find({$where:function(){return this.age>20}})
# 名字以g开头的document；通过搜索js字符串函数找到indexOf函数
db.students.find({$where:function(){return this.name.indexOf('g')==0}})
# 名字包含g的document
db.students.find({$where:function(){return this.name.indexOf('g')>=0}})
```

```bash
# 类型MySQL的limit()

# 只显示两条
db.sub.find().limit(2)
# 跳过2条
db.sub.find().skip(2)
# 跳过两条显示2条,顺序无所谓
db.sub.find().skip(2).limit(2)
db.sub.find().limit(2).skip(2)

#居然可以用循环来插入数据
for(i=0;i<15;i++){db.t1.insert({_id:i})}
```

```bash
# 投影projection, 只显示部分列
# 只显示name列
db.sub.find({},{name:1})
db.sub.find({},{name:1,count:0})
# 不显示_id
db.sub.find({},{name:1,_id:0})
```

```bash
# sort, 默认按照_id排序，因为_id前四位是时间戳，是有顺序的
# 按照name升序
db.sub.find().sort({name:1})

# 按照name降序
db.sub.find().sort({name:-1})
```

```bash
# count
db.sub.count({$where:function(){return this.count>2}})
# 或者这样
db.sub.find({$where:function(){return this.count>2}}).count()
```

```bash
# distinct
db.stu.distinct('gender',{age:{$gt:18}})
```

## MongoDB高级

### aggregate, 聚合

aggregate, 聚合, 用于统计；

`db.collection_name.aggregate([{管道1:{表达式1}}, {管道2:{表达式2}}])`

文档在一个管道处理完毕后将结果传递给下一个管道处理

#### `$group`

```bash
# data
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed489"),
    "name" : "james",
    "gender" : false,
    "age" : 22
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48b"),
    "name" : "chris",
    "gender" : true,
    "age" : 23
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48d"),
    "name" : "moris",
    "gender" : false,
    "age" : 33
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48f"),
    "name" : "tom",
    "gender" : false,
    "age" : 36
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed491"),
    "name" : "tim",
    "gender" : true,
    "age" : 43
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed493"),
    "name" : "arthor",
    "gender" : false,
    "age" : 56
}
```

```bash
# group
db.students.aggregate([
    {$group:{_id:'$gender'}}
])
#output
{
    "_id" : true
}
{
    "_id" : false
}


# sum document
# 按照gender分组，并且统计每一组document的个数
# 有一行算一个所以是num:{$sum:1}
db.students.aggregate([
    {$group:{_id:'$gender',num:{$sum:1}}}
])
# output
{
    "_id" : true,
    "num" : 2.0
}
{
    "_id" : false,
    "num" : 4.0
}


# 所有文档分到一组
db.students.aggregate([
    {$group:{_id:null,num:{$sum:1},avg_age:{$avg:'$age'}}}
])
#ouput
{
    "_id" : null,
    "num" : 6.0,
    "avg_age" : 35.5
}


# age:sum,avg,min,max,first,last,push
# age sum
db.students.aggregate([
    {$group:{_id:'$gender',sum_age:{$sum:'$age'}}}
])
#output
{
    "_id" : true,
    "sum_age" : 66
}
{
    "_id" : false,
    "sum_age" : 147
}


# age push 单个字段
db.students.aggregate([
    {$group:{_id:'$gender',array:{$push:'$age'}}}
])
#ouput
{
    "_id" : true,
    "array" : [ 
        23, 
        43
    ]
}
{
    "_id" : false,
    "array" : [ 
        22, 
        33, 
        36, 
        56
    ]
}


# push所有字段 $$ROOT
db.students.aggregate([
    {$group:{_id:'$gender',array:{$push:'$$ROOT'}}}
])
#ouput
/* 1 */
{
    "_id" : true,
    "array" : [ 
        {
            "_id" : ObjectId("5ace2b9a83c9c14727bed48b"),
            "name" : "chris",
            "gender" : true,
            "age" : 23
        }, 
        {
            "_id" : ObjectId("5ace2b9a83c9c14727bed491"),
            "name" : "tim",
            "gender" : true,
            "age" : 43
        }
    ]
}
{
    "_id" : false,
    "array" : [ 
        {
            "_id" : ObjectId("5ace2b9a83c9c14727bed489"),
            "name" : "james",
            "gender" : false,
            "age" : 22
        }, 
        {
            "_id" : ObjectId("5ace2b9a83c9c14727bed48d"),
            "name" : "moris",
            "gender" : false,
            "age" : 33
        }, 
        {
            "_id" : ObjectId("5ace2b9a83c9c14727bed48f"),
            "name" : "tom",
            "gender" : false,
            "age" : 36
        }, 
        {
            "_id" : ObjectId("5ace2b9a83c9c14727bed493"),
            "name" : "arthor",
            "gender" : false,
            "age" : 56
        }
    ]
}
```

#### `$match`

类似`find()`, 只是在`aggregate`中, 用于过滤

```bash
# 年龄大于40的男女人数及姓名
# $match后面的{}可以写入之前find()第一个参数能写的所有东西
db.students.aggregate([
    {$match:{age:{$gt:40}}}, {$group:{_id:'$gender', num:{$sum:1},item:{$push:'$name'}}}
])
# output
{
    "_id" : false,
    "num" : 1.0,
    "item" : [ 
        "arthor"
    ]
}
{
    "_id" : true,
    "num" : 1.0,
    "item" : [ 
        "tim"
    ]
}
```


#### `$project`

```bash
# $project后面的和find()的第二个参数用法相同
# 在上面的基础上不显示_id
db.students.aggregate([
    {$match:{age:{$gt:40}}}, {$group:{_id:'$gender', num:{$sum:1},item:{$push:'$name'}}}, {$project:{_id:0}}
])
# output
{
    "num" : 1.0,
    "item" : [ 
        "arthor"
    ]
}
{
    "num" : 1.0,
    "item" : [ 
        "tim"
    ]
}

# 在上面基础上，重命名
db.students.aggregate([
    {$match:{age:{$gt:40}}}, {$group:{_id:'$gender', num:{$sum:1},item:{$push:'$name'}}}, {$project:{'number':'$num','item':1}}
])
# output
{
    "_id" : false,
    "item" : [ 
        "arthor"
    ],
    "number" : 1.0
}
{
    "_id" : true,
    "item" : [ 
        "tim"
    ],
    "number" : 1.0
}
```

#### `$sort`

```bash
# 男女分组然后按照数目逆序
db.students.aggregate([
    {$group:{_id:'$gender',num:{$sum:1}}},{$sort:{num:-1}}
])
# output
{
    "_id" : false,
    "num" : 4.0
}
{
    "_id" : true,
    "num" : 2.0
}
```

#### `$limit`, `$skip`

`find().limit().skip()`中的skip, limit不分顺序；`aggregate()`中的`$skip, $limit`分顺序

```bash
# 显示3条数据
db.students.aggregate([
    {$limit:3}
])
# output
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed489"),
    "name" : "james",
    "gender" : false,
    "age" : 22
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48b"),
    "name" : "chris",
    "gender" : true,
    "age" : 23
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48d"),
    "name" : "moris",
    "gender" : false,
    "age" : 33
}


# 跳过3条数据
db.students.aggregate([
    {$skip:3}
])
# output
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48f"),
    "name" : "tom",
    "gender" : false,
    "age" : 36
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed491"),
    "name" : "tim",
    "gender" : true,
    "age" : 43
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed493"),
    "name" : "arthor",
    "gender" : false,
    "age" : 56
}

# skip & limit
db.students.aggregate([
    {$match:{age:{$gt:30}}},
    {$skip:1},
//     {$limit:2}
])
# output
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48f"),
    "name" : "tom",
    "gender" : false,
    "age" : 36
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed491"),
    "name" : "tim",
    "gender" : true,
    "age" : 43
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed493"),
    "name" : "arthor",
    "gender" : false,
    "age" : 56
}

# skip & limit
db.students.aggregate([
    {$match:{age:{$gt:30}}},
    {$skip:1},
    {$limit:2}
])
# output
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48f"),
    "name" : "tom",
    "gender" : false,
    "age" : 36
}
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed491"),
    "name" : "tim",
    "gender" : true,
    "age" : 43
}

# limit & skip
db.students.aggregate([
    {$match:{age:{$gt:30}}},
    {$limit:2},
    {$skip:1},
])
# output
{
    "_id" : ObjectId("5ace2b9a83c9c14727bed48f"),
    "name" : "tom",
    "gender" : false,
    "age" : 36
}
```

#### `$unwind`

```bash
# data
{
    "_id" : ObjectId("5aceb9a083c9c14727bed68f"),
    "item" : "t-shirt",
    "size" : [ 
        "s", 
        "m", 
        "l"
    ]
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed691"),
    "item" : "pants",
    "size" : []
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : [ 
        "s", 
        "m", 
        "l", 
        "xl", 
        "xxl"
    ]
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed695"),
    "item" : "golve",
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed697"),
    "item" : "coat",
    "size" : null
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed699"),
    "item" : "jeans",
    "size" : [ 
        "s", 
        "m", 
        "l"
    ]
}
```

`$unwind`拆开数组

用`$unwind`拆开之后可以用`$push`合并起来；

```bash
db.cloth.aggregate([
    {$match:{'item':'hat'}},
    {$unwind:'$size'}
])
# output
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "s"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "m"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "l"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "xl"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "xxl"
}
```

查看查询结果，发现对于**空数组、非数组、无字段、null**的文档，都被丢弃了
问：如何能不丢弃呢？

```bash
db.cloth.aggregate([
    {$unwind:{path:'$size',preserveNullAndEmptyArrays:true}}
])
# output
{
    "_id" : ObjectId("5aceb9a083c9c14727bed68f"),
    "item" : "t-shirt",
    "size" : "s"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed68f"),
    "item" : "t-shirt",
    "size" : "m"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed68f"),
    "item" : "t-shirt",
    "size" : "l"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed691"),
    "item" : "pants"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "s"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "m"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "l"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "xl"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed693"),
    "item" : "hat",
    "size" : "xxl"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed695"),
    "item" : "golve"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed697"),
    "item" : "coat",
    "size" : null
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed699"),
    "item" : "jeans",
    "size" : "s"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed699"),
    "item" : "jeans",
    "size" : "m"
}
{
    "_id" : ObjectId("5aceb9a083c9c14727bed699"),
    "item" : "jeans",
    "size" : "l"
}
```

```bash
db.students.aggregate([
    {$group:{'_id':'$gender',all_number:{$sum:1},docs:{$push:'$$ROOT'}}},
    {$unwind:'$docs'},
    {$limit:3}
])
# output
{
    "_id" : true,
    "all_number" : 2.0,
    "docs" : {
        "_id" : ObjectId("5ace2b9a83c9c14727bed48b"),
        "name" : "chris",
        "gender" : true,
        "age" : 23
    }
}
{
    "_id" : true,
    "all_number" : 2.0,
    "docs" : {
        "_id" : ObjectId("5ace2b9a83c9c14727bed491"),
        "name" : "tim",
        "gender" : true,
        "age" : 43
    }
}
{
    "_id" : false,
    "all_number" : 4.0,
    "docs" : {
        "_id" : ObjectId("5ace2b9a83c9c14727bed489"),
        "name" : "james",
        "gender" : false,
        "age" : 22
    }
}
```

## 索引

虽然mongoDB实在内存中搜索，如果涉及巨大的数据量，还是会慢，所以也用到索引；然而redis没有索引；

```bash
# 运行mongDB的shell既是运行mongoDB的窗口，同时也是js的编译器
# for其实就是javascript的循环语句
# 向collection中插入10w条文档
for(i=0;i<100000;i++){
    db.test.insert({name:'test'+i,age:i+1})
    }
```

查询`test10000`的文档

```bash
db.test.find({name:'test10000'})
# output
{
    "_id" : ObjectId("5acec428cb706f94dd5c3f4d"),
    "name" : "test10000",
    "age" : 10001.0
}

# 查询性能分析
# 主要是为了得到 "executionTimeMillis" : 42,
db.test.find({name:'test10000'}).explain('executionStats')
# output
/* 1 */
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "newDB.test",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "name" : {
                "$eq" : "test10000"
            }
        },
        "winningPlan" : {
            "stage" : "COLLSCAN",
            "filter" : {
                "name" : {
                    "$eq" : "test10000"
                }
            },
            "direction" : "forward"
        },
        "rejectedPlans" : []
    },
    "executionStats" : {
        "executionSuccess" : true,
        "nReturned" : 1,
        "executionTimeMillis" : 42,
        "totalKeysExamined" : 0,
        "totalDocsExamined" : 100000,
        "executionStages" : {
            "stage" : "COLLSCAN",
            "filter" : {
                "name" : {
                    "$eq" : "test10000"
                }
            },
            "nReturned" : 1,
            "executionTimeMillisEstimate" : 43,
            "works" : 100002,
            "advanced" : 1,
            "needTime" : 100000,
            "needYield" : 0,
            "saveState" : 782,
            "restoreState" : 782,
            "isEOF" : 1,
            "invalidates" : 0,
            "direction" : "forward",
            "docsExamined" : 100000
        }
    },
    "serverInfo" : {
        "host" : "grey-pc",
        "port" : 27017,
        "version" : "3.4.7",
        "gitVersion" : "cf38c1b8a0a8dca4a11737581beafef4fe120bcd"
    },
    "ok" : 1.0
}

# 建立索引，1是升序，-1是降序
db.test.ensureIndex({name:1})
# output
{
    "createdCollectionAutomatically" : false,
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "ok" : 1.0
}

# 再次查询性能
db.test.find({name:'test10000'}).explain('executionStats')
# "executionTimeMillis" : 1,
```

索引的命令：


建立唯一索引，实现唯一约束(这个值不能重复)的功能：`db.test.ensureIndex({name:1},{unique:true})`

建立联合索引: `db.test.ensureIndex({name:1, age:1})`

查看所有索引: `db.test.getIndexes()`

删除索引: `db.test.dropIndexes(index_name)`, 一般都是在robo3t直接删

```bash
db.test.getIndexes()
# output
[
    {
        "v" : 2,
        "key" : {
            "_id" : 1
        },
        "name" : "_id_",
        "ns" : "newDB.test"
    },
    {
        "v" : 2,
        "unique" : true,
        "key" : {
            "name" : 1.0
        },
        "name" : "name_1",
        "ns" : "newDB.test"
    }
]
```

## 安全

采用了**角色-用户-数据库**的安全管理方式

常用系统角色如下：

- root：只在admin数据库中可用，超级账号，超级权限
- Read：允许用户读取指定数据库
- readWrite：允许用户读写指定数据库

The **admin** database is unique in MongoDB. User with normal access to **admin** database have read and write access to **all database**

```bash
mongo
> use admin
> db.createUser({
...     user:'grey',
...     pwd:'13810455459',
...     roles:[{role:'root',db:'admin'}]
... })
> exit

# edit config file
sudo vim /etc/mongodb.conf
# Turn on/off security.  Off is currently the default
#noauth = true
auth = true

# 重启服务
service mongodb restart
mongo
# 下面的提示和开始不同了
MongoDB shell version v3.4.7
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.7
> show dbs; # 出现errormsg
> exit
# 所以需要用admin身份验证去登陆admin数据库
mongo -u grey -p --authenticationDatabase 'admin'
```

```bash
# robo3t中查看用户
db.system.users.find()
# output
{
    "_id" : "admin.grey",
    "user" : "grey",
    "db" : "admin",
    "credentials" : {
        "SCRAM-SHA-1" : {
            "iterationCount" : 10000,
            "salt" : "jFCdi7A2IN4XagD1fN8HvA==",
            "storedKey" : "FhfJdLX03vfhppgpTH51oUVAr2s=",
            "serverKey" : "Ey8mksSIIIiO8Eoo2gmCutHARnY="
        }
    },
    "roles" : [ 
        {
            "role" : "root",
            "db" : "admin"
        }
    ]
}
```

创建普通用户:

- method1: 在NoSQL Manager for MongoDB的GUI中创建
- method2: 在terminal中创建

```bash
# 登陆超级管理员，随便use一个数据库，然后
db.createUser({
    user:'moris',
    pwd:'123456',
    roles:[{role:'readWrite',db:'newDB'}]
    })
# 都会在admin/Collections/system.users下面创建用户
mongo -u moris -p 123456 --authenticationDatabase newDB

# 超级管理员修改用户
db.updateUser('moris',{pwd:'456789'})
```

## 复制(副本集)

MongoDB提供副本集：

- 备份
- 无宕机切换

复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性，并可以保证数据的安全性

客户端先访问主服务器，如果主服务器挂了，回去访问备份服务器(备份服务器每隔5ms备份一次主服务器：心跳时间)，所以备份服务器也可能丢掉一部分数据；

mongoDB自己会自动做；主从节点任意切换；

在主服务器以及从服务器之前一定有一个东西来分配：也就是**反向代理、负载均衡**

```bash
# 先关闭mongodb服务
service mongodb stop
# in xshell
ssh 192.168.128.133
```

xshell两个tab连接同一个xbuntu, 然后

```bash
# t1, t2用来保存数据库
# 以前都是存在默认数据库下面的/data/db
# 现在都存在桌面好了
mkdir t1
mkdir t2

# 测试mongod，看看有没有--bind_ip
mongod --help

# 最重要的是让副本集一致,rs0
# 分别在xshell的两个tab中输入下面两个命令
# 开启两个服务(服务器)，开启在同一个机器和不同机器没有区别
mongod --bind_ip 192.168.128.133 --port 27017 --dbpath ~/Desktop/t1 --replSet rs0
mongod --bind_ip 192.168.128.133 --port 27018 --dbpath ~/Desktop/t2 --replSet rs0
# 实际运用中，也需要createUser,这里就省略了
```

已经有两个服务器了，用xshell弄1个客户端；连接那27018端口那个服务器；

```bash
mongo --host 192.168.128.133 --port 27018

# 客户端设置主服务器，输入
> rs.initiate()
{
	"info2" : "no configuration specified. Using a default configuration for the set",
	"me" : "192.168.128.133:27018",
	"ok" : 1
}
rs0:SECONDARY> rs.status()
{
	"set" : "rs0",
	"date" : ISODate("2018-04-12T08:26:35.135Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1523521592, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1523521592, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1523521592, 1),
			"t" : NumberLong(1)
		}
	},
    # 当前的成员只有自己27018，要将27017加入进来
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.128.133:27018",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 652,
			"optime" : {
				"ts" : Timestamp(1523521592, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-04-12T08:26:32Z"),
			"infoMessage" : "could not find member to sync from",
			"electionTime" : Timestamp(1523521491, 2),
			"electionDate" : ISODate("2018-04-12T08:24:51Z"),
			"configVersion" : 1,
			"self" : true
		}
	],
	"ok" : 1
}

# 加入20017端口的服务器, 可以反复的添加形成一个集群
rs0:PRIMARY> rs.add('192.168.128.133:27017')
{ "ok" : 1 }
rs0:PRIMARY> rs.status()
{
	"set" : "rs0",
	"date" : ISODate("2018-04-12T08:29:48.203Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1523521782, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1523521782, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1523521782, 1),
			"t" : NumberLong(1)
		}
	},
    # 现场成功加入了27017端口的服务器作为SECONDARY
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.128.133:27018",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 845,
			"optime" : {
				"ts" : Timestamp(1523521782, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-04-12T08:29:42Z"),
			"electionTime" : Timestamp(1523521491, 2),
			"electionDate" : ISODate("2018-04-12T08:24:51Z"),
			"configVersion" : 2,
			"self" : true
		},
		{
			"_id" : 1,
			"name" : "192.168.128.133:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 5,
			"optime" : {
				"ts" : Timestamp(1523521782, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1523521782, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-04-12T08:29:42Z"),
			"optimeDurableDate" : ISODate("2018-04-12T08:29:42Z"),
			"lastHeartbeat" : ISODate("2018-04-12T08:29:46.568Z"),
			"lastHeartbeatRecv" : ISODate("2018-04-12T08:29:47.640Z"),
			"pingMs" : NumberLong(0),
			"configVersion" : 2
		}
	],
	"ok" : 1
}

# xshell再开一个客户端tab
mongo --host 192.168.128.133 --port 27017
# 得到secondary的样子
rs0:SECONDARY> rs.status()
{
	"set" : "rs0",
	"date" : ISODate("2018-04-12T08:42:50.620Z"),
	"myState" : 2,
	"term" : NumberLong(1),
	"syncingTo" : "192.168.128.133:27018",
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1523522562, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1523522562, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1523522562, 1),
			"t" : NumberLong(1)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.128.133:27018",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 788,
			"optime" : {
				"ts" : Timestamp(1523522562, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1523522562, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-04-12T08:42:42Z"),
			"optimeDurableDate" : ISODate("2018-04-12T08:42:42Z"),
			"lastHeartbeat" : ISODate("2018-04-12T08:42:50.329Z"),
			"lastHeartbeatRecv" : ISODate("2018-04-12T08:42:49.218Z"),
			"pingMs" : NumberLong(0),
			"electionTime" : Timestamp(1523521491, 2),
			"electionDate" : ISODate("2018-04-12T08:24:51Z"),
			"configVersion" : 2
		},
		{
			"_id" : 1,
			"name" : "192.168.128.133:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 1650,
			"optime" : {
				"ts" : Timestamp(1523522562, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-04-12T08:42:42Z"),
			"syncingTo" : "192.168.128.133:27018",
			"configVersion" : 2,
			"self" : true
		}
	],
	"ok" : 1
}
```

![](res/mongo01.png)

那么最后，SECONDARY会定时向PRIMARY拿数据


```bash
# 往主服务器中插入数据
rs0:PRIMARY> show dbs
admin  0.000GB
local  0.000GB
rs0:PRIMARY> use newDBB
rs0:PRIMARY> db
newDBB
rs0:PRIMARY> for(i=0;i<100;i++){db.stu.insert({name:'test'+i,age:i+1})}
rs0:PRIMARY> db.stu.find()
{ "_id" : ObjectId("5acf1daa36d32907b4312274"), "name" : "test0", "age" : 1 }
{ "_id" : ObjectId("5acf1daa36d32907b4312275"), "name" : "test1", "age" : 2 }
{ "_id" : ObjectId("5acf1daa36d32907b4312276"), "name" : "test2", "age" : 3 }
{ "_id" : ObjectId("5acf1daa36d32907b4312277"), "name" : "test3", "age" : 4 }
{ "_id" : ObjectId("5acf1daa36d32907b4312278"), "name" : "test4", "age" : 5 }
{ "_id" : ObjectId("5acf1daa36d32907b4312279"), "name" : "test5", "age" : 6 }
{ "_id" : ObjectId("5acf1daa36d32907b431227a"), "name" : "test6", "age" : 7 }
{ "_id" : ObjectId("5acf1daa36d32907b431227b"), "name" : "test7", "age" : 8 }
{ "_id" : ObjectId("5acf1daa36d32907b431227c"), "name" : "test8", "age" : 9 }
{ "_id" : ObjectId("5acf1daa36d32907b431227d"), "name" : "test9", "age" : 10 }
{ "_id" : ObjectId("5acf1daa36d32907b431227e"), "name" : "test10", "age" : 11 }
{ "_id" : ObjectId("5acf1daa36d32907b431227f"), "name" : "test11", "age" : 12 }
{ "_id" : ObjectId("5acf1daa36d32907b4312280"), "name" : "test12", "age" : 13 }
{ "_id" : ObjectId("5acf1daa36d32907b4312281"), "name" : "test13", "age" : 14 }
{ "_id" : ObjectId("5acf1daa36d32907b4312282"), "name" : "test14", "age" : 15 }
{ "_id" : ObjectId("5acf1daa36d32907b4312283"), "name" : "test15", "age" : 16 }
{ "_id" : ObjectId("5acf1daa36d32907b4312284"), "name" : "test16", "age" : 17 }
{ "_id" : ObjectId("5acf1daa36d32907b4312285"), "name" : "test17", "age" : 18 }
{ "_id" : ObjectId("5acf1daa36d32907b4312286"), "name" : "test18", "age" : 19 }
{ "_id" : ObjectId("5acf1daa36d32907b4312287"), "name" : "test19", "age" : 20 }
Type "it" for more
```

```bash
# 再SECONDARY服务器上面设置slaveOk()
rs0:SECONDARY> rs.slaveOk()
rs0:SECONDARY> db.stu.find().pretty()
{
	"_id" : ObjectId("5acf1daa36d32907b4312274"),
	"name" : "test0",
	"age" : 1
}
{
	"_id" : ObjectId("5acf1daa36d32907b4312276"),
	"name" : "test2",
	"age" : 3
}
{
	"_id" : ObjectId("5acf1daa36d32907b4312275"),
	"name" : "test1",
	"age" : 2
}
{
	"_id" : ObjectId("5acf1daa36d32907b431227c"),
	"name" : "test8",
	"age" : 9
}
{
	"_id" : ObjectId("5acf1daa36d32907b431227f"),
	"name" : "test11",
	"age" : 12
}
{
	"_id" : ObjectId("5acf1daa36d32907b4312277"),
	"name" : "test3",
	"age" : 4
}
{
	"_id" : ObjectId("5acf1daa36d32907b4312279"),
	"name" : "test5",
	"age" : 6
}
```

关闭主服务器后(`ctrl+c`)，再重新启动(`mongod --bind_ip 192.168.128.133 --port 27018 --dbpath ~/Desktop/t2 --replSet rs0`)，会发现原来的从服务器变为了从服务器，新启动的服务器（原来的从服务器）变为了从服务器

```bash
grey@grey-pc:~$ mongo --host 192.168.128.133 --port 27018
rs0:SECONDARY> 

grey@grey-pc:~$ mongo --host 192.168.128.133 --port 27017
rs0:PRIMARY> 
```

```bash
# 删除现在的从节点
rs.remove('192.168.128.133:27018')
```

## 手动备份(mongodump) & 恢复(mongorestore)

`mongodump -h dbhost -d dbname -o dbdirectory`

- -h：服务器地址，也可以指定端口号
- -d：需要备份的数据库名称
- -o：备份的数据存放位置，此目录中存放着备份出来的数据

```bash
mkdir mongo_bak
mongodump -u grey -p 13810455459 --authenticationDatabase 'admin' -h localhost -d newDB -o mongo_bak/
# 或者这种方法
mongodump -u grey -p 13810455459 --authenticationDatabase 'admin' -h 192.168.128.133 -d newDB -o mongo_bak/
# 或者
mongodump -u grey -p 13810455459 --authenticationDatabase 'admin' -h 192.168.128.133:27017 -d newDB -o mongo_bak/

# 一个collection对应一个bson和一个json
# 得到的结果，二进制数据放在bson中，结构放在json中
mongo_bak
└── newDB
    ├── cloth.bson
    ├── cloth.metadata.json
    ├── students.bson
    ├── students.metadata.json
    ├── test.bson
    └── test.metadata.json
```

`mongorestore -h dbhost -d dbname --dir dbdirectory`

- -h：服务器地址
- -d：需要恢复的数据库实例
- --dir：备份数据所在位置

而且这里不用手动建立数据库，直接恢复就可以了


```bash
# 恢复，必须是超级用户才能创建数据库，所以是grey
mongorestore -u grey -p 13810455459 --authenticationDatabase 'admin' -h 192.168.128.133:27017 -d newDDBB --dir mongo_bak/newDB
# 登录查看一下
mongo -u grey -p 13810455459 --authenticationDatabase 'admin'
> show dbs
admin    0.000GB
local    0.000GB
newDB    0.004GB
newDDBB  0.004GB
```

## MongDB with python

`pip install pymongo`

常用[函数](http://api.mongodb.com/python/current/api/pymongo/collection.html)：

- `insert_one()`
- `insert_many()`
- `update_one()`
- `update_many()`
- `delete_one()`
- `delete_many()`
- `find_one()`
- `find()`


### MongoClient

```python
import pymongo

# # 无安全认证
# client1 = pymongo.MongoClient('mongodb://192.168.128.133:27017')
# # 有安全认证, url方式
client = pymongo.MongoClient('mongodb://grey:13810455459@192.168.128.133:27017/admin')
# get database
db1 = client.newDB
# get collection
students = db1.students

# insert
# 返回值是_id
s1 = students.insert_one({'name': 'moriaty', 'age': 23})
print(s1)
```

```python
# update_one
students.update_one({'name':'moriaty'},{'$set':{'name':'moria'}})


# delete_one
students.delete_one({'name':'moria'})


# find_one
print(students.find_one())
# output
{'_id': ObjectId('5ace2b9a83c9c14727bed489'), 'name': 'james', 'gender': False, 'age': 22}


# find, 返回cursor对象
cursor=students.find({'gender':False})
for item in cursor:
    print(item)
# output
{'_id': ObjectId('5ace2b9a83c9c14727bed489'), 'name': 'james', 'gender': False, 'age': 22}
{'_id': ObjectId('5ace2b9a83c9c14727bed48d'), 'name': 'moris', 'gender': False, 'age': 33}
{'_id': ObjectId('5ace2b9a83c9c14727bed48f'), 'name': 'tom', 'gender': False, 'age': 36}
{'_id': ObjectId('5ace2b9a83c9c14727bed493'), 'name': 'arthor', 'gender': False, 'age': 56}


# sort
cursor=students.find({'gender':False}).sort('age',1)
for item in cursor:
    print(item['name'],item['age'])
# output
james 22
moris 33
tom 36
arthor 56


# sort with 2 fields
cursor=students.find().sort([('name', 1),('age', -1)])
for item in cursor:
    print(item['name'],item['age'])
# output
arthor 56
chris 23
james 22
moris 33
tim 43
tom 36


# skip, limit
cursor=students.find({'gender':False}).sort('age',1).skip(1).limit(3)
for item in cursor:
    print(item['name'],item['age'])
# output
moris 33
tom 36
arthor 56


# limit, skip
cursor=students.find({'gender':False}).sort('age',1).limit(3).skip(1)
for item in cursor:
    print(item['name'],item['age'])
# output
moris 33
tom 36
arthor 56
```

example: simple pymongo

```py
import pymongo

# client=pymongo.MongoClient(host='localhost', username='grey', password='xxxxxx')
client=pymongo.MongoClient('mongodb://grey:xxxxxx@localhost')

db=client['test']
collection=db['lsm']

for i in range(100):
    data={'name':f'stu-{i}', 'age':i, 'score':100+i}
    collection.update({'name':f'stu-{i}'}, {'$set':data}, upsert=True)

for j in collection.find():
    # type is dict
    print(j, type(j))

collection.insert({'name':'grey', 'data':[1, 2, 3, 4]})
collection.delete_one({'name':'grey'})
collection.find_one({'name':'stu-10'})
collection.find_one_and_replace({'name':'stu-98'}, {'name':'moris', 'data':'hello'})

data={'name':'stu-999', 'age':10}
collection.find_one_and_update({'name':'stu-99'}, {'$set':data})
```

```py
import pymongo

client=pymongo.MongoClient('mongodb://grey:xxxxxx@localhost')
collection=client.test.lsm
# collection=client['test']['lsm']

data=dict(name='grey', age=66, score=78)
collection.insert(data)

# update data
data['name']='moris'
collection.save(data)

# add new data to db
new_data={'name':'moris','IQ':100}
collection.save(new_data)

# update data
user=collection.find_one({'name':'stu-0'})
user['age']=66
collection.save(user)
collection.find_one({'name':'stu-0'}) # age: 66

# every document's age +1
collection.update({}, {'$inc':{'age':1}}, multi=True)
# every document's score-50
collection.update({}, {'$inc':{'score':-50}}, multi=True)
# query and age+10, set new field: sex
collection.update({'name':'moris'}, {'$inc':{'age':10},'$set':{'sex':1}},multi=True)

# empty collection
collection.delete_many({})

# drop collection
collection.drop()
```

pymongo new api:
- `update`: `replace_one`, `update_one`, `update_many`
- `save`:`replace_one`, `insert_one`
- `insert`:`insert_one`, `insert_many`
- `remove`: `delete_one`, `delete_many`
- `count`:`count_documents`

```py
import pymongo

client=pymongo.MongoClient('mongodb://grey:xxxxxx@localhost')
collection=client.test.lsm

# count
print(collection.count())
collection.count({'age':{'$lt':23, '$gt':10}})
collection.count_documents({'age':{'$lt':23, '$gt':10}})
collection.find({'age':{'$lt':23, '$gt':10}}).count()

# projection
for j in collection.find({}, projection = ['name', 'score']):
    print(j)

# projection with sort
for j in collection.find({}, ['name', 'score']).sort([('score', pymongo.DESCENDING), ('name', pymongo.ASCENDING)]):
    print(j)

# projection with sort
for j in collection.find({}, ['name', 'score']).sort([('score', -1), ('name', 1)]):
    print(j)

# pagination everything page with 3 items
for i in range(4):
    for j in collection.find().skip(i*3).limit(3):
        print(j)
    print('-'*20)

# query
for j in collection.find({'age':{'$lt':23, '$gt':10}, 'score':{'$gte': 5}}):
    print(j)

# find record with these fields
for i in collection.find({'IQ':{'$exists':True}}):
    print(i)

# query with regex
for j in collection.find({'name':{'$regex':r'^girl-\d$'}}):
    print(j)
```

example: cascade data

```py
import pymongo

client=pymongo.MongoClient('mongodb://grey:xxxxxx@localhost')
collection=client.test.users

collection.update({'name':'grey'}, {'$set':{'name':'grey', 'age': 23, 'data':[1, 2, 3, 4]}}, upsert=True)
collection.find_one({'data':3})

collection.update({'name':'moris'}, {'$set':{'name':'moris', 'age': 25, 'data':{'email':'grey@163.com', 'address':'beijing', 'money':1000}}}, upsert=True)
collection.update({'data.email':'grey@163.com'}, {'$set':{'data.email':'grey@foxmail.com'}}, upsert=True)

collection.find_one({'data.address':'beijing'})
collection.find_one({'data.address':'beijing'})['data']
collection.find_one({'data.address':'beijing'})['data']['email']
collection.find_one({'data.email':{"$exists":True}})
collection.find_one({'data.money':{'$gt':200}})


collection.update({'name':'tom'}, {'$set':{'name':'tom', 'age': 30, 'data':[{'gold':1000, 'fund':2000}, 'shanghai', 'beijing']}}, upsert=True)

collection.find_one({'data':'shanghai'})
collection.find_one({'data.gold':1000, 'data.fund':{'$gt': 100}})
collection.find_one({'data':{'$elemMatch':{'gold':1000, 'fund':{'$lt':3000}}}})
collection.find_one({'name':'tom'})['data'][1]
collection.find_one({'data.2':'beijing'})
```

example: mongodb [query operators](https://docs.mongodb.com/manual/reference/operator/query/)

```py
import pymongo

client=pymongo.MongoClient('mongodb://grey:xxxxxx@localhost')
collection=client.test.users

# find contains all the item in list
collection.find_one({'data':{'$all':[1,2,3]}})
# find list size=4
collection.find_one({'data':{'$size':4}})
# find list size isn't 4
collection.find_one({'data':{'$not':{'$size':4}}})

collection.find_one({'name':{'$type':2}}) # type 2 is string
collection.find_one({'data':{'$type':3}}) # type 3 is array

# remove a field
collection.update({'name':'tom'}, {'$unset':{'age':1}})
collection.update({'name':'moris'}, {'$unset':{'age':''}})

# add data to array
collection.update({'name':'grey'}, {'$push':{'data':666}})
# 加入的是list，而不是逐个加入
collection.update({'name':'grey'}, {'$push':{'data':[22,33,44]}})
# add data to array if not exist
collection.update({'name':'grey'}, {'$addToSet':{'data':111}})

# push list
collection.update({'name':'grey'}, {'$push':{'data':{'$each':[777,888,999]}}})
# push list if not exist
collection.update({'name':'grey'}, {'$addToSet':{'data':{'$each':[777,888,999]}}})

# pop last
collection.update({'name':'grey'}, {'$pop':{'data':1}})
# pop first
collection.update({'name':'grey'}, {'$pop':{'data':-1}})

# batch remove data
collection.update({'name':'grey'}, {'$pull':{'data':{'$lt':4}}})
collection.update({'name':'grey'}, {'$pullAll':{'data':[777,888,999]}})

# where query: 本质是使用js代码来代替$lt, $gt
collection.find_one({'$where':'this.age>20 && this.age<30'})
```

example: pymongo index

```py
import pymongo

client=pymongo.MongoClient('mongodb://grey:xxxxxx@localhost')
collection=client.test.users

# create index
collection.create_index([('name', pymongo.ASCENDING),])
[i for i in collection.list_indexes()]
# or 
collection.index_information()
# when modification and update, do reindex
collection.reindex()

# query analysis
collection.find({'name':'moris'}).explain()

# drop index
collection.drop_index('name_1')
# drop index except _id
collection.drop_indexes()
[i for i in collection.list_indexes()]

# unique index
collection.create_index([('name', -1)], unique=True)

collection.find({'name':'moris'}).hint([('name', -1)]).explain()
```

example: pymongo map-reduce

```json
// data
{
    "cust_id" : "a123",
    "amount" : 500,
    "status" : "a"
}
{
    "cust_id" : "a123",
    "amount" : 250,
    "status" : "a"
}
{
    "cust_id" : "b212",
    "amount" : 200,
    "status" : "a"
}
{
    "cust_id" : "b212",
    "amount" : 400,
    "status" : "a"
}
{
    "cust_id" : "a123",
    "amount" : 300,
    "status" : "d"
}
```

```py
import pymongo

client=pymongo.MongoClient('mongodb://grey:xxxxxx@localhost')
collection=client.test.users

map_func='function() {emit(this.cust_id,this.amount);}'
# map_func='function() {emit(this.cust_id,1);}' # 表示统计cust_id个数
reduce_func='function(key, values){return Array.sum(values);}'

new_collection=collection.map_reduce(map_func, reduce_func, 'myresult', query={'status':'a'})
for i in new_collection.find():
    print(i)
```

```json
// result
{
    "_id" : "a123",
    "value" : 750.0
}
{
    "_id" : "b212",
    "value" : 600.0
}
```

example: pymongo with spider

```py
import pymongo
import requests
from lxml import etree

client=pymongo.MongoClient('mongodb://grey:xxxxxx@xx.xx.xx.xx')
collection=client.test.movie

r=requests.get('https://www.meijutt.tv/new100.html')
tree=etree.HTML(r.content.decode('gbk'))

title_list=tree.xpath('//li/h5/a/@title')
type_list=tree.xpath("//li/span[@class='mjjq']/text()")
tv_list=tree.xpath("//li/span[@class='mjtv']/text()")
status_list=tree.xpath("//li/span/font/text()")

for data in zip(title_list, status_list, type_list, tv_list):
    collection.update({'title':data[0]}, {'$set':{'title':data[0], 'status':data[1], 'type':data[2], 'tv':data[3]}}, upsert=True)
```