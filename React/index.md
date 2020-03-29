# React学习笔记

### 目录

> * 基本api
> * JSX
> * props和state
> * 函数组件和class组件
> * 事件处理
> * 列表/条件渲染
> * 组件之间的交互
> * 代码分割/懒加载
> * 错误处理
> * Context API
> * Ref
> * 高阶组件
> * 整合第三方组件
> * 性能优化



### 基本API

render函数： 用于渲染视图。如返回JSX

constructor函数： 可以在里面定义state和组件的一些变量

props的用法，通常由父组件传入

```react
// 函数组件的写法
const Comp = (props) => <div>114514, {props.name}</div>
// 或者类组件的写法
class Comp extends React.Component {
  render() {
    return <div>114514, {this.props.name}</div>
  }
}
// 父组件这么传props
<Comp name="1919810" />
```

另外定义出来的组件不是单例模式，可以无尽地复用，也可以随意组合。

另外需要注意的是props的只读性，任何情况下你不能修改传入的props。

这里需要提及到 纯函数的概念*



组件简单的话可以分为两类，一种是没有状态的组件，一种是有状态的组件。无状态组件和纯函数一样，推荐直接使用函数去定义；而有状态的组件可以用class去定义，当然配合hook的话能直接在函数中去定义。



组件内state的定义以及使用

```react
class Comp extends React.Component {
	constructor(props) {
		super(props)
		this.state = {
			highLight: false
		}
	}
	componentDidMount() {
		setTimeout(() => {
			this.setState({
				highLight: true
			})
		}, 2000)
	}
	render() {
		return <div>114514, {this.props.name}, {this.state.highLight ? 'HighLight' : 'LowLight'}</div>
	}
}
```

如上，简单的描述了state是怎么定义和使用的。

另外，如果有不参与数据流的字段，可以直接在class中定义为私有变量供使用。顺带提一嘴，class组件应该始终使用props来调用父类的构造函数。

上面也出现了生命周期的钩子函数（hooks），比如下面是两种比较常用的钩子函数：

componentDidMount 在挂载完成之后会调用

componentWillUnmount 在卸载之前会调用



关于react渲染一个组件时的顺序（也就是ReactDom.render() 解析到组件时）

1. 调用组件的构造函数
2. 调用组件的render方法
3. 组件的输出被插入到DOM后，会调用componentDidMount
4. 组件在卸载之前，如果有setState被调用而且state确实被改变了的情况下，

会重新调用组件的render函数，（不会触发componentDidMount，但是有另一个钩子函数被触发，componentDidUpdate）



state使用的注意事项

- 不要直接通过赋值的方式去修改state
- 在构造函数中是state唯一可以被赋值的地方
- 出于性能考虑，setState可能会被合并起来执行，props和state的更新可能是异步的
- this.setState     为避免异步带来的问题，最好是传入函数，如

```react
this.setState((state, props) => {
	return {
		highLight: true
	}
})
```

- 浅合并、深合并 （？）



事件处理

- JSX中事件的命名要用小驼峰命名法，而且要传一个函数作为事件处理函数

```react
<button onClick={handleFunc}>click here</button>
```

- 阻止默认行为要用`e.preventDefault()`
- 为了在回调中使用this，可以在构造函数中定义

```react
this.handleFunc = this.handleFunc.bind(this)
```

如果嫌麻烦的话可以使用Public Class Fields语法

定义的时候直接：

```react
this.handleFunc = () => {
	console.log(this.state)
	// do sth...
}
// 或者在JSX中这么写：
<button onClick={() => this.handleFunc()}>Click!</button>
// 但是并不推荐这么写，因为会创建出不同的回调函数
```

- 若在循环中需要使用到回调函数来传参，可以这样

  ```react
  <button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
  
  <button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
  ```



条件渲染

在函数中可以直接使用 if 的表达式控制返回的内容，达到 Vue 中 v-if 的效果

```react
if (flag) {
	return <div>114514</div>
} else {
	return <div>1919810</div> 
}
// 另外可以使用&& （逻辑与运算符）来实现 v-if 的功能，如
render() {
	return <div>{ this.showFlag && <span>114514</span> }</div>
}
```

同理，三目运算符可以实现 v-if v-else 类似的功能

阻止渲染： render函数返回 null 即可

元素变量： JSX中的元素可以赋值给一个变量



列表中的key

- 将列表渲染为多个元素时，可以使用map来实现如：

```react
const itemList = ['a', 'b', 'c']
	render() {
		return <ul>{
			itemList.map((item, index) => {
				return <li>itemname: {item}</li>
			})
		}</ui>
}
```

