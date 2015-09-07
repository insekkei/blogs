---
layout: post
title:  "Javascript stop bubble"
date:   2015-06-16 11:20
categories: js common
---

# js阻止冒泡方法：

<pre><code>var stopBubble = function (e) {
	//如果提供了事件对象，则这是一个非IE浏览器
	if(e && e.stopPropagation) {
		//因此它支持W3C的stopPropagation()方法<!--more-->
		e.stopPropagation();
	} else {
		//否则，我们需要使用IE的方式来取消事件冒泡 
		window.event.cancelBubble = true;
	}
	return false;
};
</code></pre>

*用法：*
<pre><code>$(...).click(function(e){
	stopBubble(e);
});
</code></pre>