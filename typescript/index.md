# TypeScript 学习笔记

### 目录

> 1. 前置知识
> 2. TS的编译上下文、声明空间、模块、命名空间、动态导入表达式
> 3. 类型系统、类型注解
> 4. React JSX中使用TS
> 5. 编译选项
> 6. 常见错误以及处理方式
> 7. 工具链
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

##### 从JS迁移到TS

主要步骤：

> * 添加一个tsconfig.json的文件
> * 将文件扩展名从`.js` 改成 `.ts`，开始使用any来减少错误
> * 开始使用TS写代码，尽量减少`any`的使用
> * 回到旧代码，开始添加类型注解，并修复已识别的错误
> * 为第三方JS代码定义环境声明

减少错误

刚开始迁移的时候，TS会进行类型检查，这样会出现大量的类型错误，可以使用类型断言来减少此类错误。可以使用any进行类型断言或者类型注解

第三方JS代码的处理

因为不可能所有的代码都改成TS的，比如一些第三方库类。那么可以使用`vendor.d.ts`文件来作为针对特定库的声明文件。

```typescript
declare var $: any
// 或者显式地使用注解、并且在类型声明空间中用到它
declare type JQuery = any
declare var $: LQuery

// 引入第三方npm模块的话
declare module 'jquery'
// 在使用的时候引入
import * as $ from 'jquery'

// 引入非js资源
declare module '*.css'
// 这样就可以使用下面的语句来引入css了
import * as foo from './foo.css'
```



##### @types

可以通过安装`@types`，来添加声明文件。添加了之后可以直接在支持的编辑器上开启代码类型提示

例如

```bash
npm install @types/jquery --save-dev
```

 默认情况下TS会自动包含支持全局使用的任何定义。而通常情况下还是推荐使用模块。在安装了模块`@types`后，不需要进行特别的配置就能直接像使用模块那样使用

另外在`tsconfig.json`中能对编译选项的types选项配置有意义的类型作为全局定义。



##### 环境声明

可以通过`declare`关键字来告知TS你试图表述一个已在其他地方存在的代码

```typescript
declare var foo: any
foo = 123
```

这些声明可以放在`.ts`或者`.d.ts`文件中（推荐放在独立的`.d.ts`文件中）。

带有`.d.ts`的文件中每个根级别的声明都必须以`declare`关键字作为前缀，这里的代码就不会被TS编译成其他代码，而且开发者需要确保所声明的内容在编译的时候存在。

有些全局变量已经有社区维护的`.d.ts`文件，你并不需要重复进行声明，建议使用接口。接口允许其他人扩充这些全局变量，并且告诉TS有关这些变量的修改。

```typescript
// 添加一个函数到process里的例子
interface Process {
	exitWithLogging(code?: number): void;
}
process.exitWithLogging = function() {
  console.log('exiting')
  process.exit.apply(process, arguments)
}
```



##### 接口

接口在很多情况下和内联注解是等效的声明。接口方便在可以轻松地添加成员到现有的声明中。

TS接口是开放式的，它允许你使用接口来模仿JS的可扩展性。

使用类来实现接口吧

```typescript
interface Point {
  x: number;
  y: number;
}
class MyPoint implements Point {
  x: number;
  y: number;
}
```

在`implements`存在的情况下，外部接口的任何更改都会导致代码编译报错，所以可以确保两者的同步。也就是说接口会限制类实例的结构。

接口旨在声明JS中可能存在的任意结构。



##### 枚举

```typescript
// 枚举类型的例子
enum CardSuit {
  Clubs,
  Diamonds,
  Hearts,
  Spades
}
let Card = CardSuit.Clubs
Card = '123' // 报错，string类型的数据不能赋值给CardSuit类型
```

上面这些枚举类型的值都是数字类型的，因此称为数字枚举。

```typescript
enum Color {
  Red = 1, // 1
  Green,   // 2
  Blue     // 3
}
```



枚举中默认从0开始（通常建议从1进行初始化，这样能让你在枚举类型值中安全可靠地检查）

