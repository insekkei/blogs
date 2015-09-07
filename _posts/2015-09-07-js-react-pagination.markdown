---
layout: post
title:  "用react完成一个带省略号的分页"
date:   2015-09-07 19:18
categories: react js
---

分页其实想起来很容易：不过就是从后台拿一些数据来——当前页码、总页数、总条目数、每页显示的数量等等。

本篇将完成一个具备以下简单功能的react组件（结合Flux）：

-页码list

-首页、末页、上下页

-页码数量过多时用省略号指引<!--more-->

# 组件将被这样使用：

    <Pagination currentPn={currentPn}
            pageCount={pageCount}
            pageSize={pageSize}
            dataSize={dataSize}
            clickHandler={this.changePn}
            prevHandler={this.prevPn}
            nextHandler={this.nextPn}
            firstHandler={this.firstPn}
            lastHandler={this.lastPn}
    >
    </Pagination>

点击每一个list中的页码链接，执行`this.changePn`方法：

    changePn: function (e) {
        e.preventDefault();
        ExampleActions.setPageNumber({
            pageNumber: e.target.innerHTML
        });
    }

上下页翻页方法：

    prevPn: function (e) {
        e.preventDefault();
        ExampleActions.setPageNumber({
            pageNumber: currentPn － 1
        });
    },
    nextPn: function (e, currentPn) {
        e.preventDefault();
        ExampleActions.setPageNumber({
            pageNumber: currentPn ＋ 1
        });
    }

首页和末页方法：

    firstPn: function (e) {
        e.preventDefault();
        ExampleActions.setPageNumber({
            pageNumber: 1
        });
    },
    lastPn: function (e, currentPn) {
        e.preventDefault();
        ExampleActions.setPageNumber({
            pageNumber: pageCount
        });
    }

上述五个方法执行后，通过action调用dispatcher，dispatcher调用store，store中loadData，重新render组件这个过程更新页面中展示的数据。

# 组件将被这样实现：

定义变量：

    var pageList = '';
    var pageListArr = [];
    var prevPage = '';
    var nextPage = '';
    var firstPage = '';
    var lastPage = '';
    var thisProps = this.props;

展示页码list并不难，取到上面使用组件时传递过来的属性，将pageCount肢解，遍历，加入标签组装就可以完成这个任务：

    for (var i = 0; i < thisProps.pageCount; i++) {
        pageListArr.push(i);
    };
    pageList = pageListArr.map(function (item) {
        // 判断是否当前页
        if (item === thisProps.currentPn) {
            return (
                <li className="active"><a href="" onClick={thisProps.clickHandler}>
                    {item}</a>
                </li>
            );
        }
        else {
            return (
                <li><a href="" onClick={thisProps.clickHandler}>
                    {item}</a>
                </li>
            );
        }
    });

上下页翻页链接，要判断是否可点击，首页和末页类似，但不需要判断状态：

    // 上一页
    prevPage = (function () {
        if (thisProps.currentPn === 1) {
            return (
                <li className="disabled">
                  <span>
                    <span aria-hidden="true">上一页</span>
                  </span>
                </li>
            );
        }
        else {
            return (
                <li>
                  <a href="#" aria-label="Previous" onClick={thisProps.prevHandler}>
                    <span aria-hidden="true">上一页</span>
                  </a>
                </li>
            );
        }
    })();

    // 下一页
    nextPage = (function () {
        if (thisProps.currentPn === thisProps.pageCount) {
            return (
                <li className="disabled">
                  <span>
                    <span aria-hidden="true">下一页</span>
                  </span>
                </li>
            );
        }
        else {
            return (
                <li>
                  <a href="#" aria-label="Next" onClick={thisProps.nextHandler}>
                    <span aria-hidden="true">下一页</span>
                  </a>
                </li>
            );
        }
    })();

组装：

    <ul className="pagination pull-right">
        {firstPage}
        {prevPage}
        {pageList}
        {nextPage}
        {lastPage}
    </ul>

# 显示省略号

考虑四种情况：

- 左边右边都不需要省略号

- 左边有省略号，右边没有

