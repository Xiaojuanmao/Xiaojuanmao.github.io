---
title: React-Native笔记(三)
date: 2017-02-24 11:36:14
tags: React-Native

---
## React-Native笔记(三)

### 网络
`React Native`提供了和web标准一直的`Fetch API`来满足网络获取数据的需求。

**发起网络请求**
获取固定地址内容，不进行处理
```
fetch('https://www.baidu.com')
```

<!--more-->

获取内容并对数据进行加工

```
 fetch('http://bbs.reactnative.cn/api/category/3')
            .then((response) => response.json())
            .then((jsonData) => {
                this.setState({
                    title: jsonData.topics[0].title,
                })
            })
            .catch((error) => {
                console.warn(error);
            })
```

### WebSocket支持
对`websocket`支持，可以在单个TCP连接上提供全双工的通信通道。

```
var ws = new WebSocket('ws://host.com/path');

ws.onopen = () => {
  // 打开一个连接

  ws.send('something'); // 发送一个消息
};

ws.onmessage = (e) => {
  // 接收到了一个消息
  console.log(e.data);
};

ws.onerror = (e) => {
  // 发生了一个错误
  console.log(e.message);
};

ws.onclose = (e) => {
  // 连接被关闭了
  console.log(e.code, e.reason);
};
```

### 页面跳转

**场景(Scene)**
各种组件组合在一起，形成了一个界面，也就是一个场景

一个简单的场景如下：
```
MyScene.js

import React, { Component } from 'react';
import { View, Text } from 'react-native';

/**
 * 导出当前组件
 */

export default class MyScene extends Component {
    static defaultProps = {
        title: 'MyScene',
    };

    render() {
        return (
            <View>
                <Text>Hi ! My name is {this.props.title} </Text>
            </View>
        )
    }
}
```

**Navigator**
官方首推`Navigator`来作为导航器组件，实现界面跳转,先渲染出一个`Navigator`组件，通过`Navigator`的`renderScene`属性来渲染其他的`Scene`。将之前的那个场景给渲染进来。

**单个渲染**

```
 <Navigator
                initialRoute={{title: 'My Initial Scene', index: 0}}
                renderScene={(route, navigator) => {
                    return <MyScene title={route.title} />
                }}
            />
```
使用导航器经常会碰到“路由(route)”的概念。“路由”抽象自现实生活中的路牌，在RN中专指包含了场景信息的对象。
```


**实现跳转**
场景之间的切换，`navigator`提供了两个主要的方法：
* `push` 将`route`对象推入并渲染
* `pop` 将`route`对象弹出导航栈

**基本场景**
```
import React, { Component, PropTypes } from 'react';
import { View, Text, TouchableHighlight } from 'react-native';

export default class MyScene extends Component {
  static propTypes = {
    title: PropTypes.string.isRequired,
    onForward: PropTypes.func.isRequired,
    onBack: PropTypes.func.isRequired,
  }
  render() {
    return (
      <View>
        <Text>Current Scene: { this.props.title }</Text>
        <TouchableHighlight onPress={this.props.onForward}>
          <Text>点我进入下一场景</Text>
        </TouchableHighlight>
        <TouchableHighlight onPress={this.props.onBack}>
          <Text>点我返回上一场景</Text>
        </TouchableHighlight>    
      </View>
    )
  }
}
```

**导航器**

```
import React, {Component} from 'react';
import {AppRegistry, Navigator, Text, View} from 'react-native';

import MyScene from './MyScene';

class AwesomeProject extends Component {

    render() {
        return (
            <Navigator
                initialRoute={{title: 'My Initial Scene', index: 0}}
                renderScene={(route, navigator) => {
                    return <MyScene title={route.title}

                     onForward= { () => {
                         const nextIndex = route.index + 1;
                         navigator.push({
                             title: 'Scene ' + nextIndex,
                             index: nextIndex
                         });
                     }}

                     onBack= { () => {
                         if (route.index > 0) {
                             navigator.pop();
                         }
                     }}

                     />
                }}
            />
        );
    }

}

AppRegistry.registerComponent('AwesomeProject', () => AwesomeProject);
```