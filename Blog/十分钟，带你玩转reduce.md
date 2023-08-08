---
title: 十分钟，带你玩转reduce
date: 2019-11-20
tags: 
    - Javascript
    - TenMinutes
summary: reduce是个功能很强大，也很有意思的数组方法。以前没深入学习的时候，只是让他来做做累加，处理一些简单数据的合计功能。当我深入挖掘的时候才发现他可以做的东西还有很多，比如拼接字符串、拼接数组、过滤和映射数组、整合对象等骚操作。
---

### 简单说几句

`Array.prototype.reduce`是个功能很强大，也很有意思的数组方法。以前没深入学习的时候，只是让他来做做累加，处理一些简单数据的合计功能。当我深入挖掘的时候才发现他可以做的东西还有很多，比如拼接字符串、拼接数组、过滤和映射数组、整合对象等骚操作。

这里利用十分钟的时间，详尽地阐述`reduce`的作用，还有一些用例，让大伙可以在优雅实现功能的同时，还能节省开发时间，早点下班[手动滑稽]。

<!-- more -->

### reduce参数的简单介绍

其实`reduce`方法就接收两个参数，一个是处理迭代数据的方法函数【1】，一个是初始值【2】。然后这个传进去的函数呢就大有文章了。

【1】必填，这个函数接收四个参数，分别是当前累计的值(必须)、当前遍历到的值(必须)、当前遍历到的索引(可选)和源数组(可选)。

【2】可选，传入的是一个**任意类型的值**，表示初次累计时的值。注意这里是任意类型的值，想把`reduce`玩出花来就必须要有这个认识。

```javascript
// 打印下函数中的各参数让大家认识认识
const arr = [1, 1, 1];
const initValue = 0;     // 1)
const result = arr.reduce((total, cur, idx, arr) => {
    console.log(total, cur, idx, arr);  // 2)
    return total + cur; // 2)
}, initValue)
console.log('result => ', result)
// 第一次打印：
// 0, 1, 0, [1, 1, 1]

// 第二次打印：
// 1, 1, 1, [1, 1, 1]

// 第三次，也是最后一次打印：
// 2, 1, 2, [1, 1, 1]

// 结果打印：
// result => 3
```

这里还有几个细节要讲一下：

第一点，【2】初始值可以传入任意类型的数据，不仅是代码1)处的`0`。

第二点，后续的累计值total就是上一次`reduce`函数返回的值，就是代码2)处。

第三点，当初始值没有传递的时候，函数会默认拿源数组的第一项当作初始值，*之所以要钻这个牛角尖是因为我以前看某些文章说`reduce`函数的初始值没传的话默认是`undefined`，但是事实不是这样的*。

```javascript
[1,2,3].reduce((total, cur)  => {
    console.log(total);
    return total + cur;
}) // 1) 这里不传“初始值”
// 依次打印：
// 1

// 3

// 6
```

上述代码中，共打印三次，第一次打印的不是`undefined`，而是数组的第一项 1。如果传入初始值0的话，会打印四次，第一次是初始值——`0`。可见，如果不传值给“初始值”，那么函数默认会获取数组迭代的第一项为“初始值”。

知道上述知识之后就可以看看`reduce`能做些什么了

### reduce的一些应用

**累计器**，最普适也最常用的应该就是这个了，想我以前也只是单纯地以为`reduce`的作用仅在于此。

```javascript
function reduceAdd(arr){
    return arr.reduce((total, cur) => total + cur);
}
```



**数组扁平化**

```javascript
function flatReduce(arr){
    return arr.reduce((pre, cur) => {
        return pre.concat(cur);
    })
}
console.log(flatReduce([[0, 1], [2, 3], [4, 5]]))// [0, 1, 2, 3, 4, 5]
// 跟flat方法类似
```



**实现`filter`+`map`**

```javascript
// 同样都是实现一个获取对象数组里面id的数组的数组的方法
const data = [
    {id: 1000, name: 'William'},
    {id: 1001, name: 'Abby'},
    {id: '', name: 'skye'}
]
// 常规
function badMap(arr){
    const ids = arr.map(item => item.id); // 先遍历一次获取id数组
    const validIds = ids.filter(id => id !== '') // 获取非空id
}
// 上面的方法遍历了两次数组，时间复杂度是2On

// 其实用reduce完全可以做到filte和map的效果
function goodMap(arr){
    return arr.reduce((total, {id}) => {
        return id !== '' ? [...total, id] : total;
    }, []) // 这里初始值传入的是空数组
}
//  这里arr只迭代了一次，时间复杂度是On
```



**统计字符串中每个字符出现的次数**

