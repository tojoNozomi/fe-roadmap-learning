# 技术视野

### Flutter

以往的跨平台移动应用的解决方案有这么些：

1. 通过对移动平台的webview进行简单的封装（加个壳），使用DOM进行UI的渲染。如PhoneGap（ionic）和Cordova这些技术，在UI还原上能做的很好。当然这种看似很简单方便的技术也有很严重的问题就是性能。业务逻辑性能和渲染性能都很差
2. 将JavaScript转换到原生UI上，从而规避webview性能孱弱的问题。使用这种方案的技术有React Native和weex。不过这种技术也存在一些问题，比如复杂的UI/硬件交互需求，可能需要使用原生的技术去实现（造轮子），分平台进行特定地兼容，效率上也下降了。另外使用JS编写的业务逻辑和UI之间，需要通过JS Bridge去处理，性能也比原生的要差一些。

而Flutter的出现也解决了上面的两个问题，这也是Flutter具有前瞻性的原因。

##### 构成

相比于上面的两种技术依赖于平台实现，Flutter有自己的渲染引擎，可以做到对UI的完全控制（像素级的控制）。像素级的全屏渲染看似很消耗性能，实际上引擎对重绘渲染也有相应的优化。

Flutter有： SKia图形引擎、dart VM

Flutter封装了大量的SDK代码来提高开发的效率

Flutter 和 React很多程度上挺相似的，比如生命周期、状态管理都有React/Redux的影子。

比如Flutter的控件分为Stateful和Stateless，Stateful控件可以使用setState来更改状态。

在React中的虚拟DOM，Diff在Flutter上也有体现。Flutter在内存中保存了组件树（widget tree），在渲染的时候就对比哪些子树出现了变更，然后进行重新渲染。这种算法叫做Structural repainting using compositing

Flutter在输入到渲染会经过这些pipeline：

User input -> Animation -> Build -> Layout -> paint -> Composite -> Rasterize



所以每个flutter控件都有一个build方法来控制该控件的渲染，和React的render方法类似，build方法中不要放计算密集型的代码



因为Flutter 能够自行控制渲染，这相比React Native等方案有着太多的优势。很多时候能很简单地做出原生代码不容易达到的效果。



因为自渲染的原因，跨平台需要做的兼容工作就少很多了。

另外使用了Dart语言，能够支持AOT（Ahead Of Time）和JIT（Just In Time）两种编译模式，这样能够使用JIT支持开发时的组件热更新替换，也能在编译输出的时候编译成高效的ARM或者x86机器码。Dart是一个带有GC的语言，其AOT和Golang相似，会在编译好的执行文件中加入VM里关于内存管理的进行时（runtime），这也是架构中存在runtime的理由。



另外因为Flutter中大部分object生命周期都比较短，适用于young generational hypothesis，所以GC的效率足够高。



在语言语法方面，Dart支持mixin、泛型等高阶功能，也采用了async/await的异步模型，每个Future对象都可以看作一个独立的coroutine。另外Dart作为静态类型语言，类型上和TypeScript倒是挺相似的，类型声明虽然推荐但不强制使用，对开发者来讲倒是挺容易上手的。



[Flutter如何渲染的](https://www.v2ex.com/t/660907#reply0)

