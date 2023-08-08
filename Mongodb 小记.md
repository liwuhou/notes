以下是 MongoDB 中常用的一些命令：

1. 数据库操作：
   - `use <database>`：选择一个数据库或创建一个新数据库。
   - `show dbs`：显示所有数据库的列表。
   - `db`：显示当前所在的数据库。

2. 集合操作：
   - `db.createCollection("<collection>")`：创建一个新集合。
   - `show collections`：显示当前数据库中的所有集合。
   - `db.<collection>.drop()`：删除一个集合。

3. 文档操作：
   - `db.<collection>.insertOne(<document>)`：在集合中插入一个文档。
   - `db.<collection>.insertMany([<document1>, <document2>, ...])`：在集合中插入多个文档。
   - `db.<collection>.find(<query>)`：根据查询条件检索文档。
   - `db.<collection>.findOne(<query>)`：根据查询条件检索单个文档。
   - `db.<collection>.updateOne(<query>, <update>)`：更新符合条件的单个文档。
   - `db.<collection>.updateMany(<query>, <update>)`：更新符合条件的多个文档。
   - `db.<collection>.deleteOne(<query>)`：删除符合条件的单个文档。
   - `db.<collection>.deleteMany(<query>)`：删除符合条件的多个文档。

4. 查询条件操作符：
   - `$eq`：等于。
   - `$ne`：不等于。
   - `$gt`：大于。
   - `$gte`：大于等于。
   - `$lt`：小于。
   - `$lte`：小于等于。
   - `$in`：在给定数组中的值。
   - `$nin`：不在给定数组中的值。
   - `$and`：逻辑与。
   - `$or`：逻辑或。
   - `$not`：逻辑非。
   - `$regex`：正则表达式匹配。

5. 聚合操作：
   - `db.<collection>.aggregate([...])`：执行聚合查询操作。

6. 索引操作：
   - `db.<collection>.createIndex({ <field>: 1 })`：创建升序索引。
   - `db.<collection>.createIndex({ <field>: -1 })`：创建降序索引。
   - `db.<collection>.getIndexes()`：显示集合中的所有索引。
   - `db.<collection>.dropIndex("<indexName>")`：删除指定索引。

7. 其他操作：
   - `db.stats()`：显示当前数据库的状态和统计信息。
   - `db.dropDatabase()`：删除当前数据库。

请注意，上述命令中的 `<database>` 和 `<collection>` 分别代表数据库名称和集合名称，而 `<document>` 和 `<query>` 则分别代表要插入的文档和查询条件。这些命令可以在 MongoDB 的交互式 shell（如 `mongo` 命令）中执行。对于更复杂的操作，你还可以使用 MongoDB 的驱动程序或客户端库来与数据库进行交互。