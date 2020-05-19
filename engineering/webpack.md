# webpack配置

### 目前环境下的webpack

现在不少大框架的脚手架工具默认隐藏了webpack配置（也就是`零配置化`），毕竟这些脚手架中提供的webpack配置很多时候已经够用了，无需自己手动配置。当然在需要自己调整的时候可以使用外置的配置文件（提供了不少可配置的选项）来调整编译/开发的一些选项配置。

比如Vue在新版的vue-cli中就提供了`vue.config.js`这个配置文件，其中可以配置`devServer`、 `chunks`等，基本上可以做到非常简洁地自定义vue的开发/编译设置。

但是事情有那么简单吗？诸如面试的时候会问webpack怎么配置，一些细节等等。那么自己会搭一个webpack脚手架还是挺有必要的。

### webpack的配置

webpack的配置主要对应开发和生产两种环境，所以在配置文件上可以搞成这样的：

```javascript
webpack.config.base.js // 通用的配置文件
webpack.config.dev.js // 开发时用的配置文件
webpack.config.prod.js // 生成生产环境产物的配置文件
```

升级了之后有些项废弃了，换成相似的其他功能：

| 废弃项目                  | 替代项目                    | 功能               |
| ------------------------- | --------------------------- | ------------------ |
| UglifyjsWebpackPlugin     | .minmize                    | 代码压缩、优化     |
| ModuleConcatenationPlugin | .opconcatenateModules       | Scope hoisting     |
| CommonsChunkPlugin        | .splitChunks或.runtimeChunk | Code Splitting     |
| NoEmitOnErrorsPlugin      | noEmitOnErrors              | 编译出错时跳过编译 |

在webpack中，设置了模式之后，有些项目会自动开启，比如`development`模式下会自动开启`NamedChunksPlugin`和`NamedModulesPlugin`这两个插件以方便调试，提供更快的编译速度和更完整的错误信息。

则`production`模式下则自动开启`splitChunks`和`minmized`，所以基本上就不用怎么配置webpack了，webpack会自动帮你做好`code splitting`、压缩、`Tree Shaking`、`Scope Hoisting`。

```javascript
module.exports = {
    mode: 'development' // 生产环境 'production'
}
```



### mini-css-extract-plugin

webpack4中对css模块的支持更加完善了，新出的`mini-css-extract-plugin`就是用来替代原先的`extract-text-webpack-plugin`的。

两者最大的区别就是，新插件能在代码分割的时候，将原先内联进js chunk bundle的css，单独拆分成多个css文件。

这样拆分有个好处就是，js和css的各自改动不会影响对方，避免更新导致原有的缓存失效。

而且能自定义拆分css的粒度，不会像以前一样所有css都打包到一个css文件中。



### 压缩与优化

不过上面打包css的时候并没有对css进行压缩，那么可以使用这个选项来开启css的编译压缩功能：

```javascript
// webpack配置文件中
module.exports = {
    mode: 'production',
    minimize: true,
    minimizer: [new OptimizeCSSAssetsPlugin()]
}
```

`optimize-css-assets-webpack-plugin`这个插件默认使用`cssnano`来做css优化，能做到压缩代码、删除无用注释、去除冗余、优化css代码等。



### 关于Hash需要注意的一些地方

在webpack有个经常出现的词，就是`Hash`。相关衍生有`content hash` 和 `chunk hasn`的概念。

简单来讲，hash和每次build的产物有关，如果两次build中，没有任何修改的话，hash就会完全一样；只要改动了一点的话两次生成的hash会不一样。

而chunkhash则是根据每一个模块文件自身内容包括其依赖计算所得的hash，如果只修改一个文件的话，只有该文件的hash产生变化，不影响其他文件。

contenthash主要是为了让css不受js文件的影响。使用chunkhash的情况下，假设一个css被一个js引用了，这时两者共用一个chunkhash值，只要修改了js，就算css部分没有变动，其css文件的hash值也会变动导致缓存失效。这个时候使用contenthash的话，因为是根据内容来确定hash值的，所以改动js文件而不动css文件的话，其hash值是不会变的。（contenthash可以简单理解为moduleid +  content生成的hash）



### 开发时影响热更新速度的因素

* 没有合理设置devtool source map
* 没有正确使用exclude / include， 处理了不需要处理的node_modules
* 在开发环境中压缩了代码（uglifyjs）、提取css、使用了babel polyfill、计算hash等编译打包才需要的流程



##### 旧方案

