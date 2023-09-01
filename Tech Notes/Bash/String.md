shell 脚本对字符串十分友好，友好到一些简单的字符串都不需要 `"` 包裹，直接就能在脚本上使用。

```bash
echo hello world # output: hello world
```

与 [[Array]] 类似，获取字符串长度依然可以使用 `${#STRING}` 来获取。

```bash
str=hello
echo ${#str} # 5
```

字符串裁剪，有好用的 `${str:POS:LEN}`，可以从字符串下标 `POS` 开始截取 `LEN` 长度的字符串。
省去 `POS` 参数，会从 `0` 开始截取，而省略 `LEN` 则从 `POS` 一直截到原字符串结尾。但要注意这两个参数不能同时省略。

```bash
str="hello world"
echo ${str::5} # hello
# 等价于
echo ${str:0:5} # hello

echo ${str:1} # ello world
echo ${str::} # 而这并不是截取整个字符串，而是报错，猜不到吧，哈哈
```

字符串也支持替换，使用的语法也比较直观。使用 `${str/OLD/NEW}`，全局替换则是 `${str//OLD/NEW}`。如果字符串中，使用了空格分隔，还会很”贴心“ 地帮你把字符串后的一个空格也删掉喔。

```bash
# 定义一个字符串
STRING="to be or not to be"

# 替换遇到的第一个字符串
echo ${STRING[@]/be/eat} # to eat or not to be

# 全局替换整个字符串
echo ${STRING//be/eat} # to eat or not to eat

# 替换以 [OLD] 开关的字符串
echo ${STRING//#to be/eat now} # eat now or not to be

# 替换以 [OLD] 结尾的字符串
echo ${STRING/%be/eat} # to be or not to eat

# 配合 `$()` 动态查找或替换字符串
echo ${STRING/%be/be on ${date +%Y-%m-%d}} # to be or not to be on YYYY-mm-dd

# 删除被 [OLD] 命中的第一个字符串
echo ${STRING/be/} # to or not to be

# 当然也能全局里删除被命中的字符串
echo ${STRING//be/} # to or not to
```

以上都不会更改到原字符串，而是会生成一个新的字符串片段。

```bash
NEW_STRING=${STRING//be/eat}
echo STRING # to be or not to be
echo NEW_STRING # to eat or not to eat
```
