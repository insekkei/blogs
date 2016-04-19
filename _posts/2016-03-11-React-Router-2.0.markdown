---
layout: post
title:  "react router 2使用"
date:   2016-03-11 14:40
categories: react js router
---

> React Router keeps your UI in sync with the URL. It has a simple API with powerful features like lazy code loading, dynamic route matching, and location transition handling built right in. 

## 使用npm安装<!--more-->

    $ npm install --save react-router

## 在Layout中使用

> A Layout is something that describes an entire page structure, such as a fixed navigation, viewport, sidebar, and footer.

    import { Link } from 'react-router'

    ...

    const Layout = React.createClass({
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

    // this.props.children匹配当前加载的内容
    const PlatformContent = React.createClass({
        render: function() {
            return (
                <div className='main'>
                    {this.props.children}
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
    import { browserHistory, IndexRoute, Router, Route } from 'react-router';

    // 设置路由，component是组件名称，path写url
    const AppRoute = (
        <Router history={browserHistory}>
                <Route path="/namespace/" component={Layout}>
                    <IndexRoute component={Index}/> 
                    <Route path='index' component={Index}/>
                </Route>
        </Router>
    );

    ReactDOM.render(AppRoute, document.getElementById('xxx'));

[更多关于React-router](https://github.com/reactjs/react-router)