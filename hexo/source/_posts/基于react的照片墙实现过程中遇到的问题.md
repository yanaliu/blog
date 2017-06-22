---
title: 基于react的照片墙实现时使用的插件
date: 2017-06-21 19:47:41
tags: [react, 前端]
---

# 需要安装的npm包

1. autoprefixer-loader：自动补全css3前缀，解决浏览器的兼容性问题
2. json-loader
3. sass-loader, node-sass, webpack

这些npm包的安装方法都可以在github上搜索到，需要安装什么，请自行搜索。

<!-- more -->

# sass-loader安装

如果运行npm run serve的时候提示sass-loader没有安装的错误，如下：

> Cannot resolve module 'sass-loader' in path... 

github上搜索sass-loader，可以看到安装命令如下，因为sass-loader依赖node-sass和webpack， 所以也需要安装这两个。

```
npm install sass-loader node-sass webpack --save-dev
```

执行上述命令的过程中，如果出现安装node-sass的时候无法下载win32-x64-48_binding.node文件的问题，可以复制错误提示的路径，并在浏览器内下载，路径可能是下面的形式：

```
https://github.com/sass/node-sass/releases/download/v4.5.3/win32-x64-48_binding.node
```

文件下载完成后，命令行输入下面的命令，这是通过设置环境变量告诉程序直接使用本地的.node文件。

```
set SASS_BINARY_PATH=D:/install/help/win32-x64-48_binding.node //请把路径改成你.node文件的目录
```

之后，重新执行上面的npm install命令即可。

