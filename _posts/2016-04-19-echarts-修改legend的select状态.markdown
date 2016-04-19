---
layout: post
title:  "echarts中需要保持legend的selected状态"
date:   2016-04-19 16:57
categories: react js echarts
---

# 背景：

图表随着时间“自动更新”时，假如deSelect了某一项数据，在下次数据来临、图刷新时，依然deSelect这个数据项。

# 思路：

在公共的Chart组件中：
<!--more-->

    ...
    getInitialState() {
        return {
            selected: {}
        }
    },
    componentDidUpdate() {
        const el = this.getDOMNode();
        const myChart = echarts.init(el);

        const { selected } = this.state;
        // 如果没有改变select状态，selected将为空
        const ifSelected = this.isEmpty(selected);

        if (ifSelected) {
            myChart.setOption(this.props['data-source']);
        }
        else {
            // 手动设置legend.selected
            this.props['data-source'].legend.selected = selected;
            myChart.setOption(this.props['data-source']);
        }

        const _this = this;
        // echarts添加事件监测
        myChart.on('legendselectchanged', function(obj) {
            var selected = obj.selected;
            var legend = obj.name;
            _this.setState({
                selected: selected
            });
        });
    },
    isEmpty(obj) {
        for (var name in obj) {
            return false;
        }
        return true;
    },
    ...

[更多事件API](http://echarts.baidu.com/api.html#events)
