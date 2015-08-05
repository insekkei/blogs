---
layout: post
title:  "angularjs结合localstorage完成一个简单的备忘录"
date:   2015-08-05 14:59
categories: angularjs js
---

> 学习一种新的框架真的是痛并快乐着...而当确定了一个清晰的目标，向着这个目标努力时，成就感带来的快感将远大于茫然不解时的痛苦。

——>[预览效果](http://insekkei.com/totolist)

# 准备工作

用bower安装需要的js：

<pre><code>bower install angular
bower install angular-local-storage
</code></pre>

html中引用bower_components下对应的js文件：

<pre><code>&lt;script src="js/angular.min.js"&gt;&lt;/script&gt;
&lt;script src="js/angular-local-storage.js"&gt;&lt;/script&gt;</code></pre>

说明：npm也可以，直接下载相应js文件或者用cdn也是ok的...

# 通过[angularjs Directive](http://insekkei.com/blogs/angular/js/2015/07/09/angular.js-%E8%AF%AD%E6%B3%95%E6%B5%85%E6%9E%90.html)扩展index.html

## angular接管整个页面：

<pre><code>&lt;html ng-app="todoApp"&gt;
</code></pre>

## body中的数据由`TodoController`来控制：

<pre><code>&lt;body ng-controller="TodoController"&gt;
</code></pre>

## 添加备忘的form：

<pre><code>&lt;form ng-submit="add()"&gt;
	&lt;input placeholder="What you want to do..." type="text" ng-model="todoItem" name="totoItem"&gt;
	&lt;input type="submit" id="submit" value="add"&gt;
&lt;/form&gt;
</code></pre>

`form`提交表单时（`ng-submit`）执行`add()`方法，输入框的内容通过`ng-model`与`todoItem`关联，使用：

<pre><code>...$scope.todoItem...</code></pre>

## 添加备忘list

每条备忘中包含的信息：内容`content`、日期`creDate`、删除按钮。

<pre><code>&lt;ul&gt;
	&lt;li ng-repeat="todo in todoList"&gt;
		&lt;time&gt;{ {todo.creDate} }&lt;/time&gt;
		&lt;input type="text" ng-model="todo.content"&gt;
		&lt;span&gt;{ {todo.content} }&lt;/span&gt;
		&lt;input type="button" value="remove" ng-click="remove($index)"&gt;
	&lt;/li&gt;
&lt;/ul&gt;
</code></pre>

`ng-repeat`用来循环输出每一条备忘，`ng-model`指定当前`input`的值即备忘内容`content`，`ng-click`将`remove`按钮和`remove(index)`方法关联。

# todo.js

声明app

	var todoApp=angular.module('todoApp',[]);

定义controller

	todoApp.controller('TodoController', function($scope) {
		...
	});

定义todoList

	$scope.todoList = [];

add方法

	$scope.add = function() {
		// 如果输入框有值
        if ($scope.todoItem) {
        	// 定义一个空对象
            var todoInfo = {};
            // 对象的三个属性：时间为创建时时间
            todoInfo.creDate = new Date().getMonth() + '/' + new Date().getDate();
            todoInfo.content = $scope.todoItem;
            todoInfo.status = 'active';

            // 将todoInfo添加到todoList头部
            $scope.todoList.unshift(todoInfo);
        }
    }

remove方法

	// 删除index位置开始的1个数组元素
	$scope.remove = function(index) {
        $scope.todoList.splice(index, 1);
    }

# 改造todo.js，使用localstorage持久化

两种思路：

* 分别在`$scope.todoList`查询、添加、删除时同步操作`localstorage`；
* 监听`$scope.todoList`，当它改变时把`$scope.todoList`赋值给`localstorage`.

第一种方法明显麻烦很多，但当时不知道`$scope.$watch`可以做到监听...这里使用`$scope.$watch`：

添加localstorage模块

	var todoApp=angular.module('todoApp',['LocalStorageModule']);
	...

controller上传入服务

	todoApp.controller('TodoController', function($scope, localStorageService) {
	...

从localstorage中get数据，如果不为空，赋值给`$scope.todoList`

	var todoInStore = localStorageService.get('todoList');
	$scope.todoList = todoInStore || [];

监听$scope.todoList，当它改变时，使用localstorage的`set()`方法

	$scope.$watch('todoList', function () {
        localStorageService.set('todoList', $scope.todoList);
    }, true);

# over

这样就实现了一个简版的备忘录，只要缓存没有被清理，备忘会一直在。

另外angular local storage还提供了一些方法供开发者使用：

- remove(key)：匹配key删除localStorage条目

- clearAll()：删除全部localStorage

- isSupported：检测浏览器是否支持localStorage

- cookie：操作cookie的方法同localStorage，包含`set`、`get`、`remove`、`clearAll`。






