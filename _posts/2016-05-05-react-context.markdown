---
layout: post
title:  "react context传值"
date:   2016-05-05 17:40
categories: react-native
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


