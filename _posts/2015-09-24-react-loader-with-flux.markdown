---
layout: post
title:  "flux中使用react-loder"
date:   2015-09-24 15:07
categories: react loader
---

> 做一个react的项目，需要loading菊花的效果，找到了[react-loader](https://github.com/quickleft/react-loader)，然而README中只有一些不明觉厉的伪代码...

经过多番尝试，终于做出来了一款flux的小菊花...三木君说，这种方法藕合度挺高的。然而我目前想不到更好（kuai，四声）<!--more-->的方法来对付手头的flux项目了...如果谁有思路，请[务必告知](mailto:hancong9104@163.com)，不胜感激。

In the view:

    <Loader loaded={!this.state.data.isLoading} ... />

When send a xhr request:

    ...
    componentDidMount: function () {
        sampleStore.addChangeListener(this._change);

        if (this.isMounted()) {
            // 发请求之前都先setIsLoading
            SampleActions.setIsLoading();
            SampleActions.loadData(); 
        }
    },
    ..

And in SampleActions.js:

    ...
    setIsLoading: function (params) {
        PlatformDispatcher.dispatch({
            actionType: 'order_statistic_set_isLoading',
            item: params
        });
    },
    ...

Then, in sampleStore.js:

    PlatformDispatcher.register(function (payload) {
        var actionType = payload.actionType;

        switch (actionType) {
            ...
            case 'order_statistic_set_isLoading':
              orderStatisticStore.setIsLoading(payload.item);
              break;
            ...
        }
    });

    OrderStatisticStore.prototype = assign({}, EventEmitter.prototype, {
        setIsLoading: function (params) {
            // when send a xhr request, change state
            this.data.isLoading = true;
            this.emit('change');
        },
        loadData: function (params, callback) {
            var params = params || {};

            // send request
            PlatformRequest.getQuestHandle({
                url: ...,
                data: {
                    ...
                }
            }, function (res) {
                // when success, change the state of isLoading
                this.data.isLoading = false;
                this.data... = res;
                this.emitChange();
                callback && callback();
            }.bind(this));
        },
        ...
    });

如果再配合bootstrap-react的loading button state，也是美呆了...