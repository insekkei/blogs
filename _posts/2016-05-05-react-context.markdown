---
layout: post
title:  "react context传值"
date:   2016-05-05 17:40
categories: react context
---

用react-router的时候，希望给{this.props.children}这个组件传一个layout里的用户名...

Layout.js中：<!--more-->

    const Layout = React.createClass({
        childContextTypes: {
            username: React.PropTypes.string
        },
        getChildContext() {
            return {
                username: this.state.username
            };
        },
        ...

子组件中：

    const DateRange = React.createClass({
        contextTypes: {
            username: React.PropTypes.string
        },

        ...
        render() {
            const { username } = this.context;
            return ...
        }
        ...


[更多关于context](http://facebook.github.io/react/docs/context.html)
[更多propTypes](http://facebook.github.io/react/docs/reusable-components.html)
