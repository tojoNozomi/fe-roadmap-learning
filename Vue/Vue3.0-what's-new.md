# Vue 3.0 新亮点

> 更好的类型推导能力
>
> 更大的代码压缩空间
>
> 更好的Tree Shaking支持
>
> 更灵活的逻辑复用能力



* 性能优化
* 前置依赖
* 组合模式API  （Composition API）
* 异步依赖
* TS支持优化 （TSX）
* 自定义渲染API



与React Hook 类似

使用Composition API之后就不再推荐Mixin了

Portal -> Teleport

Suspense 基于wait的异步依赖

Vuex： API基本无变动

Nuxt跟进



### Vue3.0 Reactivity API

https://zhuanlan.zhihu.com/p/146097763

##### 副作用

```javascript
import { effect, reactive } from '@vue/reactivity'
// 使用 reactive() 函数定义响应式数据
const obj = reactive({ text: 'hello' })
// 使用 effect() 函数定义副作用函数
effect(() => {
     document.body.innerText = obj.text
})

// 一秒后修改响应式数据，这会触发副作用函数重新执行
setTimeout(() => {
  obj.text += ' world'
}, 1000)
```

使用`reactive`定义一个响应式对象，`effect`则是定义副作用函数，响应式数据和副作用函数之间会产生联系（即依赖的收集），每次响应式对象产生变化时就触发。用于将副作用从函数中抽离出来，让代码更加简洁易懂。



##### 浅响应式数据

`shallowReactive()`用来定义浅响应式数据。由此产生的对象只有第一层属性值会触发副作用函数。



##### 定义只读属性

`readonly()` 和 `shallowReadonly()` 可以定义出只读对象（和浅只读对象），对该对象的修改会报错。



