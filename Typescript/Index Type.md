### 映射类型

对象、class 在 Typescript 里对应的类型都是索引类型（Index Type），映射类型可以对索引类型作修改

```typescript
type MapType<T> = {
	[key in keyof T]: T[key]
}
```

`keyof T` 是查询索引类型中所有索引的操作，叫作 `索引查询`
`T[key]` 是取索引类型某个索引的值，叫作 `索引访问`

比如把一个索引类型的值改一个索引类型的数组

```typescript
type MapType<T> = {
	[key in keyof T]: [T[key], T[key], T[key]]
}

type res = MapType<{a: 1, b: 2}>

const result: res = {
	a: [1, 1, 1],
	b: [2, 2, 2]
}
```

映射类型就相当于将一个集合映射为另一个集合，这也是它名字的由来
![](http://cdn.liwuhou.cn/tmp/20220222104525.png)

### 重映射

除此之外，还可以重映射索引类型的索引
 
将原本的 key 加上了一个 `theKey-` 的前缀
```typescript
type Map<T> = {
	[Key in keyof T as `theKey-${Key & string}`]: T[Key]
}
```

这里的 `& string` 是因为集合的属性可以是 `string` 、`number` 和 `symbol` 类型，所以 `key` 是这三种的联合类型，这里跟 `string` 取交集剩下的就是 `string` 类型了。交叉类型会把同一类型做合并，不同的类型则舍弃掉


