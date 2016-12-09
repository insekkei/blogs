---
layout: post
title:  "charles获取express里的node请求..."
date:   2016-12-09 12:16
categories: js express charles
---

node作为代理层请求后端接口时，一般会封装一个公共的请求方法，配置好host等信息，统一处理后端异常情况。

这样的话，浏览器里就不能看到真正的api请求地址和原始的返回数据了，而在terminal里打印信息时，往往会被多重请求看得眼花缭乱...

因此需要charles抓取node里的请求和返回。<!--more-->

express里写个fetch方法：

    import fetch from 'isomorphic-fetch';
    import tunnel from 'tunnel';
    import {SERVERCONFIG} from '../config';
    const nodeEnv = process.env.NODE_ENV || 'dev';
    ...
    module.exports = {
        fetchData(req, res, apiPath, params, host) {
            let ext = {};
            params = params || {};

            if (_.isPlainObject(params)) {
                ext = {
                    method: 'POST',
                    credentials: 'same-origin',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(params)
                };
                // 设置本地调试代理，端口8888
                if (nodeEnv === 'local') {
                    ext.agent = tunnel.httpOverHttp({
                        proxy: {
                            host: 'localhost',
                            port: 8888
                        }
                    });
                }
            }

            let apiBackEndHost = SERVERCONFIG.apiBackEndHost[host];

            return fetch(apiBackEndHost + apiPath, Object.assign({}, ext)).then(response => {
                return response.json();
            }).catch(error => {
                // 统一设置错误处理
                return Promise.resolve({
                    status: 505,
                    msg: '',
                    body: { // 字段没确定
                        error: error
                    }
                });
            });
        }
    };

开启charles，配置httpProxy为8888，执行：

NODE_ENV=local npm start

就可以在charles中看到真正的后端请求和返回了～

开了shadowSocks这架小飞机怎么办？别怕，SwitchyOmega里配置Protocol为SOCKS5，Server 127.0.0.1，port 1080，
然后在charles的Proxies里选中Enable Socks proxy，设置端口1080.

备注：此时可以前往mac->system preference->network->advanced->Proxies中查看web Proxy什么的，是不是被设置了什么端口～～
