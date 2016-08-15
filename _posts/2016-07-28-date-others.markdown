---
layout: post
title:  "js中的Date另类使用"
date:   2016-07-28 14:02
categories: js Date
---

## 如何获得某个月的天数？

方法一

    const EVERY_MONTH_DAYS = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];

    function getDays(year, month) {
        if (month === 1 && isLeap(year)) return 29;
        return EVERY_MONTH_DAYS[month];
    }

<!--more-->方法二

我们发现，new Date()的第三个参数是可以大于我们所知的每个月的最后一天的的，比如：

    new Date(2016, 0, 200) //Mon Jul 18 2016 00:00:00 GMT+0800 (CST)

于是

    function getDays(year, month) {
        if (month === 1) return new Date(year, month, 29).getMonth() === 1 ? 29 : 28;
        return new Date(year, month, 31).getMonth() === month ? 31 : 30;
    }

方法三

new Date()的第三个参数传小于1的值会怎么样了，比如传0，我们就获得了上个月的最后一天。

    new Date(2016, 0, -200) //Sun Jun 14 2015 00:00:00 GMT+0800 (CST)

于是

    function getDays(year, month) {
        return new Date(year, month + 1, 0).getDate();
    }


## JavaScript 显示 Y-m-d H:i:s 的日期时间格式

方法一

    let date = new Date();
    let result = [
        [
            date.getFullYear(),
            date.getMonth() + 1,
            date.getDate()
        ].join('-'),
        [
            date.getHours(),
            date.getMinutes(),
            date.getSeconds()
        ].join(':')
    ].join(' ').replace(/\b\d\b/g, '0$&');

方法二
    
    new Date().toLocaleString('zh-CN', { hour12: false }) // "2016/8/15 15:41:44"
    new Date().toLocaleString('zh-CN', { hour12: true }) // "2016/8/15 下午3:42:06"

    var date = new Date();
    var result = date.toLocaleString('zh-CN', { hour12: false })
        .replace(/\//g, '-').replace(/\b\d\b/g, '0$&');
