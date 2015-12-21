---
layout: post
title:  "jquery和dom中操作radio和checkbox"
date:   2015-07-29 17:50
categories: jquery js
---
> DOM对象才能使用DOM中的方法，jquery对象不可以使用dom中的方法。

DOM对象可以和jquery对象互相转换：

#jquery对象转dom对象：

	$(selector).get(index);
	// or
	$(selector)[index];

#dom对象转jquery对象：<!--more-->

	// dom对象
	var object = document.getElementById('object');
	// jquery对象
	var $object = $(object);

#jquery判断radio或者checkbox是否被选中，需要jquery类型的对象。

	// 返回true or false
	$('selector').is(':checked');

#DOM方式判断radio或者checkbox是否被选中，需要用dom类型的对象。

	// 返回true or false
	$('selector')[0].checked;
	// or
	$('selector').get(0).checked;

#jquery操作radio或者checkbox使它们被选中

	//For versions higher than 1.6, use:
	$('#radio_1').prop('checked', true)
	//For versions of jQuery prior to 1.6, use:
	$('#radio_1').attr('checked', 'checked');