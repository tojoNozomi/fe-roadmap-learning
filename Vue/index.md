# Vue 复习笔记

### 未分类

##### 组件思想

函数式组件： 就像是纯函数（Pure Function）那样，输入和输出严格对应。

函数组件是无状态的，无法被实例化也没有生命周期和方法

```javascript
<template functional>
    <div class="list">
        <div class="item" v-for="item in props.list" :key="item.id" @click="props.itemClick(item)">
            <p>{{item.title}}</p>
            <p>{{item.content}}</p>
        </div>
    </div>
</template>
```

在父组件里这么使用

```javascript
<template>
    <div>
        <List :list="list" :itemClick="item => (currentItem = item)" />
    </div>
</template>

import List from '@/components/List.vue'
export default {
    components: {
        List
    },
    data() {
        return {
            list: [{
                title: 'title',
                content: 'content'
            }],
            currentItem: ''
        }
    }
}
```





路由解耦组件： 状态或者业务通过路由参数进行解耦。

```javascript
const router = new VueRouter({
    routes: [{
        path: '/user/:id',
        component: User,
        props: true // 这里开启后可以使得组件通过props接受params参数
    }]
})
// 或者通过返回函数的形式返回props
const router = new VueRouter({
    routes: [{
        path: '/user/:id',
        component: User,
        props: (route) => ({
            id: route.query.id
        })
    }]
})

export default {
    props: ['id'], // 在这里注册上
    methods: {
        getParamsId() {
            return this.id
        }
    }
```



### 高阶组件（High Order Component）

##### 概念

 一个函数，接受一个组件以及其他参数，生成的经过包装的组件。高阶组件也是在React中比较流行的思路。在Vue中倒是比较少提及到。

高阶组件主要是用于容器组件和UI组件的解耦合。

> 容器组件： 用于包含UI组件的组件，可以拥有状态并且在生命周期中改变状态，可以传入参数给UI组件。
>
> UI组件： 通过父组件获取到的props进行渲染的组件，自身为无状态的组件。

容器组件和UI组件在Vue里面大概是这样的

```Vue
<Container-Comp>
	<View-Comp />
</Container-Comp>
```



> 简化异步状态管理可以使用基于`slot-scopes`的开源库vue-promised



简单来讲，React中的HOC主要体现为

`f(Class) -> 新的Class`

而Vue中组件多为object，则体现为

`f(object) -> 新的object`

在高阶组件生成函数中，生成的组件要：

1. 生命周期中获取数据
2. 将这些数据作为props传给UI组件



通过HOC可以达到视图和逻辑的分离。其实这也是一种设计模式——装饰器模式



另外不使用Vue的`template`的话，可以使用render函数进行替代，如

```javascript
const HOCGenerator = (wrapped, promiseFunc) => {
  return {
 		data() {...},
    async mounted() {
      // 使用传入的函数获取数据
      const { requestParams } = this.$refs.wrapped
      const result = await promiseFunc(requestParams)
      // etc...
    },
    render(h) {
      const args = {
        props: {
          result: this.result,
          ...this.$attrs, // 透传属性
        },
        on: this.$listeners, // 透传监听器
        scopedSlots: this.$scopedSlots // 透传slot插槽
        ref: 'wrapped',
      }
      return h(wrapped, args)
    }
  }
}
// 这里传入的wrapped 可以是vue单文件组件，也可以是一个obejct
// 这里传入的promiseFunc是一个返回promise的函数，用来模拟异步操作
```

也可以使用JSX语法替代render函数的写法。

在HOC的概念中，非常重要的一点就是容器组件要把外部传入的属性props、监听器listeners、插槽内容scopedSlots全部传给内部的UI组件，容器组件在这里就起到一个桥梁的作用。

而容器组件如何获取到UI组件的实例？ 通过ref就可以拿到。



##### 组合

当然，按照装饰器模式的话，有时候可能需要不止包装一层逻辑，逻辑拆分地很细的情况下，就需要像这样调用高阶组件`a(b(args))`，这么写的话也太麻烦了。