- 左边没有省略号，右边有

- 左右两边都需要省略号

再考虑：

- “首页”和“末页”可以用“1”“100（假设是最后一页的页码）”放在list中代替，因为不管有没有省略号，1和100始终都要显示出来。

两种实现思路：

1、四种不同的情况下，直接拼装包含`...`的html代码

2、四种不同的情况下，先拼装pageListArr这个变量，再遍历生成html代码

考虑到react中store影响view这个特性，我决定采用第二种方法（我不会告诉你其实我先尝试了第一种方法，然后拼装变量和字符串快要崩溃...jsx都不完全懂，还是默默地去操作变量吧...何况这样真的比较清爽...）

添加变量：

    // 假设当前页码总在中间显示，pageTab就是当前页码到省略号之间的数量，比如1...4,5,6,7,8...100，pageTab就是2
    var pageTab = 4;

    // 左边的省略号，在第1页之后
    var leftEllipse = '';

    // 右边的省略号，在第100页之前
    var rightEllipse = '';

总页数小于11，全部显示（这里可以做成可配置的）：

    if (thisProps.pageCount < 11) {

        // 注意pageListArr要先清空
        pageListArr = [];

        // 从第二页开始显示，到倒数第二页
        for (var i = 1; i < thisProps.pageCount-1; i++) {
            pageListArr.push(i);
        };

        //左右都没有省略号
        leftEllipse = '';
        rightEllipse = '';
    }

总页码数大于11时，有`...`出现：

    if (thisProps.pageCount > 10) {
        // 当前页减去pageTab小于1，右边显示省略号，左边不显示
        if (thisProps.currentPn - pageTab < 1) {

            pageListArr = [];

            for (var i = 1; i < 10; i++) {
                pageListArr.push(i);
            };

            leftEllipse = '';
            rightEllipse = <li><span>...</span></li>;
        }

        // 当前页减去pageTab大于1 且 当前页加上pageTab小于总页数，两边都显示省略号
        if ((thisProps.currentPn - pageTab > 1) && ((thisProps.currentPn + pageTab) < thisProps.pageCount)) {

            pageListArr = [];

            for (var i = thisProps.currentPn - pageTab; i < thisProps.currentPn + pageTab; i++) {
                pageListArr.push(i);
            };

            leftEllipse = <li><span>...</span></li>;
            rightEllipse = <li><span>...</span></li>
        }

        // 当前页码加上pageTab大于总页数，右边不显示省略号，左边显示
        if ((thisProps.currentPn + pageTab) > (thisProps.pageCount)) {

            pageListArr = [];

            for (var i = thisProps.pageCount - (pageTab*2 + 1); i < thisProps.pageCount; i++) {
                pageListArr.push(i);
            };

            leftEllipse = <li><span>...</span></li>;
            rightEllipse = '';
        }
    }

首页和末页：

    // 首页
    firstPage = (function () {
        if (thisProps.currentPn === 1) {
            return (
                <li className="active"><a href="" onClick={thisProps.firstHandler}>
                    1</a>
                </li>
            );
        }
        else {
            return (
                <li><a href="" onClick={thisProps.firstHandler}>
                    1</a>
                </li>
            );
        }
    })();

    // 末页
    lastPage = (function () {
        if (thisProps.currentPn === thisProps.pageCount) {
            return (
                <li className="active"><a href="" onClick={thisProps.lastHandler}>
                    {thisProps.pageCount}</a>
                </li>
            );
        }
        else {
            return (
                <li><a href="" onClick={thisProps.lastHandler}>
                    {thisProps.pageCount}</a>
                </li>
            );
        }
    })();

装入`ul`：

    <ul className="pagination pull-right">
        {prevPage}
        {firstPage}
        {leftEllipse}
        {pageList}
        {rightEllipse}
        {lastPage}
        {nextPage}
    </ul>

总的组织结构：

    var Pagination = React.createClass({
        render: function () {
            ...算法／过程...
            return (
                <ul>
                    ...组装结果...
                </ul>
            );
        }
    });

备注：分页样式使用bootstrap。