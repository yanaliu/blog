---
title: Sass语法介绍高级篇
date: 2017-05-26 20:54:04
tags: [前端, Sass]
categories: 前端
---

# 响应式布局: @media

响应式布局设计的目的是为移动设备提供更好的体验，并且整合从桌面到手机的各种屏幕尺寸和分辨率。

比如，当你缩小网页的时候，网页的布局会跟着改变，这主要使用的是CSS中的media query。

Sass中的@media跟CSS中的区别是：Sass中的media query可以内嵌到css规则中，在生成css的时候才会被提到样式的最高层级。这样做的好处是避免了重复书写选择器或者打乱样式表的流程。

<!-- more -->

``` scss
//定义
@mixin col-sm($width:50%) {
  @media (min-width: 768px) {
    width: $width;
    float: left;
  }
}
//使用
.webdemo-sec {
  @include col-sm();
}
```

对应的css代码如下：

``` scss
@media (min-width: 768px) {
  .webdemo-sec {
    width: 50%;
    float: left;
  }
}
```

# 嵌套问题:@at-root

虽然Sass提供了很棒的嵌套能力，但CSS最佳实践告诉我们，浏览器在解析CSS文件的时候是按从右往左的顺序进行的，也就是说嵌套的代码会从内往外解析，一步步向上查找，直到HTML一级，这样会导致渲染的低效。

而且通过嵌套也有一些副作用：增加了样式修饰的权重，制造了样式位置的依赖。

最佳实践是给每个元素命名，但这样又不如嵌套清晰，易维护。

解决方案，在嵌套的时候使用Sass的at-root指令，明确指定被嵌套的属性输出到样式表的底层。如下，变量里是字体的定义，[看这里](https://yanaliu.github.io/2017/05/26/Sass%E5%85%A5%E9%97%A8%E4%BB%8B%E7%BB%8D/#more)

``` scss
.main-sec {
  font-family:$main-sec-ff;
  @at-root {
    .main-sec-headline {
      font: {
        family: $main-sec-ff;
        size: 16px;
      }
    }

    .main-sec-detail {
      font-size: 12px;
    }
  }
}
```

生成的css文件如下，各个样式之间不存在嵌套关系。

``` css
.main-sec {
  font-family: Arial, Verdana, Helvetica, sans-serif;
}
/* line 16, ../sass/screen.scss */
.main-sec-headline {
  font-family: Arial, Verdana, Helvetica, sans-serif;
  font-size: 16px;
}

/* line 23, ../sass/screen.scss */
.main-sec-detail {
  font-size: 12px;
}
```

# 参数校验

为了是col-sm代码块更具健壮性，我们给传入的宽度值增加校验代码如下，这里我们要求传入的width必须为百分值。

``` scss
@mixin col-sm($width:50%) {
  @if type_of($width) != number {
    @error "$width必须是一个数值类型，你输入的width是: #{$width}.";
  }
  @if not unitless($width) { //非不带单位 == 带单位
    @if unit($width) != "%" { //$width的单位不是%（px和%都是单位）
      @error "$width应该是一个百分值，你输入的width是： #{$width}.";
    }
  } @else { //不带单位
    @warn "$width应该是一个百分值，你输入的width是： #{$width}.";
    $width: (percentage($width) / 100);
  }
  @media (min-width: 768px) {
    width: $width;
    float: left;
  }
}
```

如果使用compass watch观察这个代码，在输入的width值不是百分值的情况下，会输出相应的错误提示信息。

# CSS输出样式

``` scss
output_style = :expanded or :nested or :compact or :compressed
```

expanded是默认样式，输出的css是展开的，跟手动书写css没有区别。

nested输出的css文件根据嵌套关系相应的缩进。

compact所有属性写在一行，这种情况下，更多的是关注选择器之间的关系，而不是属性

compressed将样式表压缩以减少空间，不可读。

# 中文支持

如果你跟我的安装过程一样，然后发现代码中出现中文的时候会报错，即使是注释中的中文也会报错，那么可以参考这个步骤来解决

> 找到ruby的安装目录，里面也有sass模块，类似这样的路径：
>
> C:\Ruby\lib\ruby\gems\2.2.0\gems\sass-3.4.22\lib\sass
>
> 在这里面有个文件叫engine.rb,添加一行代码
>
> Encoding.default_external = Encoding.find('utf-8')
>
> 放在所有的require xxxxxx 之后即可。

---

文字资料来自[慕课网课程](http://www.imooc.com/learn/364) 