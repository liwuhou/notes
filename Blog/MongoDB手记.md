---
title: MongoDB手记
date: 2019-10-07
tags: MongoDB
summary: MongoDB的一下简单的操作指令
---

MongoDB是一个NoSQL的数据库，不需要写SQL语句，而且里面存取的格式都是前端友好的JSON对象的形式。

<!-- more -->

### MongoDB的安装

安装好MongoDB数据库后，我们需要启用服务端才能使用。启用服务的命令是：Mongod。

1. 打开命令行:先打开运行（快捷键win+R），然后输入cmd后回车，就可以打开命令行工具。
2. 执行mongod:在命令中直接输入mongod，但是你会发现服务并没有启动，报了一个exception，服务停止了。
3. 新建文件夹:出现上边的错误，是因为我们没有简历Mongodb需要的文件夹，一般是安装盘的根目录，建立data/db,这两个文件夹。
4. 运行mongod：这时候服务就可以开启了，链接默认端口是27017。

**链接服务：**

服务器开启之后，可以使用`mongo`命令来链接服务端

查看存在数据库命令：`show dbs`

查看数据库版本命令： `db.version()`

### MongoDB的基本命令

虽然在`Mongo Shell`中，语法跟`javascript`很相像，但是在打印信息的时候，并不是调用`console.log()`，而是`print()`，这个稍有区别

``` javascript
var x = 'Hello World!';
print(x);
var y = function(){
    return 'william';
}
print(y());
```

**MongoDB的存储结构**

以前关系型数据库的数据结构都是**顶层是库，库下面是表，表下面数据**，但是`MongoDB`有所不同：**顶层还是库，库下面是集合，集合下面是文件**

 在学习中我们可以对比记忆，这样才能更好的了解这些名词，其实数据表就是集合，数据行就是文件，当然这只是为了记忆，实质还是有区别的。 

**一些基本的`Shell`命令**

了解存储结构后，就可以开始学习我们的基础Shell命令了，因为这些命令比较基础，我会以列表形式展现，具体使用方法可以到视频中进行观看。

- `show dbs` :显示已有数据库，如果你刚安装好，会默认有`local`、`admin(config)`，这是`MongoDB`的默认数据库，我们在新建库时是不允许起这些名称的。
- `use [dbsName]`： 进入数据，也可以理解成为使用数据库。成功会显示：`switched to db admin`。
- `show collections`: 显示数据库中的集合（关系型中叫表，我们要逐渐熟悉）。
- `db`:显示当前位置，也就是你当前使用的数据库名称，这个命令算是最常用的，因为你在作任何操作的时候都要先查看一下自己所在的库，以免造成操作错误。

**数据操作的基本命令**

- `use db`（建立数据库）：`use`不仅可以进入一个数据库，如果你敲入的库不存在，它还可以帮你建立一个库。但是在没有集合前，它还是默认为空。
- `db.[collectionName].insert( )`:新建数据集合和插入文件（数据），当集合没有时，这时候就可以新建一个集合，并向里边插入数据。Demo：`db.user.insert({“name”:”William”})`
- `db.[collectionName].find( )`:查询所有数据，这条命令会列出集合下的所有数据，可以看到MongoDB是自动给我们加入了索引值的。Demo：`db.user.find()`
- `db.[collectionName].findOne( )`:查询第一个文件数据，这里需要注意的， **所有MongoDB的组合单词都使用首字母小写的驼峰式写法。** 
- `db.[collectionName].update({查询},{修改})`:修改文件数据，第一个是查询条件，第二个是要修改成的值。这里注意的是可以多加文件数据项的，比如下面的例子。demo:`db.user.update({name: 'william'}, {name: 'skye', age: '18'})`

- `db.[collectionName].remove(条件)`：删除文件数据，注意的是要跟一个条件。Demo:db.user.remove({“name”:”William”})
- `db.[collectionName].drop( )`:删除整个集合，这个在实际工作中一定要谨慎使用，如果是程序，一定要二次确认。
- `db.dropDatabase( )`:删除整个数据库，在删除库时，一定要先进入数据库，然后再删除。实际工作中这个基本不用，实际工作可定需要保留数据和痕迹的。

