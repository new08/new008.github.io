---
layout: post
title:  "使用 DevTools 的工作区设置持久化"
date:   2017-12-27 21:00
categories: work
tags: chrome debug
---

# 使用 DevTools 的工作区设置持久化

转自 [chrome.doc: setup persistence with DevTools Workspaces](https://developers.google.com/web/tools/setup/setup-workflow)

试想一种场景，产品上线培训时有一位客户发现某个页面打不开。在你使用他的电脑查看问题时，发现是该页面有 js 语法错误。现在，需要让这位客户立刻可以使用此功能，但又没法快速地进行服务器部署。这种时候除了向客户道歉外，还有另一种解决方式 —— 使用 DevTools 现场改浏览器中运行的代码解决问题。

我们在使用 chrome DevTools 时，都知道可以在 Sources 视图里查看代码和 debug。但较人知道如何直接修改代码并运行。

以 Chrome 63.0 为例，

1. 打开 DevTools > 进入 sources > 在左侧资源清单里找到需要编辑的文件。

2. 在文件上右键点击 'Add folder to workspace', 根据提示选择本地文件夹作为 workspace。期间，浏览器会提示授权，点击同意即可。

按上述部署设置后，就可以直接在 source 里面修改 js 文件了。但是需要注意，如果页面刷新，修改的文件就会还原成服务器下载的版本了。

另外，如果是压缩过的js，修改时就要注意了。如果使用 Pretty Print 功能，DevTools 会开启只读的新页签显示格式化后的代码。这里是不能编辑的。折中的做法是：复制 Pretty Print 后的 js 代码，替换掉 .min.js 里面的内容。

todo

1. 持久化保存

2. 灰度部署

3. 部署前 ES5 语法检查





### 参考资料

[1] [使用Chrome DevTools的Timeline分析页面性能](http://horve.github.io/2015/10/26/timeline/)   
[2] [网页性能管理详解](http://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html)   
[3] [404 Large Data Sets and Performance](http://ui-grid.info/docs/#/tutorial/404_large_data_sets_and_performance)   
[4] []()   
[5] []()   
[6] []()   