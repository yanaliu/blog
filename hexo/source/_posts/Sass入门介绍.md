---
title: Sass入门介绍
date: 2017-05-26 16:17:31
tags: [前端, Sass]
categories: 前端
---

# Sass入门介绍

CSS预处理器，用一种专门的语言进行页面Web样式设计，编译生成正常的CSS文件以供使用，可以让CSS更加简洁、适应性更强、可读性更佳、更易于代码的维护。

优秀的CSS预处理器语言包括Sass、LESS、Stylus等，本文主要介绍Sass。

> Sass是最成熟、稳定、强大、专业的CSS扩展语言。

<!-- more -->

# Sass的两种文件后缀

## .sass

这种后缀的Sass文件使用类Ruby的语法，如下，属性之间通过换行分割，而不是分号。这种语法对于我这种熟悉C++、C等后端语言的前端开发者来说有点不太友好了。

``` 
h1
  color: #000
  background: #fff
```

## .scss

幸好Sass在3.0版本中引入Scss语法，这种语法是CSS语法的超集，如下，这种语法我就比较喜欢了！
```scss
h1 {color: #000; background: #fff}
```
两种语法没有好坏之分，如果你习惯Python或者Ruby，那Sass对你来说无疑是更好的选择，更对于正统的前端开发者来说，应该会和我一样喜欢Scss语法了。

当然你也可以在一个项目中混合使用两种语法，但注意，不要在一个文件中混合使用两种语法。

两种文件之间的转换也很简单，如下，哪里不会转哪里，so easy。

```
sass-convert main.scc main.sass
```

# Sass安装

## 安装Ruby

对于使用Windows是我来说，这一步再简单不过了。从[这里](https://rubyinstaller.org/downloads/) 下载相应版本的Ruby进行傻瓜安装即可。由于要在命令行使用Ruby，安装时要把Ruby可运行文件加载到PATH里。命令行输入*ruby -v*  检查Ruby是否安装成功。

Ruby安装完成之后，gem也就安装好了，这就和安装好node.js之后，npm就安装好了一样。

由于国内网络的原因，Ruby原有的sources经常间歇性的链接不上，所以我们首先修改Ruby sources，命令如下：

```
//移除原有的sources
gem sources --remove https://rubygems.org/
//添加国内淘宝的sources
gem sources -a https://ruby.taobao.org/
//如果上面那个失败的话，可以试试下面这个中山大学的sources（感谢慕课小伙伴提供）
gem sources -a http://mirror.sysu.edu.cn/rubygems/
//检查
gem sources -l
```

## 安装sass

```
gem install sass
//检查sass是否安装成功
sass -v

//也可以指定需要安装的sass的版本,如下：
gem install sass --version=3.3.0
//卸载某一个版本的sass
gem uninstall sass --version=3.3.0
```

# Sass语法介绍（基础篇）

## 变量

sass的变量使用$符号定义

```scss
$headline-ff: Braggadocio, Arial, Verdana, Helvetica, sans-serif;
$main-sec-ff: Arial, Verdana, Helvetica, sans-serif;
```

变量的使用：

```scss
.headline {
  font-family: $headline-ff;
}

.main-sec {
  font-family:$main-sec-ff;
}
```

Sass也提供了很方便的嵌套语法，如下：

```scss
.main-sec {
  font-family:$main-sec-ff;
  .headline {
    font-family: $main-sec-ff;
  }
}
```

这种用法相当于下面的写法：

```scss
.main-sec {
  font-family:$main-sec-ff;
}

.main-sec .headline{
  font-family:$main-sec-ff;
}
```

但是对于hover则会出现问题，比如：

```scss
a {
  :hover {
    color:blue;
  }
}
```

它生成的css文件是下面这样的，显然这种结果不符合我们的预期

```css
<!-- a后面有一个空格 -->
a :hover { 
  color:blue;
}
```

为了解决这个问题，Sass提供了父选择器&，如下：

```scss
a {
  &:hover {
    color:blue;
  }
}
```

我们一般将变量定义放在一个专门的文件里，比如_variables.scss（加下划线表明是一个局部文件，因为它不需要编译生成对应的CSS文件），在需要使用变量的文件（宿主文件）里，只需要引入这些变量即可，如下：

```scss
@import "variables";
```

## Sass里的@import 和CSS里原生的@import 

Sass的@import和CSS里原生的@import 指令的不同，CSS里原生的@import指令有两大弊端：

1. 必须放在代码最前面，否则不起作用。

2. 对性能不利。

   比如a引用b，需要浏览器把a下载下来，解析渲染a的时候读到a中@import "b"的指令的时候才会去下载b，此时浏览器处于阻塞状态，延迟页面渲染时间。

而Sass重新定义了@import指令的功能，它在编译阶段将被引用文件合并输出到了相应的CSS文件，而且@import指令可以用在文档的任何地方。

当然，如果你非要使用CSS原生的@import也不是没有可能，一切都有既定的规则，如果满足下面4个条件中的任何一个，Sass就会认为你想使用CSS原生的@import。

1. 当@import后面的文件名以".css"结尾的时候
2. 当@import后面跟的是以"http://"开头的字符串的时候
3. 当@import后面跟的是一个url()函数的时候
4. 当@import后面带有"media queries"的时候

从上面的@import指令可以看到，我们没有给"variables"文件指定后缀名，这是基于Sass的既定规则：

1. 没有文件后缀名的时候，Sass会添加.scss或者.sass
2. 统一目录下，局部文件和非局部文件不能重名（所有文件都不能重名）

## Sass注释

Sass支持下面这两种格式的注释，区别在于第二种格式的注释在编译时会生成的对应的css文件里。

```scss
//这是注释
/* 这也是注释*/
```

---

文字资料来自[慕课网课程](http://www.imooc.com/learn/364) 