#### **`update`方法中还有很多修改器**

|    名称     | 作用                                                              | 示例                                                                                                                                            |
| :---------: | :---------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------- |
|   `$set`    | 修改一个指定的键值                                                | `db.test.update({name: 'william'}, {$set: {sex: 1}})`<br />`db.test.update({name: 'william'}, {$set: {'interest.2': 'code'}})`                  |
|  `$unset`   | 将key删除                                                         | `db.test.update({name: 'william'}, {$uset: {age: ''}})`                                                                                         |
|   `$inc`    | 对数据进行计算                                                    | `db.test.update({name: 'william'}, {$inc: {age: -2}})`                                                                                          |
|   `$push`   | 数组数据追加(可内嵌文档)                                          | `db.test.update({name: 'william'}, {$push: {interest: 'code'}})`<br />`db.test.update({name: 'william'}, {$push: {'skill.skuFour': 'coding'}})` |
|    `$ne`    | 查有不执行，无则执行                                              | `db.test.update({name: 'william', interest: {$ne: 'playGame'}}, {$push: {interest: 'playGame'}})`                                               |
| `$addToSet` | 查找是否存在，不存在就直接`$push`                                 | `db.test.update({name: 'william'}, {$addToSet: {interest: 'read'}})`                                                                            |
|   `$each`   | 批量追加                                                          | `db.test.update({name: 'william'}, {$addToSet: {interest: {$each: ['singing', 'dance', 'code']}}})`                                             |
|   `$pop`    | 删除开头/结尾数组值<br /> 1： 从数组尾部删除; -1：从数组开头删除; | `db.test.update({name: 'william'}, {$pop: {interest: 1, 'skill.skillFour': -1}})`                                                               |
|  `upsert`   | 查有则改，无则添加                                                | `db.test.update({name: 'william'}, {$set: {age: 20}}, {upsert: true})`                                                                          |
|   `multi`   | 对所有数据进行复数操作                                            | `db.test.update({}, {$set: {interest: []}}, {multi: true})`                                                                                     |

其中，`upsert`和`multi`可以简写，在`update`中第三个参数为`upsert`，第四个参数为`multi`，即

```javascript
// 给所有男同胞加薪1000
db.test.update({sex: 'man'}, {$inc: {money: 1000}}, false, true);
等同于
db.test.update({sex: 'man'}, {$inc: {money: 1000}}, {upsert: false, multi: true});
```

#### `find`命令

`db.[collection].find(query, fields)`命令用来查找、筛选集合中的数据

`query:Object`指查询条件

`fields:Object`指显示的条件

**简单查找**

第一个参数传递query字段，即可完成筛选，例如：

```javascript
// 查找name为william的所有数据
db.test.find({name: 'william'})
```

**筛选字段**

有时候，find返回来的数据项太多，太乱，可以让返回的值按照我们想要的字段展示

```javascript
// 第二个参数传入一个对象，对要显示的字段赋值true，这里_id字段默认为true，若要忽略需要手动传false,而其他的字段默认是false
db.test.find({name: 'william'}, {name: true, age: true})
// 返回结果
{"_id": ObjectId("5a611350c4e36dee6008987a"), "name": "william", "age": 18}
```

**不等修饰符**

- 小于($lt):英文全称less-than
- 小于等于($lte)：英文全称less-than-equal
- 大于($gt):英文全称greater-than
- 大于等于($gte):英文全称greater-than-equal
- 不等于($ne):英文全称not-equal

如查找下公司内年龄小于30大于25岁的人员：

```javascript
db.test.find(
    {age: {$lt: 30, $gt: 25}},
    {name: true, age: true, _id: false}
)
```

**日期查找**

 MongoDB也提供了方便的日期查找方法，现在我们要查找注册日期大于2018年1月10日的数据，我们可以这样写代码。 

