---
layout: post
title:  "react router使用"
date:   2016-02-19 11:40
categories: react js
---

> React Router keeps your UI in sync with the URL. It has a simple API with powerful features like lazy code loading, dynamic route matching, and location transition handling built right in. 

## 使用npm安装<!--more-->

    $ npm install --save react-router

## 在Layout中使用RouteHandler

> A Layout is something that describes an entire page structure, such as a fixed navigation, viewport, sidebar, and footer.

    var React = require('react');
    var RouteHandler = require('react-router').RouteHandler;
    var Link = require('react-router').Link;

    ...

    var Layout = React.createClass({
        ...
        render: function () {
            return (
                <div className="wrapper">
                    <PlatformHeader/>
                    <PlatformNav/>
                    <PlatformContent/>
                </div>
            );
        }
    });

    // RouteHandler是react router的核心组件之一，代表与当前路由（url）匹配的组件。
    var PlatformContent = React.createClass({
        render: function() {
            return (
                <div className='main'>
                    <RouteHandler/>
                </div>
            );
        }
    });

    // PlatformNav中有url链接（路由需求），例如index
    var PlatformNav = React.createClass({
        render: function() {
            return (
                <div className='nav'>
                    ...
                    <Link to="index">首页</Link>
                    ...
                </div>
            );
        }
    });

    ...
    module.exports = Layout;



## index页

    var Index = React.createClass({
        render: function() {
            return (
                <div>
                    Welcome!
                </div>
            );
        }
    });

    module.exports = Index;

## 在app.js中配置路由

    var Layout = require('./components/Layout');
    var Index = require('./components/Index');

    // 引入需要的组件
    var Router = require('react-router').Router;
    var Route  = require('react-router').Route;
    var DefaultRoute = require('react-router').DefaultRoute; // 默认路由，首页

    // 或使用es6
    import { Router, Route, DefaultRoute } from 'react-router'

    // 设置路由，handler中是组件名称，path写url
    var routes = (
        <Route>
            <Route path="/" handler={Layout}>
                <DefaultRoute handler={Index}/>
                <Route path='index' name='index' handler={Index}/>
            </Route>
        </Route>
    );

    Router.run(routes, Router.HistoryLocation, (Root) => {
        React.render(<Root/>, document.body);
    });
    
这样，点击nav中的“首页”链接时，浏览器地址栏中会出现'index'，同时页面上能看到“Welcome”。而刷新页面时，也能根据给当前url分配的组件Index，加载想要看到的内容。

[更多关于React-router](https://github.com/reactjs/react-router)