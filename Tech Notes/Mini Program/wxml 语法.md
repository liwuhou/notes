像 vue 又不像 vue，像 HTML 又不像 HTML。这大概就是很多人对 wxml 的印象。确实是这样，说是 HTML，看到的都是 `<view>`、`<text>` 的标签， 其中又嵌套了很多 `wx:if`、`wx:for` 属性的语法，就是 wxml 的个性。而 `<view>` 这些标签就是小程序框架设计的一套标签语言。WXML 语法的后缀名是 `.wxml`。虽说 WXML 的语法跟 HTML 的语法十分接近，但是有些许不同的是，WXML 要求标签是严格闭合的，如果没有闭合的话将会导致报错。同时属性是大小写敏感的。

### 行内属性
WXML 中提供的所有标签都具有以下相同的行内属性：

- `id` 标签的id选择器名
- `style` 标签的行内样式
- `class` 标签的类名
- `data-*` 标签的自定义属性
- `hidden` 是否显示标签，默认为 false
- `bind*/catch*` 绑定相关事件或劫持相关事件

### 数据绑定

在传统的web开发中，开发者需要使用 Javascript 使用相关的 DOM api 去界面进行实时的更新。而在小程序中，WXML 语言提供了一种数据的绑定功能，WXML 能通过 `{{变量名}}` 来将其与对应的 js 文件中的 `data` 对象属性进行一个绑定。在js 修改了对应的 data 上的值的时候，会将最新的值渲染到页面中。

### 逻辑语法



