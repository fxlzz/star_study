这个简直了，安装也是麻烦的很，启动也是麻烦的很... 不过用起来还是很方便的，不错不错

进入正题，

mongodb 是非关系型数据库，相对的 mysql 是关系型数据库

启动服务：

```
mongod --dbpath data文件路径，例如：
mongod --dbpath D:\software\mongodb\server\data
```

对了，mongodb 的 shell 程序需要自行下载，下载 mongosh

```
mongosh
```

# 基础概念

- `db`：和`mysql`的概念一致
- `collection`：集合，类似于`mysql`中的表
- `document`：每个集合中的文档，类似于`mysql`中的记录

- `Primary Key`：和`mysql`中的主键含义一致，每个`document`都有一个主键
- `field`：文档中的字段

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1741782759311-4dbaab52-a8ac-4ea7-a1bd-897ccd2b73dc.png)

`mongodb`属于`nosql`中的文档型数据库，每个文档相当于是一个对象，它没有列的概念，也没有表关系

由于它是一个`nosql`数据库：

- 无`sql`语句
- 使用极其简单，学习成本非常低
- 由于没有集合之间的关联，难以表达复杂的数据关系
- 存取速度极快

由于它是一个`文档型`数据库：

- 数据内容非常丰富和灵活
- 对数据结构难以进行有效的限制

# mongodb->node.js

## mongoose

```
npm i mongoose
```

## 连接数据库

使用`mongoose`node.js 的 mongodb 的驱动，连接 mongodb 数据库

```
const mongoose = require("mongoose");
mongoose.connect("mongodb://localhost/test");

mongoose.connection.on("open", () => {
  console.log("连接已打开");
});
```

创建 Schema，就跟 node.js 中的 ORM 模型，差不多，通过模型来定义数据库的表结构，当然在这里是文档结构

## 创建 Schema

```
const addressSchema = require("./addressSchema");
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const userSchema = new Schema({
  // 用户文档结构
  loginId: {
    type: String, // 类型是字符串
    required: true, // 必填
    trim: true,
    minlength: 3,
    maxlength: 18,

    index: true, // 将该字段设置为索引
    unique: true, // 特殊索引，唯一索引
  },
  loginPwd: {
    type: String, // 类型是字符串
    required: true, // 必填
    trim: true,
    select: false, // 默认情况下，查询用户时不会查询该字段
  },
  name: {
    type: String, // 类型是字符串
    required: true, // 必填
    trim: true,
    minlength: 2,
    maxlength: 10,
  },
  age: {
    type: Number,
    default: 18,
  },
  loves: {
    type: [String], // 类型是一个数组，数组每一项是一个字符串
    required: true,
    default: [], // 默认值为一个空数组
  },
  address: {
    type: addressSchema,
    required: true,
  },
});

// User 是文档名，结构是 userSchema
module.exports = mongoose.model("User", userSchema);
```

# CRUD

## 插入数据

### 原生插入数据方法

