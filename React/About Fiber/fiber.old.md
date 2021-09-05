<!-- # React简介（一）

### React的架构

---

#### React16之前的架构
- Reconciler（协调器）
  - 负责找出变化的组件
- Renderer（渲染器）
  - 负责将变化的组件渲染到页面上

[Reconciler官方文档的解释](https://zh-hans.reactjs.org/docs/codebase-overview.html#reconcilers)
>即便 React DOM 和 React Native 渲染器的区别很大，但也需要共享一些逻辑。特别是协调算法需要尽可能相似，这样可以让声明式渲染，自定义组件，state，生命周期方法和 refs 等特性，保持跨平台工作一致。
为了解决这个问题，不同的渲染器彼此共享一些代码。我们称 React 的这一部分为 “reconciler”。当处理类似于 setState() 这样的更新时，reconciler 会调用树中组件上的 render()，然后决定是否进行挂载，更新或是卸载操作。
Reconciler 没有单独的包，因为他们暂时没有公共 API。相反，它们被如 React DOM 和 React Native 的渲染器排除在外。

[Renderer官方文档解释](https://zh-hans.reactjs.org/docs/codebase-overview.html#renderers)
>React 最初只是服务于 DOM，但是这之后被改编成也能同时支持原生平台的 React Native。因此，在 React 内部机制中引入了“渲染器”这个概念。<br>
渲染器用于管理一棵 React 树，使其根据底层平台进行不同的调用。

在React中可以通过 ```this.setState ```、```this.forceUpdate```、```ReactDOM.render```等API触发更新.

>this.forceUpdate：默认情况下，当组件的 state 或 props 发生变化时，组件将重新渲染。如果 render() 方法依赖于其他数据，则可以调用 forceUpdate() 强制让组件重新渲染。调用 forceUpdate() 将致使组件调用 render() 方法，此操作会跳过该组件的 shouldComponentUpdate()。但其子组件会触发正常的生命周期方法，包括 shouldComponentUpdate() 方法。如果标记发生变化，React 仍将只更新 DOM。<br><br>
ReactDOM.render()：ReactDOM.render() 会控制你传入容器节点里的内容。当首次调用时，容器节点里的所有 DOM 元素都会被替换，后续的调用则会使用 React 的 DOM 差分算法（DOM diffing algorithm）进行高效的更新。ReactDOM.render() 不会修改容器节点（只会修改容器的子节点）。可以在不覆盖现有子节点的情况下，将组件插入已有的 DOM 节点中。

当每次组件发生更新时，Reconciler（协调器）的工作：
* 调用函数组件或class组件的render方法，将返回的JSX转化为虚拟DOM
* 将虚拟DOM和上次更新时的虚拟DOM进行对比
* 通过对比找出本次更新中变化的虚拟DOM
* 通知Renderer将变化的虚拟DOM渲染到页面上

##### React16之前架构的缺点
在Reconciler中，mount的组件会调用mountComponent，update组件会调用updateComponent。这两个方法都会递归更新子组件，当递归一旦开始，中途就无法中断。当层级很深时，递归更新时间超过了16.67ms（主流浏览器刷新率60HZ），页面就会卡顿。基于这些原因React15 —> React16有很大的变动。如果需要产生不卡顿的效果，那就需要在60HZ内完成更新，而在React应用中能够在16.67ms内完成更新场景极少，解决这个问题的办法就是利用异步可中断的更新来代替同步的更新。

#### React16之后的架构
- Scheduler（调度器）
  - 调度任务的优先级，高优先级任务优先进入Reconciler
- Reconciler（协调器）
  - 负责找出变化的组件
- Renderer（渲染器）
  - 负责将变化的组件渲染到页面上

##### Scheduler（调度器）

以浏览器剩余时间作为任务中断的标准，需要一种机制，当浏览器有剩余时间时通知我们。虽然浏览器提供了这个API requestIdLeCallback。但是因为该API还存在：<br>
1、浏览器兼容性 <br>
2、触发频率不稳定的因素<br>
React的团队实现了功能更加完备的requestIdLeCallback polyfill，这就是Scheduler。

##### Reconciler（协调器）
React15中的版本是递归处理虚拟DOM的，在React16中的Reconciler更新工作从递归变成了可以中断的循环过程。每次循环的过程中都会调用shouldYield判断当前循环是否有剩余时间。在React16中，Reconciler与Renderer不再是交替工作。当Scheduler将任务交给Reconciler后，Reconciler会为变化的虚拟DOM打上代表增、删、更新的标记。整个Scheduler与Reconciler的工作都在内存中进行。只有当所有组件都完成Reconciler的工作，才会统一交给Renderer。

##### Renderer（渲染器）
Renderer根据Reconciler为虚拟DOM打的标记，同步执行对应的DOM操作。在Scheduler和Reconciler过程中步骤随时可能因为以下原因被打断：<br>
1、有其他更高优的任务需要更新<br>
2、当前帧没有剩余时间<br>
由于Scheduler和Reconciler的工作都是在内存中进行，不会更新页面上的DOM，所以即使反复中断，用户也不会看见更新的DOM。

### Fiber架构的[心智模型](https://overreacted.io/zh-hans/how-are-function-components-different-from-classes/)
---
#### [什么是代数效应](https://overreacted.io/zh-hans/algebraic-effects-for-the-rest-of-us/)

代数效应是函数时编程中的一个概念，用于将副作用从函数调用中分离。
>函数的副作用：调用函数时，除了函数返回值外，还对主调用函数产生附加的影响。（例如修改全局变量（函数外的变量）、http请求、数据库的操作等等）

#### 代数效应在React中的应用
代数效应与React之间的关系最明显的例子就是hooks。对于类似useState、useReducer、useRef这样的hook，不需要关注FC的State在hook中是如何保存的，React会处理，只需要知道useState返回的是所需的state，并编写业务逻辑即可。

#### 代数效应与Generator
从React15到React16，协调器（Reconciler）重构的一大目的：将老的同步更新的架构变为***异步可中断更新***。异步可中断更新可以理解为：更新在执行过程中可能会被打断（浏览器时间分片用尽或有更高优先级任务插队），当可以继续执行时恢复之前执行的中间状态。这就是代数效应中try...handle的作用。<br><br>
浏览器中原生支持类似的实现Generator。[但是Generator的一些缺陷使React团队放弃了它](https://github.com/facebook/react/issues/7942)：<br>
>1、类似async，Generator也是传染性的，使用了Generator则上下文的其他函数也需要做出改变。这样心智负担比较重。<br>
2、Generator执行的中间状态是上下文关联的。<br><br>
每当浏览器有空余时间时都会依次执行其中一个doExpensiveWork，当时间用尽则会中断，当再次恢复时会从中断位置继续执行。只考虑“单一优先级任务的中断与继续”情况下Generator可以很好的实现异步可中断更新，但是考虑到“高优先级任务插队”的情况，如果此时已经完成doExpensiveWorkA与doExpensiveWorkB计算出x与y。此时B组件接收到一个高优先级，由于Generator执行的中间状态是上下文关联的，所以计算y时无法复用之前已经计算出的x，需要重新计算。如果通过全局变量的保存之前的中间状态，又会引入新的复杂度。基于以上种种原因，React没有采用Generator实现协调器。

React Fiber可以理解为：<br>
React内部实现的一套状态更新机制。支持任务不同优先级，可中断与恢复，并且恢复后可以复用之前的中间状态。其中每个任务更新单元为React Element对应的Fiber节点。

### Fiber的含义
---
- 从架构来说，React15的Reconciler采用递归的方式执行，数据保存在递归调用栈中。React16的Reconciler基于Fiber节点实现，被称为Fiber Reconciler。
每个Fiber节点有个对应的React element，多个Fiber节点是如何连接成树的？
- 作为静态数据来说，每个Fiber节点对应一个React element，保存了该组件的类型（函数组件/类组件/原生组件等）、对应的DOM信息。
- 作为动态的工作单元，每个Fiber节点保存了本次更新该组件改变的状态、要执行的工作（需要被删除/被插入到页面中/被更新）

在React中最多会同时存在两颗Fiber树。当前在屏幕上显示内容对应的Fiber树称为current Fiber树，正在内存中构建的Fiber树称为workProgress Fiber树。current Fiber树中的Fiber节点被称为current fiber，workInProgress Fiber树中的Fiber节点称为workInProgress fiber，通过alternate属性连接。
```js
// v17.0.2  ReactFiber.old.js -> 256
......
let workInProgress = current.alternate;
  if (workInProgress === null) {
    // react使用双缓存技术。一棵树最多只需要两个版本
    workInProgress = createFiber(
      current.tag,
      pendingProps,
      current.key,
      current.mode,
    );
    workInProgress.elementType = current.elementType;
    workInProgress.type = current.type;
    workInProgress.stateNode = current.stateNode;

    if (__DEV__) {
      // DEV-only fields
      workInProgress._debugID = current._debugID;
      workInProgress._debugSource = current._debugSource;
      workInProgress._debugOwner = current._debugOwner;
      workInProgress._debugHookTypes = current._debugHookTypes;
    }

    workInProgress.alternate = current;
    current.alternate = workInProgress;
    ......
```
React应用根节点通过使current指针在不同Fiber树的rootFiber间切换来完成current Fiber树指向的切换。即当workInProgress Fiber树结构建完成交给Renderer渲染在页面上后，应用根节点的current指针指向workInProgress Fiber树，此时workInProgress Fiber树就变为current Fiber树。每次状态更新都会产生新的workInProgress Fiber树，通过current与workInPorgress的替换，完成DOM更新。

```jsx
export default function App(){
  const [num, setCount] = useState(0);
  return <div>
    <header>
      <img ... />
      <p onClick={() => setCount(num => num  + 1)} ></p>
    </header>
  </div>
}
```
#### mount时
1、首次执行ReactDOM.render会创建fiberRootNode（源码中叫fiberRoot）和rootFiber。其中fiberRootNode是整个应用的根节点，rootFiber是<App />所在组件树的根节点。在React应用中可以多次调用ReactDOM.render渲染不同的组件树，他们会拥有不同的rootFiber。但是整个应用的根节点只有一个，那就是fiberRootNode。fiberRootNode的current会指向当前页面上已渲染内容对应Fiber树，即current Fiber树。

>由于是首屏渲染，页面中还没有挂载任何DOM，所以fiberRootNode.current指向的rootFiber没有任何子Fiber节点（即current Fiber树为空）。

2、接下来进入render阶段，根据组件返回的JSX在内存中依次创建Fiber节点并连接在一起构建Fiber树，被称为workInProgress Fiber树。
在构建workInProgress Fiber树时会尝试复用current Fiber树中已有的Fiber节点内的属性，在首屏渲染时只有rootFiber存在对应的current fiber（即rootFiber.alternate）
图中右侧已构建完的workInProgress Fiber树在commit阶段渲染到页面。此时DOM更新为右侧树对应的样子。fiberRootNode的current指针指向workInProgress Fiber树使其变为current Fiber树。

#### update时
接下来我们点击p节点触发状态改变，这会开启一次新的render阶段并构建一颗新的workInProgress Fiber树。
和mount时一样，workInProgress fiber的创建可以复用current Fiber树对应的节点数据。
workInProgress Fiber树在render阶段完成构建后进入commit阶段渲染到页面上。渲染完毕后，workInProgress Fiber树变为current Fiber树。 -->