* 在开发时不使用路由懒加载。具体的话就是封装一个import函数，用来识别当前环境是否是生产环境，来决定是否进行懒加载

```javascript
// 开发环境
module.exports = file => require('@/views/' + file + '.vue').default
// 生产环境
module.exports = file => import('@/views/' + file + '.vue')
```

不过webpack import机制的问题，有一定的副作用，上面的写法会导致`@/views/`下的所有vue文件都会被打包。不管是否用到都会多打包一些无关、用不到的代码。

##### 新方案

babel提供了一个plugin用来将所有的`import()`转换成`require()`，也就是`babel-plugin-dynamic-import-node`。使用这个插件的话可以将所有的异步组件以同步形式进行引入，并且结合`BABEL_ENV`这个babel环境变量，让这种形式只作用于开发环境。

这么做能在不影响业务代码本身（侵入性小），解决重复打包的问题。路由部分的代码可以按照官方文档中路由懒加载的部分来写。剩下的交给babel处理，不香吗？

具体操作：

package.json中：

```json
{
    "scripts": {
        "dev": ”BABEL_ENV=development webpack-dev-server ...“ // 在原先的npm scripts前面添加
    }
}
```

然后在babel配置文件`.babelrc`中：

```json
{
    "env": {
        "development": {
            "plugin": ["dynamic-import-node"]
        }
    }
}
```

路由文件中这么写： 

```javascript
{
    path: '/login', component: () => import('@/views/login/index')
}
```



### 打包速度

影响打包速度的因素有不少，那么首先是需要知道打包中到底是哪个环节比较耗费时间。

可以使用`speed-measure-webpack-plugin`插件来监控webpack编译中哪个步骤比较费时。

一般来讲，编译中最花时间的部分多是用在`Uglifyjs`压缩代码。使用新版本的`UglifyJsPlugin`的时候可以加上

`cache: true` `parall: true`的选项，能提高代码打包速度。

另外就是庞大的第三方库的打包也会非常占用时间，可以选择将这部分的库给`exrernals`出去，然后通过script标签进行引入；或者使用dll的方式进行打包。

另外可以选择使用并行执行webpack的库如`parallel-webpack`、`happypack`等。

还可以通过升级node、升级机器配置等提高node性能，进而提高编译速度。

最后需要注意的是，编译打包的速度固然重要，但是需要做好平衡，工程师应该注重业务，而不是把大量的时间都用在工具的微量提升上。



### Tree Shaking

老生常谈的Tree Shaking功能。webpack4中新增了`JSON Tree Shaking` 和 `sideEffects`来更好地shaking。

webpack4中默认支持的，不过很多时候会因为babel配置的原因失效。

Tree Shaking是基于ES6 modules的静态特性检测来找出未使用的代码的。如果用了babel插件如`babel-preset-env`时，会默认将模块打包成commonjs，也就使得Tree Shaking失效了。

只要让babel不将模块转换成commonjs就行了，配置如下：

```json
// .babelrc
{
  "presets": [
    ["env", {
      modules: false,
      ...
    }]
  ]
}
```



### 分包策略

webpack4最大的改动就是将`CommonChunkPlugin`换成`optimization.splitChunks`。

新版本下的webpack在production模式下会自动开启`Code Splitting`功能。这种情况下（也就是使用默认配置，没有自己改配置），入口文件依赖的文件会被打包进`app.js`，而大于30kb的第三方库类则会被单独打包成一个个独立的`bundle`。

`Code Splitting`的分割策略如下：

* 新的chunk是否被共享或者来自node_modules
* 新的chunk体积在压缩之前是否大于30kb
* 按需加载chunk的并发请求数量小于等于5个
* 页面初始加载时的并发请求数量小于等于3个



有些情况就是，一些第三方组件/包未达到30kb的大小（比如在15到30kb之间），但是被多个页面引用，会导致这个包被打包到多个bundle中，显然是不合理的。

这种情况建议直接将多个页面能共用的组件单独抽出来合并到一个bundle中。



### 分包策略的优化

一般情况下各类型的代码/库类的使用率 / 共用率 / 更新频率

| 类型            | 共用率 | 使用频率 | 更新频率 | 例子                                                         |
| --------------- | ------ | -------- | -------- | ------------------------------------------------------------ |
| 基础库类        | 高     | 高       | 低       | vue/react、vuex/mobx、axios等                                |
| UI组件库        | 高     | 高       | 中       | element-ui/antd等                                            |
| 必要组件/函数   | 高     | 高       | 中       | Nav/Header/Footer组件、路由定义、权限验证、全局State、全局配置等 |
| 非必要函数/组件 | 高     | 高       | 中       | 封装的Select/Radio组件、utils函数等（必要/非必要函数组件可以合并） |
| 低频组件        | 低     | 低       | 低       | 富文本、echarts、Dropzone等                                  |
| 业务代码        | 低     | 高       | 高       | 业务组件、业务模块、业务页面等                               |

