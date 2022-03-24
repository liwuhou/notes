实现一个 `ParseQueryString` 

```ts
function parseQueryString<Str extends string>(str: Str): ParseQueryString<Str> {
  if (!queryStr || !queryStr.length) { 
    return {} as any; 
  }
   
  const queryObj = {} as any; 
  const items = queryStr.split('&'); 
  items.forEach(item => {
    const [key, value] = item.split('='); 
    if (queryObj[key]) {
      if(Array.isArray(queryObj[key])) {
        queryObj[key].push(value); 
      } else {
        queryObj[key] = [queryObj[key], value] 
      } 
    } else { 
      queryObj[key] = value; 
    } 
  }); 
  
  return queryObj as any;  
}
```

实现这个 ParseQueryString 高级类型的思路是，首先通过模板字符串`&` 套取到 QueryString 对，再通过 `=` 套取对象的键值对，通过递归合并值、合并各个对象类型。

```ts
type ParseQueryString<Str extends string> =
  Str extends `${infer One}&${infer Rest}`
    ? MergeParams< // 把两个对象合并起来
      ParseParam<One>, // 将 `a=b` 的字符串字面量类型 转为 {a: b} 索引类型
      ParseQueryString<Rest> // 递归调用
    > 
    : ParseParams<Str>
```

这里的 `ParseParam` 比较容易实现，将 `a=b` 这类类型转为 `{a: b}` 的索引类型

```ts
type PraseParam<Str extends string> =
  Str extends `${infer Key}=${infer Value}`
    ? {
      [K in Key]: Value
    } : Record<string, any>
```

接着实现合并两个对象字面量类型的合并

```ts
type MergeParams<
  One extends Record<string, any>,
  Other extends Record<string, any>,
> = {
  [Key in keyof One | keyof Other]:
    Key extends keyof One
      ? Key extends keyof Other
        ? MergeValue<One[Key], Other[Key]>
        : One[Key]
      : Key extends keyof Other
        ? Other[Key]
        : never
}
```

然后相同 key 值的合并类型方法的实现：

```ts
type MergeValue<First, Second> =
  First extends Second
    ? Frist
    : Second extends unknown[]
      ? [First, ...Second]
      : [First, Second]

```

至此为此，`ParseQueryString` 就都实现了