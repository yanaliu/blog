---
title: JavaScript设计模式与开发实践：高阶函数（一）
date: 2017-06-21 21:39:33
tags: [js, 高阶函数]
---

# 是什么

高阶函数是指至少满足下列条件之一的函数：

1. 函数可以作为参数被传递：回调函数、Array.prototype.sort
2. 函数可以作为返回值输出：判断数据的类型、getSingle

<!--more-->

## 判断数据类型

在这个例子里，要判断一个数据的类型，使用Object.prototype.toString进行计算。该函数的返回值如下：

```javascript
Object.prototype.toString.call([6,6,6])  // "[object Array]"
Object.prototype.toString.call("str")  // "[object String]"
Object.prototype.toString.call(66)  // "[object Number]"
```

可以使用循环语句，来批量注册判断类型的isType函数，如下：

```javascript
var Type = {};
for(var i = 0, type; type=['String', 'Array', 'Number'][i++]; ) {
	(function(type) {
		Type['is'+type] = function(obj) {
			return Object.prototype.toString.call(obj) === '[object ' + type +']';
		} 
	})(type)
};
console.log(Type.isArray([])); //true
console.log(Type.isString("str")); //true
```

简单解释一下这个程序，type是数组['String', 'Array', 'Number']中的第i个元素，并作为参数传递到循环内立即执行的函数表达式。这个函数表达式的每次执行都给Type对象注册一个['is'+type]的函数。

## getSingle

这是个**单例模式**的例子，后续的设计模式系列博客里会介绍，这里只简单说一说。

```javascript
var getSingle = function(fn) {
	var ret;
	return function() {
		return ret || (ret = fn.apply(this, arguments));
	};
};

//效果
var getScript = getSingle(function() {
	return document.createElement("script");
});
var script1 = getScript();
var script2 = getScript();
console.log(script1 === script2);
```

在getSingle这个高阶函数中，既把函数当做参数传递，有在函数执行后返回另一个函数。

在这个函数里，有几个问题需要明确：

1. 函数里的this指的是什么：我们在getSingle中显示返回了一个对象（函数），返回的这个函数作为普通函数被调用时，this指向全局对象，在浏览器里就是window.
2. arguments到底是哪个函数的参数：arguments是getSingle里被返回的那个函数的参数，在这个程序里，getScript指向被返回的函数，所以arguments是getScript函数被调用执行的时候的参数。
3. 两次执行getScript函数得到的结果分别是什么：
   1. 首先明确的是这是一个闭包结构，所以getScript的每次执行都能访问到函数定义时所在的作用域里的ret变量。
   2. 函数第一次执行的时候，由于ret是undefined的，所以执行的是逻辑表达式(||)的第二部分，也就是作为参数传递进去的fn函数被执行，并把执行结果报存在ret里。
   3. 所以，函数第二次执行的时候，由于ret里已经有值了，所以会直接返回ret的值，而不会再次执行fn函数。当然，以后的每次执行也都是直接返回ret里已经报错的值。

# 高阶函数实现AOP

AOP(面向切面编程)的主要作用是把一些跟核心业务逻辑模块无关的功能（日志记录、安全控制、异常处理等）抽离出来，再以“动态织入”的方式掺入业务逻辑模块中。这样可以保持业务逻辑模块的纯净和高内聚性，而且可以很方便的复用日志记录能功能模块。

在JS中实现AOP，是指把一个函数“动态织入”另一个函数中，下面是通过扩展Function.prototype来实现的方法，这是一种**装饰者模式**，以后也会介绍。

```javascript
Function.prototype.before = function(beforefn) {
	var _self = this; //保存原函数的引用
	return function() {  //返回包含了原函数和新函数的“代理”函数
		beforefn.apply(this, arguments);  //执行新函数，修正this
		return _self.apply(this, arguments); //执行原函数
	}
};

Function.prototype.after = function(afterfn) {
	var _self = this;
	return function() {
		var ret = _self.apply(this, arguments);
		afterfn.apply(this, arguments);
		return ret;
	}
};

var func = function() {
	console.log(2);
};

func = func.before(function() {
	console.log(1);
}).after(function() {
	console.log(3);
});

func();
```

## 需要明确的问题

在这个函数里，我们需要弄清楚before和after里的_self分别指什么，返回的函数是什么样的，最后的func函数又变成了什么样。

1. _self的指向

   1. before函数里，_self指向before函数的调用者（this作为对象的方法调用时的绑定规则），也就是原始的func函数，如下：

      ```javascript
      function() {
      	console.log(2);
      };
      ```

   2. 同理，after函数里，_self也指向after函数的调用者，也就是before函数返回的结果，详见下面的分析。

2. 返回的函数

   1. before函数的返回函数如下：

      ```javascript
      function () { 
      	beforefn.apply(this, arguments); //this指向window
      	 return _self.apply(this, arguments); 
      }
      //由于_self执行原始的func函数，所以相当于：
      function () { 
      	beforefn.apply(this, arguments); //console.log(1)
      	return func.apply(this, arguments);  //console.log(2)
      }
      ```

   2. after函数的返回函数如下：

      ```javascript
      function () {
      	var ret = _self.apply(this, arguments); //_self指向before的返回函数
      	afterfn.apply(this, arguments); //console.log(3)
      	return ret;
      }
      ```

3. func函数

   根据上面的分析可知，func函数最后会变成下面的样子：

   ```javascript
   function () { 
   	beforefn.apply(this, arguments); //console.log(1)
   	func.apply(this, arguments);  //console.log(2)
     	afterfn.apply(this, arguments); //console.log(3)
   }
   ```

4. 最后的疑问？

   不明白为什么before函数和after函数里最后都有一个return语句，如果有明白的朋友，请告诉我，不胜感激！

   我测试过了，去掉return语句之后，函数的执行结果并不会改变，而且我觉得语义还更清晰了呢。

---

笔者才疏学浅，如有错误，欢迎[联系我](https://yanaliu.github.io/about/)批评指正。