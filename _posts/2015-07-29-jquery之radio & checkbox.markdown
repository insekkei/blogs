---
layout: post
title:  "jquery中操作radio和checkbox"
date:   2015-07-29 17:50
categories: jquery js
---

#jquery判断radio或者checkbox是否被选中

<pre><code>// 返回true or false
$('selector').is(':checked');
</code></pre>

#jquery操作radio或者checkbox使它们被选中

<pre><code>//For versions higher than 1.6, use:
$('#radio_1').prop('checked', true)
//For versions of jQuery prior to 1.6, use:
$('#radio_1').attr('checked', 'checked');
</code></pre>