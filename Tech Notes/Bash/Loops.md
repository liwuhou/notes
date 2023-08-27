shell 中也有循环的语句，可以用来遍历一些列表、执行条件循环。

loop 迭代

```bash
for arg in [list]; do
  commands...
done
```

用来遍历一些列表数据，或是配合命令行执行遍历操作

```bash
NAMES=(Joe Jenny Sara Tony)
for NAME in ${NAMES[@]} ; do
  echo "The name is $NAME"
done

for f in $( ls . ) ; do
  echo "File is :$f"
done
```

也有 `while` 循环

```bash
COUNT=4
while [ $COUNT >= 0 ]; do
  echo "Value of count is $COUNT"
  COUNT=$(($COUNT - 1))
done
```

也有 `do...while` 循环

```bash
COUNT=1
until [ $COUNT -gt 5 ] ; do
  echo "Value of count is: $COUNT"
  COUNT=$(($COUNT + 1))
done
```

跟其它编程语言一样，循环语句中也有 `break` 和 `continue` 语句

`break` 是结束整个循环，`continue` 则是结束当前的循环。

```bash
COUNT=0
while [ $COUNT >= 0 ]; do
  $((COUNT + 1))
  if [ $(($COUNT % 2)) = 0 ]; then
    continue
  fi
  if [ $COUNT == 5 ]; then
    break
  fi
  echo $COUNT
done

# output: 1, 3
```