枚举有一个很好的用途就是用来作为标记。可以用来检查一组条件中的某个条件是否为真。

```typescript
// 比如一个权限的表，可以参考Linux中的文件权限
enum Perssion = {
	None = 0,
  Read = 1 << 0,
  Write = 1 << 1,
  Exec = 1 << 2
}
// 通过左移的位运算符，得到这些数字 001, 010, 100（对应1，2，4）
// 当使用枚举标记的时候，位运算符将带来很大的帮助(|或 &与 ~非)
// 使用 |= 添加一个标记
// 组合使用 &= 和 ~ 来清除一个标记
// 使用 | 来合并标记
```



而枚举类型不止数字枚举，还有字符串枚举、常量枚举。



##### lib.d.ts

当安装TS的时候，会顺带安装一个`lib.d.ts`的声明文件。这个文件包含了JS运行时以及DOM中存在的各种常见的JS环境声明。

这个文件会自动包含在TS项目的编译上下文中，能让你快速开始书写经过类型检查的JS代码。

编译目标的更改会导致声明文件中包含更多的环境声明。

想要解耦编译目标和环境库支持之间的关系的话，可以使用`--lib` `--target`选项或者使用`tsconfig.json`中`compilerOptions` 的`lib`选项和`target`选项。如果需要使用polyfill的话可以引入`core-js`包来支持



##### 函数

比如下面的函数有着参数注解、返回值注解（这个是可选的，不写的话可以根据编译器来推断）

```typescript
function foo (sampleParam: {bar: number}): number {
  return sampleParam.bar
}
```

可选参数、参数默认值：

```typescript
// 使用? 可以标记为可选出参数
function foo (bar: number, bas?: string): void {
  console.log(bar, bas)
}
// 也可以给参数设置默认值
function foo (bar: number, bas: string = 'bas') {
  console.log(bar, bas)
}
```

TS的函数支持重载（略）



声明函数

```typescript
type Foo = {
  (a: number): number;
}
type Bar = (a: number) => number
```

 这里的两种写法是等效的，但是想要使用函数重载的话只能使用第一种方式。



你可以使用类型别名或者接口来表示一个可以被调用的类型注解

```typescript
interface ReturnString {
  (): string
}
declare cosnt foo: ReturnString
const bar = foo() // bar会被推断为字符串

// 一个相对复杂的可被调用的类型注解
interface Com {
  (foo: string, bar?: number, ...others: boolean[]): number
}
```

箭头函数的注解

```typescript
const simple: (foo: number) => string = foo => foo.toString()
```

可实例化的例子

```typescript
interface Bar {
  new (): string
}
// 用例
declare const Foo: Bar
const boo = new Foo() // boo会被推断为string类型
```



##### 类型断言

```typescript
interface Foo {
  a: number;
  b: string;
}
const foo = {} as Foo
foo.a = 1
foo.b = 'b'

const bar = {}
bar.a = 1 //  报错，因为bar被推断为{}，不能为它添加属性a或b
bar.b = 'b' // 报错
```

使用类型断言可以规避上面的报错。不过使用断言之后，如果忘了某个属性，编译的时候不会报错，需要规避这种情况。

这个时候可能需要创建一个临时变量来通过类型推断的检查。

双重断言，例如 

```typescript
function handle(event: Event) {
  const el = (event as any) as HTMLElement
}
```

TS如何确定单个断言是否足够

基本上，当S类型是T类型的子集，或者T类型是S类型的子集时，S能够被成功断言成T。这也是为了在进行类型断言时的安全性而考虑的，毫无根据的断言是危险的，如果你想这么做，可以使用any。

##### Freshness

这是用于方便检查对象的字面量类型，确保对象字面量在结构上类型兼容。

例如，React中就很好地使用了Freshness。这可能需要给所有成员都标记为可选，这样就能捕获到拼写错误了。

