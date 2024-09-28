+++
title = 'X-UA-Compatible IE=edge,chrome=1'
slug = 'x-ua-compatible'
date = 2016-04-13T21:07:37+08:00
draft = false
tags = ['浏览器兼容', 'HTML']
image = 'cover.png'
+++

为了让IE浏览器使用最高版本文档模式渲染页面，常常在`html` 中加入这句：

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
```

## X-UA-Compatible用法

起初我以为这句分别指定了IE浏览器和chrome浏览器文档模式，还纳闷了半天光指定这两个浏览器，那firefox等浏览器该怎么办，后来才知道这句仅仅是针对IE浏览器的。`IE=edge` 指明使用浏览器可用的最高版本文档模式渲染页面，其他可用的取值可以参考[MSDN:2.1.3.5 X-UA-Compatibility Meta Tag and HTTP Response Header](https://msdn.microsoft.com/en-us/library/ff955275(v=vs.85).aspx)；`chrome=1` 源于google开发的一款IE专用插件: Google Chrome Frame(GCF), 该插件可以让用户在使用IE浏览器浏览网页时，外观不变但实际上使用的是Chrome的内核。如果用户已安装GCF，则使用GCF渲染页面，否则使用最高版本IE内核渲染。不过GCF在2014年已经退休，所以现在只需要设置`IE=edge` 就可以了。

## 注意点

在 [MSDN](https://msdn.microsoft.com/zh-cn/library/jj676915.aspx) 中有这么一句：

> The X-UA-Compatible header isn't case sensitive; however, it must appear in the header of the webpage (the HEAD section) before all other elements except for the title element and other meta elements.

之前做项目时，因为同事把埋点脚本插入到X-UA-Compatible标头前面，导致条件注释判断有误，排查了半天，这个坑 [99css](https://www.99css.com/) 在六年前也踩过（[IE8 X-UA-Compatible 失效？](https://www.99css.com/463/)）。

## 浏览器模式&文档模式

前面多次提到文档模式(document mode)，究竟什么是文档模式？它和浏览器模式(browser mode)有什么不同？

概括地说，就是 **浏览器模式** 决定IE发给服务器的用户代理(UA)字符串、浏览器的默认文档模式以及IE如何解析条件注释；**文档模式** 指定IE引擎如何渲染页面，修改文档模式会刷新页面，但不会重新向服务器发送UA字符串以及从服务器获取新标记。这里先点到为止，以后有空再细说，感兴趣的可以看看这篇博客: [关于浏览器模式和文本模式的困惑](https://imququ.com/post/browser-mode-and-document-mode-in-ie.html)

## 总结
1. 通过在网页的 `head` 中指定 `X-UA-Compatible` 信息，设置文档模式:
```html
<meta http-equiv="X-UA-Compatible" content="IE=Edge">
```

2. X-UA-Compatible标头前面只能是title元素或其他meta元素，不然设置会失效。

## 参考

- [MSDN:Specifying legacy document modes](https://msdn.microsoft.com/zh-cn/library/jj676915.aspx)
- [MSDN:2.1.3.5 X-UA-Compatibility Meta Tag and HTTP Response Header](https://msdn.microsoft.com/en-us/library/ff955275(v=vs.85).aspx)
- [IE8 X-UA-Compatible 失效？](https://www.99css.com/463/)
- [关于content="IE=edge,chrome=1"介绍-让网页优先采用Chrome渲染](http://ziren.org/html-css/content-ie-edge-chrome-1-introduction-web-page-using-chrome-rendering.html)
- [IE's Compatibility Features for Site Developers](https://blogs.msdn.microsoft.com/ie/2010/06/16/ies-compatibility-features-for-site-developers/)
- [关于浏览器模式和文本模式的困惑](https://imququ.com/post/browser-mode-and-document-mode-in-ie.html)