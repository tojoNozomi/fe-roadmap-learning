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



### 高阶组件