```javascript
var queryDate = new Date('01/01/2018');
db.test.find(
	{regeditTime: {$gt: queryDate}},
    {name: true, regeditTime: true, _id: false}
)
```

**多重条件筛选**

| 名称   | 作用                           | 示例                                                                                 |
| ------ | ------------------------------ | ------------------------------------------------------------------------------------ |
| `$in`  | 查询一键多值的数据             | `db.test.find({age: {$in: [18, 28]}})`找出年龄18和28的数据                           |
| `$nin` | `$in`的取反操作                | `db.test.find({age: {$nin: [18, 28]}})`找出年龄不为18或28的数据                      |
| `$not` | 找出不为某项值得数据           | `db.test.find({age: {$not: {$lte: 30, $gte: 20}}})`年龄大于30或小于20的数据          |
| `$or`  | 或者处理，整合查询条件做或逻辑 | `db.test.find({$or: [{age: {$lt: 20}}, {age: ${gt: 30}}]})`年龄大于30或小于20的数据  |
| `$nor` | `$or`的取反操作                | `db.test.find({$nor: [{age: {$lte: 20}}, {age: ${gte: 30}}]})`大于20并且小于30的数据 |
| `$and` | 与且处理，整合查询条件做与逻辑 | `db.test.find({$and: [{age: {$lt: 30}}, {age: {$gt: 20}}]})`大于20并且小于30的数据   |
| `$not` | 获取查询条件之外的值           | `db.test.find({age: {$not: {$gte: 30, $lte: 20}}})`大于20并且小于30的数据            |

**数组查询**

| 名称         | 作用                                 | 示例                                                                                                                                |
| ------------ | ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| 精确等值查询 | 会匹配所有相同项的数组               | `db.test.find({interest: ['画画','聚会']})`匹配爱好是画画和聚会的用户<br />`db.test.find({interest: ['画画']})`匹配只喜欢画画的用户 |
| 包含值查询   | 匹配数据数组中包含值的数据           | `db.test.find({intereste: '画画'})`匹配爱好中有画画的用户                                                                           |
| `$all`       | 匹配数据数组中包含数组值的数据       | `db.test.find({interest: {$all: ['看书', '看电影']}})`匹配爱好中有看书**和**看电影的用户                                            |
| `$in`        | 匹配数据数组中含有某些数组数据的数据 | `db.test.find({interest: {$in: ['看书', '看电影']}})`匹配爱好中有看书**或者**看电影的用户                                           |
| `$size`      | 根据查询数组的长度返回结果           | `db.test.find({interest: {$size: 5}})`找出拥有5个爱好的用户                                                                         |
| `$slice`     | 对显示选项的数组结果进行截取         | `db.test.find({}, {interest: {$slice: 2}})`显示前两项爱好<br />`db.test.find({}, {interest: {$slice: -1}})`显示最后一项爱好         |

**find后续方法**

| 名称    | 作用                                          | 实例                                            |
| ------- | --------------------------------------------- | ----------------------------------------------- |
| `limit` | 限制返回的数量                                | `db.test.find().limit(2)`只显示2条数据          |
| `skip`  | 跳过多少个数据显示                            | `db.test.find().skip(2)`跳过前两个数据显示      |
| `$sort` | 数据的排序方式，从小到大使用1，从到大小使用-1 | `db.test.find().sort({age: 1})`年龄从小到大排序 |

利用这些工具，可以很轻易的实现数据的分页效果

```javascript
// 假设获取请求参数中的分页和当前页码
const {limit, page, query, sort} = request.query;
// 链接数据库
const db = connect('company');
const res = db.test.find(query).limit(limit).skip(limit * （page - 1)).sort(sort);
// res的结果即为满足分页的筛选结果
```

**$where修饰符**

这是一个非常强大的修饰符，他允许我们在条件里使用`javascript`的语句来进行复杂查询，但是这样也影响数据库的压力和安全性，有一定风险，在工作中要尽量减少`$where`修饰符的使用

