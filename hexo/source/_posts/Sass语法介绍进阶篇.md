---
title: Sass语法介绍进阶篇
date: 2017-05-26 19:10:19
tags: [前端, Sass]
categories: 前端
---

变量的操作分为两种：一、直接操作变量（即变量表达式）；二、通过函数。

函数又分为两种：一、跟代码块无关的函数，多是自己的内置函数，称functions；二、可重用的代码块，称mixin，类似于C语言中的宏。

mixin又细分为两种概念：一、使用时以赋值拷贝的方式存在的，通过@include方式调用；二、使用时以组合声明的方式存在的，通过@extend的方式调用。

<!-- more -->

# 变量表达式和functions

两者的使用范围都较窄，歪果仁关于表达式变量的例子经常是下面这个样子的：

``` scss
p {
  height: (500px / 2);
}
```

从这个例子中可以看出两点：一、歪果仁的数学不咋地；二、变量表达式的运算可以带单位。

Sass官网上列出了所有的functions，[看这里](http://sass-lang.com/documentation/Sass/Script/Functions.html) 。比如hsl()，HSL是CSS3新增的颜色表示模式，HSL就是色调(Hue)、饱和度(Saturation)和亮度(Lightness)的缩写。Sass是CSS的超集，所以当然也支持这种模式。可以通过@functions声明函数。

# mixin：@include

## 定义

mixin的定义以@mixin开头，如下：

``` scss
@mixin col-6 {
  width: 50%;
  float: left;
}
```

也可以增加参数信息使mixin更具有可配置性，如下：

``` scss
@mixin col($width) {
  width: $width;
  float: left;
}
```

当然，mixin也支持默认参数，如下：

``` scss
@mixin col($width:50%) {
  width: $width;
  float: left;
}
```

## 使用

也许你会说，mixin代码块的定义是如此简单，那么，你可能会觉得代码块的使用更简单，只需要使用@include引入即可，如下：

``` scss
@include col-6();
@include col(25%);
```

# mixin：@extend

## 场景介绍

首先考虑一个样式正交性的问题，设想这样一个场景，假设你有下面两个div：

``` html
<div class="error">今天没有学习!!!!</div>
<div class="serious-error">这两天都没有学习！！！</div>
```

现在你想让error类显示红色的字体，而serious-error类除了字体为红色之外，还要有一个红色的边框，那么你可能会这样设计两个类的样式，然后再给serious-error的div增加一个error类名。

``` css
.error {
  color: #f00;
}
.serious-error {
  border: 1px solid #f00;
}
```

或者这样设计类的样式：

``` css
.error {
  color: #f00;
}
.serious-error {
  color: #f00;
  border: 1px solid #f00;
}
```

显然，这两种做法都不太好，冗余，不好维护。

## extend登场

Sass的extend属性可以帮助我们解决这个问题，明确指定一个类去继承另一个类的样式。

``` css
.serious-error {
  @extend .error;
  border: 1px solid #f00;
}
```

编译生成的css文件如下：

``` css
.error, .serious-error {
  color: #f00;
}
.serious-error {
  border: 1px solid #f00;
}
```

extend工作的原理很简单，就是把继承者的选择器（.serious-error）插入到被继承者的选择器（.error）出现的地方。

## 继承多个和连续继承

继承多个类的情况下，可以写多个extend，每个后面跟一个类名，也可以写一个extend，后面的类名以逗号分隔。

连续继承也很简单，如下，存在多个继承和连续继承的情况：

``` scss
.error {
  color: #f00;
}
.normal {
  font-size: 15px;
}
.serious-error {
  @extend .error, .normal;
  border: 1px solid #f00;
}

.dead-error {
  @extend .serious-error;
  border-color: #000;
}
```

## extend的两个知识点

1. extend不可以继承选择器系列

   ``` scss
   .A .B { 
     color: #f00;
   }
   .C { 
     @extend .A .B; //错误的
   }
   ```

   但是这样做却是可以的，虽然你可能并不想要这样的效果。

   ``` scss
   .A .B { 
     color: #f00;
   }
   .C { 
     @extend .B;
   }
   .D {
     @extend .A;
   }
   ```

   生成的CSS代码如下：

   ``` css
   .A .B, .D .B, .A .C, .D .C {
     color: #000;
   }
   ```

2. 可以使用%，用来构建只用来继承的选择器。

   ``` scss
   %error {
     color: #f00;
   }
   .serious-error {
     @extend %error;
     border: 1px solid #f00;
   }
   ```

   error的样式不会输出到生成的css文件里。

---

文字资料来自[慕课网课程](http://www.imooc.com/learn/364) 