shell 脚本的变量会在赋值的时候被创建，变量可以保存数字、字符及字符串。变量名大小写敏感，可以包含字符 `_`，但不能有 `$`，因为这个在 shell 脚本中有额外的语法含义。

赋值的时候，`=`的两边一定不能有空格，否则赋值语句不会生效。

```bash
VARIABLE=10
STR=HELLO_WORLD
S="Hello         World"
```

当字符串中需要使用特殊字符时，可以使用 `\` 转义。

```bash
echo "A apple for \$5."
```

使用变量有几种方式，一种是直接使用变量名

```bash
V=5
some_method v # 传参
v=6 # 重新赋值
```

在字符串中则 `$` + 变量名。

```bash
Greet=Hello

echo $Greet # Hello
echo "$Greet world" # Hello world
```

在 `""` 中也可以使用 `${}` 包裹，避免歧义

```bash
echo "${Greet} world" # Hello world
```

同时可以使用 `` ` `` 或者 `$()` 包裹来赋值/插入一些命令的执行结果。

```bash
Current_File_List=`ls .` # 当前目录文件列表

echo "Show the list of current directory: $(ls)"
```
