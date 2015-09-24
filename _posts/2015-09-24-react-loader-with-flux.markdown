---
layout: post
title:  "flux中使用react-loder"
date:   2015-09-24 15:07
categories: react loader
---

> 做一个react的项目，需要loading菊花的效果，找到了[react-loader](https://github.com/quickleft/react-loader)

三木君说，这种方法藕合度挺高的。然而我目前想不到更好（kuai，四声）的方法来对付手头的flux项目了...如果谁有思路，请[务必告知](hancong9104@163.com)，不胜感激。

<!--more-->

In the view:

    <Loader loaded={!this.state.data.isLoading} ... />

When send a xhr request:

    ...
    componentDidMount: function () {
        sampleStore.addChangeListener(this._change);

        if (this.isMounted()) {

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

Then in sampleStore.js:

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