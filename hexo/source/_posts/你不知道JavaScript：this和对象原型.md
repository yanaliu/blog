---
title: 你不知道的JavaScript：this和对象原型
date: 2017-05-18 19:39:44
tags: [前端, js]
categories: 前端
---
this在调用时绑定，完全取决于函数的调用位置。
## 绑定规则
### 默认绑定
独立函数调用，绑定到全局对象，但严格模式下this会绑定到undefined
``` javascript
function foo() {
	console.log(this.a);
}
var a = 2;
foo(); // 2
```
<!-- more -->

### 隐式绑定
当函数引用有上下文对象时，this绑定在这个上下文对象上，最后一层调用位置起作用
``` javascript
function foo() {
	console.log(this.a);
}
var obj = {
	a: 2,
	foo: foo
}
var a = "hello";
obj.foo(); // 2
```

**隐式丢失**
``` javascript
function foo() {
	console.log(this.a);
}
var obj = {
	a: 2,
	foo: foo
}
var bar = obj.foo; //实际上指向foo本身，独立函数调用
var a = "hello world";
bar(); // hello world
```
### 显示绑定
call和apply,第一个参数是对象，函数调用时将this绑定到这个对象上
硬绑定
``` javascript
function foo() {
	console.log(this.a);
}
var obj = {
	a: 2
}
var bar = function() {
	foo.call(obj);
}
bar(); // 2
setTimeout(bar, 1000); // 2
var a = "hello world";
bar.call(window); // 2
```
### new绑定
用new调用函数时，会创建一个新对象，并将这个新对象绑定到函数调用的this上。
``` javascript
function foo(a) {
	this.a = a;
}
var bar = new foo(2); //将返回的对象bar绑定在foo的this上
console.log(bar.a); //2
```

## 绑定规则优先级
### 显示绑定高于隐式绑定
``` javascript
function foo() {
	console.log(this.a);
}
var obj1 = {
	a: 2,
	foo: foo
}
var obj2 = {
	a: 3,
	foo: foo
}
obj1.foo();//2
obj2.foo();//3
obj1.foo.call(obj2);//3,先将foo中的this绑定到obj2，在被obj1调用
obj2.foo.call(obj1);//2
```
### new绑定高于隐式绑定
``` javascript
function foo(something) {
	this.a = something;
}
var obj1 = {
	foo:foo
};
var obj2 = {};
obj1.foo(2);
console.log(obj1.a); //2
obj1.foo.call(obj2,3);
console.log(obj2.a); //3
var bar = new obj1.foo(4);
console.log(bar.a);//4
console.log(obj1.a);//2
```
### new绑定高于硬绑定（new 和call或者apply不能同时使用）
``` javascript
function foo(something) {
	this.a = something;
}
var obj1 = {};
var bar = foo.bind(obj1);
bar(2);
console.log(obj1.a); //2
var baz = new bar(3);
console.log(obj1.a); //2
console.log(baz.a);//3
```
new中使用硬绑定，可以预先设置一些参数，使用new初始化时只需传入其他的参数，柯里化的一种
``` javascript
function foo(p1, p2) {
	this.val = p1 + p2;
}
var bar = foo.bind(null, "p1"); //不关心硬绑定到哪里，this会被new修改
var baz = new bar("p2");
console.log(baz.val);//p1p2
```

