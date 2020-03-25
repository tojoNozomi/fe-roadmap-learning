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

