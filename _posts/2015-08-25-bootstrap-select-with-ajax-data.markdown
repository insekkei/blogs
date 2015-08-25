---
layout: post
title:  "bootstrap-select.js使用ajax get到的option数据"
date:   2015-08-25 11:38
categories: bootstrap js
---

> 第一次用到，搜了很多博文都没用，一篇抄一篇而且整得特复杂...还是官方文档比较给力，然而最近红杏什么的都没法用了，代理jquery到国内cdn后才能正确显示效果，非常之坑...有些东西遮遮掩掩真的有用吗？

[bootstrap-select.js官方文档](http://silviomoreto.github.io/bootstrap-select/)

html：

<pre><code>&lt;select class="form-control selectpicker"  data-live-search="true"&gt;
	...
&lt;/select&gt;
</code></pre>

js部分：

<pre><code>$.ajax({
	// get请求地址
    	url: basePath,
    	dataType: "json",
    	success: function (data) {
	    	var optArr = [];
	        for (var i = 0; i < data.length; i++) {
	            $('.selectpicker').append("&lt;option value=" + data[i].userName + "&gt;" + data[i].userName + "&lt;/option&gt;");
	        }

	        // 缺一不可
	        $('#adeName').selectpicker('refresh');
	        $('#adeName').selectpicker('render');
    }
});
</code></pre>