```typescript
interface State {
  foo?: string;
  bar?: string;
}
this.setState({foo: 'abc'}) // 可以通过类型检查
this.setState({foos: 'abc'}) // 报错，对象只能指定已知的属性
this.setState({foo: 123}) // 报错，不能将number类型赋值给string类型
```



##### 类型保护

typeof 使用了这个关键字之后，TS会自动推断类型是否符合，以及是否拼写错误

instanceof 一个判断实例是否来自一个类。

in 安全地检查一个对象上是否存在一个属性

也可以使用`==` `===` `!==` `!=` 这些来区分字面量类型

自定义类型保护

```typescript
interface Foo {
  foo: number;
  common: string;
}
interface Bar {
  bar: number;
  common: string;
}
// 用户自定义的类型保护
function isFoo(arg: Foo | Bar): arg is Foo {
  return (arg as Foo).foo !== undefined
}
// 用例
function doStuff(arg: Foo | Bar) {
  if (isFoo(arg)) {
    console.log(arg.foo) // 正确
    console.log(arg.bar) // 错误
  } else {
    console.log(arg.foo) // 错误
    console.log(arg.bar) // 正确
  }
}

```



类型保护和回调函数

TS不能假设类型保护在回调中一直有效的，所有这么假设是危险的。

可以使用临时变量将推断的安全值存放起来，防止被外部更改。

```typescript
declare var foo: {bar?: {baz: string}}
function immediate(callback: () => void) {
	callback()
}

// 类型保护
if (foo.bar) {
  console.log(foo.bar.baz) // 正确
  someFunc(() => {
    console.log(foo.bar.baz) // TS错误： 对象可能为undefined
  })
}

// 类型保护
if (foo.bar) {
  console.log(foo.bar.baz) // 正确
  const bar = foo.bar
  someFunc(() => {
    console.log(bar.baz) // 正确
  })
}

```



##### 字面量类型

可以将字符串字面量当作一个类型来使用，虽然不是很实用。

```typescript
let foo: 'Hello'
foo = 'bar' //错误： bar不能赋值给Hello类型
```



##### readonly

可以在一个接口中使用readonly，能以一种更安全的方式工作

```typescript
function foo (config: {readonly bar: number, readonly bas: number}) {
  // ...
}
const config = {bar: 1, bas: 2}
foo(config)
// 这样能确保config不会再改变了

type Foo = {
  readonly bar: number;
  readonly bas: number;
}
const foo: Foo = {bar: 1,bas: 2}
foo.bar = 123 // 错误： foo.bar是只读属性
```

readonly和const的区别：

前者用于属性，而因为别名的原因，该属性可以被改变

后者用于变量，不能被重新赋值



##### 泛型

设计泛型的关键动机是在成员之间提供有意义的类型约束，这些成员可以是类的实例成员、类的方法、函数的参数、函数返回值

假设JS实现了一个队列，可以传入和弹出任何类型的数据。想要限制出入的数据的类型的话，可以创建一个特殊类用来约束。但是这么做的话需要不少成本来修改原有的代码。

而使用泛型的话，可以很简单地完成这件事。

```typescript
class QueueNumber {
  private data = []
  push = (item: number) => this.data.push(item)
  pop = (): number => this.data.shift()
}
const queue = new QueueNumber()
queue.push(1) 
queue.push('1') // 报错，不能放入string类型的数据

// 使用泛型
class Queue<T> {
  private data: T[] = []
  push = (item: T) => this.data.push(item)
  pop = (): T | undefinded => this.data.shift()
}
// usage
const queue = new Queue<number>()
queue.push(0)
queue.push('1') // 报错，不能放入string类型的数据，只能放入number类型的
```



设计模式

```typescript
declare function parse<T>(name: string): T
// 等同于下面的类型断言
declare function parse(name: string): any
const sth = parse('sth') as TypeOfSth
```

仅使用一次的泛型并没有类型断言安全。

再来一个例子

