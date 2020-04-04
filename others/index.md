# 未分类

### webpack的externals使用



场景： 比如在webpack构建的项目中引入线上的百度地图库，而又要符合es module规范的话，可以使用externals来解决ESlint报错等问题。当然这种方式不支持AMD、CommonJS等规范的库（待考证）。

```javascript
{
  externals: {
    'BMap': 'BMap' // 或者window.BMap， 通过捕获全局变量来使用
  }
}
```

通过externals来使用线上的第三方JS库，在原来的html中插入引用的script标签，可以节省服务器资源

```javascript
import Bmap from 'BMap' // import进来后使用
```

