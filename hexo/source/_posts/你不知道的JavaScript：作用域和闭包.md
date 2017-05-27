---
title: 你不知道的JavaScript：作用域和闭包
date: 2017-05-18 16:09:14
tags: [前端, js]
categories: 前端
---
最近看了《你不知道的JavaScript》这本书的上卷，感觉相当不错，一些以前模糊不清的感念从这本书里都弄懂了，下面是我的简单的笔记，写在这里权做记录。

## 变量的赋值
编译器在当前作用域中声明变量，运行时引擎在作用域中查找该变量，并为其赋值

``` javascript
var a = 2; //相当于var a; 和 a = 2;两条指令
```
<!-- more -->

## LHS和RHS
实际上相当于左引用和右引用，也可以说是定值和引用，RHS查询失败抛出referenceError异常，LHS查询失败会隐式创建一个全局变量（非严格模式下），或者抛出referenceError异常（严格模式下）
referenceError作用域查找不到， typeError作用域链查找到了，但是类型错误。

``` javascript
function foo(a){
	console.log(a + b); //referenceError
	b = a;//创建一个全局变量
}
foo(2);
```

## 欺骗词法
会被严格模式影响，导致无法在编译时对作用域查找进行优化。
### eval
eval修改其所处的词法作用域，就像eval中的代码写在这里一样
``` javascript
function foo(str, a) {
	eval(str); //欺骗
	console.log(a, b);
}
var b = 2;
foo("var b = 3", 1); //1, 3
```
严格模式下eval有自己的词法作用域。
``` javascript
function foo(str) {
	"use strict";
	eval(str); 
	console.log(a); // referenceError
}
foo("var a = 2"); //1, 3
```
### with
根据传递给它的对象创建一个新的词法作用域，将对象的属性作为该对象中的标识符
``` javascript
function foo(obj) {
	with(obj) {
		a = 2;
	}
}
var o1 = {
	a:3
};
var a2 = {
	b:3
};
foo(o1);
console.log(o1.a); // 2

foo(o2);
console.log(o2.a); //undefined
console.log(a); //2
```

## 函数声明和函数表达式
名称标识符绑定的位置不同（函数所在作用域、函数本身）。
``` javascript
//foo被绑定在所在作用域中
var a = 2;
function foo() {
	var a = 3;
	console.log(a); //3
}
foo();
console.log(a);//2

//函数声明，foo只能在函数内部使用，不会污染外部作用域
var a = 2;
(function foo() {
	var a = 3;
	console.log(a); //3
})();
console.log(a);//2
```
## 匿名函数缺点
调用栈更难追踪；自我引用更难；代码更难理解
## 块作用域
### with
用with从对象中创建的作用域仅在with声明中有效, 
### catch
``` javascript
try {
	undefined();//执行一个非法操作来强制制造出一个异常
}
catch(err) {
	console.log(err);//可以正常执行
}
console.log(err);//referenceError: err not found
```
### let
let为其声明的变量隐式的劫持了所在的块作用域(不会在块内进行变量提升)
``` javascript
var foo = true;
if(foo) {
	let bar = foo *２;
	bar = something(bar);
	console.log(bar);
}
console.log(bar);//referenceError
```
### const
创建块级作用域，值固定。

## 变量提升
函数优先，且重复的var声明会被忽略，但重复的函数声明会覆盖之前的声明。
## 闭包
函数当成值传递的时候，会出现闭包，因为函数不是在其定义时所在的作用域里执行，但执行时仍保存了书写位置的作用域
``` javascript
function foo() {
	var a = 2;
	function bar() {
		console.log(a);
	}
	return bar;
}
var baz = foo();
baz();//2,这就是闭包的效果
```
## 循环和闭包
共享闭包作用域、共享变量
``` javascript
for(var i = 0; i < 3; i++) {
	setTimeout(function timer() {
		console.log(i);  //3,3,3
	}, 0);
}
```
IIFE(立即执行函数表达式)形成的作用域里并没有内容，仍然不行
``` javascript
for(var i = 0; i < 3; i++) {
	(function () {
		setTimeout(function timer() {
			console.log(i);  //3,3,3
		}, 0);
	})();
} 
```
每个迭代生成一个新的作用域，保存了正确的变量值
``` javascript
for(var i = 0; i < 3; i++) {
	(function () {
		var j = i;
		setTimeout(function timer() {
			console.log(j);  //0,1,2
		}, 0);
	})();
} 
```
let劫持块作用域
``` javascript
for(var i = 0; i < 3; i++) {
	let j = i;  
	setTimeout(function timer() {
		console.log(j);  //0,1,2
	}, 0);
}
```
let表明i每次迭代声明一次，并用上次迭代结束时的值初始化这个变量
```
for(let i = 0; i < 3; i++) {
	setTimeout(function timer() {
		console.log(j);  //0,1,2
	}, 0);
}
```
## 模块
1.必须有外部封闭函数，且至少执行一次；
2.封闭函数必须返回至少一个内部函数
``` javascript
function CoolModule() {
	var something = 'cool';
	var another = [1, 2, 3];
	function doSomething() {
		console.log(something);
	}
	function doAnother() {
		console.log(another.join('!'));
	}
	return {
		ds: doSomething,
		da: doAnother
	};
}
var foo = CoolModule();
foo.ds();
foo.da();
```