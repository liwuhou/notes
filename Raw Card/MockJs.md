Mockjs 的语法规范包含两部分：
 1. 数据模板定义规范（Data Template Definition, DTD）
 2. 数据占位符定义规范（Data Placeholder Definition, DPD）

### DTD
数据模板中的每个属性由 3 个部分构成：属性名（name）、生成规则（rule）、属性值（value）。格式如下：

```js
{
  'name|rule': value
}
```

注意：
- 属性名和生成规则之间要用竖线 `|` 分隔
- 生成规则是可选的
- **生成规则的含义需要依赖属性值的类型才能确定**
- 属性值中可以含有 `@点位符`
- 属性值还指定了最终值的初始值和类型
- 生成规则有 7 种格式：
  1.  `'name|min-max': value`
  2.  `'name|count': value`
  3.  `'name|min-max.dmin-dmax': value`
  4.  `'name|min-max.dcount': value`
  5.  `'name|count.dmin-dmax': value`
  6.  `'name|count.dcount': value`
  7.  `'name|+step': value`

详细规则见[Syntax Specification · nuysoft/Mock Wiki (github.com)](https://github.com/nuysoft/Mock/wiki/Syntax-Specification#%E6%95%B0%E6%8D%AE%E6%A8%A1%E6%9D%BF%E5%AE%9A%E4%B9%89%E8%A7%84%E8%8C%83-dtd)

### DPD
Mockjs 内置了可以生成常见数据的字符串，诸如姓名、时间、地址、段落、指定大小的图片链接、符合某个正则表达式的字符串等，占位符只是在属性值字符串中占个位置，并不出现在最终的属性值
占位符的格式为：

```plain
@点位符
@占位符(参数[, 参数])
```

注意：
1. 用 `@` 来标识其后的字符串是占位符
2. 占位符引用的是 `Mock.Random`中的方法
3. 通过 `Mock.Random.extend()` 来扩展自定义占位符
4. 占位符也可以引用数据模板中的属性
5. 占位符会优先引用数据模板中的属性
6. 占位符支持相对路径和绝对路径

```js
Mock.mock({
    name: {
        first: '@FIRST',
        middle: '@FIRST',
        last: '@LAST',
        full: '@first @middle @last'
    }
})
// =>
{
    "name": {
        "first": "Charles",
        "middle": "Brenda",
        "last": "Lopez",
        "full": "Charles Brenda Lopez"
    }
}
```

