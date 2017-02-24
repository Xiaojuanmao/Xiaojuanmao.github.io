---
title: React-Native笔记(二)
date: 2017-02-24 11:36:10
tags: React-Native

---

## React-Native笔记(二)

### 图片

**静态图片**
引用图片只需要将图片文件放在项目文件夹中，指定路径引用

```
<Image source={require('./my-icon.png')} />
```
系统会从一个引用了图片的组件所在的根目录出发，去寻找这张图片。
提供`@2x`,`@3x`来指明图片的精度。
使用`my-icon.ios.png`以及`my-icon.android.png`来提供不同平台的图片。

<!--more-->

在`require()`里面的图片名字必须为静态字符串

> 加入新的图片资源后，可能需要重启packager引入新的图片资源

**原生与React Native**
使用react native的时候也能够使用原生打包的资源文件

```
<Image source={{uri: 'app_icon'}} style={{width: 40, height: 40}} />
```

> 这一做法并没有任何安全检查。你需要自己确保图片在应用中确实存在，而且还需要指定尺寸。


**网络图片**
使用网络图片的时候需要指定图片的尺寸

```
// 正确
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}}
       style={{width: 400, height: 400}} />

// 错误
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}} />
```

### ScrollView
`ScrollView`是一个通用的可滚动容器，支持竖直、水平滚动

```
    	<ScrollView> 

    		<Text style={{fontSize:32}}>Scroll me plz</Text>

    		<Text style={{fontSize:32}}>Scroll me plz</Text>

    		<Image source={require('./test.jpg')} />
    	</ScrollView>
```

**可点击组件**
使用`Touchable`开头的一系列组件能够捕捉用户点击操作。
`onPress`属性接受一个处理点击事件的函数，在点击操作开始并终止于组件的时候，相应的函数会被调用。
`onLongPress`属性用来处理长按事件。

```
_onPressButton() {
    console.log("You tapped the button!");
  }

<TouchableHighlight onPress={this._onPressButton}>
        <Text>Button</Text>
      </TouchableHighlight>
```

可以通过使用不同组件提供给用户不同的触摸反馈:

* `TouchableHighlight`会在用户手指按下的时候变暗
* `TouchableNativeFeedback`会在Android上体现出原生的视觉效果
* `TouchableOpacity`会在用户按下的时候改变透明度
* `TouchableWithoutFeedback`没有任何反馈


###ListView
动态渲染屏幕上可见的元素
`dataSource`属性，设定列表的数据源
`renderRow`属性，诸葛解析数据源中的数据，染回一个设定好格式的组件来渲染

```
class ListViewBasics extends Component {
  // 初始化模拟数据
  constructor(props) {
    super(props);
    const ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});
    this.state = {
      dataSource: ds.cloneWithRows([
        'John', 'Joel', 'James', 'Jimmy', 'Jackson', 'Jillian', 'Julie', 'Devin'
      ])
    };
  }
  render() {
    return (
      <View style={{flex: 1, paddingTop: 22}}>
        <ListView
          dataSource={this.state.dataSource}
          renderRow={(rowData) => <Text>{rowData}</Text>}
        />
      </View>
    );
  }
}
```