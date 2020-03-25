# TypeScript 学习笔记

### 目录

> 1. 前置知识
> 2. TS的编译上下文、声明空间、模块、命名空间、动态导入表达式
> 3. 类型系统、类型注解
> 4. React JSX中使用TS
> 5. 编译选项
> 6. 常见错误以及处理方式
> 7. 工具链
> 8. 最佳实践（小技巧、代码风格）
> 9. 编译原理



### 前置知识

使用TS需要准备哪些工具？

* TypeScript编译器，可以在npm上面获取
* 编辑器，可以使用VS Code



为何使用TS？

* 可以使用类型系统、提高代码可维护性和质量
* 利于重构和debug
* 文档上有优势（比如函数输入输出的参数、中间的变量都标有类型）
* TS是JS的超集



优势？

* JS和TS无缝衔接，TS下可以直接使用JS
* 类型可以是显式/隐式的（写不写类型，TS都会进行类型检查）
* 类型可以是结构化的（定义interface）
* 类型错误不会影响JS的正常运行

* 支持JS的新特性（ES6+），以后新增的ECMAScript标准都会添加进来



数字的安全使用： 可以考虑使用big.js来规避一些问题

检查是否为NaN： 使用Number.isNaN(NaN)，不能使用NaN === NaN



类：

静态属性： static 类中定义了的静态属性可以被所有实例共享

访问修饰符： public protected private

抽象：abstract 类中定义的抽象函数不能直接被调用（直接被实例化），需要在子类中实现这个功能



### TypeScript项目构成

##### 编译上下文

类似于ts的配置文件，可以用来做文件分组，区分哪些文件有效/无效的。也可以设置一些编译选项。

比如这些选项都可以设置

* ECMAScript的目标版本、使用的模块规范（如CommonJS或者AMD这些）
* 输出的文件、目录路径
* 类型检查的一些配置
* source map的相关配置



##### 声明空间

主要有类型声明空间和变量声明空间

类型声明空间包含用来作为类型注解的内容，如下

```typescript
class Foo {}
interface Bar {}
type Bas = {}
// 使用这些类型作为类型注解
let foo: Foo
let bar: Bar
let bas: Bas
```

变量声明空间则是用于将类作为变量来传递/使用

```typescript
class Foo {}
const val = Foo
```

而声明出来的变量，并不能用作类型注解



##### 模块

因为TypeScript支持使用全局命名空间，那么可以做到这样的事：

在一个ts文件中定义一个全局变量，然后在另外一个ts文件中可以直接使用到这个变量。这样是很危险的，很容易造成命名冲突。

为避免这种情况，推荐使用文件模块来规避命名冲突。

比如

```typescript
// a.ts
export const foo = 114514;

// b.ts
import { foo } from './a'
console.log(foo)
```

因为支持的不少的模块规范，所以选择上还是有一定讲究的，比如使用CommonJS和es6 module这些。

模块路径： 相对模块路径和动态查找

前者可以根据路径规则直接指定引入的文件

后者是指按照模块解析的策略自动搜索模块所在位置。



##### 命名空间

es5时代使用命名空间的方式

```javascript
(function(something){
  something.foo = 114514
})(something || (something = {}))
```

而TS则是使用`namespace`来作为定义命名空间的关键字

```typescript
namespace Util {
  export function log (msg) {
    console.log(msg)
  }
  export function error (msg) {
    console.log(msg)
  }
}
  // usage
Util.log('114')  
Util.error('514')
```

编译成js 之后就和上面es5的命名空间写法是差不多了

##### 动态导入表达式（暂略过？）

这是ECMAScript的新功能，允许你在程序的任意位置异步地加载一个模块。

webpack里面可以使用`import()` 和`require.ensure()`来实现代码分割



### 类型系统

##### 注解

比如下面的代码使用了变量、函数参数、函数返回值的类型注解

```typescript
// 基本类型的注解，有number string boolean等
const num: number = 123
function foo(num: number): number {
  return num
}
```



而数组也可以使用注解，如

```typescript
let arr: boolean[];
arr = [true, false]
```



还有个概念是接口，它能将多个类型注解合并成一个类型注解，如

```typescript
// 看起来像是针对对象用的
interface Name {
  first: string;
  last: string
}
let name: Name
name = {
  first: 'kooji'，
  last: 'tadokoro'
}
```



与接口注解不太一样，可以使用内联类型注解。

对于出现次数不多的情况可以使用这种方式去注解；如果多次出现的话还是选择接口注解会好一些

```typescript
let name: {
  first: string;
  last: string
}
name = {
  first: 'kooji',
  last: 'tadokoro'
}
```



除了基础类型之外，TS还提供了特殊类型，如： `null` 、 `undefined`、 `void`、  `any`

1. any

   可以用于兼容所有类型，相当于把类型检查关闭。在迁移JS代码到TS的时候会非常方便，但应该尽量减少使用这种类型。

2. null 和 undefined

   这两种类型的字面量可以被赋值给任意类型的变量

3. void

   用来表示一个函数没有返回值，如

   ```typescript
   function foo(msg: string): void {
     console.log(msg)
   }
   ```



和其他强类型语言一样，TS中有泛型的概念。不少数据结构和算法不会依赖于对象的实际类型，但是你依旧想对每个变量进行约束。如接受一个数组并进行反转，这里的约束就介于传入函数和函数的返回值之间

```typescript
function reverse<T>(items: T[]): T[] {
	let result = []
	for (let i = items.length - 1; i >= 0; i--) {
    result.push(items[i])
  }
  return result
}

const arr = [1,2,3]
const result = reverse(arr)
console.log(result) // 3, 2, 1

result[0] = '1' // 报错
result[1] = 2 // 正确
```

也就是说，当传入元素为number的数组，返回的数组的元素也都为number，这种情况下的类型是安全的。经过这样的函数输出的值是不能在变更类型的

当希望属性是多种类型之一时，可以使用联合类型注解，即使用 `|` 作为类型注解，例如一个字符串格式化的函数：

```typescript
function foo(msg: string[] | string): string {
  let result = ''
  if (typeof msg === 'string') {
    result = msg.trim()
  } else {
    result = msg.join(' ').trim()
  }
  return result
}
```

在JS中会经常用到extends这种模式。在这种模式下，可以根据两个对象来创建一个新的对象，新对象会拥有两个对象所有的功能。在TS里你可以使用交叉类型来安全地使用这种模式，如：

```typescript
function extend<T, U>(first: T, second: U): T & U {
  const result = <T & U>{}
  for (let id in first) {
    (<T> result)[id] = first[id]
  }
  for (let id in second) {
    if (!result.hasOwnProperty(id)) {
      (<U>result)[id] = second[id]
    }
  }
  return result
}
const x = extend({a: 114}, {b: '514'})
console.log(x) // {a: 114, b: '514'}
```

元组类型（略）

为了方便注解的使用，可以使用类型别名来表示注解，如：

```typescript
type StrOrNum = string | number
let a: StrOrNum
a = 114
a = '514'

// 可以为任意的类型注解提供类型别名，这是一些实例
type Text = string | {text: string}
type Coordinates = [number, number]
type Callback = (data: string) => void
```

需要注意的是，如果类型注解有层次结构的话，应该使用`interface`。它能使用`implement` 和 `extends`。