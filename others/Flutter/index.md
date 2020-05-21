# Flutter笔记

### 组件

##### 布局组件

Row：只有横向（布局类似于Flex）

Column： 只有纵向 （像传统的块级元素，上下排列）

Stack： 堆叠（类似绝对定位）

Wrap： 类似Flex（可以做流式布局）

Container： 类似div（有完整的盒子模型）

Padding



##### 基础组件

Text： 文字，可以配置样式，不可以换行

Expanded： 扩展，类似于Flex中的设置（自动占满）

MediaQuery.of(content).size.width： 获取当前内容的实际宽度，值为double，可以运算



点击：

* Material UI组件库中有button
* flat button 大块的点击
* Gesture Dector 手势检测（点击用onTap事件）

网络请求： 使用Dio （类似于Axios）



### Dart语法

```dart
List<Widget> = []; // 泛型。List中元素限制为Widget类型。

@override
function funcName() {} // 函数重写，用于继承后函数的改写
```

路由： Navigation中提供了路由栈，根据push pop操作



### 安卓下开启Webview的chrome devtool inspector

`Flutter_webivew_plugins` 安卓下开启webview在chrome下检查功能

添加`webview inspect`支持：

 

在本地的flutter安装目录下找到 `.pub-cache`文件夹，

进入hosted中`pub.flutter-io.cn` 找到引用的包`flutter_webview_plugin-0.3.5`

进入android的源码下找到：

`/flutter/.pub-cache/hosted/pub.flutter-io.cn/flutter_webview_plugin-0.3.5/android/src/main/java/com/flutter_webview_plugin`

 有4个java文件

```
BrowserClient.java

FlutterWebviewPlugin.java

ObservableWebView.java

WebviewManager.java
```

在WebviewManager.java中，找到WebviewManager的构建函数，在构建函数的第一行加入：

 ```dart
WebView.setWebContentsDebuggingEnabled(true);
 ```

 然后在flutter项目中，使用flutter clean 清除掉已经编译过的flutter package。

然后再使用 flutter build apk --release 进行apk构建，用于预发布版的webview检查

或者F5 进行debug

 

另外，webview路由返回，可以使用history,length来判断，当前是否需要直接返回原生界面



##### 坑： 

有使用了MediaQuery的场合， 在release版本下widget不显示出来。

使用print输出之后发现都是0

原因是在 release版本下，flutter 加载要快于MediaQuery查询到的参数

（同理window对象下获取到的屏幕参数也是这种情况）

 另： 使用adb logcat 可以查看安卓系统全局的日志记录

 

##### 安卓应用签名： 

使用java sdk 自带的keytool （一般放在jdk 的bin中）

使用命令生成key.jks 

需要填写：

密码

开发者的相关信息

 

配置则需要在build.gradle中 设置签名的信息

（签名分 release 和 debug）可分别设置

 

微信开放平台中添加移动应用：

> 1.  需要 应用名称、包名、官网、签名（上面key.jks 使用keytool 查询出来的md5 去掉冒号，16进制的字母使用小写）
> 2.  打包的时候，需要应用名称 包名，签名一直，才能调用微信的sdk
> 3.  iOS 要以release版本运行，只能使用 flutter run --release 这种模式进行安装（）

 

##### 最佳实践

1.  类似css中的全局样式， 使用枚举类型定义一些预置样式

2.  模块化



### iOS相关



##### 打包前确认info.plist是否正确

```sh
plutil -p ./info.plist
```



##### iOS证书操作

1.  App Store
2.  申请App bundle id （切记不能重复）如果被占用的话需要删除掉前面注册的bundle id

​        如果是使用非认证账号的话，需要等7天这个bundle id过期了才能申请

3. 申请证书

   > iOS的开发证书、生产证书（有些证书需要导入到开发机的keychain中）
   >
   > 需要使用推送功能的话，需要申请ANPS
   >
   > 结合极光，需要从macOS的keychain导出P12文件（类似密钥？）

4.  TestFlight

   >简单来讲，在APP Connect中直接添加用户（这种是内部用户）需要是已注册为Apple ID的邮箱？）
   >
   >添加用户之后在TestFlight中添加到测试用户组中，这里可以作职能分类
   >
   >按邮件1 加入APP Connect组中
   >
   >按邮件2 流程下载TestFlight（在App Store中），然后把邮件中的邀请码兑换了之后可以下载使用。