- 当渲染组件时（不是元素），需要传入key     用来避免来自React的提醒（这个key应该是用来追踪的，让虚拟DOM和真实DOM能对应上） 

用来识别React中哪些元素改变了，建议使用唯一的字符串（列表内，即在兄弟节点中唯一），key需要在组件外部指定并传入（不是在组件定义的地方）

- key不能作为属性值，如果需要用到key的值的话，可以另外传一个属性值去代替
- 在JSX中可以使用map来渲染列表，但是需要注意的是做好粒度细分，防止嵌套过深，不利于维护



 受控组件

- state成为组件的唯一数据源时，会使得修改和验证用户的输入变得简单

【补充： select上和input的实现】

- select如果上复选的话，则需要给value传入数组。

- 文件类型的input是非受控组件，因为这时的value是只读的，需要通过JS的File API去读取和处理

- 处理多个输入的情况下：     使用一个通用的处理函数，在input中插入name，处理数据时根据name来分情况处理

  ```react
  this.setState({
  	[name]: value // 这里使用了es6的计算属性名称
  })
  ```

  

状态提升（父子组件之间的通信）

- 当需要两个子组件之间state共享时，可以通过状态提升来实现，state则统一由父组件去管理。（MVC）

  【这里需要补充一个】简单来讲： 

  ​	[img]

  （其实这里的render 会让父组件再次调用render，更新自身和两个子组件的视图）

* 在React应用中，任何可变数据应当只能有唯一的“数据源”，需要同步state的时候（两个相邻的组件），即将state提取到共同的父组件中，依靠“自上而下的数据流”。相比于双向绑定的模式，这种单向的数据流虽然烦琐一些，但是易于排查bug，因为仅有组本身能更改state，可使用自定义逻辑或者转换用户输入

  [img]





组合模式： 推荐组合而非继承来实现代码复用

包含关系： 使用children prop 来传递子组件到渲染结果（有点像Vue中的slot）

```react
	// Comp1组件中render函数
	<div>
		{ props.children }
	</div>
	// 父组件中对Comp1的children属性进行传值
	<Comp1>
		<span>114514</span>
	</Comp1>
// 上面props.children能获取到 span元素 
```

同样的如果需要用到类似Vue的具名slot那样的功能的话，可以：

```react
	<Comp1 
		left={CompChild1}
		right={CompChild2}
	/>
	// 类似于Vue的
	<Comp1 slot="left" />
	<Comp1 slot="right" />
	
	<slot name="left"></slot>
	<slot name="right"></slot>
```

*实际上React元素的本质上是Object（对象），可以当作props来传

- 特例关系： 基于组合，类似给定值的函数 【这段是干嘛的？】
- 继承：     UI上仅通过props和组合即可完成多种情况下的复用，基本上不需要用到继承来实现代码复用。对于需要功能复用的，直接通过提取为公共方法或者库来调用
- 高阶组件： 和组合有点像 【待整合整理】



语义化

- html的语义化：aira-xxx  React中的语义化写法 aira-label={labeltext}
- 无障碍表单：     React中使用htmlFor="这里填表单输入框的id"
- 焦点的管理：需要父组件的某个子组件被focus时，可以使用refs获取到该子组件的DOM，就能直接使用到DOM的API
- 复杂组件的设计模式【？】
- 鼠标事件【？】



代码分割

Export / import 语法 同Vue（都基于webpack）

Next.js / Nuxt.js

Babel： babel-plugin-syntax-dynamic-import 插件



React 懒加载

```react
// 原先
import Comp from './comp'
// 使用后
const Comp = React.lazy(() => import('./comp'))
```

使用懒加载的React组件应该用Suspense组件去包裹，fallback里面放一个loading指示器就挺好的

```react
//  动态引入
<suspense fallback={<span>这里是等待加载中渲染出来的元素/组件， 加载完了之后就没这个元素/组件的事了</span>}>
	<TargetComp />
</suspense>
```





异常捕获边界

- 模块加载出现错误时（例如网络问题），会抛出错误。异常捕获边界会捕获到错误，并根据此来处理，保障用户体验 /     管理恢复事宜
- 【错误捕获边界】？
- 一般来讲，ui的渲染错误不应该导致整个应用崩溃，使用错误捕获边界可以渲染出备用的UI
- 无法捕获到的情况：
  1. 事件处理中抛出的错误
  2. 异步代码中抛出的错误
  3. 服务器渲染抛出的错误
  4. 自身抛出的错误（不是子组件）

