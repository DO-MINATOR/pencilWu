### 聚合操作简介

MongoDB聚合框架是一个计算框架作用在一个或几个集合对集合中的数据进行**一系列**运算，将这些数据转化为期望的形式。

```js
db.orders.insertMany(
[
{
 zip:"000001",
 phone:"13101010101",
 name:"LiuBei",
 status:"created",
 shippingFee:10,
 orderLines:[
 	{product:"Huawei Meta30 Pro",sku:"2002",qty:100,price:6000,cost:5599},
 	{product:"Huawei Meta40 Pro",sku:"2003",qty:10,price:7000,cost:6599},
 	{product:"Huawei Meta40 5G",sku:"2004",qty:80,price:4000,cost:3700}
 ]
},

{
 zip:"000001",
 phone:"13101010101",
 name:"LiuBei",
 status:"created",
 shippingFee:10,
 orderLines:[
 	{product:"Huawei Meta30 Pro",sku:"2002",qty:100,price:6000,cost:5599},
 	{product:"Huawei Meta40 Pro",sku:"2003",qty:10,price:7000,cost:6599},
 	{product:"Huawei Meta40 5G",sku:"2004",qty:80,price:4000,cost:3700}
 ]
},

{
 zip:"000001",
 phone:"13101010101",
 name:"LiuBei",
 status:"created",
 shippingFee:10,
 orderLines:[
 	{product:"Huawei Meta30 Pro",sku:"2002",qty:100,price:6000,cost:5599},
 	{product:"Huawei Meta40 Pro",sku:"2003",qty:10,price:7000,cost:6599},
 	{product:"Huawei Meta40 5G",sku:"2004",qty:80,price:4000,cost:3700}
 ]
}]
);
```

添加两个字段，值为每个订单的原价总价和 订单总额

```js
db.orders.aggregate([{$addFields: {
  totalPrice:{  $sum: "$orderLines.price"},
  totalCost: {  $sum: "$orderLines.cost"},
}}]);
```

按 totalPrice 进行排序

```js
[
{
    $addFields: {
        totalPrice: {
            $sum: "$orderLines.price"
        },
        totalCost: {
            $sum: "$orderLines.cost"
        },
    }
}
,   // stage 1 
{
    $sort: {
        totalPrice: -1
    }
}
]     // stage 2 
```

第一个阶段，针对整个集合添加两个字段，totalPrice，totalCost，并将结果传到第二个阶段，按订单总价进行排序。

### 聚合操作

- $project  对输入文档进行再次投影
- $match 对输入文档进行筛选
- $limit 筛选出管道内前 N 篇文档
- $skip 跳过管道内前N篇文档
- $unwind 展开输入文档中的数组字段
- $sort 对输入文档进行排序
- $lookup 对输入文档进行查询操作
- $group 对输入文档进行分组
- $out 对管道中的文档输出

```js
db.userInfo.insertMany(
 [
  {nickName:"zhangsan",age:18},
  {nickName:"lisi",age:20}]
 );
//集合中的 nickName 投影成 name
db.userInfo.aggregate({$project:{ name:"$nickName"}});
//控制输出格式
db.userInfo.aggregate({$project:{ name:"$nickName",_id:0,age:1}});
```

```js
//文档筛选
db.userInfo.aggregate({$match:{ nickName:"lisi"}});
```

筛选管道操作和其他管道操作配合时候时，尽量放到开始阶段，这样可以减少后续管道操作符要操作的文档数，提升效率

```js
//筛选和文档投影结合使用
db.userInfo.aggregate(
 [ 
   { $match:   { nickName: "lisi"}},
   { $project:  { _id:0, name: "$nickName", age:1}}
]
);
```

```js
//集合数量筛选和跳过
db.userInfo.aggregate({$limit:1});
db.userInfo.aggregate({$skip:1});
```

```js
db.userInfo.insertOne(
{ "nickName" : "xixi", 
  "age" : 35, 
  "tags" : ["80","IT","BeiJing"]
}
);
//数组展开
db.userInfo.aggregate(
{  $unwind:{path:"$tags"}}
);
```

```js
//文档排序
db.userInfo.aggregate({$sort:{age:-1}});
```

```js
db.account.insertMany(
 [
 {_id:1,name:"zhangsan",age:19},
 {_id:2,name:"lisi",age:20}
]
);
db.accountDetail.insertMany(
[
{aid:1,address:["address1","address2"]}
]
);
//$lookup关联查询
db.accountDetail.aggregate(
{  
  $lookup:  
         {  
           from:"account",
           localField:"aid",
           foreignField:"_id",
           as: "field1"
        }
   }
);
```

**group分组操作**

- $addToSet ：将分组中的元素添加到一个数组中，并且自动去重
- $avg   返回分组中的平均值， 非数值直接忽略
- $first   返回分组中的第一个元素
- $last    返回分组中的最后一个元素
- $max  返回分组中的最大元素
- $min  回分组中的最小元素
- $push   创建新的数组，将值添加进去
- $sum    求分组数值元素和

```js
db.sales.insertMany(
[
{ "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, "date" : ISODate("2014-01-01T08:00:00Z") },
{ "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, "date" : ISODate("2014-02-03T09:00:00Z") },
{ "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 5, "date" : ISODate("2014-02-03T09:05:00Z") },
{ "_id" : 4, "item" : "abc", "price" : 10, "quantity" : 10, "date" : ISODate("2014-02-15T08:00:00Z") },
{ "_id" : 5, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:12:00Z") },
{ "_id" : 6, "item" : "xyz", "price" : 5, "quantity" : 10, "date" : ISODate("2014-02-15T09:12:00Z") }
]
);
```

