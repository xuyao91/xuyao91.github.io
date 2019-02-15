---
layout: post
title: "ES6中如何删除数组中的重复项"
date: 2019-02-15 16:00:59 +0800
comments: true
categories: 
---

&emsp; ES6中，如何删除一个数组中的重复数据呢，以下是我总结的，从数组中过滤掉重复项并且返回唯一值的三种方法。我最喜欢的是使用Set因为它是最简单和最简单的😁
<!-- more -->
```
const array = ['🐷',1, 2,'🐷','🐷', 3]

// 1: 'Set'
[... new Set(array)]

// 2: 'Filter'
array.filter((item, index) => array.indexOf(item) === index)

// 3: 'Reduce'
array.reduce((unique, item) => unique.includes(item) ? unique : [...unique, item], [])

// 返回结果
[ '🐷', 1, 2, 3 ]
```
#### 1. 使用Set
我先解释一下什么是Set:
>Set是ES6中引入的新数据对象。Set只允许您存储唯一值。当传入数组时，它会删除任何重复的值。  

ok，我们重新回到刚才的代码来分析一下发生了什么事情，（上面的代码）做了两件事情。
1、首先，我们通过传递一个数组来创建了一个新的Set，因为Set只允许存储唯一的值，所以数组中那些重复的值会被删除。
2、创建一个新的Set对象后，重复值已经被删除了，现在我们使用...运算符，将其转换回数组。
```
const array = ['🐷',1, 2,'🐷','🐷', 3]

// Step 1
const uniqueSet = new Set(array)
# Set { '🐷', 1, 2, 3 }

// Step 2
const backToArray = [...uniqueSet]
# [ '🐷', 1, 2, 3 ]
```
##### 使用Array.from 将Set转换为Array
或者你也可以使用Array.from将Set转化为数组
```
const array = ['🐷',1, 2,'🐷','🐷', 3]

Array.from(new Set(array))
# [ '🐷', 1, 2, 3 ]
```
#### 2.使用Filter
为了便于理解，我们先来看下indexOf和filter会做什么事情
##### indexOf
indexOf方法会将我们提供的参数，在数组中找到该元素的第一个索引值并返回。
```
const array = ['🐷',1, 2,'🐷','🐷', 3]

array.indexOf('🐷') #=> 0
array.indexOf(1) #=> 1
array.indexOf(2) #=> 2
array.indexOf(3) #=> 5
```
#### filter
filter()方法会根据我们传递的条件，来创建一个新的元素数组。换句话说，如果元素通过并返回true，它将包含在已经过滤的数组中。任何失败元素或者返回为false的，都不会在过滤后的数组中。
让我们一步步来分析数组循环时发生的事情
```
const array = ['🐷',1, 2,'🐷','🐷', 3]

array.filter((item, index) => {
	console.log(
		// a. Item
		item,
		// b. Index
		index,
		// c. IndexOf
		array.indexOf(item),
		// d. Condition
		array.indexOf(item) === index,
	);
	return array.indexOf(item) === index
})
```
以下是上面显示的console.log的输出。重复项是索引与indexOf不匹配的位置。因此，在这些情况下，条件将为false，不会包含在我们的过滤数组中。
![image.png](https://upload-images.jianshu.io/upload_images/1796624-e1340356c5bd98fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 检索重复值
我们还可以使用filter方法从数组中检索重复值。我们可以通过简单地修改我们的条件就可以做到：
```
const array = ['🐷',1, 2,'🐷','🐷', 3]
array.filter((item, index) => array.indexOf(item) !== index)
# [ '🐷', '🐷' ]
```
再次，如果我们一步一来执行上面的代码，可以查看输出：
![image.png](https://upload-images.jianshu.io/upload_images/1796624-37f124574645e5c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3.使用 reduce
reduce方法用于减少数组的元素，并根据您传递的某些reducer函数将它们组合成最终数组。

在这个例子中，我们的reducer函数检查我们的最终数组是否包含该项。如果没有，请将该项目推送到我们的最终数组中。否则，跳过该元素并按原样返回我们的最终数组（基本上跳过该元素）。

reduce总是有点难以理解，所以让我们进入每个案例并查看输出：
```
array.reduce((unique, item) => {
	console.log(
		// a. Item
		item,
		// b. Final Array (Accumulator)
		unique,
		// c. Condition (Remember it only get pushed if this return 'false')
		unique.includes(item),
		// d. Reducer Function Result
		unique.includes(item) ? unique : [...unique, item],
	);
	unique.includes(item) ? unique : [...unique, item]
}, [])

# [ '🐷', '🐷' ]
```
这是console.log的输出：
![image.png](https://upload-images.jianshu.io/upload_images/1796624-8de3d95d35e6bf25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 参考资料
*   [MDN Web Docs — Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
*   [MDN Web Docs — Filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
*   [MDN Web Docs — Reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
*   [GitHubGist: Remove duplicates from JS array](https://gist.github.com/telekosmos/3b62a31a5c43f40849bb)
*   [CodeHandbook: How to Remove Duplicates from JavaScript Array](https://codehandbook.org/how-to-remove-duplicates-from-javascript-array/)

#### 说明
本文翻译自[Samantha Ming](https://medium.com/@samanthaming)的博客，原文地址  
https://medium.com/dailyjs/how-to-remove-array-duplicates-in-es6-5daa8789641c
