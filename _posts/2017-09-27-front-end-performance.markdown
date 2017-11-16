---
layout: post
title:  "前端作业性能调优"
date:   2017-09-27 17:49
categories: work
tags: angular ui-grid front-end chrome-devtools
---

# 前端作业性能调优

**环境** ng 1.2.32

### 背景说明

分销系统费用模组《对账结算单维护作业》的列表页进入详情页能感觉明显卡顿，有时要 5 秒时间。


### 分析

1. 要打开的页面包含 `42个只读栏位` 和 `7个表格`，每个表格的栏位数目不一，最多的有 `42列`。页面元素较多。
2. 页面打开时没有复杂运算逻辑：打开过程中，只需要调用一个查询服务，将返回的数据显示在画面傻瓜。
3. 使用 chrome devTools 的网络面板检查，微服务返回的数据不大（20KB ），请求时间也很短时（几十毫秒），页面打开需要 5 秒。

初步判断

1. 页面打开的等待时间过长和后端服务无关，前端是瓶颈点。
2. 在详细测试中，服务调用不会是主要影响因子，可以在前端使用静态数据进行模拟测试。

### 处理方案

1. 使用 chrome devTools 测试，
    记录页面打开的等待时间，并作为基准线
2. 使用 performanceTool 分析瓶颈   
3. 修改代码并测试，与基准线作比较
4. 重复 2、3 两个步骤
5. 确认最终优化方案

### 使用 chrome devTools 测试

**测试目的**

+ 记录页面打开的等待时间
+ 希望记录页面打开和操作过程中的事件，用来分析等待原因

**为什么使用 chrome devTools 进行测试？**

+ 通过 devTools 提供的工具可以准确记录页面加载，交互，动画的开销
+ devTools 的 `网络面板` 可以记录页面请求和下载的资源文件的信息，包括请求消耗时间和传输大小等关键信息
+ devTools 的 `性能面板` 可以通过记录和查看网站生命周期内发生的各种事件来提高页面的运行时性能
+ 普通的计时工具受到的干扰因素较多，不准确

**测试步骤**

测试主要使用 [performance tool](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance) 工具。测试前，修改页面代码直接加载静态数据，排除网络影响（同时使用本地的 web 服务器部署前端项目）。

1. 关闭系统的其他软件，关闭浏览器分析系统外网页，关闭浏览器插件
2. 打开 `devTools` > `Performance`。 勾选 Screenshots 选项
   *勾选 Screenshots 后，Timeline 面板可以在页面加载时捕捉屏幕截图，结合 Timeline 上的其他信息可以直观地观察页面打开过程*
3. 首先进入列表页，然后点击 Record 按钮开始记录。等待3s后点击网页上的详情按钮
4. 等待详情页面完全打开，等待3s点击Timeline记录的stop按钮
5. `Ctrl + S` 保存记录
6. 点击 Clear 按钮清空数据。重复 3~5 步骤，每种情况记录 3 次数据

### 使用 performanceTool 分析瓶颈

**检查 performance 记录**