那么要如何解决这个问题呢？ 从React那边借鉴来的思想，当然也是从React全家桶中找答案了。Redux里面有个compose函数就是专门做函数高阶化的。

```javascript
function compose(...funcs) {
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

`compose(a, b, c)`函数会返回这么一个函数，这个函数会嵌套执行abc这几个函数

```javascript
(..args) => a(b(c(...args)))
```

不过要使用compose的话，只能改造高阶组件函数的输入和返回值了。因为在使用compose只能接受一个参数。比如

```javascript
const HOCGenerator = (promiseFunc) => {
  return function wrap(wrapped) {
    return {
			// Vue组件对象的内容
      mounted() {}
    }
  }
}
```

于是使用的时候

```javascript
const composed = compose(HOCGenerator1(promiseFunc), HOCGenerator2)
const hoc = composed(view)
```



##### 在实际应用场景中

Vue-Router中已经帮我们实现了异步加载路由。但是如果我们想在路由加载的时候显示骨架屏或者loading提示的话，Vue中有loading选项可以支持这些操作，而Vue-Router并不能帮我们做这件事。

那么可以通过渲染一个统一的路由加载容器组件，用来包装这部分逻辑

```javascript
const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
})

function lazyLoadView (AsyncView) {
  const AsyncHandler = () => ({
    component: AsyncView,
    loading: require('./Loading.vue').default,
    error: require('./Timeout.vue').default,
    delay: 400,
    timeout: 10000
  })

  return Promise.resolve({
    functional: true,
    render (h, { data, children }) {
      // 这里用 vue 内部的渲染机制去渲染真正的异步组件
      return h(AsyncHandler, data, children)
    }
  })
}
  
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: () => lazyLoadView(import('./Foo.vue'))
    }
  ]
})
```



### 路由懒加载

[route lazy load](https://github.com/vuejs/vue-router/blob/8975db3659401ef5831ebf2eae5558f2bf3075e1/docs/zh-cn/advanced/lazy-loading.md)

如果直接打包构建应用，那么会导致构建产物中JS文件硕大无比，导致页面加载慢。如果能按照不同路由对应的组件进行分割，只有在路由被访问到时才加载对应的组件，那么体验会更好，效率就更高了。

可以结合Vue的异步组件和Webpack的代码分割功能，实现路由组件的懒加载。

首先定义一个异步组件，返回一个Promise的工厂函数（这个promise会resolve组件本身）

```javascript
const Foo = () => Promise.resolve({/*组件定义对象*/})
```

接着在Webpack中，使用动态import语法来定义代码分块点（split point）

```javascript
import('./Foo.vue')
```

如果使用的是Babel，则要添加`syntax-dynamic-import`来正确识别这个语法。

最后就在router.js中，定义路由配置

```javascript
const Foo = () => import('./Foo.vue')
// 路由配置则和其他时候写法一样，没有变化
const router = new VueRouter({
  routes:[{
    path: '/foo',
    component: Foo
  }]
})
```

当需要把某个路由下的所有组件都打包到同一个异步块的时候，可以使用`命名chunk`，一个特殊的注释语法来提供chunk name

```javascript
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```





### Vue常用技巧

##### 路由组件参数解耦



##### 函数式组件



##### 样式穿透



##### $watch的高阶使用



##### 事件参数



##### 自定义组件的双向绑定



##### 事件监听器



##### 手动挂载组件



##### Vuex

* action可以异步，而mutation只能进行同步操作
* 开启严格模式： 在创建store时加入`strict: true`的选项
* state快照的设置： 深拷贝+添加webpack插件



##### element-ui二次开发

* 使用element-theme工具，编辑生成的变量文件即可修改样式

* 需要深层次定制时候，进入node_modules/theme_default/src 下面修改源码（因为et工具通过这个npm包里面的源码进行编译）

* element-theme源码是使用css4

* https://github.com/Molunerfinn/theme-default 二次开发指南