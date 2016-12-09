---
layout: post
title:  "express和webpack配合时启动文件配置"
date:   2016-12-09 12:16
categories: js express webpack
---

* 项目结构：react-redux-webpack-express-antui

* client中存放react代码，分为src和build两个目录

* server中存放express代码，分为src和build两个目录

* client里的ajax请求包装形式为：/request.ajax?path=xxx<!--more-->

## client里请求路径映射到server里的文件：

    import _ from 'lodash';
    import fs from 'fs';
    import express from 'express';
    import xml2js from 'xml2js';
    import path from 'path';
    import fetch from 'isomorphic-fetch';

    import {SERVERCONFIG} from '../config';
    const {nodeEnv, isMock, apiDir} = SERVERCONFIG;
    const serverPath = '../' + apiDir.replace(/[\\\/]?$/, '/');
    let router = express.Router();

    function getLocalApi(req, res, apiPath, params) {
        // 获取文件路径
        let file = path.resolve(__dirname, serverPath + apiPath + (isMock ? '-mock.js' : '.js'));

        // 判断文件是否存在，若文件不存在返回错误
        let isExist = fs.existsSync(file);
        if (!isExist) {
            return {errorCode: 500, message: 'error in server: invalid API path.', data: null};
        }

        // 如果文件存在，返回请求到的数据
        let result = require(file);
        let resp = _.isPlainObject(result)
            ? new Promise(function (resolve) { resolve(result); })
            : result(req, res, apiPath, params);
        return resp;
    }

    router.all('/request.ajax*', function (req, res, next) {
        // 解析出path
        let body = req.body;
        let path = body.path || req.query.path;
        if (!path) {
            return res.json({errorCode: 500, message: 'error in server: empty API path.', data: null});
        }

        // 解析出参数
        let params = typeof body.params === 'string' ? JSON.parse(body.params) : body.params;

        // 获取对应文件的数据
        getLocalApi(req, res, path, params).then(function (result) {
            console.log('=================response result================');
            console.log(result);
            res.json(result);
        });

        // 设置超时
        // 设置所有HTTP请求的超时时间
        req.setTimeout(20000);
        // 设置所有HTTP请求的服务器响应超时时间
        res.setTimeout(20000);
    });

    export default router;


## express 启动文件配置

express作为server时，webpack只需要负责编译就好了，辣么，单页应用中必然会出现刷新404的情形，怎么破？

    /**
     * @file express server
     * @author hancong05@baidu.com
     * @date 2016-10-29
     */

    'use strict';

    // 非production环境里可以使用express-session来存储用户名等信息，但生产环境下不允许，可以用cookie-session或者Redis之类的方案代替它
    import cookieSession from 'cookie-session';
    import express from 'express';
    import path from 'path';
    // express接收到浏览器里传过来的的req.body不能直接拿来用，需要解析成正常的格式
    import bodyParser from 'body-parser';
    // 专业解决单页应用刷新问题，类似webpack-history-api-callback什么的
    import fallback from 'express-history-api-fallback';

    // 登录
    import loginFilter from './router/loginFilter';
    // 退出登录
    import logout from './router/logout';
    // 这是一个前端请求map后端js文件的方法，按照路径映射到express server层下对应的文件里
    import apiMap from './router/apiMap';

    import {SERVERCONFIG} from './config';
    const {port, nodeEnv, postLimit} = SERVERCONFIG;

    let app = express();

    // use cookieSession
    app.use(cookieSession({
        name: 'posterlab',
        keys: ['userName'],
        // 30 days
        maxAge: 30 * 24 * 60 * 60 * 1000
    }));

    // login router
    app.use(loginFilter);
    app.use(logout);
    // static files
    let root = path.resolve(__dirname, '../../client/build/');
    app.use(express.static(root));

    // SPA refresh 404 resolution
    app.use(fallback('index.html', {root}));


    // map request path to api file
    app.use(bodyParser.urlencoded({limit: `${postLimit}MB`, extended: true}));
    app.use(bodyParser.json({limit: `${postLimit}MB`}));
    app.use(apiMap);

    // server start
    if (!module.parent) {
        app.listen(port, function () {
            console.log('\n Express app listening on port ' + port + '.');
        });
    }

## SERVERCONFIG：

    module.exports = {
        // app server启动配置
        SERVERCONFIG: {
            apiDir: 'api',
            // 后端接口host
            apiBackEndHost: {
                product: 'http://10.94.245.18:8888/ceres/launch/api',
                promote: 'http://10.94.245.18:8889/ceres/show/api',
                preview: 'http://10.94.245.18:8889/ceres/show/api'
            },
            // 是否mock，true时请求${path}-mock.js
            isMock: 0,
            // app port
            port: process.env.NODE_ENV === 'production' ? 8080 : 8886,
            // 开发／生产环境
            nodeEnv: process.env.NODE_ENV || 'dev',
            // 单位：MB
            postLimit: 5
        },
        // UUAP配置
        UUAPCONFIG: {
            production: {
                protocol: 'https',
                hostname: 'uuap.baidu.com',
                validateMethod: '/serviceValidate',
                pathname: '/login',
                logoutMethod: '/logout',
                uicUrl: 'http://uuap.baidu.com:8086/ws/',
                appKey: 'uuapclient-5-1E1FF0L10eb2TthZmAcc'
            },
            dev: {
                protocol: 'http',
                hostname: 'itebeta.baidu.com',
                port: '8100',
                pathname: '/login',
                validateMethod: '/serviceValidate',
                logoutMethod: '/logout',
                uicUrl: 'http://itebeta.baidu.com:8102/ws/',
                appKey: 'UICWSTestKey'
            },
            API: {
                UserRemoteService: 'UserRemoteService?wsdl',
                EmailGroupRemoteService: 'EmailgroupRemoteService?wsdl'
            }
        }
    };

## .babelrc

    {
        "presets": ["es2015", "react", "stage-0"],
        "plugins": [["import", {
            "style": false,
            "libraryDirectory": "lib",
            "libraryName": "antd",
            "camel2DashComponentName": true,
        }]]
    }

## 最外层package.json命令：

    "scripts": {
        // 服务启动
        "start": "nodemon server/src/app.js --ignore client/ --exec babel-node",
        // client里编译
        "build": "webpack --watch --config client/webpack-dev-server.config.js",
        // client里发布
        "release": "webpack --watch --config client/webpack-production.config.js",
        // server里编译
        "buildServer": "babel server/src -d server/build"
    },