![performance-overview](http://zillionmile.com:8080/images/blog/17-9-27-performance.png)

在 Timeline 面板中查看加载的 profile 文件，需要重点关注的是

+ Overview
+ Details
+ 火焰图

**Overview 信息分析**

![performance-overview](http://zillionmile.com:8080/images/blog/17-9-27-performance-overview.png)

```
Overview 窗格包含以下三个图表：

FPS: 每秒帧数。绿色竖线越高，FPS 越高。 FPS 图表上的红色块表示长时间帧，很可能会出现卡顿。
CPU: CPU 资源。此面积图指示消耗 CPU 资源的事件类型。
NET: 每条彩色横杠表示一种资源。横杠越长，检索资源所需的时间越长。
每个横杠的浅色部分表示等待时间（从请求资源到第一个字节下载完成的时间）。
深色部分表示传输时间（下载第一个和最后一个字节之间的时间）。
不同类型的资源现实色彩不同：HTML（蓝色），脚本（黄色），样式表（紫色），媒体文件（绿色），其他资源（灰色）。
```

鼠标在 Overview 窗格上左右移动时，下方会同步显示当前时间点的屏幕截图。   
同时，通过选择时间范围可以筛选火焰图和Details的信息。

**Details 信息**

Summary 页签以饼图的方式呈现一个时间片段内浏览器动作的时间分布   
![performance-overview](http://zillionmile.com:8080/images/blog/17-9-27-performance-summary.png)   

```
蓝色(Loading)：网络通信和HTML解析
黄色(Scripting)：JavaScript执行
紫色(Rendering)：样式计算和布局，即重排
绿色(Painting)：重绘
灰色(other)：其它事件花费的时间
白色(Idle)：空闲时间
```

可以发现，占用时间最多的是 Scripting 和 Rendering。   
在下面的火焰图可以检查它们的详细信息。

**火焰图**

火焰图是 CPU 堆叠追踪的可视化图形，其 x 轴是时间轴， y 轴是方法调用栈。   
图形中每一个矩形框代表一个[事件](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/performance-reference) 。其中有些矩形框的右上角有红色箭头，这代表 devTools 认为这里可能存在性能瓶颈。

![performance-overview](http://zillionmile.com:8080/images/blog/17-9-27-performance-fire.png)   

[Scripting](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/performance-reference#scripting) 是 JavaScript 执行记录，用黄色显示 。  

[Rendering](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/performance-reference#rendering) 是页面渲染的过程, 用紫色显示。   
通过查看火焰图，可以发现 `Rendering` 时间主要是 ui-grid 产生的 `Forced reflow`。

通过检查 profile 文件，可以推测打开页面等待的瓶颈点是 ui-grid 呈现。

*PS. 其实在页面上注释表格代码后，实测打开速度很快。亦能说明表格元素是造成等待时间长的主要因素.*

### 页面优化

首先尝试优化 ui-grid 的使用：根据 ui-grid 官方文档，gridOption 有两个可选参数 `flatEntityAccess` 和 `fastWatch`。

```
if you have a data array that consists purely of flat objects - and each column in your grid is tied to a single field in the entities in that array (i.e. the way we think 90% of people use the grid), then you can turn off the support for complex binding, and the grid will run faster.
...

 the grid watches the data and the column defs to determine if anything has changed. Watches get called very frequently in angularJS (on every digest cycle, which can be many times per second), and watching each row of a large data collection can be expensive. The fastWatch gridOption watches only the data reference and it's length - in theory this means we might not notice if you replaced a row in the data, or replaced a column in the columnDefs. If using fastWatch you should always do deletes and adds separately, never a swap. Fastwatch cannot be dynamically toggled - you need to set it upon grid creation.

```

通过设置两个参数，可以显著改善 ui-grid 的效率。

使用 performance tool 进行测试后，同样的数据进行显示。等待时间由 5.3s 左右降低到了 2.6s。

**进一步优化**

在当前数据量下，2.6s 的打开时间依然非常长，在 ui-grid 本体优化空间有限的情况下。只能简化显示的内容。

检查表格栏位的 HTML 模板。发现大量单元格都使用了 empty-text 控件，该控件的主要作用是当没有值时，可以显示 `<空>`。通过代码替换去除它们的使用后。页面等待时间进一步降低到 1.7s 左右。

**其它优化方向**

通常来说，在使用 angularjs 时，我们都会推荐不可以修改的元素使用单向绑定替换双向绑定。通过修改 HTML 模板 —— 将 ng-model 修改为 ng-value。获得了不足 0.1s 的1提升。  

### 优化方案总结

**表格优化：**  
1. 对于几乎全部页面。都可以通过设置 gridOption.flatEntityAccess 来改善性能。   
2. 对于几乎全部详情页面。都可以通过设置 gridOption.fastWatch 来改善性能。但是，新增和修改页面需要慎用此参数。  
3. 当表格包含很多栏位或笔数很多时，需要尽量避免使用 `cellTemplate`。  

**其他优化：**   
1. 尽可能使用单向绑定。   

### 下阶段目标

相同条件下，使页面 Loading 时间降到 `1s` 以内。

1. 为 ui-gird 的栏位设定固定宽度应该也有改善（未验证）。 
2. 通过限制 ui-grid 控件的宽度和高度能减少计算压力。但该方案需要和 SA 沟通取得同意才能使用。
3. 测试过程中有观察到疑似内存泄漏现象（待单独验证）。   


### 参考资料

[1] [使用Chrome DevTools的Timeline分析页面性能](http://horve.github.io/2015/10/26/timeline/)   
[2] [网页性能管理详解](http://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html)   
[3] [404 Large Data Sets and Performance](http://ui-grid.info/docs/#/tutorial/404_large_data_sets_and_performance)   
[4] []()   
[5] []()   
[6] []()   