---
layout: post
title:  "Javascript反序输出字符串"
date:   2015-06-16 11:49
categories: js function
---

这是一个我在至少4次web前端笔试或者面试中遇到过的题目，就这么一行字的题目，最近一次看到却还让我如遭五雷轰顶，罪过，罪过。 

*解决方案一：*

<pre><code> >'abcd ed'.split('').reverse().join(''); 
<-"de dcba"
</code></pre>
如果写成：
<pre><code> >'abcd ed'.split('').reverse().join(); 
</code></pre>
结果会是：
<pre><code> <"-d,e, ,d,c,b,a"
</code></pre>

注意``join(seperator)``方法是通过指定的字符连接数组所有元素，默认连接符是``.``。

*解决方案二：*

<pre><code> >"abcd ed".split('').sort(function(a,b){return 1;}).join('');
<-"de dcba"
</code></pre>

``sort(sortby)``方法，如果没有sortby参数，返回数组按照字母顺序，即字符编码顺序进行排列。现在，按照``sort()``方法的说明，return 1，也就是一个大于0的数，表示排序后的数组中a应该出现在b之后。

*解决方案三：*

<pre><code> >str='abcd ed';Array.prototype.map.call(str,function(it, i){return str[str.length-i-1];}).join('');
<-"de dcba"
</code></pre>

对@yfwz100的这个方法我表示五体投地地膜拜Orz。
map()方法会给数组中的每个元素都按顺序调用一次 callback 函数。比如：

<pre><code>>var numbers = [1, 4, 9]; 
>numbers.map(function(num) { return num * 2; });
<-[2, 8, 18]
</code></pre>

方案三使用了``array.map(callback[, thisArg])``，并且，字符串被当作数组来使用...，callback中第一个参数是数组中当前被传递的元素，这里即字符串中当前被传递的字符，第二个参数是当前被传递元素的索引，第三个参数是调用map方法的数组。
例1，获取每个字符串对应的ASCLL码： 

<pre><code>>Array.prototype.map.call("Hello World", function(x) { return x.charCodeAt(0); }) 
<-[72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100]
</code></pre>

通常情况下，map 方法中的 ``callback`` 函数只需要接受一个参数，就是正在被遍历的数组元素本身。但这并不意味着 ``map`` 只给 ``callback`` 传了一个参数，第三个参数callback会忽视，但第二个参数``index``会被使用。

例2，进制数转换：

<pre><code>>["1", "2", "3"].map(parseInt);
<-[1, NaN, NaN]
</code></pre>

例2中，``parseInt()``方法有两个参数，``element``和``进制数``，于是它将``index``视为了进制数。因此正确的方法应该是：

<pre><code>>function returnInt(element){ return parseInt(element,10); }
>["1", "2", "3"].map(returnInt);
<-[1, 2, 3]
</code></pre>
