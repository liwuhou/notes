---
title: ES6语法的新特性
date: 2018-05-08
tags: Javascript
summary: ES6的一些语法笔记
---

#### let和const
**let**和**const**将变量的的作用域限制在块中，可以避免因在执行JavaScript代码前，变量被提升所带来的问题。
> let在同一作用域中只能声明一次，但可以重复赋值
> const在同一作用域中只能声明一次，并且初始化的时候必须赋值，随后无法重新赋值

- 当你打算为变量重新赋值的时候，则使用`let`
- 当你不打算为变量重新复制的时候，则使用`const`

<!-- more -->

<br>

#### 模板字面量
**模板字面量**本质是包含嵌入式表达式的字符串字面量
模板字面量用倒引号(`)表示的，可以包含用${expression}表示占位符，这样更容易构建字符串。

```JavaScript
const Day = 'sunday';

const student = 'xiaoming';

//使用字符串连接运算符
let message = xiaoming + ' like ' + Day +'!';

//使用模板字面量
let message = `${student} like ${Day}!`;

```

<br>

#### 解构
ES6中，可以使用**解构**从数组和对象中提取值并赋给独特的变量

```JavaScript
const point = [10,25,-34];
const studen = {
    name:'xiaoming',
    age:18
}

//ES5
var x = point[0];
var y = point[1];
var z = point[2];

name = studen.name;
age = studen.age;

//ES6

let [x,y,z] = point;
let [name,age] = studen;

```

灵活选取

```JavaScript
//数组,因为数组有序，这里用逗号隔开，
const arr = [0,1,2,3,4,5,6,7];
let [zero,,two,,,,,seven] = arr;//0,2,7

const goods = {
    type:'food',
    color:'red',
    count:'ten',
    weight:'19kg'
};
let {color,weight} = goods;//red,'19kg'
```
将对象的方法赋予变量之后的this指向问题

```JavaScript
const circle = {
  radius: 10,
  color: 'orange',
  getArea: function() {
    return Math.PI * this.radius * this.radius;
  },
  getCircumference: function() {
    return 2 * Math.PI * this.radius;
  }
};

let {radius, getArea, getCircumference} = circle;
getArea();  //NaN;

```

<br>
#### 对象字面量简写法
使用和所分配的变量名称相同的名称初始化对象，这样可以去掉很多重复的代码，如果没有简写的话，就会如下示例：

```JavaScript
let type = 'quartz';
let color = 'rose';
let carat = 21.29;

const gemstone = {
  type: type,
  color: color,
  carat: carat
};

```
可以看到重复的地方为：`type:type`、`color:color`、`carat:carat`，实在是很费劲呐。
现在ES6中，如果属性名称和所分配的变量一样，那么就可以从对象属性中删掉这些重复的变量名称。

```JavaScript
let type = 'quartz';
let color= 'rose';
let carat = 21.29;

let gemstone = {type,color,carat};
```

为对象添加方法也有一种简写方法

```JavaScript
let type = 'quartz';
let color= 'rose';
let carat = 21.29;

//ES5
let gemstone = {
    type=type,
    color=color,
    carat=carat,
    calculateWorth:function(){
        something;
    }
};

//ES6
let gemstone = {
    type,
    color,
    carat,
    calculateWorth(){   //不需要function关键字
        something;
    }
};
```

<br>
#### For…of 循环
for…of循环用于循环访问任何可迭代的数据类型。其跟for…in循环基本一样，只是将`in`替换为`of`，可以忽略索引。

```JavaScript
const digits = [0,1,2,3,4,5,6,7,8,9,0];
for (const digit of digits){
    console.log(digit);
}
```
你也可以随时退出或停止for…of循环

```JavaScript
const digits = [0,1,2,3,4,5,6,7,8,9,0];
for (const digit of digits){
    if(digit%==2){
        continue;
    }
    console.log(digit);//odd;
}
 
```

<br>
#### 展开运算符
**展开运算**用三个`...`表示，是ES6的新概念，是你能够姜*字面量对象*展开为多个元素

```JavaScript
let arr = [0,1,2,4,5,6,7,8,9];
console.log(...arr);
```
> **Print:** 0 1 2 3 4 5 6 7 8 9 

以下是具体应用

```JavaScript
const fruits = ['apples','bananas','pears'];
const vegetables = ['corn','potatoes','carrots'];

const produce = [...fruits,...vegetables];
console.log(produce);
```
> **Print:**['apples','bananas','pears','corn','potatoes','carrots']

<br>
#### 剩余参数
**剩余参数**也用`...`三个点来表示，使你能够将不定数量的元素表示为数组，在多种情形下都比较有用。
一种情形是将变量赋值数组值时。例如，

```JavaScript
const order = [20.17, 18.67, 1.50, "cheese", "eggs", "milk", "bread"];
const [total, subtotal, tax, ...items] = order;
console.log(total, subtotal, tax, items);

```

> **Print:**20.17 18.67 1.5 ["cheese","eggs","milk","bread"]

该代码将order数组的值分配给单个变量。数组中的前三个值被分配给了total、subtotal和tax，然后通过使用剩余参数，数组中剩余的值被作为数组，分配给了items。

可变参数函数，在不适用函数中的参数对象的时候，可以使用剩余参数，使得函数更简练，更易读懂。

```JavaScript
function sum(...nums){
    let total = 0;
    for(const num of nums){
        total+=num;
    }
    return total;
}
```

关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)
