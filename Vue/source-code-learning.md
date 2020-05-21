# Vue 源码学习笔记

> 来源于《Vue技术内幕》

### Vue的data选项

在一个Vue组件挂载前，会将data选项中上的对象进行遍历，并将使用Object.defineProperty这些选项转换成getter / setter。

getter / setter 对于用户来讲是无感知的，在内部是为了让Vue去追踪依赖，在属性被访问 / 修改的时候就通知变化。

每个组件的实例都有相应的watcher实例对象，会将组件渲染过程中依赖的属性记录为依赖，在依赖项的setter变化时，会通知watch重新计算，更新组件。



### $watch 响应式原理

在组件挂载前对data选项上的对象进行深层次的遍历，并对该对象上每个属性都设置上getter / setter。

其中setter会触发dep中收集的依赖，而getter会触发将该依赖收集到dep中的动作。

data经过def（Object.defineProperty的简单封装）处理之后，会添加上`__ob__`属性，此属性不可枚举。



dep在Vue中有两层含义：

1. 通过闭包引用的dep。在属性值修改时，会触发其中收集的依赖。因为被闭包引用所以在被处理过的data中看不到。
2. `childOb.dep`，也就是`__ob__`中的dep。在使用`$set`或者`Vue.set`给数据对象添加属性的时候触发。（`__ob__`中的dep主要是为了能在添加 / 删除属性的时候有能力触发依赖，也就是`Vue.set`和`Vue.delete`的原理了）

在`defineReactive`函数中，会创建一个被闭包引用的dep，即上面1中的dep。并且在为data属性定义getter / setter时，不会覆盖原先这个属性自带的getter / setter（如果有的话），会被缓存起来，在触发Vue层面的getter / setter时一同调用。



##### get函数如何收集依赖？

get函数中如果已经缓存过原有的getter的话，通过这个函数获取属性的值，这个值会在最后return出去。

而依赖怎么收集呢？

在`$watch`中，会传入要监听的属性字段名，和对应的回调函数。执行的时候会把这个回调函数传给`Dep.target`，可以理解为一个全局变量。然后再触发这个属性的getter。

触发getter之后，getter中有收集依赖的方法，可以将触发watch时传到Dep.target的依赖直接收集到dep实例中。另外还会有算法来阻止重复收集依赖。

```javascript
// 这里简略地实现watch
function watch(exp, fn) {
    Dep.target = fn
    data[exp]
}

// get函数的实现
{
    get: function(val) {
        // 判断val是来源于原有的getter还是本身
        const value = getter ? getter.call(obj) : val
        dep.depend() // 这里就收集到依赖了
        if(childOb) {
            childOb.depend() // 通过Vue.set设置的ob，来收集对应的依赖
        }
    }
}
// depend如何实现？(猜想)
Dep.prototype.depend = function() {
    this.store.push(this.target)
    this.target = null
}
```

收集依赖的触发时机就是当setter被执行时。setter中有一行是通知dep的，如`dep.notify()`。而对于想要在data上新增属性时，可以`Vue.set`。在`Vue.set`中会触发`__ob__.dep.notify()`。因为JS的限制，只使用Object.defindProperty()是无法拦截到给对象添加属性的操作的。（Vue3中使用了Proxy解决了这个问题）



##### set函数如何触发依赖

set函数中会先获取原来的值，然后和传入的newVal进行对比，如果一致的话就直接return掉。

如果两者不相同的话，且原有setter存在的话，先触发原有setter更新值，然后使用`dep.notify()`通知dep，触发依赖执行。

```javascript
// set函数的实现
{
    set: function(newVal) {
        // 获取原来的val
        const value = getter ? getter.call(obj) : val
        if (newVal === val) { // 简单来讲，两个值相同就没有更新的必要，直接返回掉
            return 
        }
        if (setter) { // 原有setter的话，更新到那边，否则直接更新
            setter.call(obj, newVal)
        } else {
            val = newVal
        }
		childOb = !shallow && observe(newVal) // 重写外部的__ob__
        dep.notify()
    } 
}
// 简略的实现？（猜想）
Dep.prototype.notify = function() {
    this.store.map(item => item()) // 触发dep中收集的所有依赖
}
```



### 拦截原有的数组方法

在Vue中要拦截数组变异的方法的话，需要对数组原有的方法进行改写，在不影响原有数组功能的同时，额外执行了诸如通知Vue数组已经发生改变的行为。（算是劫持原型方法吧。）

```javascript
// 要拦截的数组变异方法
const mutationMethods = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]

const arrayMethods = Object.create(Array.prototype) // 实现 arrayMethods.__proto__ === Array.prototype
const arrayProto = Array.prototype  // 缓存 Array.prototype

mutationMethods.forEach(method => {
  arrayMethods[method] = function (...args) {
    const result = arrayProto[method].apply(this, args)

    console.log(`执行了代理原型的 ${method} 函数`)

    return result
  }
})

// 修改这个arr的原型
const arr = []
arr.__proto__ = arrayMethods

arr.push(1)
```

不过在旧IE系列的浏览器上，并不支持`__proto__`属性，那么还可以在数组实例上直接添加变异后的同名函数，这样就能优先执行变异方法了。（原型链）

```javascript
// arraykeys是需要劫持的方法列表
// arrayMethods是已经变异过的方法
arrayKeys.forEach(method => {
  Object.defineProperty(arr, method, {
    enumerable: false,
    writable: true,
    configurable: true,
    value: arrayMethods[method]
  })
})
```

