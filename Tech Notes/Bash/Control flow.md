跟主流的编程语言一样，Shell 语言也有流程控制语句，就是看起来长点有得”怪怪的“。

```bash
if [expression]; then
  code # if expression is true
fi
```

也能接 `elif` 和 `else`

```bash
NAME="William"

if [ "$NAME" = "William" ]; then
  echo "Handsome boy"
elif [ "$NAME" = "Abby" ]; then
  echo "Beautiful girl"
else
  echo "Somebody"
fi
```

这里要注意的点是，字符、字符串变量，要使用 `""` 包裹起来。

除此之外，如果是条件表达式的话，就不能用 `[]` 了，而是要用 `[[]]` 包裹起来。

```bash
if [[ $VAR_A[0] -eq 1 && ($VAR_B = "bee" || $VAR_T = "tee") ]]; then
  # code
fi
```

同时 shell 也有关系比较级操作符：

数字类型比较符：

```plain
comparison    Evaluated to true when
$a -lt $b    $a < $b
$a -gt $b    $a > $b
$a -le $b    $a <= $b
$a -ge $b    $a >= $b
$a -eq $b    $a is equal to $b
$a -ne $b    $a is not equal to $b
```

字符类型的比较符：

```plain
comparison    Evaluated to true when
"$a" = "$b"     $a is the same as $b
"$a" == "$b"    $a is the same as $b
"$a" != "$b"    $a is different from $b
-z "$a"         $a is empty
```

也有类型 `switch` 的控制语句

```bash
case "$variable" in
  "$condition1" )
    # code
  ;;
  "$condition2" )
    #
  ;;
esac
```

也可以省掉缩进，写在一行。

```bash
variable=1
case $variable in
  1) echo 1;;
  2) echo 2;;
  3) echo 3;;
  4) echo 4;;
  5) exit
esac


```