```react
	static getDerivedStateFromError() {
	// 定义如何处理错误
	}
	componentDidCatch() {
	// 定义如何处理错误
}

	// 可以上报错误和打印日志，或者修改state（state中标志出错了的字段）
	render() {
		if (this.state.hasError) {
			return <span>The App got Wrong</span>
		}
		// render element
	}
```

- 一般错误边界放在哪？     需要根据组件的粒度来确定，或者统一放在顶层的路由组件，或者在单独组件中（比如频繁网络请求、频繁计算、或者用户交互的地方）
- React 16之后，未被捕获的错误会导致整个React树被卸载
- 错误追踪： 组件栈追踪（dev模式下默认开启，production模式必须关闭）
- 另外，try / catch     是命令式代码，不太适合React这种声明式的框架
- 事件处理器出现的错误可以用 try /     catch来捕获



基于路由的代码分割？ 有点像Vue-router的【这里详细了解后作辨析】

命名导出

Keyword： Tree Shaking



Context API

- 类似于props透传（但是不用写的麻烦），不用逐层传递 

  ```react
  	// 定义
  	const ThemeContext = React.createContent()
  	// JSX里面，父组件
  	<ThemeContext.Provider value="dark">
  	<Comp />
  	</ThemeContext.Provider>
  	// 子组件里
  	static contextType = ThemeContext
  console.log(this.context)
  ```

  另外，要谨慎使用context，这会让复用性变差



- 为了避免深层次的嵌套，可以将子组件的子组件一起拿到父组件中传递下去
- 结合children prop，可以覆盖不少情况



Ref

- 在当前组件中使用Ref

  ```react
  // 定义
  this.textInput = React.createRef()
  // 直接进行DOM操作
  this.textInput.current.focus()
  // 在render中指定哪个元素要ref
  render() {
  	return <input ref={this.textInput} />
  }
  ```

  推荐使用ref的场合：

  1. 焦点管理、文本选择或者媒体播放
  2. 触发强制动画
  3. 集成第三方DOM库

  - 高阶组件的ref 需要做     forwardRef 来转发
  - 注意不能滥用ref



Fragments

用于返回多个节点（而不产生额外的父节点）（主要在props.children中使用？）

```react
<React.Fragment>
	// 这里是多个节点，可以带key
</React.Fragment>
// 短语法写法，此时不支持key 或者属性：
<></>
```



高阶组件

以组件为参数，返回值也是组件的（类似高阶函数的概念）

将相似的逻辑抽象出来，实现逻辑的共享

（有高阶组件了就不用mixins了）

*注意这里不能修改传入的组件

【装饰者模式？】

一种实现： 

```react
	function addDataGetAblitity (Comp, data) {
		return class extends React.Component {
			// 获取数据等操作
			// state管理等
			render() {
				return <Comp data={data} /> // 这里的comp 应为纯函数
			}
		}
	}
```

约定1： 无关的props应该透传给被包裹的组件

```react
render() {
	const { extraProp, ...passThroughProps } = this.props
	return (<WrapComp injectedProp={injectedProp} {...passThroughProps} />)
}
```

- 约定2：最大化可组合性

（大概是 拆得越散越好？）

*函数式编程的核心： 数据的映射（输入->输出）

- 约定3:     包装显示名称，以便于轻松调试（在React Devtools中）

 

- 需要注意的是，不要在render方法中使用高阶组件，diff算法在更新的时候会卸载前一个子树，再重新渲染（浪费性能），而且状态会全部丢失

在render和部分生命周期钩子函数之外调用可以避免这个问题

在高阶组件的构造中，务必复制静态方法，不然高阶组件会获取不到原先的静态方法

```react
function enhance (WrapComp) {
	return class Enhance extends React.Component {
		constructor() {
			Enhance.staticMethod = WrapComp.staticMethod
		}
		render() {}
	}
}
```

另一种方法就是通过import / export 将静态方法给导出来

高阶组件不会转发原先的ref，需要使用forwardRef来实现



与第三方库的协同

- React不会理会React之外的DOM操作，这样可能会造成冲突，比如React更新视图后将DOM给替换了（原先第三方库的DOM操作就被覆盖了）

这里可以使用ref 来让其他的第三方DOM操作库 进入React组件的生命周期中

也就是需要 手动地维护这些操作了



4、JSX语法：

```react
const element = <div>114514</div>
```

这是一种JS的语法扩展，并不是HTML

可以嵌入表达式 {} ，一对花括号包含住

在中间可以是函数（的返回值）、表达式

