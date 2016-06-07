---
layout: post
title:  "react和css实现一个高度自适应的textarea"
date:   2016-06-07 15:21
categories: js react css
---

textarea默认有滚动条，设置height: auto或者固定高度的话，书写体验又不够好；若使用div，设置contentEditable为true，兼容起来比较麻烦，而且这家伙复制粘贴时带着源格式，不好操作。
所以还是想办法使用textarea吧。<!--more-->

考虑文字换行等情况，这里使用 <pre> 符号，将textarea的值同步到 <pre> 中，通过这个看不见的 <pre> 盒子的高度自增长，撑高 autoHeight ，然后设置textarea绝对定位，top为0，覆盖<pre>的位置，且高度撑满 autoHeight （100%）。

js:

    ...
    getInitialState() {
        return {
            text: 'react和css实现一个高度自适应的textarea'
        }
    },
    changeText(e) {
        this.setState({
            text: e.target.value
        });
    },
    render() {
        return <div className="autoHeight">
            <pre>{this.state.text}</pre>
            <textarea onChange={this.changeText}></textarea>
        </div>;
    }
    ...

css:

    .autoHeight {
        position: relative;

        pre{
            display: block;
            visibility: hidden;
            min-height: 10em;
            padding: 6px 7px; /*需要与textarea保持一致*/
            white-space: pre-wrap;
            line-height: 1.7em;
        }

        textarea {
            resize: none;
            position: absolute;
            height: 100%;
            top: 0;
            line-height: 1.7em;
        }
    }

理论上，这样就可以实现目的了，但总有点瑕疵——pre和textarea的高度并非完全相等，于是，我将pre的visibility和textarea的top注释掉，对比两个在页面上的表现，发现：

* pre 每段话是不换行的，横向一撑到底，相当于设置了white-space: no-wrap；
* pre 每一段（即textarea中的return换行）的段落间距比较大（其实是line-height）；
* textarea 有预置的padding

于是改成：

    pre{
        display: block;
        visibility: hidden;
        min-height: 10em;
        padding: 6px 7px; /*需要与textarea保持一致*/
        white-space: pre-wrap; /*按照textarea中的内容，该换行换行*/
        line-height: 1.7em; /*需要与textarea保持一致*/
    }

    textarea {
        resize: none;
        position: absolute;
        height: 100%;
        top: 0;
        line-height: 1.7em;
    }

完美。