### Tuple
`Tuple` 即为元组，可以理解为长度固定的，并且值的类型顺序也固定的特殊数组

例如二维坐标的数组类型恒为：

```typescript
type Position = [number, number]
```

这样坐标点的值就能被类型正确约束了

```javascript
const pos: Position = [0, 0] // correct

const wrongPos: Position = [0] // incorrect
const wrongPos1: Position = ['0', '0'] // incorrect
```

### Array
`Array` 是 js 中的一种数据结构，长度可变，类型各异，可以经过元组类型约束之后变为元组。

```javascript
const arr = [0, 1, 'string']
```

### difference
普通的数组较为灵活，`typeof` 获取二者的类型也有所不同

```typescript
const arr = [0, 1, 'string']

type Arr = typeof arr // (number | string)[]

const tul: [number, number, string] = arr

type Tul = typeof tul // [number, number, string]
```

普通数组加上 `as const` 可以强制转为元组，只是数组中的值的类型会被约束为其字面量类型

```typescript
const arr = [0, '1'] as const

type Arr = typeof arr // readonly [0, '1']
```

二者的 `indexed` 的 `length` 也有所不同

```typescript
const a = [0, '1']
const b: [number, string] = a
const c = a as const

type LA = typeof a['length'] // number
type LB = typeof b['length'] // 2
type LC = typeof c['length'] // 2
```