JSX中的元素属性值可以使用花括号，填入表达式来计算获得

JSX的本质实际上是语法糖，通过Babel将JSX代码转换成React的API 用来渲染DOM



JSX深入学习

a、React应该在JSX的作用域内（无论组件有没有直接用到React的API）

不打包，直接用script标签引入的情况下则要确保React是全局变量

b、JSX类型中使用点语法

```react
const comp = {
	subcomp: () => <span>114514</span>
}
 // 那么可以这样用
<Comp.subcomp />
```

c、用户定义的组件要以大写字母开头

d、不能将表达式作为React的元素类型（即在运行时选择类型）

```react
const type = {
'a': Comp1,
'b': Comp2
}
<type['a'] /> // 这样会报错的
```

应该用一个变量将结果计算出来 （这个变量的首字母要大写）

e、关于JSX的props的值

（1）、JS表达式可以作为props的值（运算符ok，但是流程控制不可以）

（2）、字符串字面量可以

（3）、如果没有赋值，则默认为true

（4）、属性展开 

```react
<Comp {...injectProps} />
```

f、关于JSX的子元素的值

（1）、字符串字面量

（2）、JSX子元素（就套娃呗）

（3）、JS表达式（用花括号包起来）

（4）、函数（还是用花括号包起来嗷）

（5）、Boolean类型值、undefined、null会被忽略

（6）、巧用 && 和三目运算符等等



29、 Portals

用于将React子元素 渲染到其他DOM节点上，类似于 对话框组件，悬浮卡 的场景

这里的事件冒泡： 按照React树往上走，而不是DOM树

 

30、Profiler API

用来测试 渲染一个React应用要多久以及付出的代价【好像React Devtools支持这个】

 

31、协调 Diff 过程

- 调用一次render函数后会产生一棵React元素的UI树，下一次state或者props变更后，要再次调用相同的render方法，产生一棵不同的UI树，需要判断两棵UI树的差异，并判断如何高效地更新视图。用常规的方法的话     时间复杂度为O(n的三次方)
- 启发式算法的时间复杂度为O(n)

a、两个不同类型的元素会产生不同的树

b、开发者可以通过key（prop）暗示哪些元素在不同的渲染下保持稳定

- diff算法

a、根节点为不同类型的元素时，直接拆卸原有的树并重建。拆卸会导致对应的DOM节点销毁，执行componentWillUnmount方法

建立新树则会有 componentWillMount -> componentDidMount

所有与之前相关的state会被销毁

b、对比相同元素：保留当前节点，仅对比有更新的属性。当前节点处理完了就继续对子节点进行递归

c、对比相同类型的组件： 当一个组件更新后实例不变的情况下，state在跨越不同渲染保持一致。React会更新props以保持一致，然后调用componentWillReceiveProps和componentWillUpdate。完了之后调用render方法，接着将之前的结果和新的结果进行递归

d、对子节点进行递归： 递归DOM节点的子元素时，React会遍历前后两个子元素列表，当产生差异时，生成一个mutation

（1）、自动在尾部插入会比较节省资源

（2）、在头部插入会比较浪费资源 【这里涉及到重绘和重排（回流）？ 该学一学嗷】

所以差异只出现在头部的话，直接全列表进行mutate会更快

用key能解决（key能快速判断哪些元素是新来的，就移动哪些元素，key在列表中要确保唯一性）

（不推荐使用数组下标作为key，改变排序（key）顺序会造成重排重绘影响性能）

一棵子树能在其兄弟之间移动，其他情况会造成重新渲染

- 建议1: 两个不同组件之间的频繁切换，而输出内容近似的情况下，建议改成同一类的组件
- 建议2:     key应当稳定，可预测，以及确保列表中的唯一性
- 不稳定的（如随机生成的）key会导致许多组件和DOM节点被不必要地重新创建



Render Props

利用props传函数，来实现代码共享

```react
<DataProvider render={data => (<span>hello {data.message}</span>)} />
```

Render props接受渲染函数后，在自身render()中也把传入的函数给执行了

可嵌套 能和高阶组件配合

注意： 这只是一种模式，并没有限制prop的名字

其他

- 静态类型： FLOW /     PropType （与Vue类似） 可作类型检查
- 严格模式：     有助于检查出不安全的生命周期 / 识别出过时API / 过时的context API / 检测出意外的副作用
- 非受控组件： 与受控组件相反：     表单数据由DOM处理，取值可以通过ref获取

默认值： 在input标签中 指定defaultValue

另外 文件类型的input始终都是非受控组件