## 绑定例外
### 把null或者undefined传入call,apply,bind等，忽略this
``` javascript
function foo() {
	console.log(this.a);
}
var a = 2;
foo.call(null);//2,实际使用默认绑定
```
### 间接引用
``` javascript
function foo() {
	console.log(this.a);
}
var a = 2;
var o = {a:3, foo:foo};
var p = {a:4};
o.foo();//3
(p.foo = o.foo)();//2,返回值是目标函数的引用，所以是默认绑定
```
### 软绑定
``` javascript
function foo() {
	console.log("name: " + this.name);
}
var obj = {name:obj},
	obj2 = {name:obj2},
	obj3 = {name:obj2};
var fooOBJ = foo.softBind(obj);
fooOBJ(); //name:obj
obj2.foo = foo.softBind(obj);
obj2.foo();//name:obj2
fooOBJ.call(obj3);//name:obj3
setTimeout(obj2.foo, 10);//name:obj
```
## 箭头函数
根据外层作用域来决定this，箭头函数的绑定无法修改
``` javascript
function foo() {
	return (a) => {
		//this继承自foo()
		console.log(this.a);
	}
}
var obj1 = {a:2};
var obj2 = {a:3};
var bar = foo.call(obj1);
bar.call(obj2);//2
```
## 数组
添加数字类属性，数组长度增加
``` javascript
var myArray = ["foo", 42, "bar"];
myArray.baz = "baz";
console.log(myArray.length);
myArray["3"] = "baz";
console.log(myArray.length);
```
## 属性描述符
writable,是否可以修改属性的值；configurable，属性描述符是否可修改，为false的话禁止删除这个属性；Enumerable，可枚举
## getter / setter
``` javascript
var myObject = {
	get a() {
		return this._a_;
	},
	set a(val) {
		this._a_ = val * 2; 
	}
};
myObject.a = 2;
console.log(myObject.a);
```
属性的存在性，值为undefined或者不存在都返回
``` javascript
undefined
var myObject = {
	a:2
};
"a" in myObject; // true, 会检查原型链
"b" in myObject; //false
myObject.hasOwnProperty("a");//true，只检查对象本身
myObject.hasOwnProperty("b");//false
```
## 混入

### 显示混入

``` javascript
function mixin(sourceObj, targetObj) {
	for(var key in sourceObj) {
		if(!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}
	return targetObj;
}

var Vehicle = {
	engine: 1,
	ignition: function() {
		console.log("turning on my engine.");
	},
	drive: function() {
		this.ignition();
		console.log("steering and moving forward!");
	}
};

var Car = mixin(Vehicle, {
	wheels: 4,
	drive: function() {
		Vehicle.drive.call(this);
		console.log("rolling on all" + this.wheels + "wheels!");
	}
});
```

### 混合混入
先进行复制，之后再特殊化,效率低，不常用

``` javascript
function mixin(sourceObj, targetObj) {
	for(var key in sourceObj) {
		targetObj[key] = sourceObj[key];
	}
	return targetObj;
}

var Vehicle = { 
	//...
};

var Car = mixin(Vehicle, {});
mixin({
	wheels: 4,
	drive:function(){
		//...
	}
}, car);
```
### 寄生继承

``` javascript
function Vehicle() {
	this.engine = 1;
}
Vehicle.prototype.ignition = function() {
	console.log("turning on my engine.");
};
Vehicle.prototype.drive = function() {
	this.ignition();
	console.log("steering and move forward!");
};
function Car() {
	var car = new Vehicle();
	car.wheels = 4;
	var vehDrive = car.drive;
	car.drive = function() {
		vehDrive.call(this);
		console.log("rolling on all" + this.wheels + "wheels!");
	}
	return car;
}
var mycar = new Car();
mycar.drive();
```
### 隐式混入

``` javascript
var Something = {
	cool: function() {
		this.greetng = "hello world";
		this.count = this.count ? this.count + 1 : 1;
	}
};

Something.cool();
Something.greetng; // hello world
Something.count; // 1

var Another = {
	cool: function() {
		Something.cool.call(this);
	}
};
Another.cool();
Another.greetng; //hello world
Another.count; // 1, count 不是共享状态
```

**隐式屏蔽**

``` javascript
var anotherObject = {
	a: 2
};
var myObject = Object.create(anotherObject);
anotherObject.a; // 2
myObject.a; //2
anotherObject.hasOwnProperty("a"); //true
myObject.hasOwnProperty("a"); //false
myObject.a++; //隐式屏蔽
anotherObject.a; //2
myObject.a;//3
myObject.hasOwnProperty("a");//true
```
## 构造函数 || 调用
``` javascript
function nosp() {
	console.log("don't mind me!");
}
var a = new nosp(); //don't mind me!
a; // {} 
```
## 委托模式
``` javascript
Task = {
	setID: function(ID) {this.id = ID;},
	outputID: function() {console.log(this.id);}
};

//让XYZ委托Task
XYZ = Object.create(Task);

XYZ.prepareTask = function(ID, Lable) {
	this.setID(ID);
	this.Lable = Lable;
};

XYZ.outputTaskDetails = function() {
	this.outputID();
	console.log(this.Lable);
};
```
