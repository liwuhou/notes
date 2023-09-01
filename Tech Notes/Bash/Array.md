Array 是具有多个值的变量，是一种集合类型，Array 变量的初始化使用一个 `()` 包裹，元素之前使用空格分隔。

```bash
array=(apple banana "Fruit Basket" orange)
array[2]=watermelon
```

与其它编程语言相同，shell 中的 Array 下标也是从 0 开始的（zero-based)。与变量略有不同，在 bash 中，往字符串中插入数组个项是要使用 `${}` 包裹的，但在 zsh 里，可以直接使用 `$array[0]`，取值。
同时，bash 中使用整个数组也是要用 `${}` 包裹，如果直接使用 `$array`，只会打印数组的第一项。

```bash
echo ${array[1]} # banana
echo $array # apple
echo $array[1] # array[1] 根本不是数组里的值
```

zsh 里就友好多了

```zsh
echo ${array[1]} # banana
echo $array # apple banana "Fruit Basket" orange
echo $aaray[1] # banana
```

但为了兼容性，还是尽量使用 `${}` 来包裹变量。获取到数组的长度可以使用 `${#array[@]}` 来获取，所以数组最后一个元素可以这样直接获取 `${array[${#array[@]}]}`。

```bash
# 获取数组长度
echo ${#array[@]}

# 获取数组最后的元素
echo ${array[${#array[@]}]}
```

然后就是在[[Loops|遍历数组]]的时候，一定要用 `${array[@]}`，否则也是不能正确遍历的。