```typescript
const getJSON = <T>(config: {url: string; headers?:{[key: string]: strimg}}): Promise<T> => {
  const fetchConfig = {
    method: 'GET',
    Accept: 'application/json',
    'Content-Type': 'application/json',
    ...(config.headers || {})
  }
  return fetch(config.url, fetchConfig).then<T>(response.json())
}

type LoadUserResponse = {
  user: {
    name: string;
    email: string;
  }[];
}
function loaderUser() {
  return getJSON<LoadUserResponse>({url: 'http://example.com/users'})
}
```

使用`Promise<T>`相比`Promise<any>`要好就是前后是匹配的。

另外泛型可以被用于函数参数

```typescript
declare function send<T>(arg: T): void
send<stm>({
  x: 123,
  // 能自动补全
})
```



##### 类型推断

TS可以根据一些简单的规则来推断（然后检查）变量的类型。

* 定义变量

  定义两个不同类型的变量之后，将其中一个赋值给另外一个变量时会报错

* 函数返回类型

  自动根据return 返回的数据判断

* 赋值

* 结构化

  用字面量定义了一个对象，其中的属性的类型会自动识别并检查类型

* 解构

  使用解构赋值得到的变量也会识别/检查



##### 类型兼容性（略）

协变： 只在同一个方向兼容

逆变： 只在相反的方向兼容

双向协变： 双向兼容

不变： 如果类型不同，则不兼容



##### never

TS需要一个可靠的类型来代表哪些永远不会发生的事情

never类型是TS的底部类型，有这么些例子：

* 一个从来不会有返回值的函数
* 一个总会抛出错误的函数

你也可以将never作为类型注解，但never类型只能被另外一个never类型赋值。

和void的区别： 当一个函数返回空值，它会返回一个void；但当一个函数根本没有返回值，或者总抛出错误时，它会返回一个never



##### 联合类型

定义了联合类型之后，作类型检查的时候就以能推断出类型是来自联合类型中的哪一个

```typescript
interface Square {
  kind: 'square';
  size: number;
}
interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}
type Shape = Square | Rectangle // 联合类型的定义
```



##### 索引签名

使用字符串访问JS中的对象，并保存对其他对象的引用。

当向索引签名传入一个其他对象时JS会先调用`toString`方法

而在TS中，索引签名会限定在number和string类型上（Symbol类型也是可以的），如果要传入对象的话需要显式带上`toString()`，其中有个理由就是默认执行的`toString` 非常糟糕，如V8引擎总是返回`[object Object]`。



声明一个索引签名可以使用下面的这种结构

```typescript
const foo: {
  [index: string]: { message: string };
}
foo['a'] = { message: 'abc' } // 正确
foo['a'] = { messages: 'abc' } // 报错，提示要包含message
console.log(foo['a'].message) // 正确
console.log(foo['a'].messages) // 报错，提示messages不存在
```

上面代码中的index除了提高可读性之外是没有意义的。比如这个数据结构是存储用户名的，可以这么写`[username: string]: {message: string}`

number类型的索引可以这么写：`[const: number]: Type`，这里的Type可以替换成你想要的类型

```typescript
interface Foo {
  [key: string]: number;
  x: number; // 这里的类型只能是number，和上面一致
  y: number; // 换成其他类型的话会报错
}

// 使用一组有限的字符串字面量
type Index = 'a' | 'b' | 'c'
type FromIndex = {
  [k in Index]?: number
}
const good: FromIndex = {
  b: 1,
  c: 2
}
const bad: FromIndex = {
  b: 1,
  c: 2,
  d: 3
} // 错误，对象字面量只能指定已知类型，d并不存在于FromIndex类型上
```

同时拥有string和number类型的索引签名可以这么定义

```typescript
interface ArrStr {
  [key: string]: string | number;
  [index: number]: string;
  length: number
}
```



##### 类型移动

如果需要移动一个类，你可能尝试这么做

```typescript
class Foo {}
const Bar = Foo
let bar: Bar //报错： 找不到名称Bar
```