```javascript
function countChar(str = ''){
    return str.split('').reduce((p, k) => (p[k]++ || (p[k] = 1), p), {});
}
console.log(countChar('aaabbc'));
// {a: 3, b: 2, c: 1}
```





**拼接字符串**，开发的时候，可能会碰到一些接口是`get`请求，然后要手动拼接get的请求参数的，当然如果你用一些工具函数也是可以，这里用`reduce`方法实现一下

```javascript
function getParamsForGET(params = {}, url = ''){
    // 这里接收的params是对象类型的数据，所以先用es6里的entries获取一下其中可迭代属性先
    const paramsWrapArr = Object.entries(params); // 类似[['key', 'value'], ...]的数据
    // 这里利用解构，更优雅的处理数据
    const query = paramsWrapArr.reduce((str, [key, value], idx) => {
        // 这里将cur参数结构了，等同于 const [key, value] = cur;
        // get请求参数第一个参数是用?符号隔开的
        const next = idx === 0 ? `?${key}=${value}` : `&${key}=${value}`;
        // 拼接参数
        return str + next; 
    }, '');  // 传入初始值空字符串，用于拼接 
    return url + query;
}
// 示例
const params = getParamsForGET({wd: '阿五', foo: 'bar'}, 'baidu.com/s');
// 返回： baidu.com/s?wd=阿五&foo=bar
```



**深层对象属性漫步**，访问一个对象的深层属性，不用自己去点点点

```javascript
function walkInObj(data, keys = []){
    // 这里的keys可以接受符号隔开的字符串或者数组
    const path = Array.isArray(keys) ? keys : keys.split(/,|\.|，/);
    if(!path.length) return data;
    // 主要代码
    return path.reduce((pre, cur) => {
        if(!(cur in pre)){
            console.log(`${cur}属性不存在于对象中。`)
            return pre;
        }else{
            return pre[cur];
        }
    }, data); // 这里传入对象data为初始值，可以让reduce去深层访问属性
}
// 示例
const data = {
    home: {
        furniture: {
            chest: {
                name: 'clothes'
            }
        },
        electric: {
            TV: {
                channel: 'TVB星河'
            }
        }
    }
}
console.log(
	walkInObj(data, 'home.furniture,chest，name'),
    walkInObj(data, ['home', 'electric', 'TV', 'channel'])
)
// 打印：
// clothes TVB星河
```

看着好像没什么用，但是只要稍一改动就可以用来检验一些属性的有效性，跟后端对接的时候，经常碰到一些随心所欲的后端大爷，接口返回值不规范。比如你需要一个list字段的数据，有数据的时候是一个数组类型，没数据返回给空数组或者null就行，但是我家后端大爷经常没数据的时候返回一个空字符串(`''`)，或者干脆这个属性就不给了，更过分的，返回一个空对象都有！（说是某p*p的框架的缺陷，咱也不知道咱也不敢问）

所以秉着减少重复代码早下班，远离996的想法，可以写出一些**校验是否是有效数据的方法**，比如下面就是我常用的用来校验后端传过来的对象或者数值是有是有效值的方法，现在看起来有点臃肿了，但是效果还是不错的。

![项目中用到的一个巨稳妥的判断函数](https://blogs-1257826393.cos.ap-shenzhen-fsi.myqcloud.com/code-1573911710584.png)

### 最后BB几句

以上，只是`reduce`方法的主要用法了，在工作中，可以灵活地利用这个方法来极大程度的简化代码，有些人可能会想：“OK，又懂了一个我可能不会用上的东西”。其实不然，只要你知道这个方法的一些细节，再跟着敲一下一些例子，我相信你已经能称得上摸透这个方法了。当你掌握了某种工具之后，你会在不知不觉中，潜移默化地使用他，等你发觉的时候，你已经运用的很熟练了，然后再总结出一篇跟我上述文字差不多的文章出来了...

最后留个家庭作业吧，利用`reduce`实现一个**数组去重**的功能，答案就在下面，最好自己先思考下然后再往下翻着看。OK，先这样吧。





















```javascript
// 家庭作业
function unique(arr = []){
    return arr.reduce((total, cur) => (
        total.incldues(cur) ? total : [...total, cur];
    ), [])
}

// 兼容es3
function uniqueProfill(arr){
    var arr = arr || [];
    return arr.reduce(function(total, cur){
        return !~total.indexOf(cur) ? [...total, cur] : total;
    }, [])
}

// 当然我项目中都是这样简单粗暴的...
function uniqueForce(arr){
    return [...new Set(arr)]
}
```



> 每次花个十分钟，懂一个前端知识点，走得慢，但坚持走下去，足以致千里。

关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)
