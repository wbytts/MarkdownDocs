# Js 与 jQuery 的基本写法

JavaScript

```js
window.onload = function() {
    // ......
}
```

jQuery的写法：

```javascript
$(function() {
    // ......
})
```

# 原生 js 与 jQuery 入口函数的加载模式区别

区别一：

- 原生js会等待DOM元素加载完毕，并且图片也加载完毕才会执行
- jQuery会等待DOM元素加载完毕，但不会等待图片也加载完毕，就会执行

区别二：

- 原生的js如果编写了多个入口函数，后面编写的会覆盖前面编写的
- jQuery的入口函数，后面的不会覆盖前面的

# jQuery 入口函数的其他写法

```javascript
$(function() {
    
})
```

```javascript
jQuery(function() {
    
})
```

```javascript
$(document).ready(function() {
    
})
```

```javascript
jQuery(document).ready(function() {
    
})
```

一般就采用第一种语法

# jQuery 的 $ 符号冲突问题

- 首先释放 `$` 的使用权：`jQuery.noConflict()`
  - 注意：释放必须在编写其他jQuery代码之前编写，释放之后就不能使用 `$`
- 释放的同时执行新的指代符号：`var by = jQuery.noConflict()`



# jQuery 核心函数

`$()` 就是 jQuery 的核心函数

- **接收一个函数**
  - 表示入口函数
- **接收一个字符串**
  - 接收一个字符串选择器
    - 获取选择器对应的那些元素
  - 接受一个代码片段
    - 返回一个jQuery对象
- **接收一个DOM元素**
  - 将DOM元素包装成一个jQuery的对象

# jQuery 核心对象

`$` 就是 jQuery 的核心对象

jQuery对象是一个伪数组

*什么是伪数组：有0到length-1的属性，并且有length属性*

# jQuery 的一些静态方法

## each 方法

原生js的forEach：这个只能遍历数组，不能遍历伪数组

```javascript
arr.forEach(function(value, index) {
	/*
		第一个参数value：遍历到的元素
		第二个参数index：当前遍历到的索引
	*/
});
```

jQuery的each：也可以遍历伪数组

```javascript
$.each(arr, function(index, value) {
    // 第一个参数：当前遍历到的索引
    // 第二个参数：遍历到的元素
});
```



## map 方法

原生js的map方法：不能遍历伪数组

```javascript
arr.map(function(value, index, array) {
    /*
    	第一个参数：当前遍历的值
    	第二个参数：当前遍历的索引
    	第三个参数：遍历的数组本身
    */
})
```

jQUery的map方法：同时可以遍历伪数组

```javascript
$.map(arr, function(value, index){
    // 可以返回一个值，作为map的结果
});
```

## each 与 map 的区别

- each 默认的返回值就是遍历的对象，遍历谁就返回谁
- map 默认的返回值是一个空数组
- each 不支持在回调函数中对遍历的数组进行处理
- map 可以再回调函数中通过 return 对遍历的数组进行处理，然后生成一个新的数组返回

## trim 方法

去除字符串两边的空白，返回值是去除之后的字符串

```javascript
s = "   asdasd  ";
s = $.trim(s);
```

## isWindow 方法

判断当前对象是不是window 

`$.isWindow(xxx)`

## isArray 方法

判断对象是不是一个`真`数组

## isFunction 方法

判断是不是一个function

注：jQuery框架本质上是一个函数

## holdReady 方法

暂停入口函数的执行

```javascript
$.holdReady(true);

$(function() {
    
})
```

在需要的时候恢复入口函数的执行

# jQuery 选择器

# 属性相关操作

## 属性和属性节点

什么是属性？

- 对象身上保存的变量就是属性

如何操作属性？

- `对象.属性名 = 值`
- `对象[属性名字符串] = 值`

什么是属性节点？

- 在编写HTML时，在标签中添加的属性就是属性节点

如何操作属性节点？

- `DOM元素.setAttribute("属性节点名称", "值")`
- `DOM元素.getAttribute("属性节点名称")`

属性和属性节点的区别？

- 任何对象都有属性
- 只有DOM对象才有属性节点

## attr 方法

`attr()`

- 如果传递一个参数，表示获取属性节点的值
- 如果传递两个参数，表示设置属性节点的值
- 注：
  - 如果是获取：无论找到多少个元素，都只会返回第一个元素指定的属性节点的值
  - 如果是设置，则找到多少个就设置多少个

`removeAttr()`

- 移除相关属性节点
- 会删除所有找到的元素的属性节点

## prop 方法

`prop()`

- 获取或者设置属性
- 不仅可以操作属性，还可以操作属性节点
- 官方推荐在操作属性节点的时候，具有true、false两个属性的属性节点使用 prop，其他的使用 attr

`removeProp()`

- 移除属性



特点与 attr 方法一致

# 类相关操作

# 文本值相关操作

# 样式操作方法







































