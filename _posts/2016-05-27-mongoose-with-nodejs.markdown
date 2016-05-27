---
layout: post
title:  "nodejs中使用mongodb"
date:   2016-05-27 16:37
categories: nodejs mongo
---

需求背景：线上出问题时可以找出近期数据相关的操作历史，但日志只在内网访问。

于是可以把日志相关的数据保存在mongodb中。

yog2的框架中，后端文件结构如下：<!--more-->

    server | - action/api/xx.es
             - lib/dataFetch.es..
             - model | - schema/appfilterlog.es
                       - appfilterlog.es
             - config.es
             - router.es

首先定义表结构，model/schema/appfilterlog.es中：
    
    'use strict';

    import * as mongoose from 'mongoose';
    let logSchema = mongoose.Schema({
        time: Date,
        operate: String,
        operator: String,
        requestData: String
    });

    let appfilterlog = mongoose.model('appfilterlog', logSchema, 'Appfilterlog');

    export default appfilterlog;

存储数据的方法，在model/appfilterlog.es中，需要保存时间、操作（增加／删除／修改）、操作人、请求数据：

    'use strict';

    import appfilterlog from './schema/appfilterlog';

    export  async function newLog (operate, operator, requestData) {
        appfilterlog.create({
            time: new Date(),
            operate: operate,
            operator: operator,
            requestData: requestData
        });
    }

在action中调用增加／删除／修改的参数时，也调用newLog的方法，例如action/api/add.es中：

    import {newLog} from '../../../../../../model/appfilterlog';
    ...
    newLog('add', user.email, JSON.stringify(req.query));
    ...