```js
//查看每天，卖出哪几种商品项目，按每天分组， 将商品加入到去重数组中 
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } },
           itemsSold: { $addToSet: "$item" }
         }
     }
   ]
)
```

```js
//$avg:  求数值的平均值
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: "$item",
           avgAmount: { $avg: { $multiply: [ "$price", "$quantity" ] } },
           avgQuantity: { $avg: "$quantity" }
         }
     }
   ]
)
```

```js
//$push，创建新的数组，存储，每个分组中元素的信息
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } },
           itemsSold: { $push:  { item: "$item", quantity: "$quantity" } }
         }
     }
   ]
)
```

group 阶段有 100m内存的使用限制， 默认情况下，如果超过这个限制会直接返回 error，可以通过设置  allowDiskUse 为 true 来避免异常， allowDiskUse 为 true 将利用临时文件来辅助实现group操作。

```js
//$out将聚合结果写入另一个文档
db.sales.aggregate(
   [
     {
       $group:
         {
           _id: { day: { $dayOfYear: "$date"}, year: { $year: "$date" } },
           itemsSold: { $push:  { item: "$item", quantity: "$quantity" } }
         }
     },
    {  $out:"output"}
   ]
)
```

### 聚合优化

1. 投影优化
   聚合管道可以确定它是否仅需要文档中的字段的子集来获得结果。如果是这样，管道将只使用那些必需的字段，减少通过管道的数据量。 
2. 管道符号执行顺序优化
   对于包含投影阶段($project或$unset或$addFields或$set)后跟$match阶段的聚合管道，MongoDB 将$match阶段中不需要在投影阶段计算的值的任何过滤器移动到投影前的新$match阶段
3. $sort+$match
   如果序列中带有$sort后跟$match，则$match会移动到$sort之前，以最大程度的减少要排序的对象的数量
4. $project/ $unset + $skip序列优化
   当有一个$project或$unset之后跟有$skip序列时，$skip 会移至$project之前。 
5. $limit+ $limit合并
   当$limit紧接着另一个时 $limit，两个阶段可以合并为一个阶段 $limit，其中限制量为两个初始限制量中的较小者。
6. skip+ $skip 合并
   当$skip紧跟另一个$skip，这两个阶段可合并成一个单一的$skip，其中跳过量为总和的两个初始跳过量。
7. $match+ $match合并
   当一个$match紧随另一个紧随其后时 $match，这两个阶段可以合并为一个单独 $match的条件 $and

```js
{ $match: { year: 2014 } },
{ $match: { status: "A" } }
//优化后 
{ $match: { $and: [ { "year" : 2014 }, { "status" : "A" } ] } }
```

### 索引

索引是特殊的数据结构，它以一种易于遍历的形式存储集合数据集的一小部分。索引存储一个或一组特定字段的值，按字段的值排序。索引项的排序支持有效的**相等匹配**和基于**范围的查询**操作。此外，MongoDB可以通过使用索引中的排序返回**排序后**的结果。

**单键索引**，如基于主键ID 进行的B+ tree 数据结构。**复合索引**，只支持前缀子查询

**默认id索引**，在创建集合期间，MongoDB 在_id字段上创建唯一索引。该索引可防止客户端插入两个具有相同值的文档。你不能将_id字段上的index删除。

```js
//创建一个单键索引
db.members.createIndex({name:1});
//索引的默认名称是索引键和索引中每个键的方向(即1或-1)的连接，使用下划线作为分隔符， 也可以通过指定 name 来自定义索引名称；对于单字段索引和排序操作，索引键的排序顺序(升序或降序)并不重要，因为MongoDB可以从任何方向遍历索引。 
db.members.createIndex({name:1}，{ name: "whatever u like."});

//创建复合索引，复合索引中列出的字段的顺序具有重要意义。如果一个复合索引由 {name: 1, age: -1} 组成，索引首先按name 升序排序，然后在每个name值内按 age 降序 排序。
db.members.createIndex({ name:1,age:-1});

//创建多键索引，MongoDB使用多键索引来索引存储在数组中的内容。如果索引包含数组值的字段，MongoDB为数组的每个元素创建单独的索引项。数组字段中的每一个元素，都会在多键索引中创建一个键
db.members.createIndex( { tags:1});
```

**覆盖查询**

当查询条件和查询的<投影>只包含索引字段时，MongoDB直接从索引返回结果，而不扫描任何文档或将文档带入内存。这些覆盖的查询可能非常高效。

```js
db.members.explain().find().sort( {name:1 ,age:-1}) ;              
```

使用已创建索引的字段进行排序，能利用索引的顺序，不需要重新排序，效率高

**唯一索引**

索引的unique属性使MongoDB拒绝索引字段的重复值。除了唯一性约束，唯一索引和MongoDB其他索引功能上是一致的

```js
db.members.createIndex({age:1},{unique:true});
```

- 如果文档中的字段已经出现了重复值，则不可以创建该字段的唯一性索引
- 如果新增的文档不具备加了唯一索引的字段，则只有第一个缺失该字段的文档可以被添加，索引中该键值被置为null。

复合键索引也可以具有唯一性，这种情况下，不同的文档之间，其所包含的复合键字段值的组合不可以重复。

使用唯一索引+稀疏索引可以保存多个具有null值的索引。

**索引生存时间**

针对日期字段，或者包含了日期元素的数组字段，可以使用设定了生存时间的索引，来自动删除字段值超过生存时间的文档。

```js
db.members.insertMany( [ 
    {
     name:"zhangsanss",
     age:19,tags:["00","It","SH"],
     create_time:new Date()}
    ] );
```

```js
//在create_time字段上面创建了一个生存时间是30s的索引，超过30s自动删除该索引及其记录
db.members.createIndex({ create_time: 1},{expireAfterSeconds:30 });
```