于是对应上面的各种类型的代码来制定处理方案：

* 基础类库 chunk-libs

这是构建项目必不可少的基础类库，比如vue全家桶（vue、vuex、vue-router、axios），它们的升级频率并不高，但每个页面都需要用到它们。（一些体积不大的第三方类库也可以一同放入其中，如：nprogress、js-cookie、clipboard等）

* UI 组件库

理论上UI组件库是可以归入基础类库的范畴中的，但是单独拿出来是因为这类组件库的体积太大了，无论是element-ui还是antd，在经过gzip压缩之后还可能需要200kb（可能比其他libs中所有库加起来还要大）。另外考虑到UI组件的更新频率比较高，需要经常更新来获取新的功能或者解决一些bug。于是建议将UI组件库单独拆分成一个包

* 自定义组件/函数 chunk-commons

这里的commons主要分为必要和非必要。

必要组件是指那些在项目里必须加载它们才能正常运行的组件或者函数。比如路由表、全局state、全局侧边栏/Header/Footer等组件、自定义SVG图标等。也就是你在入口文件就加载/依赖的东西，它们会默认打包进app.js中。

而非必要组件则是指被大部分页面使用，但不被入口文件所引用的模块。比如自己封装的table组件，由于体积不大会被默认打包到每一个懒加载页面的chunk中，造成不少的浪费（积少成多）。这些被大量共用的组件应该单独打包成chunk-commons。

当然必要组件和非必要组件可以一同打包进app.js中也没问题。

* 低频组件

低频组件和上述的共用组件chunk-commons的最大区别就是，它们只在特定的场景下使用，如富文本编辑器等。一般这些库体积会比较大，所有webpack4会默认打包成单独的一个bundle中，无需特别处理。小于30kb则会被打包进使用它的页面bundle中。

* 业务代码

业务代码一般按照页面的划分进行打包，如在vue中，使用路由懒加载的方式加载页面的话，webpack会默认将它打包成一个独立的bundle。

webpack中配置方式如下：

```javascript
splitChunks: {
  chunks: "all",
  cacheGroups: {
    libs: {
      name: "chunk-libs",
      test: /[\\/]node_modules[\\/]/,
      priority: 10,
      chunks: "initial" // 只打包初始时依赖的第三方
    },
    elementUI: {
      name: "chunk-elementUI", // 单独将 elementUI 拆包
      priority: 20, // 权重要大于 libs 和 app 不然会被打包进 libs 或者 app
      test: /[\\/]node_modules[\\/]element-ui[\\/]/
    },
    commons: {
      name: "chunk-commons",
      test: resolve("src/components"), // 可自定义拓展你的规则
      minChunks: 2, // 最小共用次数
      priority: 5,
      reuseExistingChunk: true
    }
  }
}
```

而实际的业务场景也没那么简单，很多时候都需要根据情况来调整分包策略，比如共用组件chunk-commons包含了不少组件，可能会出现这种情况：加载某个页面，只需要其中某个很小的组件，却需要下载整个chunk-commons。也就是需要开发者去考虑哪些组件拆出来放一起比较好，如何拆（调整minChunks值）等等。



其实分包优化配置这块就是一场博弈，首次加载快和cache利用率高两者不可兼得。需要注意一点就是，拆包切不能过分追求颗粒化，过于分散的bundle会使得加载一个页面需要请求十来个js文件，在非HTTP2的情况下，请求阻塞会很明显的（受限制于浏览器的并发请求数量）。

> 在支持HTTP2的情况下，可以使用webpack4中新增的maxSize选项，这能让chunk在minSize的范围内更加合理地拆分，这样可以更好地利用HTTP2来进行长缓存。（HTTP2的环境下，缓存策略会稍有变化）



持久化缓存

简单来讲，有这么个成熟的缓存方案：

> * 针对html文件，不开启缓存，把html放道自己的服务器上，关闭服务器的缓存（或者设置成协商缓存？）
> * 针对静态的js、css、图片等文件： 开启cdn和缓存，将静态资源上传到cdn服务器中，并对资源开启长期缓存，因为资源路径具有唯一性，可以确保线上用户访问的稳定性。
> * 每次发布更新时，先将静态资源上传到cdn服务器上，再上传html文件，这样技能确保老用户正常访问，又能让新用户看到新的页面。



