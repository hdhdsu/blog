初识react native
=====
##### 什么是react native
react是一个前端开发框架，现在微博前端比较流行的有三个框架，react ,angular,vue。而react native 是facebook出的基于react的用来开发app端的框架，React有三个特点：

*	交互式的
> 应用是基于状态的，应用会随着数据的变化切换到不同的状态，状态改变，视图改变

*	基于组件的（component）
> 把管理状态view封装成一个个的component,然后再把这些component组合起来实现复杂UI

*	learn Once,Write Anywhere(一次编译，到处运行)
> 这也是facebook鼓吹react native的一句话，其实之所以能理论上实现这样，主要是virture dom 的功劳

##### 搭建react native开发环境
https://reactnative.cn/docs/0.51/getting-started.html#content

PS:这是react native的中文官网，里面干货满满


##### react native的文件的基本结构

	import React, { Component } from 'react'
	import {
	    StyleSheet,
	    TouchableOpacity,
	    View,
	    Text,
	    Image
	} from 'react-native'
	import { constant } from '../Theme'
	import Line from '../ui/Line'
	import InquiryOrderItem from '../functional/InquiryOrderItem'
	import RefreshList from '../functional/RefreshList'
	import DimenUtil from '../../utils/DimenUtil'
	import topicDetailInfo from '../../res/data/topicDetail.json'
	
	class InquiryOrderView extends Component {
    constructor(props) {
        super(props)
        this.state = {
            topicDetailList: topicDetailInfo.topicDetail,
        }
    }

    render() {
        return (
            <View style={{ flex: 1 }}>
                <RefreshList
                    style={{ backgroundColor: 'white' }}
                    data={this.state.topicDetailList}
                    renderItem={this._renderItem}
                />
            </View>
        )
    }

    _renderItem = ({ item }) => (
        <InquiryOrderItem />
    );
	}

	const styles = StyleSheet.create(
	    {
	        container: {
	            flex: 1,
	            justifyContent: 'center',
	            alignItems: 'center',
	            backgroundColor: '#FFFFFF',
	        },
	    }
	);
	
	export default InquiryOrderView


首先，刚才介绍了，既然react native是基于组件式的，当然会继承Component,而我们要从'react-native'这个库导入这个类，
在React Native开发中，component是一个非常重要的概念，它类似于iOS的UIView或者Android中的view,将视图分成一个个小的部分。如果继承了Component,就有了生命周期

![组件生命周期](https://images2017.cnblogs.com/blog/1202156/201708/1202156-20170809111027464-1788797193.png)

###### 创建阶段

* constructor

	什么时候调用：在组件初始化的时候调用

	作用：初始化state

* componentWillMount

	什么时候调用：即将加载组件的时候调用

	作用：在render之前做事情

* render

	什么时候调用：渲染组件的时候调用

	作用：通过这个方法渲染界面

* componentDidMount

	什么时候调用：组件渲染完成之后调用

	作用：在render之后做事情，比如发送请求

###### 更新阶段

* componentWillReceiveProps

	什么时候调用：每次传入Props的时候就会调用

	作用：拦截Props
* shouldComponentUpdate

	什么时候调用：每次Props或者State改变就会调用

	作用：控制界面是否刷新
* componentWillUpdate

	什么时候调用：组件即将更新的时候调用

	作用：在render更新前做事情
* componentDidUpdate

	什么时候调用：组件更新完成之后调用

	作用：在render更新后做事情

###### 销毁阶段
* componentWillUnmount

	什么时候调用：组将即将销毁的时候调用

	作用：移除观察者，清空数据

知道了组件的生命周期，就可以在对应的方法里面做对应的事

state和props这两种数据很重要，具体可以参照官网的解释

[state相关](https://reactnative.cn/docs/0.51/state.html#content)

ps:只要this.setState()就可以更新界面，而且是局部更新，是不是很爽呢

[props相关](https://reactnative.cn/docs/0.51/props.html#content)

##### render
我们已经知道了render()方法事渲染布局的，我们都知道，前端是js和html，css分割开的，而reactnative采用JSX方式来写布局，这个实际上是JavaScript语言的扩展，它并不改变JS本身语法。使用起来类型XML，React会对JSX的代码进行编译，生成JavaScript代码，用来描述React中的Element如何渲染。就是js的语法糖，而且它实现了css in html，也就是样式，和view集中到一起，

这里如果要使用什么组件都要导入，如果是react native的库里面就有的组件，那么import {} from 'react-native'，如果是导入自己定义的组件，那么就需要路径导入

#####  StyleSheet
这个是view的样式，它也是要导入的，在这里可以定义不同的样式

##### 对其他组件开放
export default InquiryOrderView 这句话是用来对于其他组件开放的，没有这句话，其他组件不能引用这个组件


