---
title: LRU缓存淘汰算法
date: 2020-04-08
tags: 
  - Javascript
  - Algorithm
summary: LRU缓存淘汰算法(Lastest Recently Used)，即最近最少使用的缓存数据会被淘汰的算法。于前端而言，日常接触到的浏览器中的缓存策略、Vue中的keep-alive，都涉及到了该算法，名称听上去好像有些高大上，其实原理和实现还是很简单的，赶紧进来点亮下技能吧！
---

![](http://cdn.liwuhou.cn/blog/20200408075041.png)

### 简单说几句

各位看官先不要被这个什么 LRU 给劝退了，这个东西其实很简单。LRU 就是 Lastest Recently Used，即**最近最少使用**，说人话就是数据中，如果数据最近被访问过，那么将来被访问的几率也更高 ，优先淘汰最近没有被访问到的数据。

放张图：

<!-- more -->

![](http://cdn.liwuhou.cn/blog/20200408073340.png)

1. 最开始，内存都是空的，内存容量假定是 3，依次存入 A、B、C
2. 而当 D 存入的时候就发生问题了，因为内存空间不够，所以最久没被访问的 A 就被淘汰了
3. 当 B 被重新引用的时候，它的顺序就被重新调整了，也就是被放置到最前面来。
4. 当再次存入数据的时候，又面临内存空间不足的问题，由于 B 最近被使用了，所以 C 成了最久未被使用的数据而被淘汰了。

### 浏览器中的缓存机制

当你用浏览器访问网页，浏览器会先检测本地有没有保存相应的资源副本，如果有的话就不费事请求了，直接结束请求并返回缓存文件，这样节省流量请求的同时还能提升访问速度。

![](http://cdn.liwuhou.cn/blog/20200408083543.png)

既然是本地缓存，就必然缓存大小是有限的。当我们请求数量过多的时候，就会根据这个`LRU缓存淘汰策略`来确定哪些数据会被保留，哪些数据会被移除。
淘汰策略除了`LRU`(最近最少使用)，还有`FIFO`(先进先出)和`LFU`(最少使用)等。

### LRU 算法在 Vue 中 keep-alive 的实现

熟悉`Vue`的同学应该会知道`keep-alive`的作用——保持动态组件的状态，以避免在反复渲染中损失性能。[来自 vue 文档中关于 keep-alive 的简述](https://cn.vuejs.org/v2/api/#keep-alive)。
同样的，既然是本地缓存，就必然有缓存容量的限制。同浏览器的缓存机制相似，这里也用到了 LRU 淘汰缓存算法。如果一个组件你越久不去激活它，那么它被淘汰删除的可能性也就越高。

**keep-alive**

- props
  - `include` 字符串或正则表达式。只有名称匹配的组件会被缓存。
  - `exclude` 字符串或正则表达式。只有名称匹配的组件都不会被缓存。
  - `max` 数字。最多可以缓存多少组件实例。2.5.0 版本新增。
- 用法

```vue
<keep-alive>
  <component :is="view"></component>
</keep-alive>
```

`keep-alive`在 vue 中的实现

```ts
  const patternTypes: Array<Function> = [String, RegExp, Array];

  export default {
    name: 'keep-alive',
    // 标记为抽象组件，不会渲染成DOM元素，也不会出现在父组件链中
    abstract: true,

    props: {
      // 符合缓存的条件
      include: patternTypes,
      // 符合不缓存的条件
      exclude: patternTypes,
      // 限制缓存组件的大小
      max: [String, Number]
    },

    created () {
      // 初始化用于存储缓存的 cache 对象
      this.cache = Object.create(null);
      // 初始化用于存储VNode key值的 keys 数组
      this.keys = [];
    },

    destroyed () {
      // keep-alive组件销毁时，删除所有缓存的组件
      for (const key in this.cache) {
        pruneCacheEntry(this.cache, key, this.keys)
      }
    },

    mounted () {
      /**
       * 监听缓存，include中，满足条件的缓存；exclude中满足条件的不缓存
       * purneCache 传入实例和过滤的方法，用以过滤实例里的cache
       * matches		判断组件名是否与条件匹配
       */
      this.$watch('include', val => {
        pruneCache(this, name => matches(val, name))
      })
      this.$watch('exclude', val => {
        pruneCache(this, name => !matches(val, name))
      })
    },

    render () {
      // 获取第一个子元素的slot
      const slot = this.$slots.default
      const vnode: VNode = getFirstComponentChild(slot)
      const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions

      if (componentOptions) {
        // check pattern
        const name: ?string = getComponentName(componentOptions)
        const { include, exclude } = this
        if ( // 检查组件名如果不在include中，或者在exclude中的，就直接返回VNode，不缓存
          // 不在include中
          (include && (!name || !matches(include, name))) ||
          // 在exclude中
          (exclude && name && matches(exclude, name))
        ) {
          return vnode
        }

        // 满足缓存条件的
        const { cache, keys } = this
        // 定义键名
        const key: ?string = vnode.key == null
          // same constructor may get registered as different local components
          // so cid alone is not enough (#3269)
          ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '')
          : vnode.key

        // 这里就是LRU算法的实现了
        if (cache[key]) { // 组件已经在cache中缓存了的话
          vnode.componentInstance = cache[key].componentInstance
          // make current key freshest
          // 将当前组件的key值放置到最新的位置（删掉原本的，再push到队尾）
          remove(keys, key)
          keys.push(key)
        } else { // 组件如果还没缓存进cache里的话
          // 将组件放进cache缓存，将key追加进keys队尾
          cache[key] = vnode
          keys.push(key)
          // prune oldest entry
          // 判断当前缓存大小是否超过限制
          if (this.max && keys.length > parseInt(this.max)) {
            // 超出限制就将最久未被使用的组件删除（keys队首）
            pruneCacheEntry(cache, keys[0], keys, this._vnode)
          }
        }

        vnode.data.keepAlive = true
      }
      return vnode || (slot && slot[0])
    }
  }

  // 补充下pruneCacheEntry方法的实现
  function pruneCacheEntry (
    cache: VNodeCache,
    key: string,
    keys: Array<string>,
    current?: VNode
  ) {
    const cached = cache[key]
    if (cached && (!current || cached.tag !== current.tag)) {
      // 卸载组件
      cached.componentInstance.$destroy()
    }
    // 清除组件在cache中的缓存
    cache[key] = null
    // 删除key
    remove(keys, key)
  }

  // remove 方法（位于shared/util.js中）
  /**
   * Remove an item from an array.
   */
  export function remove (arr: Array<any>, item: any): Array<any> | void {
    if (arr.length) {
      const index = arr.indexOf(item)
      if (index > -1) {
        return arr.splice(index, 1)
      }
    }
  }
```

[vue 中 keep-alive 的源码](https://github.com/vuejs/vue/blob/dev/src/core/components/keep-alive.js)

根据上面的源码可以看出来，`keep-alive`组件包裹的动态组件中，会缓存`includes && !excldues`条件的组件，而不是销毁他们。
当`keep-alive`中的 cache 缓存的组件数量超过`max`时，就会采取 LRU 淘汰算法，将最久未被使用到的组件从缓存中删去。

### 下面让我们尝试下实现一个 LRU 算法吧

> 设计和构建一个“最近最少使用”缓存，该缓存会删除最近最少使用的项目。缓存应该从键映射到值(允许你插入和检索特定键对应的值)，并在初始化时指定最大容量。当缓存被填满时，它应该删除最近最少使用的项目。
>
> 它应该支持以下操作： 获取数据 get 和 写入数据 put 。
>
> 获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
> 写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。
>
> 来源：[力扣（LeetCode）](https://leetcode-cn.com/problems/lru-cache-lcci)

[我的 leetcode 题解](https://leetcode-cn.com/problems/lru-cache-lcci/solution/javascriptshi-xian-lrutao-tai-huan-cun-suan-fa-by-/)

关注本人公众号

![](https://blogs-1257826393.cos.ap-shenzhen-fsi.myqcloud.com/qrcode_for_gh_373ae200ef34_344.jpg)
