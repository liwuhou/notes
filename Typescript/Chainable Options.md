> In this challenge, you need to type an object or a class - whatever you like - to provide two function `option(key, value)` and  `get()`. In `option`, you can extend the current config type by the given key and value. We should about to access the final result via `get`.

```typescript
declare const config: Chainable

const result = config
  .option('foo', 123)
  .option('name', 'type-challenges')
  .option('bar', { value: 'Hello World' })
  .get()

// expect the type of result to be:
interface Result {
  foo: number
  name: string
  bar: {
    value: string
  }
}
```

implement

```typescript
type Chainable<T extends Record<string, unknown>> = {
  option<Key extends string, Value>(key: Exclude<K, keyof T>, v: Value): Chainable<T & Record<Key, Value>>
  get(): T
}
```

搞懂这种写法

```typescript
type T = <T>(T) => T
type T = {
  option<T>(t: T): T
}
```