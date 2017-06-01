---
title: Grunt-beginner
date: 2017-06-01 21:57:06
tags: [前端, Grunt, Yeoman, Bower]
---

# 前端集成解决方案

## 是什么

什么是前端集成解决方案，我们先来看看草根派和学员派的解释，

草根派：用来解决前段工程的根本问题

学院派：一套包含**框架**和**工具**，便于开发者**快速**构建**美丽实用**的web应用程序的工作流，同时这套工作流必须是稳健强壮的。

<!-- more -->

## 解决什么问题

那么，这些解决方案到底用来解决前端的什么问题呢？

1. 开发团队代码风格不统一，如何强制开发规范
2. 前期开发的组件库如何维护和使用
3. 如何模块化前端项目
4. 服务器部署前必须的压缩，检查流程如何简化，流程如何完善。

## 主流方式

1. Yeoman
2. Bower
3. Grunt | Gulp

# Yeoman, Bower, Grunt简介和安装

## Yeoman

Yeoman：现代Web APP的脚手架工具。在Web项目的立项阶段，使用yeoman来生成项目的文件，代码结构。Yeoman自动将最佳实践和工具整合进来，大大加速和方便了我们后续的开发。

安装方法如下：

``` 
npm install -g yo
```

注意，windows下检查是否安装成功的命令是：

``` 
yo --version
```

## Bower

Bower：web的包管理器

安装和检查：

``` 
npm install -g bower
bower -v
```

## grunt

grunt: build tool, 目的是自动化，减少压缩、编译、单元测试、代码校验这种重复且无业务关联的工作。

安装：

``` 
npm install -g grunt-cli
```

安装完成之后，输出grunt会看到一个错误信息，但这也证明grunt已经成功安装。

# Yeoman实践

为了更好的使用Yeoman，我们需要各种各样的生成器，要使用某个生成器时，可以从[看这里](http://yeoman.io/generators/)查找安装及使用信息。

以angular为例，安装命令如下：

``` 
npm install -g generator-angular
```

使用Yeoman时，首先新建一个项目目录并cd进入：

``` 
mkdir my-new-project && cd $_
```

运行yo angular，同时可以指定项目的名字：

``` 
yo angular [app-name]
```

如上，就可以完成项目的初始化。

如果运行yo angular的时候报karma相关的错误，应该是因为没有安装karma生成器，使用如下命令安装即可。

``` 
npm install -g generator-karma 
```

# bower实践

bower是一个包管理器，这里以jquery和bootstrap为例，进行说明。

首先，安装jquery和bootstrap，如下：

``` 
bower install jquery
bower install bootstrap
```

如果bower中没有注册某个组件或者框架，就不能像上面这样通过名字安装。此时可以通过github短写或者完整的github地址来安装。如果框架或者组件没有在github上，bower也可以通过URL安装。

之后使用bower init生成bower.json文件。

windows用户请到cmd窗口运行bower init命令，在git bash窗口会报错。

# Grunt实践
