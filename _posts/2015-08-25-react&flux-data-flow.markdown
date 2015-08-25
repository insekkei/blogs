---
layout: post
title:  "react&flux之BasicSelect组件实现"
date:   2015-08-25 14:03
categories: react js flux
---

应工作需求，用到flux，为了解react设计思想及其flux强调的数据流，查找了各种在线资料，终于可以开工了。然而编码的时候时常产生一种怀疑：我真的是一个前端？

# React

对于react，我是从[这里](http://facebook.github.io/react/docs/getting-started.html)开始了解的，然而这儿仅仅能了解到react的语法。

对于习惯了html＋css＋js文件的我来说，乍一看到react的代码，第一反应是：html写在js中？这不就是前端一直强调的“结构、样式、行为分离”思想的反面教材吗？不能再颠覆啊！我感到无所适从。

于是，我想应该先了解下react的设计思想，因此，我跳过了[Tudorial](http://facebook.github.io/react/docs/tutorial.html)，从[Thinking in React](http://facebook.github.io/react/docs/thinking-in-react.html)开始看起。

在[Thinking in React](http://facebook.github.io/react/docs/thinking-in-react.html)页面我看到了一个很实用的栗子，然后快速地抓住了使用react编码的两个关键字：拆分、组件复用。

更多关于“理解react设计思想”的文章，可以在许多知名博客中搜索到，文码并茂，看多了就理解了...

# flux

什么是flux？

> It's more of a pattern rather than a formal framework. 

与angular双向数据绑定不同，它的数据流向永远是单向的，因此带来的好处是：我不必思考当前数据应该传递给谁。

在flux中，view和controller是一体的，同时它增加的两层内容：dispatcher和stores，前者负责创建actions，然后将actions按照名称（`actionType`）分发给stores，stores中对数据进行修改，触发一些change相关的方法，将改动渲染到view层（一般先是最高层级的组件收到新的数据，再把这些数据传递到子组件中）。简单说就是：

Action -> Dispatcher -> Store -> View...

实战总是比理论让人理解得更加深刻，这是当年学习依赖注入、控制翻转时得到的结论。于是在写了几个查询数据的模块之后，将一点心得在这里记录下来，方便以后查看。

# 一个select组件实例

这里将通过一个select组件的实现过程描述flux的数据流。

## 静态数据：config.js

<pre><code>terminalArr: [
    {
        key: 'x',
        val: '0'
    },
    {
        key: 'xx',
        val: '1'
    },
    {
        key: 'xxx',
        val: '2'
    }
]
</code></pre>

## Example.js

顶层view组件，监测stores变化，并将新的queryObj传递给组件的formData属性，由顶级组件向下分发数据。

<pre></code>var Example = React.createClass({
	
    componentDidMount: function () {
        exampleStore.addChangeListener(this._change);

        if (this.isMounted()) {
            exampleStore.loadData();
        }
    },
    componentWillUnmount: function () {
        exampleStore.removeChangeListener(this._change);
    },
    _change: function () {
        this.setState(exampleStore.queryObj);
    },
    render: function () {
        return (
        	&lt;div&gt;
        		...
                &lt;ExampleForm formData={this.state.queryObj}/&gt;
                ...
            &lt;/div&gt;
        );
    }
});
</code></pre>

中间组件，从formData中取出特定数据，传递给再下游的组件作为属性（props）值。

<pre><code>var ExampleForm = React.createClass({
    render: function () {
        var formData = this.props.formData;
        return (
            &lt;form&gt;
                ...
                &lt;TerminalsSelect terminal={formData.terminal}&gt;&lt;/Terminals&gt;
                ...
            &lt;/form&gt;
        );
    }
});
</code></pre>

继续向下，当select change时，调用action中的方法，其中select使用了已经定义好的BasicSelect组件，因此需要传递一些参数。

<pre><code>var TerminalsSelect =  React.createClass({
    changeTerminalHandler: function (e) {
    	// view -> action
        ExampleActions.setTerminal({
            terminal: e.target.value
        });
    },
    render: function () {
        return (
            &lt;div className="form-group"&gt;
                &lt;label htmlFor="terminal" className="form-control-static"&gt;Terminal：&lt;/label&gt;
                &lt;BasicSelect
                    &lt;!-- option数据 --&gt;
                    optionData={terminalArr}
                    &lt;!-- 引用的config里的数据的字段名（不是所有数据都是val,key） --&gt;
                    optKey="key"
                    optVal="val"
                    &lt;!-- 将上层组件获取到的formData.terminal传递给BasicSelect --&gt;
                    defaultVal={this.props.terminal}
                    &lt;!-- select name --&gt;
                    nameText="terminal"
                    &lt;!-- change方法 --&gt;
                    changeHandler={this.changeTerminalHandler}
                &gt;&lt;/BasicSelect&gt;
            &lt;/div&gt;
        );
    }
});

</code></pre>

最底层的组件：为了能够最大程度复用，所有文字内容都避免写死而是以变量（参考上一段代码注释）传入。

<pre><code>var BasicSelect = React.createClass({
render: function () {
        // option内容
        var optionContent;
        // option html字段
        var optKey = this.props.optKey;
        // option value字段
        var optVal = this.props.optVal;

        // 遍历optionData的值（此处是terminalArr），返回所有option html代码
        optionContent = this.props.optionData.map(function (item) {
            return (
                &lt;option value={item[optVal]}&gt;{item[optKey]}&lt;/option&gt;
            );
        });

        // select
        return (
            &lt;select 
                className="form-control"
                defaultValue={this.props.defaultVal}
                name={this.props.nameText}
                onChange={this.props.changeHandler}
            &gt;
                &lt;option value=""&gt;全部&lt;/option&gt;
                {optionContent}
            &lt;/select&gt;
        );
    }
});
</code></pre>

## ExampleActions.js

Dispatcher创建一个action。

<pre><code>setTerminal: function (params) {
    // action -> dispatcher
    PlatformDispatcher.dispatch({
        actionType: 'province_flow_set_terminal',
        item: params
    });
}
</code></pre>

## ExampleStore.js

处理actionType，调用具体方法。

<pre><code>PlatformDispatcher.register(function (payload) {
    var actionType = payload.actionType;

    switch (actionType) {
    	...
        case 'province_flow_set_terminal':
        	// dispatcher -> store
            provinceFlowStore.setTerminal(payload.item);
            break;
        ...
    }
});
</code></pre>

对应的方法触发emit，使store改变，view重新渲染。

<pre><code>ExampleStore.prototype = assign({}, EventEmitter.prototype, {

    ...

    // store -> view
    setTerminal: function (params) {
        this.queryObj.terminal = params.terminal;
        this.emit('change');
    },
    addChangeListener: function (action) {
        this.on('change', action);
    },
    removeChangeListener: function (action) {
        this.removeListener('change', action);
    },
    emitChange: function () {
        this.emit('change');
    }
});
</code></pre>


至此，一个flux下实现select组件的栗子就完成了，然而，如果要级联呢？怎么办？

# select级联

比如我们要实现一级目录和二级目录的级联，数据结构是这样的：

## config.js

<pre><code>dirs: [
    {children:[{dirName:'xx',dirId:'secondDir0'},
    {dirName:'xx',dirId:'secondDir1'},
    {dirName:'xx',dirId:'secondDir2'},
    {dirName:'xx',dirId:'secondDir3'},
    {dirName:'xx',dirId:'secondDir4'},
    {dirName:'xx',dirId:'secondDir5'},
    {dirName:'xx',dirId:'secondDir6'},
    {dirName:'xx',dirId:'secondDir7'},
    {dirName:'xx',dirId:'secondDir8'}],dirName:'情感',dirId:'firstDir0'},
    ...
    // first1...2...3...
]
</code></pre>

# Example.js

此时，ExampleForm组件之下，使用一个DirsContent组件，传入上层分发下来的数据：

<pre><code>&lt;DirsContent firstDir={formData.firstDir}
    secondDir={formData.secondDir}
&gt;
&lt;/DirsContent&gt;
</code></pre>

实现DirsContent组件：

<pre><code>var DirsContent = React.createClass({

    // 两个select的change方法，action、store中类似上面terminal的写法
    changeFirstDirHandler: function (e) {
        ContentFlowActions.setFirstDir({
            firstDir: e.target.value
        });
    },
    changeSecondDirHandler: function (e) {
        ContentFlowActions.setSecondDir({
            secondDir: e.target.value
        });
    },
    render: function () {
        var firstDir = this.props.firstDir;
        var secondDir = this.props.secondDir;
        // 二级目录select选项初始为空
        secondDirsArr = [];
        // 如果选择了一级目录
        if (firstDir) {
        	// 二级目录取对应的children字段的值（以firstDir0、firstDir1、firstDir2...最后一位的数字为索引）
            secondDirsArr = dirsArr[firstDir.split('Dir')[1]].children;
        }
        return (
            &lt;div className="form-group"&gt;
                &lt;div className="form-group"&gt;
                    &lt;label htmlFor="" className="form-control-static"&gt;一级目录：&lt;/label&gt;
                    &lt;BasicSelect
                    	&lt;!-- 一级目录的数据，可以直接从config中获取 --&gt;
                        optionData={dirsArr}
                        optKey="dirName"
                        optVal="dirId"
                        changeHandler={this.changeFirstDirHandler}
                        nameText="firstDir"
                        defaultVal={firstDir}
                    &gt;&lt;/BasicSelect&gt;
                &lt;/div&gt;

                &lt;div className="form-group"&gt;
                    &lt;label htmlFor="" className="form-control-static"&gt;二级目录：&lt;/label&gt;
                    &lt;BasicSelect
                    	&tl;!-- 二级目录的数据，为render方法中计算出来的数据 --&gt;
                        optionData={secondDirsArr}
                        optKey="dirName"
                        optVal="dirId"
                        defaultVal={secondDir}
                        nameText="secondDir"
                        changeHandler={this.changeSecondDirHandler}
                    &gt;&lt;/BasicSelect&gt;
                &lt;/div&gt;
            &lt;/div&gt;
        );
    }
});
</code></pre>

# 补充

到现在为止，写了五个大同小异的查询数据模块，猿生第一次写这么多的js代码（用了React不得不写这么多啊...），从开始的迷茫到逐渐拨开云雾见到彩虹，感谢[三木君](http://yansm.github.io/fromone)耐心的帮助和清晰的讲解。学习一种新的东西，或许，可以先上手再理解...