于是需要让webpack给静态资源生成一个可靠的hash，让它能自动在合适的时候更新hash，并确保hash值的唯一性。只要打包内容不一样，那么hash值就不一样。

webpack4中持久化缓存的一些相关问题：

* RuntimeChunk(mainfest)
* Module vs Chunk
* HashedModulesPlugin
* NamedChunksPlugin

##### RuntimeChunk(mainfest)

webpack4提供了这个选项让我们方便提取mainfest，以前需要这么配置

```javascript
// before
new webpack.optimize.CommonsChunkPlugin({
    name: 'mainfest',
    minChunks: Infinity
})

// now
{
    runtimeChunk: true
}
```

其作用就是将包含chunks映射关系的list单独从app.js中提取出来，因为每一个chunk的id基本都是基于内容hash出来的，所以每次改动都会影响它，如果不提出来的话，也就相当于app.js每次都会改变，也就让缓存失效了。

实际上打包出来的runtime.js非常小，gzip之后一般只有几kb，但这个文件会经常改动，每次都需要重新请求它，其http请求耗时远大于其执行时间，所以建议是不要单独拆包而是直接内联到index.html中（index.html也是每次打包都会变的，正好）

要实现这样的功能的话可以选用`script-ext-html-webpack-plugin`。

```javascript
const ScriptExtHtmlWebpackPlugin = require("script-ext-html-webpack-plugin");

// 注意一定要在HtmlWebpackPlugin之后引用
// inline 的name 和你 runtimeChunk 的 name保持一致
new ScriptExtHtmlWebpackPlugin({
  //`runtime` must same as runtimeChunk name. default is `runtime`
  inline: /runtime\..*\.js$/
});
```



##### Module vs Chunk

在webpack中chunk是指代码中引用的文件（如js、css、图片等）会根据配置合并成一个或者多个包，合并生成的这个包就称为chunk。

module则是指代码按照功能拆分，分解成离散功能块。拆分之后的代码就叫做module。可以简单地理解为一个export/import就是一个module。



##### HashedModuleIdsPlugin

如果按照默认的webpack配置，给filename设置了chunkhash，然后引入一个新的文件，再打包时会发现有多个文件发生了变化（这些文件的module id都产生变化，而其他的都没变）。

这是因为： **webpack 内部维护了一个自增的 id，每个 module 都有一个 id。所以当增加或者删除 module 的时候，id 就会变化，导致其它文件虽然没有变化，但由于 id 被强占，只能自增或者自减，导致整个 id 的顺序都错乱了。**

这样使用chunkhash作为输出名是不够的，chunk内部都有id，webpack会默认使用递增的数字作为moduleId，如果引入或者删除了一个文件会导致其他文件的moduleId发生变化，那么缓存就失效了。

为了解决这个问题，可以使用`HashedModuleIdsPlugin`，避开使用自增id。

> 或者使用`optimization.moduleIds` [v4.16.0](https://github.com/webpack/webpack/releases/tag/v4.16.0) 新发布，文档还没有。查看 [源码](https://github.com/webpack/webpack/blob/master/lib/WebpackOptionsApply.js#L374)发现它有`natural`、`named`、`hashed`、`size`、`total-size`。这里我们设置为`optimization.moduleIds='hash'`等于`HashedModuleIdsPlugin`。源码了也写了`webpack5`会优化这部分代码。

其原理是使用文件路径作为id，将路径hash之后的值作为moduleId。这样可以确保moduleId的唯一性以及固定性。



##### NamedChunkPlugin

和moduleId同理，chunk id也是需要固定下来的，不然在增减chunk的时候会导致chunk顺序错乱，进而使得缓存失效。

但是这玩意只对有name的chunk生效，也就是异步加载的页面是无效的。

目前有三种解决方案：

* records
* webpackChunkName
* 自定义nameResolver

> [太长了，贴个原文](https://juejin.im/post/5b5d6d6f6fb9a04fea58aabc)

其中使用webpackChunkName的话，在vue中是这样的

```javascript
// Vue的路由表
{
    path: '/test',
    component: () => import(/* webpackChunkName: "test" */ '@/views/test')
},
```

当然这种做法也太繁琐了，相当于每个路由页面都要写一个注释。



于是可以使用 自定义nameResolver 

→ To Be Continue