创建或插入操作用于将新[文档](https://www.mongodb.com/zh-cn/docs/manual/core/document/#std-label-bson-document-format)添加到[集合](https://www.mongodb.com/zh-cn/docs/manual/core/databases-and-collections/#std-label-collections)中。如果集合当前不存在，插入操作会创建集合。

MongoDB 提供以下方法将文档插入到集合中：

- [db.collection.insertOne()](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.insertOne/#mongodb-method-db.collection.insertOne)
- [db.collection.insertMany()](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.insertMany/#mongodb-method-db.collection.insertMany)

例如：在 test 数据库中，在 test 文档中添加一条数据

```
db.test.insertOne({name: "tom", age: 18})
```

### mongoose 插入数据

mongoose 提供两种插入数据的方法，

方法一：创建 model 对象，通过 new 关键字来添加数据

```
let User = mongoose.model("User", userSchema);

//向文档中添加数据
let user = new User({
  loginId:"abc", 
  loginPwd: "123456",
  name: "tom"
})
user.save();
```

方法二：model.create() 调用 api

```
let User = mongoose.model("User", userSchema); 
User.create({
  loginId:"abc", 
  loginPwd: "123456",
  name: "tom"
}).then(res => console.log(res))

// 调用 create() API可以传入多个对象，也可以传入数组对象[{}, {}]
```

【注意】插入很多条数据，还是使用`User.insertMany()`一次性添加到数据库中，mongoose 中提供的 create API 是添加一个数据，创建一个数据，速度比较慢

## 读取数据

### 原生读取数据方法

根据条件、投影查询指定集合，返回**游标**

[db.collection.find([filter], [projection])](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)

【补充】

使用 _id 作为查询条件时，需要调用 ObjectId 函数

例如，

```
db.users.find({
  _id: ObjectId('67d2858ddb800758715e53ae')
})
```

### **游标** cursor

查询返回的是一个**游标对象**，它类似于**迭代器（很像链表的指针，只不过这个游标不能回退）**，可以在查询结果中进行迭代

`cursor`的成员：

- `next()`：游标向后移动，并返回下一个结果，如果没有结果则报错
- `hasNext()`：判断游标是否还能向后移动，返回`boolean`
- `skip(n)`：跳过前面的`n`条数据，**返回**`**cursor**`
- `limit(n)`：取当前结果的`n`条数据，**返回**`**cursor**`
- `sort(sortObj)`：按照指定的条件排序，**返回**`**cursor**` **-1 表示降序 1 表示升序**
- `count()`：得到符合`filter`的结果数量，返回`Number`
- `size()`：得到最终结果的数量，返回`Number`

由于某些函数会继续返回`cursor`，因此可以对其进行**链式编程**，返回`cursor`的函数成为了链中的一环，无论它们的调用顺序如何，始终按照下面的顺序执行：

```
sort -> skip -> limit
```

例如：

```
// 查询user文档中的数据
db.user.find().skip(2).limit(3).sort({age: 1}) 
// 先根据age升序排序，跳过2条数据，取3条数据，注意这里返回的还是cursor

db.user.find().limit(3).size() // 这个得出的是经过filter、经过limit后的结果长度 3
db.user.find().count() // 这个
```

### 条件运算符

`find`函数的第一个参数是查询条件`filter`，它的写法极其丰富，下面列举了大部分情况下我们可能使用到的写法。

【注意】条件与条件之间默认是**并且**的关系

```
// 查询所有 name="曹敏" 的用户
{
  name: "曹敏" 
}

// 查询所有 loginId 以 7 结尾 并且 name 包含 敏 的用户
{
  loginId: /7$/ , 
    name: /敏/  
}

// 查询所有 loginId 以 7 结尾 或者 name 包含 敏 的用户
{
  $or: [
    {
      loginId: /7$/,
    },
    {
      name: /敏/  
    },
  ],
}

// 查询所有年龄等于18 或 20 或 25 的用户
{
  age: {
    $in: [18, 20, 25]
  }
}

// 查询所有年龄不等于18 或 20 或 25 的用户
{
  age: {
    $nin: [18, 20, 25]
  }
}

// 查询所有年龄在 20~30 之间的用户
{
  age: {
    $gt: 20,
    $lt: 30
  }
}
```

查询中出现了一些特殊的属性，它以`$`开头，表达了特殊的查询含义，这些属性称之为`操作符 operator`

查询中的常用操作符包括：

- `$or`：或者
- `$and`：并且
- `$in`：在...之中
- `$nin`：不在...之中
- `$gt`：大于
- `$gte`：大于等于
- `$lt`：小于
- `$lte`：小于等于
- `$ne`：不等于

【注意】，由于 mongodb 支持 js 操作，在筛选条件中，查询数组还可以当作对象使用

`{"loves.0": "吃饭"}`

### 投影

也就是 find 查询的**第二个参数**，类似于 mysql 中的 `select name, age from user;`这条语句中只展示 name，age 两个参数。

就是控制输出结果要展示那些数据，

```
// 查询结果中仅包含 name、age，以及会自动包含的 _id
{
  name: 1,
  age: 1
}

// 查询结果不能包含 loginPwd、age，其他的都要包含
{
  loginPwd: 0,
  age: 0
}

// 查询结果中仅包含 name、age，不能包含_id
{
  name: 1,
  age: 1,
  _id: 0
}

// 错误：除了 _id 外，其他的字段不能混合编写
{
  name: 1,
  age: 0
}
```

例如：

```
db.users.find({
  name: /^M/
}, {
  name: 1,
  age: 1
})

// 返回的数据
{
  _id: ObjectId('67d28b127ee0768e42472113'), // _id 是默认携带的
  name: 'Michael',
  age: 53
}
```

### mongoose中的查询

```
<Model>.findById(id); // 按照id查询单条数据
<Model>.findOne(filter, projection); // 根据条件和投影查询单条数据
<Model>.find(filter, projection); // 根据条件和投影查询多条数据
```

`findOne`和`find`如果没有给予回调或等待，则不会真正的进行查询，而是返回一个`DocumentQuery`对象，可以通过`DocumentQuery`对象进行链式调用进一步获取结果，直到传入了回调、等待、调用`exec`时，才会真正执行。

链式调用中包括：

- `count`
- `limit`
- `skip`
- `sort`

例如，

```
async function test(){
const res = await User.find({ age: { $gte: 20, $lte: 30 } }, { name: 1, age: 1, _id: 0 })
    .skip((page - 1) * limit)
    .limit(limit)
    .sort({
      age: 1,
    });
}
test();
```

## 更新操作

### 原生更新数据方法

更新操作用于修改[集合](https://www.mongodb.com/zh-cn/docs/manual/core/document/#std-label-bson-document-format)中的现有[文档](https://www.mongodb.com/zh-cn/docs/manual/core/databases-and-collections/#std-label-collections) 。

- [db.collection.updateOne(filter, update, [options])](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.updateOne/#mongodb-method-db.collection.updateOne)
- [db.collection.updateMany()](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.updateMany/#mongodb-method-db.collection.updateMany)
- [db.collection.replaceOne()](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.replaceOne/#mongodb-method-db.collection.replaceOne)

![](https://cdn.nlark.com/yuque/0/2025/svg/51701855/1741833639732-2f97bc28-286c-4395-b32b-eacd6b095820.svg)

其中，filter 读取数据中的条件运算符，也可以使用

update 要更新的数据，以及相匹配的运算符

options 相关配置，只需了解一个即可，

- `upsert`：默认`false`，若无法找到匹配项，则进行添加

### 更新运算符

第二个参数决定了更新哪些字段，它的常见写法如下：

1. 字段操作

```
// 将匹配文档的 name 设置为 邓哥，address.city 设置为 哈尔滨
{
  $set: { name:"邓哥", "address.city": "哈尔滨" }
}

// 将匹配文档的 name 设置为 邓哥，并将其年龄增加2
{
  $set: { name:"邓哥" },
  $inc: { age: 2 } // 在原有基础上增加 2 increase
}

// 将匹配文档的 name 设置为 邓哥，并将其年龄乘以2
{
  $set: { name:"邓哥" },
  $mul: { age: 2 } // 在原有基础上乘以 2 multiply
}

// 将匹配文档的 name 字段修改为 fullname
{
  $rename: { name: "fullname" } // 修改字段名
}

// 将匹配文档的 age 字段、address.province 字段 删除
{
  $unset: {age:"", "address.province":""}
}
```

2. 数组操作

```
// 向 loves 添加一项：秋葵
// 若数组中不存在则进行添加 若存在则不进行任何操作
{
  $addToSet: {
    loves: "秋葵"
  }
}

// 向 loves 添加一项：秋葵
// 无论数组中是否存在，都必定会添加
{
  $push: {
    loves: "秋葵"
  }
}

// 向 loves 添加多项：秋葵 香菜
{
  $push: {
    loves: { $each: ["秋葵", "香菜"]}
  }
}

// 删除 loves 中满足条件的项: 是秋葵 或 香菜
{
  $pull: {
    loves: {$in: ["秋葵","香菜"]}
  }
}

// 将所有loves中的 其他 修改为 other
// 该操作符需要配合查询条件使用
db.users.updateOne({
  loves: "其他"
}, {
  $set: {
    "loves.$": "other"
  }
})
```

### mongoose 更新文档

```
async function test() {
  const u = await User.findById("67d29640266a9a4d9eec632b");
  // 得到对象，直接操作即可
  u.address.province = "黑龙江";
  u.loves.push("秋葵", "香菜");
  await u.save(); // 此时会自动对比新旧文档，完成更新
}
test();
```

## 删除操作

删除操作用于从集合中删除文档，mongoose 删除文档差不多

- [db.collection.deleteOne(filter)](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.deleteOne/#mongodb-method-db.collection.deleteOne)
- [db.collection.deleteMany(filter)](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.deleteMany/#mongodb-method-db.collection.deleteMany)

在 MongoDB 中，删除操作针对的是单个[集合](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary/#std-term-collection)。

![](https://cdn.nlark.com/yuque/0/2025/svg/51701855/1741833648515-1edd93eb-7e8c-480c-bebd-f78510e91a9d.svg)

# 索引

## 原生创建索引

```
// 为某个集合创建索引
db.<collection>.createIndex(keys, [options]);
```

- `keys`：指定索引中关联的字段，以及字段的排序方式，1为升序，-1为降序

```
// 索引关联 age 字段，按照升序排序
{ age: 1 }
```

- `options`索引的配置

- `background`：默认`false`，建索引过程会阻塞其它数据库操作，是否以后台的形式创建索引
- `unique`：默认`false`，是否是唯一索引
- `name`：索引名称

在mongodb中，索引的存储结构是B-树

## 其他索引操作

```
// 查看所有索引
db.<collection>.getIndexes()
// 查看集合索引占用空间
db.<collection>.totalIndexSize()
// 删除所有索引
db.<collection>.dropIndexes()
// 删除集合指定索引
db.<collection>.dropIndex("索引名称")
```

## mongoose 创建索引

这个简单，在创建 Schema 时，直接在某个属性（想要其成为索引）配置一下就可以了~

```
index: true, // 将该字段设置为索引
unique: true, // 特殊索引，唯一索引
```