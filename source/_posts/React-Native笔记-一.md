---
title: React-Native笔记(一)
date: 2017-02-24 11:36:05
tags: React-Native

---

## React-Native笔记(一)
创建项目 `react-native init projectname`

手动运行packager，用于调试项目 `react-native start`

查看日志 `adb locat *:S ReactNative:V ReactNativeJS:V`

<!--more-->


## 代码

### import
在index.android.js中，导入需要用到的组件,例如`Component`，`AppRegistry`，`View`等等
```
 import React, { Component } from 'react';
 import { AppRegistry, StyleSheet, Text, View} from 'react-native';
```

### AppRegistry
应用作为最大的一个Component，需要在最后通过`AppRegistry.registerComponent('AppName', () => AppName)`注册并运行

### props(属性)
常见基础组件有各种属性(props)，例如`Image`就有`source`和`style`等属性来控制图片地址和尺寸,下列代码中的name就是自定义的props

```
class Greeting extends Component {
  render() {
    return (
      <Text>Hello {this.props.name}!</Text>
    );
  }
}

```

### state(状态)
`props`由父组件指定，在生命周期中不会再改变，而`state`能够用于改变的数据。需要再`Constructor`中对`state`进行初始化，实例如下:

```
class Blink extends Component {
	constructor(props) {
	  super(props);
	  this.state = { showText: true};

	  setInterval(() => {
	  	this.setState({ showText: !this.state.showText});
	  }, 1000);
	}


	render() {
		let display = this.state.showText ? this.props.text : ' ';
		return (
			<Text>{display}</Text>
		);
	}
}
```

### 样式
所有核心组件都接受`style`属性，样式名称要求使用驼峰命名法。
使用过程中建议用`StyleSheet.create`来集中定义组件样式，类似于`Android`中定义各种`style`一样。
借鉴了`CSS`中层叠覆盖的做法，属性能够被覆盖

```
const styles = StyleSheet.create({
	bigblue: {
		color: 'blue',
		    	<View style={{
   			padding: 10,
    	}}>

    		<TextInput
    			style={{height: 40}}
    			placeholder="Type here to translate!"
    			onChangeText={(text) => this.setState({text})} /> 

    		<Text style={{padding: 10, fontSize: 42}} >
    			{this.state.text.spilt(' ').map((word) => word && 'woo').join(' ')}
    		</Text>
    	</View>
fontWeight: 'bold',
		fontSize: 30,
	},
	red: {
		color: 'red',
	}
})
```


### 控件尺寸

**固定尺寸**
在样式中设置`width`和`height`。
```
<View style={{width: 50, height: 50}} />
```

**弹性(flex)尺寸**
在组件样式中使用`flex`来指定控件所占比例，`flex:1`指定组件撑满剩余空间，多个并列的`flex:1`平分剩余控件
> 组件能够撑满剩余空间的前提是其父容器的尺寸不为零。如果父容器既没有固定的width和height，也没有设定flex，则父容器的尺寸为零。其子组件如果使用了flex，也是无法显示的。


```
<View style={{flex:1}}>
    		<View style={{flex:1, backgroundColor: 'powderblue'}} />
    		<View style={{flex:2, backgroundColor: 'skyblue'}} />
    		<View style={{flex:3, backgroundColor: 'steelblue'}} />
    	</View>
```

### Flexbox布局

[布局属性文档](http://reactnative.cn/docs/0.41/layout-props.html)


`Flexbox`规则用来指定某个组件的子元素布局。

> React Native中的Flexbox的工作原理和web上的CSS基本一致，当然也存在少许差异。首先是默认值不同：flexDirection的默认值是column而不是row，而flex也只能指定一个数字值。

**Flex Direction**
`flexDirection`样式决定布局的**主轴**，子元素沿着**主轴**方向排列，默认为`column`方向,下面的三个`view`横向排列
```
<View style={{flex:1, flexDirection: 'row'}}>
	...
</View>
```

**Justify Content**
`justifyContent`决定子元素沿着**主轴**的排列方式。可选属性如下：

| 名称 | 功能 |
|--------|--------|
|    默认    |    相邻无间距排列    |
|    flex-start    |    同上    |
|    center    |    从中间开始依次排列    |
|    flex-end    |   从父布局尾端开始排列     |
|    space-around    |    每个子布局周围距离一致    |
|    space-between    |    子布局之间距离一致，和父布局无距离    |

```
<View style={{
    		flex:1,
    		flexDirection: 'row',
    		justifyContent: 'flex-start',
    	}}>
        ...
    	</View>
```

**Align Items**
`alignItems`决定子元素**次轴**排列方式，**次轴**和**主轴**方向垂直。

| 名称 | 功能 |
|--------|--------|
|    flex-start    |    次轴顶端排列    |
|    center    |    次轴中间开始排列    |
|    flex-end    |   次轴尾端开始排列     |
|    stretch   |    每个子布局周围距离一致    |


> 要使stretch选项生效的话，子元素在次轴方向上不能有固定的尺寸。


### 处理文本输入
使用`TextInput`来允许用户输入文本。

`onChangeText`属性接受一个函数，在文本发生变化的时候调用该函数。
`onSubmitEditting`属性会在文本被提交之后调用(用户按下软键盘上的提交按钮)。
