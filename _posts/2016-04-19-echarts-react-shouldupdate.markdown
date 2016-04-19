---
layout: post
title:  "echarts多图时页面resize事件以及重绘需要判断"
date:   2016-04-19 14:20
categories: react js echarts
---

# resize事件

当页面上有>1个图表时，

    window.onresize = myChart.resize;

并不能使所有的图响应浏览器resize事件，只有一个随机的图能随着resize。

于是参考[DOM Event Listeners in a Component](https://facebook.github.io/react/tips/dom-event-listeners.html)里所写，添加了resize事件：<!--more-->

    ...
    getInitialState() {
        let innerWidth = window.innerWidth;
        let acWidth;
        if (innerWidth > 991) {
            acWidth = (innerWidth - 144) / 3;
        }
        else {
            acWidth = innerWidth - 60;
        }
        return {
            width: acWidth
        };
    },
    handleResize() {
        let innerWidth = window.innerWidth;
        let acWidth;
        if (innerWidth > 991) {
            acWidth = (innerWidth - 144) / 3;
        }
        else {
            acWidth = innerWidth - 60;
        }
        this.setState({
            width: acWidth
        });
    },
    componentDidMount() {
        window.addEventListener('resize', this.handleResize);
    },
    componentWillUnmount() {
        window.removeEventListener('resize', this.handleResize);
    },
    render() {
        const { width } = this.state;
        let option = {
            // 根据echarts配置项手册配置option
        }
        return <Chart data-source={option} height="300" width={width}/>;
    }

这样，当浏览器窗口缩放时，所有图都会随着重新渲染。但是出现了又一个问题：

# 判断是否需要重新渲染

很多时候的重新渲染都是不必要的，如果开启了echarts的animation，就可以看到，页面中其它组件的状态变化也会使echarts重新渲染一遍。

此时需要`shouldComponentUpdate`来限制重绘：

Chart组件中添加以下代码：

    shouldComponentUpdate: function(nextProps, nextState) {
        let renderFlag = true;

        if ((this.props.width === nextProps.width)
            && (JSON.stringify(nextProps['data-source']) === JSON.stringify(this.props['data-source']))) {
            renderFlag = false;
        }
        return renderFlag;
    }

`shouldComponentUpdate`的方法返回false时，不重绘。如此，当宽度和数据都没有变化的时候，图将不会重新渲染；页面上图比较多的时候，这样做非常有必要。

[更多关于React-router](https://github.com/reactjs/react-router)