```javascript
// 查询大于30岁的人员
db.test.find(
	{$where: 'this.age >= 30'},    // 这里this指向的是test(查询集合)本身
    {name: 1, age: 1, _id: 0}
)
```



### 用js文件写MongoDb命令

`MongoDB`支持导入js脚本来对数据库进行操作，只需在js中调用`MongoDB`暴露的相应api就可以了

链接数据库方法：`connect('dbName')`，返回链接的数据库的实例，所有db下的shell命令都可以调用

打印数据到控制台：`print('something')`，能在控制台上打印信息，还有一个以json对象的格式打印`pringjson({})`

执行js文件也很简单，直接在控制台中

```shell
mongo xxx.js
```

以下模拟一个登陆日志表信息的功能

```javascript
// 在js脚本中
// 建议用var，这样在终端中多次load('./filePath.js')的时候，不会报因为const和let声明过的错误
var userName = 'william'; 	// 声明一个登陆名
var loginTime = Date.now();  // 声明登陆时的时间戳
var db = connect('log');	// 链接一个叫‘log’的数据库
db.login.insert({userName, loginTime});		// 数据库log下面的login集合插入一条数据，保存登陆名和登陆时间

var resultMessage = db.runCommand({getLastError: 1});
if(resultMessage.ok === 1){
	print('insert data success!'); 		// 没有错误即提示插入成功    
}else{
    print('insert data fail!')
}
```

**打印查询结果**

```javascript
var db = connect("company")  //进行链接对应的集合collections
var result = db.test.find() //声明变量result，并把查询结果赋值给result
//利用游标的hasNext()进行循环输出结果。
while(result.hasNext()){
    printjson(result.next())  //用json格式打印结果
}
```

在终端中，只要`load()`文件目录即可执行。

`MongoDB`也提供了一个`forEach`的循环方法来辅助输出结果，代码可以优雅地改成这样

```javascript
var db = connect('company');
var result = db.test.find();
result.forEach(function(res){
    printjson(res);
});
// 或者更优雅的传入回调
// result.forEach(printjson);
```



### MongoDB的应答式写入

`MongoDB`的写操作是没有任何返回值的，这个虽然减少了写操作的等待时间，但是客观上也造成了MongoDB不管有没有写入到磁盘或者有没有遇到错误，它都不会报错。但一般我们不放心这么做，这时候既可以调用`runCommand()`方法来获取操作的返回值。

```javascript
var db = connect('test');
db.testCollection.update({foo: 'bar'}, {$set: {baz: 'foobar'}})；
var resultMessage = db.runCommand({getLastError: 1});
printjson(resultMessage);
// 打印如下内容
{
        "connectionId" : 1,
        "updatedExisting" : true,
        "n" : 2,
        "syncMillis" : 0,
        "writtenTo" : null,
        "err" : null,
        "ok" : 1
}
```

**getLastError**： 表示返回功能错误，详细访问[MongoDB的getLastError]( http://www.mongoing.com/docs/reference/command/getLastError.html#dbcmd.getLastError )

这里再介绍下`db.listCommands()`方法，可以查看所有Command命令

`db.runCommand({ping: 1});`返回与数据库连接的状态，如果返回`{ok: 1}`则表示链接正常

**findAndModify**

从字义上看的出事查找并修改的意思。配置它可以在修改后给我们返回修改的结果

```javascript
var myModify={
    findAndModify:"test",
    query:{name:'William'},
    update:{$set:{age:18}},
    new:true    //更新完成，需要查看结果，如果为false不进行查看结果
}
var ResultMessage=db.runCommand(myModify);
 
printjson(ResultMessage)
```

`findAndModiy`的性能没有直接的使用`db.[coolection].update的好，但是在实际工作中都是使用它，毕竟要商用的程序，这点性能的损失跟安全性比起来还是值得的。

`findAndModify`的属性值：

* query： 需要查询的条件/文档
* sort： 进行排序
* remove：[boolean]是否删除查找到的文档
* new：[boolean]返回更新前的文档还是更新后的文档
* filelds：需要返回的字段
* upsert：[boolean]没有这个值时是否增加

