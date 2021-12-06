### 介绍

一种非关系型数据库，noSQL风格，不用查询语句查询的数据存储系统。其记录是一个文档，是由字段和值组成的数据结构，类似json格式，方便和对象形式进行转化。

关系型数据库和文档型数据库主要概念对应

|              | **关系型数据库** | **文档型数据库**       |
| ------------ | ---------------- | ---------------------- |
| **模型实体** | 表               | 集合                   |
| **模型属性** | 列               | 字段                   |
| **模型关系** | 表关联           | 内嵌数组，引用字段关联 |

### 基本操作

#### 插入

```js
db.集合.insertOne(<JSON对象>)   // 添加单个文档
db.集合.insertMany([{<JSON对象1>},{<JSON对象2>}])   // 批量添加文档
db.collection.insertMany(
[   {doc } , {doc }, ....],
  {
   writeConcern: 安全级别, 
   ordered: true/false
  }
)
```

writeConcern 定义了本次文档创建操作的安全写级别简单来说， 安全写级别用来判断一次数据库写入操作是否成功，安全写级别越高，丢失数据的风险就越低，然而写入操作的延迟也可能更高。

writeConcern 决定一个写操作落到多少个节点上才算成功。 writeConcern的取值包括

- 0： 发起写操作，不关心是否成功
- 1- 集群中最大数据节点数： 写操作需要被复制到指定节点数才算成功
- majority: 写操作需要被复制到大多数节点上才算成功

ordered:  觉得是否按顺序进行写入

- true：顺序写入时，一旦遇到错误，便会退出，剩余的文档无论正确与否，都不会写入
- false：乱序写入，则只要文档可以正确写入就会正确写入，不管前面的文档是否是错误的文档

```js
db.emp.insertOne(
{
 name:"zhangsan",
 age:20,
 sex:"m"}
);
```

插入文档时，如果没有显示指定主键，MongoDB将默认创建一个主键，字段固定为_id,ObjectId() 可以快速生成的12字节id 作为主键，**ObjectId** 前四个字节代表了主键生成的时间，精确到秒。主键ID在客户端驱动生成，一定程度上代表了顺序性，但不保证顺序性， 可以通过ObjectId("id值").getTimestamp() 获取创建时间。

#### 查询

```js
db.inventory.find({})               //查询所有的文档 
db.inventory.find({}).pretty()      //返回格式化后的文档

//精准等值查询
 db.inventory.find( { status: "D" } );
 db.inventory.find( { qty: 0 } );
//多条件查询
 db.inventory.find( { qty: 0, status: "D" } );
//嵌套对象精准查询
 db.inventory.find( { "size.uom": "in" } );
//返回指定字段(文档投影)，主键默认显示，非主键默认不显示，需通过设值为1显示
db.inventory.find( { }, { item: 1, status: 1 } );
//条件查询 and 
db.inventory.find({$and:[{"qty":0},{"status":"D"}]}).pretty(); 
//条件查询 or
db.inventory.find({$or:[{"qty":0},{"status":"D"}]}).pretty();
```

```js
构造一组数据:
db.members.insertMany([
{
  nickName:"曹操",
  points:1000
},
{
  nickName:"刘备",
  points:500
}
]);
//积分不小于100 的
db.members.find({points: { $not: { $lt: 100}}}  );
//昵称等于曹操， 积分大于 1000 的文档 
db.members.find({$and : [ {nickName:{ $eq : "曹操"}}, {points:{ $gt:1000}}]});
//名字为刘备或者积分大于1000的文档
db.members.find({$or : [{nickName:{ $eq : "刘备"}},{points:{ $gt:1000}}]});
```

**复合主键**

```js
db.demeDoc.insert(
{
	_id: {  product_name: 1,  product_type: 2},
	supplierId:" 001",
	create_Time: new Date()
})
```

注意复合主键，字段顺序换了，会当做不同的对象被创建，即使内容完全一致

**文档游标**

```js
db.movies.find().skip(1).count();
db.movies.find().skip(1).limit(1).count();
db.movies.find().skip(1).limit(1).count(true);
```

默认情况下 ， 这里的count不会考虑 skip 和 limit的效果，如果希望考虑 limit 和 skip ，需要设置为 true。 分布式环境下，count 不保证数据的绝对正确

**排序**

```js
cursor.sort({field:ordering})
ordering:1/-1
1自然序，-1逆序
```

当同时应用  sort， skip， limit 时 ，应用的顺序为   sort， skip， limit

**文档投影**

```js
db.collection.find(  查询条件,  投影设置)
//投影设置：主键字段只能选为0，非主键字段只能选为1。
db.members.find({},{_id:0 ,nickName:1, points:1})
```

**数组查询**

可以使用 $slice 返回数组中的部分元素

```js
db.members.insertOne(
{
 _id: {uid:3,accountType: "qq"},
 nickName:"张飞",
 points:1200,
 address:[
 {address:"xxx",post_no:0000},
 {address:"yyyyy",post_no:0002}
 ]}
);
//返回数组第一个元素
db.members.find(
{},
{_id:0, 
 nickName:1,
 points:1,
 address: 
    {$slice:1}
});
//返回倒数第一个
db.members.find(
{},
{_id:0, 
 nickName:1,
 points:1,
 address:{$slice:-1}}
 );
```

slice：

-  1： 数组第一个元素
- -1：最后一个元素
- -2：最后两个元素
- slice[ 1,2 ] : skip, limit  对应的关系 

slice: 值

 1： 数组第一个元素

-1：最后一个元素

-2：最后两个元素

slice[ 1,2 ] : skip, limit  对应的关系 

还可以使用 $elementMatch 进行数组元素进行匹配

```js
db.members.insertOne(
{
 _id: {uid:4,accountType: "qq"},
 nickName:"张三",
 points:1200,
 tag:["student","00","IT"]}
);
//查询tag数组中第一个匹配"00" 的元素
db.members.find(
  {},
  {_id:0,
   nickName:1,
   points:1,
   tag: {  $elemMatch:  {$eq:  "00" } }
});
```

#### 更新

- updateOne/updateMany 方法要求更新条件部分必须具有以下之一，否则将报错
- $set   给符合条件的文档新增一个字段，有该字段则修改其值
- $unset  给符合条件的文档，删除一个字段
- $push： 增加一个对象到数组底部 
- $pop：从数组底部删除一个对象 
- $pull：如果匹配指定的值，从数组中删除相应的对象
- $pullAll：如果匹配任意的值，从数据中删除相应的对象
- $addToSet：如果不存在则增加一个值到数组

```js
db.collection.update( <query>,<update>,<options>)
//<query> 定义了更新时的筛选条件
//<update> 文档提供了更新内容
//<options> 声明了一些更新操作的参数
```

```js
db.userInfo.insert([
{ name:"zhansan",
  tag:["90","Programmer","PhotoGrapher"]
},
{  name:"lisi",
   tag:["90","Accountant","PhotoGrapher"] 
}]);
//将tag 中有90 的文档，增加一个字段： flag: 1
db.userInfo.updateMany(
{tag:"90"},
{$set:{flag:1}}
);
//只修改一个,更新文档操作只会作用在第一个匹配的文档上
db.userInfo.updateOne(
{tag:"90"},
{$set:{flag:2}}
);
```

#### 删除

```js
db.collection.remove(<query>,<options>)
//默认情况下，会删除所有满足条件的文档， 可以设定参数 { justOne:true},只会删除满足添加的第一条文档
db.collection.drop( { writeConcern:<doc>})
//这个指令不但删除集合内的所有文档，且删除集合的索引
```

