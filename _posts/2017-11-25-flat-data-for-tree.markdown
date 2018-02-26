---
layout: post
title:  "扩展 angularjs 指令"
date:   2017-11-15 20:29
categories: post
tags: front-end angularjs
---

# 扩展 angularjs 指令

转自 [angular.wiki: understanding directives](https://github.com/angular/angular.js/wiki/Understanding-Directives#)

当我们在使用第三方指令时，如果你希望扩展它，但又不要直接修改代码的话。有几种方式可以实现。

### 全局配置(global configuration)

一些设计良好的指令(比如 AngularUI 的指令)可以使用全局配置而不是为每个实例单独进行设置。

```
// AngularUI 的文档通过特别图例标明了指令的哪些配置属性是可以进行全局设置的
// 比如 uib-datepicker-popup 的 close-text 和 clear-text 就是可以通过全局配置进行设置
// 示例如下，

angular.module('ui.bootstrap.demo', ['ui.bootstrap']).config(function (datepickerConfig) {
  datepickerConfig.closeText = '确定';
  datepickerConfig.clearText = '取消';
});

// 对应的单独配置则是,

<input type="text" class="form-control" uib-datepicker-popup="{{format}}" 
  ng-model="dt" is-open="popup1.opened" datepicker-options="dateOptions" 
  ng-required="true" close-text="确定" clear-text="取消" 
  alt-input-formats="altInputFormats" />

```

### 依赖指令(require directives)

假定第一个指令已经应用了，我们可以在它之上再创建一个新指令。如果需要在新指令里使用另一个指令的方法，需要第一个指令通过 controller 暴露这个方法(但是需要向前者的开发者发起 pull request 或 feature request 来实现)。

```
// 为了保证被依赖的指令存在，需要在父级或同级 DOM 元素上使用这个指令
// 理论上保证 a 的定义在 b 之前执行也是可以的 —— 通过 requirejs 就能做到

// <div a b></div>
ui.directive('a', function() {
  return {
    controller: function() {
      this.data = {}
      this.changeData = function( ... ) { ... }
    },
    link: function($scope, $element, $attributes, controller) {
      controller.data = { ... }
    }
  }
})
myApp.directive('b', function() {
  return {
    require: 'a',
    link: function($scope, $element, $attributes, aController) {
      aController.changeData()
      aController.data = { ... }
    }
  }
})
```

### 指令栈(stacking directives)

我们可以使用已有指令的名字创建新指令。此时两个指令都会生效。这种情况下，可以用指令优先级来控制哪一个指令先生效(因为通常我们不会特意设置指令优先级，所以这种情况下可能以意味着又需要提 pull request 了)。

```
// <div a></div>
ui.directive('a', ... )
myApp.directive('a', ... )
```

### 模板(templating)

我们可以使用 <ng-include> 创建新指令来包裹住已有指令——或者，直接写一段使用了原指令的 HTML。

```
// <div b></div>
ui.directive('a', ... )
myApp.directive('b', function(){
  return {
    template: '<div a="someOptions"></div>'
  }
})
```

### 修改指令配置(specialized the directive configuration)

有时候我们可能会希望早不干扰原有指令的前提下创建一个新的指令——起个新名字，或者修改一些指令配置（比如 templateUrl）。我们可以在定义新指令时通过指令名获取原指令的定义方法。修改后用于新指令的定义。

注意：定义指令时，会把他注册为 指令名 + 'Directive' 后缀的服务。比如 <my-dir> 就是 myDirDirective。当我们在新指令的构造函数里注入这个指令时，就可以利用它得到指令配置。指令配置会是一个数组（未测试：应该是使用同名多次注册一个指令时才会有多笔。排序可能和指令优先级有关）。只要选择第一笔（假定），做一个浅拷贝，修改后再返回就可以了。

```
// <div b></div>
ui.directive('a', ... )
myApp.directive('b', function(aDirective){
  return angular.extend({}, aDirective[0], { templateUrl: 'newTemplate.html' });
})
```

实际上，上面的做法理论上可以修改原指令构造函数的任意内容。  

 + 通过复制原指令的 controller 并修改（参考依赖指令）——推荐。  
 + 若原指令只使用了 link，依然能进行修改，

```
ui.directive('a', ... )
myApp.directive('b', function(aDirective){
  var directive =  _.cloneDeep(aDirective[0]);
  function newLinkFn() {
  ...
  }
  var directive.link = newLinkFn;
   return directive;
})
```

**大误**
```
ui.directive('a', ... )
myApp.directive('b', function(aDirective){
  var directive =  _.cloneDeep(aDirective[0]);
  var newLinkFn = function() {};
  var strLinkFn = 'var newLinkFn = ' + directive.link.toString();
  ...
  eval(strLinkFn);
  var directive.link = newLinkFn;
   return directive;
})
```


### 相关资源

[细说ES7 JavaScript Decorators](http://www.cnblogs.com/whitewolf/p/details-of-ES7-JavaScript-Decorators.html)

[Angular 2 Decorators(装饰器)](https://github.com/semlinker/angular2-ionic2/issues/9)

[Decorating AngularJS Services](http://odetocode.com/blogs/scott/archive/2013/06/06/decorating-angularjs-services.aspx)

[Overriding Core And Custom Services In AngularJ](https://www.bennadel.com/blog/2927-overriding-core-and-custom-services-in-angularjs.htm)

### overriding angular controller

``` javascript
.controller('childController', function ($scope, $controller) {
    'use strict';
    $controller('parentController', {$scope: $scope});
    $scope.someFunction=function(){}
})
```

### overriding angular directive

### 参考资料

[1] [使用Chrome DevTools的Timeline分析页面性能](http://horve.github.io/2015/10/26/timeline/)   
[2] [网页性能管理详解](http://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html)   
[3] [404 Large Data Sets and Performance](http://ui-grid.info/docs/#/tutorial/404_large_data_sets_and_performance)   
[4] []()   
[5] []()   
[6] []()   