这里报错是因为const只是把Foo复制到了一个变量声明空间，但并不能当作一个了类型注解使用。正确方式是使用`import`关键字。

```typescript
namespace importing {
  export class Foo {}
}
 import Bar = importing.Foo
let bar: Bar // 正确
```

如果需要捕获一个变量的类型，可以这么做

```typescript
let foo = 123
let bar: typeof foo
bar = 456 // 正确
bar = '123' // 报错： string不能赋值给number类型

// 捕获类成员的类型
class Foo {
  foo: number;
}
declare let _foo: Foo
let bar: typeof _foo.foo
```



##### 异常处理

JS中有一个`Error`类，可以通过`throw`关键字来抛出错误，并通过 `try/catch`来捕获错误。

错误子类型

* RangeError 当数字类型的变量或参数超出有效范围时，报的错
* ReferenceError 引用无效
* SyntaxError 解析到无效的JS代码（多是语法错误）
* TypeError 变量或参数不是有效类型
* URIError `encodeURI()` 或者`decodeURI`传入无效参数



##### 混合（略）

JS和TS中，类只支持严格的单继承，不能实现多重继承。



### JSX

在TS里如何使用JSX？

> * 使用后缀为.tsx的文件
> * 在tsconfig.json中的compilerOption中设置jsx选项为react
> * 安装依赖 @types/react @types/react-dom
> * 在tsx文件中导入react： import * as React from 'react'

函数组件的定义

```tsx
type Props = {
  foo: string;
}
const myCom: React.FunctionComponent<Props> = props => {
  return <span>{props.foo}</span>
}
<MyCom foo="bar" />
```

类组件的定义

```tsx
type Props = {
  foo: string;
}
class MyCom extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}
<MyCom foo="bar" />
```

React类型声明文件中提供了`React.ReactElement<T>`，可以通过你传入的`<T />`，来注解类组件的实例化结果

```tsx
// 使用了上面的组件
const foo: React.ReactElement<MyCom> = <MyCom />
```

泛型组件

```tsx
// 一个泛型组件
type SelectProps<T> = { item: T[] }
class Select<T> extends React.Component<selectProps<T>, any> {}

// 用例
const Form = () => <Select<string> items={['a', 'b']} />
```

泛型函数

```tsx
function foo<T>(x: T): T {
  return x
}
// 并不能直接使用箭头函数
// 这里报错了，提醒T标签没闭合
const foo = <T>(x: T) => T        
```

想要使用的话用extends来提示编译器这是个泛型

```tsx
const foo = <T extends {}>(x: T) => x
```



### TS编译选项

`noImplicitAny`用于标记出无法被推断的类型（未指明类型而且无法推断的话就报错），开启后不会被自动标为any



`strictNullChecks` 开启后会区别出null和undefined，两者不能互相赋值



### TS错误处理

常见的错误

* TS2304 Cannot find xxx  未声明
* TS2307 Cannot find module 'xxx' 把第三方库当作模块使用了，并且缺少对应的环境声明文件
* TS1148
* 捕获不能有类型注解的变量。 需要使用类型保护



### 开发&测试工具

在Jest中使用TS

> * 安装Jest包
> * 安装@types/jest
> * 安装TS预处理器ts-jest。这能让Jest动态转化成TS，并添加内置的source-map
> * 写入devDependencies

配置Jest

```javascript
module.exports = {
  'roots': [
    '<rootDir>/src' // 指定目录
  ],
  'transform': {
    '^. + \\.tsx?$': 'ts-jest' // 告知Jest使用ts-jest来处理ts/tsx文件
  }
}
```

运行测试

``` sh
# 运行观察模式
npm test --watch
# 普通运行
npm test
```

使用Jest的理由

* 内置断言库
* 强大的TS支持
* 可靠的测试观察模式
* 快照测试
* 内置覆盖率报告
* 内置async/await支持



Prettier 代码格式化工具

Husky 防止错误提交、推送以及其他问题。也可以配合Prettier使用

ESlint 也可以用于TS



### 编译原理（略）

