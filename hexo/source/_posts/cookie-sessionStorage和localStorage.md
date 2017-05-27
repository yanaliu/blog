---
title: cookie, sessionStorage, localStorage和userData
date: 2017-05-22 19:14:53
tags: [前端, js]
categories: 前端
---

# cookie

cookie(即HTTP cookie)是网站为了标识用户身份而储存在用户本地终端上的数据（通常经过加密）。

**cookie数据始终在同源的http请求中携带**（即使不需要），即会在浏览器和服务器间来回传递。

**sessionStorage和localStorage不会自动把数据发给服务器**，仅在本地保存。

**cookie在设置的过期时间之前一直有效，即使窗口或者浏览器关闭**

<!--more-->
``` javascript
//由于js中读写cookie不是非常直观，所以需要一些函数来简化cookie的功能。
var cookieUtil = {
   //读取cookie
    get: function(name) {
        var cookieName = encodeURIComponent(name) + "=",
            cookieStart = document.cookie.indexOf(cookieName),
            cookieValue = null;
        if(cookieStart > -1) {
            var cookieEnd = document.cookie.indexOf(";", cookieStart);
            if(cookieEnd == -1) {
                cookieEnd = document.cookie.length;
            }
            cookieValue = decodeURIComponent(document.cookie.substring(cookieStart + cookieName.length, cookieEnd));         
        }
        return cookieValue;
    }

    //写入cookie
    set: function(name, value, expires, path, domain, secure) {
        var cookieText = encodeURIComponent(name)+ "=" + encodeURIComponent(value);
        if(expires instanceof Date) {
            cookieText += "; expires=" + expires.toGMTString();
        }
        if(path) {
            cookieText += "; path=" + path;
        }
        if(domain) {
            cookieText += "; domain=" + domain;
        }
        if(secure) {
            cookieText += "; secure";
        }
        document.cookie = cookieText;
    } 

    //删除cookie
    unset: function(name, path, domain, secure) {
        this.set(name, , new Date(0), path, domain, secure);
    }
} ;

```

## cookie的缺点

1. cookie的数量和长度都有限制。每个domain最多只能有20条cookie，每个cookie长度不能超过**4KB**，否则会被截掉。
2. 安全性问题。如果cookie被人拦截了，那人就可以取得所有的session信息。即使加密也与事无补，因为拦截者并不需要知道cookie的意义，他只要原样转发cookie就可以达到目的了。
3. 有些状态不可能保存在客户端。例如，为了防止重复提交表单，我们需要在服务器端保存一个计数器。如果我们把这个计数器保存在客户端，那么它起不到任何作用

## cookie隔离

如果静态文件都放在主域名下，那静态文件请求时会把带有的cookie数据提交给server，非常浪费流量，所以需要隔离。
因为cookie有域的限制，不能跨域提交请求，所以使用非主要域名的时候，请求头中就不会带有cookie数据。
这样可以降低请求头的大小，降低请求时间，从而达到降低整体请求延时的目的。
同时，这种方法不会将cookie传入server，也减少了server对cookie的处理分析环节，提高了webServer的http请求解析速度。

# 浏览器本地存储

在较高版本的浏览器中，js提供了sessionStorage和globalStorage。在HTML5中提供了localStorage来取代globalStorage。
sessionStorage用于本地存储一个会话（session）中的数据，**这些数据只有在同一个会话中的页面才能访问并且当会话结束后数据也随之销毁。**因此sessionStorage不是一种持久化的本地存储，仅仅是会话级别的存储。
 **而localStorage用于持久化的本地存储，除非主动删除数据，否则数据是永远不会过期的。**
localStorage和sessionStorage也有存储大小的限制，但比cookie大的多，可以达到**5M**或更大。

``` javascript
//clear(): 删除所有值；firefox没有实现
//getItem(name): 根据name获取值
//setItem(name, value): 
//key(index): 获得index位置处的值的名字
//removeItem(name): 删除name名值对。

//使用方法存储数据
sessionStorage.setItem("name", "yanaliu");

//使用属性存储数据
sessionStorage.book = "professional javascript";

//遍历sessionStorage中的值
//方法一
for(let key of sessionStorage) {
    var value = sessionStorage.getItem(key);
    alert(key + "=" + value);
}
//方法二
for(var i = 0, len = sessionStorage.length; i < len; i++) {
    var key = sessionStorage.key(i);
    var value = sessionStorage.getItem(key);
    alert(key + "=" + value);
}

```


# web storage和cookie的区别


Web Storage的概念和cookie相似，区别是它是为了更大容量存储设计的。Cookie的大小是受限的，并且每次你请求一个新的页面的时候Cookie都会被发送过去，这样无形中浪费了带宽，另外cookie还需要指定作用域，不可以跨域调用。
除此之外，Web Storage拥有setItem,getItem,removeItem,clear等方法，不像cookie需要前端开发者自己封装setCookie，getCookie。
但是Cookie也是不可以或缺的：Cookie的作用是与服务器进行交互，作为HTTP规范的一部分而存在 ，而Web Storage仅仅是为了在本地“存储”数据而生
浏览器的支持除了IE７及以下不支持外，其他标准浏览器都完全支持(ie及FF需在web服务器里运行)，值得一提的是IE总是办好事，例如IE7、IE6中的UserData其实就是javascript本地存储的解决方案。通过简单的代码封装可以统一到所有的浏览器都支持web storage。
localStorage和sessionStorage都具有相同的操作方法，例如setItem、getItem和removeItem等


# userData


在IE5.0中，微软通过一个自定义行为引入了持久化用户数据的概念。userData允许每个文档最多**128KB**数据，每个域名最多**1MB**数据。
``` javascript
//首先，使用CSS在某个元素上指定userData行为。
<div style="behavior:url(#default#userData)" id="dataStore"></div>

//使用setAttribute()方法保存数据
var dataStore = document.getElementById("dataStore");
dataStore.setAttribute("name", "yanaliu");
dataStore.setAttribute("book", "professional javascript");
//指定数据空间的名称
dataStore.save("bookInfo");

//获取数据
dataStore.load("bookInfo");
alert(dataStore.getAttribute("name"));
alert(dataStore.getAttribute("book"));

//删除数据
dataStore.removeAttribute("name");
dataStore.removeAttribute("book");
//指定数据空间的名称
dataStore.save("bookInfo");

```
---
从今天开始写端面试题系列，本系列只作为本人的学习笔记使用，无任何商业用途，侵删。