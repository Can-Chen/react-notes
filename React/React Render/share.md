
>基于React@17.0.2

## React首次渲染执行流程

### [渲染器](https://zh-hans.reactjs.org/docs/codebase-overview.html#renderers)
---

>渲染器用于管理一棵 React 树，使其根据底层平台进行不同的调用。

在React开发的项目中，react有对应不同版本的渲染器（Renderer）。有负责浏览器渲染的<b>ReactDOM</b>、渲染APP的<b>ReactNative</b>这两种常见的。还有<b>ReactTest</b>输出纯js对象进行测试、<b>ReactArt</b>渲染到Canvas、svg或VML（IE8）。

### React启动模式

react应用有三种启动模式，官网上对于[这3种启动模式的介绍](https://zh-hans.reactjs.org/docs/concurrent-mode-adoption.html#why-so-many-modes), 基本说明如下:

1. legacy 模式: `ReactDOM.render(<App />, rootNode)`. 这是当前 React app 使用的方式. 这个模式可能不支持[这些新功能(concurrent 支持的所有功能)](https://zh-hans.reactjs.org/docs/concurrent-mode-patterns.html#the-three-steps).
   
2. [Blocking 模式](https://zh-hans.reactjs.org/docs/concurrent-mode-adoption.html#migration-step-blocking-mode): `ReactDOM.createBlockingRoot(rootNode).render(<App />)`. 目前正在实验中, 它仅提供了concurrent模式的小部分功能, 作为迁移到concurrent模式的第一个步骤.
  ```js
   // BolckingRoot
   // 1. 创建ReactDOMRoot对象
   const reactDOMBolckingRoot = ReactDOM.createBlockingRoot(
     document.getElementById('root'),
   );
   // 2. 调用render
   reactDOMBolckingRoot.render(<App />); // 不支持回调
   ```
3. [Concurrent 模式](https://zh-hans.reactjs.org/docs/concurrent-mode-adoption.html#enabling-concurrent-mode): `ReactDOM.createRoot(rootNode).render(<App />)`. 目前在实验中, 未来稳定之后，打算作为 React 的默认开发模式. 这个模式开启了所有的新功能.

   ```js
   // ConcurrentRoot
   // 1. 创建ReactDOMRoot对象
   const reactDOMRoot = ReactDOM.createRoot(document.getElementById('root'));
   // 2. 调用render
   reactDOMRoot.render(<App />); // 不支持回调
   ```

在这里我主要分享的是以ReactDOM.render启动的方式，也就是legacy模式.<br><br> 

ReactDOM.render做了什么工作？

> ReactDOM.render() 会控制你传入容器节点里的内容。当首次调用时，容器节点里的所有 DOM 元素都会被替换，后续的调用则会使用 React 的 DOM 差分算法（DOM diffing algorithm）进行高效的更新。<br><br>
> 不会修改容器节点（只会修改容器的子节点）。可以在不覆盖现有子节点的情况下，将组件插入已有的 DOM 节点中。<br><br>

render方法接收三个参数分别为所需要渲染的节点信息（element）、容器（一般为document.getElement("id")）、渲染成功或者更新后的回调（callback）。<br><br>

第一个参数为渲染所需的节点信息element。而在日常中编写的代码都是采用JSX语法的形式进行编码的，那JSX是什么东西呢？<br><br>

>[官网的描述](https://react.docschina.org/docs/introducing-jsx.html)。它是一个 JavaScript 的语法扩展。我们建议在 React 中配合使用 JSX，JSX 可以很好地描述 UI 应该呈现出它应有交互的本质形式。JSX 可能会使人联想到模板语言，但它具有 JavaScript 的全部功能。

JSX在编译时会被Babel编译为React.createElement方法。当然JSX并不是只能被编译成React.createElement，这与你配置的babel插件有关。

```jsx
// v17之前 显式声明React
// @babel/plugin-transform-react-jsx
<div className="Can-Chen" >
  <a href="https://github.com/Can-Chen" >个人介绍</a>
</div>
```

```js
// 经过babel编译后后成下面这样的结构
React.createElement(
  "div",
  {className: "Can-Chen"},
  React.createElement(
    "a",
    {href: "https://github.com/Can-Chen"},
    "个人介绍"
  ))
```

createElement主要处理了传入React节点类型、属性、children，最后调用ReactElement方法，返回一个包含组件数据的对象<br><br>

```js
export function createElement(type, config, children) {

  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  if (config != null) {
    // 处理config 并赋值给props
  }

  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
  //  处理children
  }

  // Resolve default props
  if (type && type.defaultProps) {
  //  处理default props
  }

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}

const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 标记这是个 React Element
    $$typeof: REACT_ELEMENT_TYPE,

    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner,
  };

  return element;
};
```

下图代表render方法执行的所有的调用栈
----
![render_process](/React/assets/img.Render/render_process.png)

在图中我以Green代表绿框、Red代表红框、Blue代表蓝框。
> 1. Green在第一次执行React.render()方法时调用执行。
> 2. Red在第一次执行React.render()和更新都会执行。
> 3. Blue代表着开始把需要挂载的内容渲染到页面上。

render方法。省略部分代码<br>

很简洁，主要就是调用`legacyRenderSubtreeIntoContainer`方法
```js
export function render(
  element: React$Element<any>,
  container: Container,
  callback: ?Function,
) {

  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
};
```

legacyRenderSubtreeIntoContainer方法。省略部分代码<br>

在`legacyRenderSubtreeIntoContainer`方法中就会对这次执行进行判断。首次进入到该方法中时，因为在react的容器container中还未初始化react应用的环境，所以`container._reactRootContainer`返回的root字段为undefined，需要对root进行初始化，创建ReactDOMRoot对象，初始化react应用环境。从以下代码可以看出首次加载和更新都有调用`updateContainer`方法，该方法是在首次加载时不需要进行批量更新。

```js
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  container: Container,
  forceHydrate: boolean,
  callback: ?Function,
) {

  let root: RootType = (container._reactRootContainer: any);
  let fiberRoot;
  if (!root) {
    // 首次调用
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    
    // 更新容器，先调用unbatchedUpdates，更改执行上下文legacyUnbatchedContenxt，之后调用updateContainer进行更新.
    unbatchedUpdates(() => {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {

    // 更新会走这一步流程
    updateContainer(children, fiberRoot, parentComponent, callback);
  }
  return getPublicRootInstance(fiberRoot);
}
```

关于root的定义（省略部分代码）
```js
export type RootType = {
  render(children: ReactNodeList): void,
  unmount(): void,
  _internalRoot: FiberRoot,
  ...
};
```

跟踪`legacyCreateRootFromDOMContainer`.最后调用new ReactDOMBlockingRoot(container, LegacyRoot, options);

```js
function legacyCreateRootFromDOMContainer(
  container: Container,
  forceHydrate: boolean,
): RootType {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);

  return createLegacyRoot(
    container,
    shouldHydrate
      ? {
          hydrate: true,
        }
      : undefined,
  );
}

export function createLegacyRoot(
  container: Container,
  options?: RootOptions,
): RootType {
  // LegacyRoot固定值常量
  return new ReactDOMBlockingRoot(container, LegacyRoot, options);
}
```
所以在`legacy`模式下调用`ReactDOM.render`有两个核心步骤：
1. 创建`ReactDOMBlockingRoot`对象，初始化react应用环境。
2. 调用`updateContainer`进行更新<br><br>

ReactDOMBlockingRoot（省略部分相关代码）

```js
function ReactDOMBlockingRoot(
  container: Container,
  tag: RootTag,
  options: void | RootOptions,
) {
  // 创建fiberRoot对象，并将其挂载到this._internalRoot上
  this._internalRoot = createRootImpl(container, tag, options);
}

ReactDOMBlockingRoot.prototype.render = function(
  children: ReactNodeList,
): void {
  const root = this._internalRoot;

  updateContainer(children, root, null, null);
};

ReactDOMBlockingRoot.prototype.unmount = function(): void {
  const root = this._internalRoot;
  const container = root.containerInfo;
  updateContainer(null, root, null, () => {
    unmarkContainerAsRoot(container);
  });
};
```
`ReactDOMBlockingRoot`的特性

1. 调用`createRootImpl`创建`fiberRoot`对象, 并将其挂载到`this._internalRoot`上.
2. 原型上有`render`和`umount`方法, 且内部都会调用`updateContainer`进行更新.

### 创建fiberRoot对象（省略部分代码）.

```js
function createRootImpl(
  container: Container,
  tag: RootTag,
  options: void | RootOptions,
) {

  // 创建fiberRoot
  const root = createContainer(container, tag, hydrate, hydrationCallbacks);
  
  // 标记DOM对象，把DOM对象和fiber对象关联起来
  markContainerAsRoot(root.current, container);
  const containerNodeType = container.nodeType;

  return root;
}
```

创建fiberRoot对象，创建`react`应用的首个`fiber`对象.

```js
export function createContainer(
  containerInfo: Container,
  tag: RootTag,
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
): OpaqueRoot {
  // 创建fiberRoot对象
  return createFiberRoot(containerInfo, tag, hydrate, hydrationCallbacks);
}

export function createFiberRoot(
  containerInfo: any,
  tag: RootTag,
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
): FiberRoot {
  // 创建fiberRoot对象
  const root: FiberRoot = (new FiberRootNode(containerInfo, tag, hydrate): any);

  // 创建了`react`应用的首个`fiber`对象, 称为`HostRootFiber`
  const uninitializedFiber = createHostRootFiber(tag);
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;
  // 初始化HostRootFiber的updateQueue
  initializeUpdateQueue(uninitializedFiber);

  return root;
}
```

在创建`HostRootFiber`时，其中`fiber.mode`属性，会与3种`RootTag`(`ConcurrentRoot`,`BlockingRoot`,`LegacyRoot`)关联

```js
export function createHostRootFiber(tag: RootTag): Fiber {
  let mode;
  if (tag === ConcurrentRoot) {
    mode = ConcurrentMode | BlockingMode | StrictMode;
  } else if (tag === BlockingRoot) {
    mode = BlockingMode | StrictMode;
  } else {
    mode = NoMode;
  }
  
  // mode属性是由RootTag决定的
  return createFiber(HostRoot, null, null, mode);
}
```

`fiber`树中所节点的`mode`都会和`HostRootFiber.mode`一致(新建的 fiber 节点, 其 mode 来源于父节点),所以**HostRootFiber.mode**非常重要, 它决定了以后整个 fiber 树构建过程. 此时`react`应用的初始化完毕。

### 更新的入口函数`updateContainer`（省略部分代码）

```js
function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function,
): Lane {
  // current类型为Fiber，指向的是rootFiber（Fiber树的跟节点）
  const current = container.current;
  // 获取当前时间戳
  const eventTime = requestEventTime();
  // 创建一个优先级变量(车道模型)，计算本次更新的优先级
  const lane = requestUpdateLane(current);

  // 根据车道优先级, 创建update对象
  const update = createUpdate(eventTime, lane);

  //设置update对象的payload, 这里element需要注意(是tag=HostRoot特有的设置, 指向<App/>) tag=0
  update.payload = {element};

  // 将新建的一个update对象加入的更新队列中（链表结构） fiber.updateQueue.pending队列
  enqueueUpdate(current, update);

  // 进入reconciler运作流程中的`输入`环节
  scheduleUpdateOnFiber(current, lane, eventTime);

  return lane;
}
// 省略部分代码
function createUpdate(eventTime: number, lane: Lane): Update<*> {
  const update: Update<*> = {
    eventTime, // 创建update的当前时间
    lane, // 调度优先级

    tag: UpdateState, // 状态标记
    payload: null, // 
    callback: null,

    next: null, // next指针
  };
  return update;
}
```
* 获取本次更新的优先级
* 初始化`current`对象的`updateQueue`队列
* 调度和更新`current`(`HostRootFiber`)对象


### reconciler阶段(render过程)
---
#### scheduleUpdateOnFiber 省略部分代码
```js
function scheduleUpdateOnFiber(
  fiber: Fiber,
  lane: Lane,
  eventTime: number,
) {
  // 检查最大更新深度 50次 超过抛出错误
  checkForNestedUpdates();

  // 首次挂载传入的fiber为根fiber节点，fiber.return 为null tag为3 === HostTag(根节点)
  // 返回了fiber.stateNode
  const root = markUpdateLaneFromFiberToRoot(fiber, lane);

  // // legacy下, lane===Sync
  if (lane === SyncLane) {
    if (
      // 检查是否在批处理中
      (executionContext & LegacyUnbatchedContext) !== NoContext &&
      // 检查是否还没有渲染
      (executionContext & (RenderContext | CommitContext)) === NoContext
    ) {
      // 如果是本次更新是同步的，并且当前还未渲染，意味着主线程空闲，并没有React的
      // 更新任务在执行，那么调用performSyncWorkOnRoot开始执行同步任务
      performSyncWorkOnRoot(root);
    } else {
    }
  } else {
  }
}

function performSyncWorkOnRoot(root) {

  let lanes;
  let exitStatus;

  // 初次挂载 workInProgressRoot为空
  if (
    root === workInProgressRoot &&
    includesSomeLane(root.expiredLanes, workInProgressRootRenderLanes)
  ) {
  } else {
    lanes = getNextLanes(root, NoLanes);
    // 传入FiberRoot对象, 执行同步render
    exitStatus = renderRootSync(root, lanes);
  }

  // render结束之后, 设置fiberRoot.finishedWork(指向root.current.alternate)
  const finishedWork: Fiber = (root.current.alternate: any);
  root.finishedWork = finishedWork;
  root.finishedLanes = lanes;
  commitRoot(root);

  // 再次对fiberRoot进行调度，退出之前保证fiberRoot没有需要调度的任务
  ensureRootIsScheduled(root, now());

  return null;
}
```
* 调用`performSyncWorkOnRoot`方法
* 该方法中会分别执行render和commit步骤

#### renderRootSync（省略部分代码）

```js
function renderRootSync(root: FiberRoot, lanes: Lanes) {
  // 缓存executionContext 描述当前在react执行栈中位置
  const prevExecutionContext = executionContext;

  // 位运算 更新在react执行栈中位置
  executionContext |= RenderContext;

  if (workInProgressRoot !== root || workInProgressRootRenderLanes !== lanes) {
    // 为当前render设置全新的stack，并设置公共变量workInProgress,workInProgressRoot,workInProgressRootRenderLanes等
    prepareFreshStack(root, lanes);
    startWorkOnPendingInteractions(root, lanes);
  }

  do {
    try {
      // 执行工作循环
      workLoopSync();
      break;
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);

  //在React生成执行之前调用，以确保“readContext”`
  //无法在渲染阶段之外调用。
  resetContextDependencies();

  // 重置该变量
  executionContext = prevExecutionContext;

  // 表明当前并没有进行的渲染
  workInProgressRoot = null;
  workInProgressRootRenderLanes = NoLanes;

  return workInProgressRootExitStatus;
}
```
* 更新`executionContext`
* `prepareFreshStack`重置工作空间, 避免先前工作循环中的变量污染.
* `workLoopSync()`. 执行工作循环

##### workLoopSync 工作循环
```js
function workLoopSync() {
  // 第一次render, workInProgress=HostRootFiber
  // 循环执行 performUnitOfWork, 这里的workInProgress是从FiberRoot节点开始,依次遍历直到所有的Fiber都遍历完成
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
```

performUnitOfWork（省略部分代码）
```js
function performUnitOfWork(unitOfWork: Fiber): void {

  // 第一次render时，unitOfWork = HostRootFiber，alternate已经初始化
  const current = unitOfWork.alternate;

  let next;
  if (enableProfilerTimer && (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    // 创建Fiber节点
    next = beginWork(current, unitOfWork, subtreeRenderLanes);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork(current, unitOfWork, subtreeRenderLanes);
  }

  unitOfWork.memoizedProps = unitOfWork.pendingProps;
  if (next === null) {
    // 深度优先的方式，到达深节点后，先进行completeWork，下一步可能进行兄弟节点的beginWork或继续completeWork父节点
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }

  ReactCurrentOwner.current = null;
}
```

performUnitOfWork对当前传入Fiber节点开始, 进行深度优先循环处理.<br><br>

把workLoopSync的调用逻辑全部串联起来.
![work-loop](../../React/assets/img.Render/work-loop.jpg)

与工作循环相关主要有4个主要函数：

* `performUnitOfWork(unitOfWork: Fiber): void`
* `beginWork(current: Fiber | null, workInProgress: Fiber, renderLanes: Lanes,): Fiber | null`
* `completeUnitOfWork(unitOfWork: Fiber): void`
* `completeWork(current: Fiber | null, workInProgress: Fiber, renderLanes: Lanes,): Fiber | null`

每个 Fiber 对象的处理过程分为 2 个步骤:
1. `beginWork(current, unitOfWork, subtreeRenderLanes)`, `diff`算法在这里实现(由于初次 render 没有需要进行比较的对象, 都是新增, 正式的`diff`在`update`阶段)

  `beginWork(current, unitOfWork, subtreeRenderLanes)`, DOM差分算法在这里实现(由于初次 render 没有需要进行比较的对象, 都是新增, 正式的`diff`在`update`阶段)
   - 根据 reactElement 对象创建所有的 Fiber 节点, 构造 Fiber 树形结构(根据当前 Fiber 的情况设置`return`(父级)和`sibling`(兄弟)指针)
   - 给当前 Fiber 对象设置`effectTag`标记(二进制位, 用来标记 Fiber 的增,删,改状态)
   - 给抽象类型的 Fiber(如: class )对象设置`stateNode`(此时: `fiber.stateNode=new Class()`)

  `completeUnitOfWork(unitOfWork)`, 处理 beginWork 阶段已经创建出来的 Fiber 节点.
   - 给 Fiber 节点(tag=HostComponent, HostText)创建 DOM 实例, fiber.stateNode 指向这个 DOM 实例,
   - 为 DOM 节点设置属性, 绑定事件(这里不做详细说明).
   - 把当前 Fiber 对象的 effects 队列添加到父节点 effects 队列之后, 更新父节点的`firstEffect`, `lastEffect`指针.
   - 根据 beginWork 阶段设置的`effectTag`判断当前 Fiber 是否有副作用(增,删,改), 如果有, 需要将当前 Fiber 加入到父节点的`effects`队列, 等待 commit 阶段处理.

##### beginWork（省略部分代码）
```js
function beginWork(
  // 当前组件对应的Fiber节点在上一次更新时的Fiber节点，即workInProgress.alternate
  current: Fiber | null,
  // 当前组件对应的Fiber节点
  workInProgress: Fiber,
  // 优先级相关
  renderLanes: Lanes,
): Fiber | null {
  const updateLanes = workInProgress.lanes;

  // 初次渲染 只有rootFiber存在currentFiber树，其余的都为null
  // 先不用看更新
  if (current !== null) {
  } else {
    didReceiveUpdate = false;
  }

  workInProgress.lanes = NoLanes;

  switch (workInProgress.tag) {
    case IndeterminateComponent: {
      return mountIndeterminateComponent(
        ...
      );
    }
    case LazyComponent: {
      const elementType = workInProgress.elementType;
      return mountLazyComponent(
        ...
      );
    }
    case FunctionComponent: {
      ...
      return updateFunctionComponent(
        ...
      );
    }
    // 类组件
    case ClassComponent: {
     ...
      return updateClassComponent(
        ...
      );
    }
    // 根节点
    case HostRoot:
      return updateHostRoot(current, workInProgress, renderLanes);
    // 普通节点
    case HostComponent:
      return updateHostComponent(current, workInProgress, renderLanes);
    // 文本节点
    case HostText:
      return updateHostText(current, workInProgress);
      ...
}
```
这个函数是针对所有的 Fiber 类型, 其中的每一个 case 处理一种 Fiber 类型.

`updateXXX`函数(如: updateHostRoot, updateFunctionComponent 等)的主要逻辑有 3 个步骤:

1. 收集整合当前 Fiber 节点的必要状态属性(如: state, props)
   - 更新当前 Fiber 的`effectTag`
2. 获取下级`reactElement`对象
   * class 类型的 Fiber 节点
   - 构建`React.Component`实例,
     - 设置`fiber.stateNode`指向这个新的实例
     - 执行`render`之前的生命周期函数
     - 执行`render`方法, 获取下级`reactElement`
   - 更新当前节点的`effectTag`
   * function 类型的 Fiber 节点
   - 执行 function, 获取下级`reactElement`
     - `Fiber.memoizedState`指向`hook`队列
     - 初始化`Fiber.memoizedState`队列中的每一个`hook`对象, 使其拥有独立的`memoizedState`
   - 更新当前节点的`effectTag`
   * HostComponent 类型(如: div, span, button 等)的 Fiber 节点
   - `pendingProps.children`作为下级`reactElement`
     - 如果下级节点是文本节点,则设置下级节点为 null. 准备进入`completeUnitOfWork`阶段
   - 更新当前节点的`effectTag`
   * ...
3. 生成`Fiber`子树
   - `diff`算法, 设置子树 Fiber 节点的`effectTag`

updateHostComponent（省略部分代码）
```js
function updateHostComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
) {
  pushHostContext(workInProgress);

  if (current === null) {
    // return void
    tryToClaimNextHydratableInstance(workInProgress);
  }

  // 收集当前Fiber节点的必要状态属性
  // 节点类型
  const type = workInProgress.type;
  const nextProps = workInProgress.pendingProps;
  const prevProps = current !== null ? current.memoizedProps : null;

  let nextChildren = nextProps.children;
  // 判断当前子节点是否只存在一个文本节点，如果是并不会对这个子文本节点生成fiber节点
  const isDirectTextChild = shouldSetTextContent(type, nextProps);

  if (isDirectTextChild) {
    nextChildren = null;
  } else if (prevProps !== null && shouldSetTextContent(type, prevProps)) {
    // 从文本子节点转换为正常的字节点，需要把文本节点重置
    workInProgress.flags |= ContentReset;
  }

  // 更新当前fiber的effectTag
  markRef(current, workInProgress);
  // 生成fiber子树
  reconcileChildren(current, workInProgress, nextChildren, renderLanes);
  return workInProgress.child;
}
```

updateHostText（省略部分代码）
```js
function updateHostText(current, workInProgress) {
  if (current === null) {
    tryToClaimNextHydratableInstance(workInProgress);
  }
  return null;
}
```
很显然，不管是在更新还是挂载阶段，对文本节点不会进行特殊的处理，直接就返回了。
<br>

updateHostRoot（省略部分代码）

```js
function updateHostRoot(current, workInProgress, renderLanes) {
  pushHostRootContext(workInProgress);

  // 收集整合当前Fiber节点的必要状态属性(例如: state, props)
  const updateQueue = workInProgress.updateQueue;
  const nextProps = workInProgress.pendingProps;
  const prevState = workInProgress.memoizedState;
  const prevChildren = prevState !== null ? prevState.element : null;

  // clone一个updateQueue, 分离current和workInProgress对updateQueue的引用.
  // (以前是同一个引用, clone之后引用不同)方便后面processUpdateQueue
  cloneUpdateQueue(current, workInProgress);

  // 处理updateQueue,设置workInProgress的memoizedState,lanes等属性
  processUpdateQueue(workInProgress, nextProps, null, renderLanes);
  const nextState = workInProgress.memoizedState;

  // 获取下级的reactElement对象, 用于生成Fiber子树(HostRoot比较特殊, 直接拿到初始的react对象)
  const nextChildren = nextState.element;
  if (nextChildren === prevChildren) {
    resetHydrationState();
    return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
  }
  const root: FiberRoot = workInProgress.stateNode;
  if (root.hydrate && enterHydrationState(workInProgress)) {
  } else {
    // 生产fiber子树，diff算法，设置子树fiber节点的effectTag
    reconcileChildren(current, workInProgress, nextChildren, renderLanes);
    resetHydrationState();
  }
  return workInProgress.child;
}
```

reconcileChildren（省略部分代码）<br><br>
首次加载进入mountChildFibers，而mountChildFibers方法与reconcileChildFibers方法区别就只是传参的区别，而这个bool的状态代表着本次reconciler是否需要处理副作用。
```js
export const reconcileChildFibers = ChildReconciler(true);
export const mountChildFibers = ChildReconciler(false);
```

```js
function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderLanes: Lanes,
) {
  // 初次挂载的时候 只有根节点会存在current
  if (current === null) {
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderLanes,
    );
  } else {
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes,
    );
  }
}
```
ChildReconciler（省略部分代码）<br><br>
协调阶段子节点fiber的生成。

```js
// 传入bool，由于是初次挂载，传入为false
function ChildReconciler(shouldTrackSideEffects) {
  function deleteChild(returnFiber: Fiber, childToDelete: Fiber): void {
    // 不存在需要处理的副作用，直接返回
    if (!shouldTrackSideEffects) {
      // Noop.
      return;
    }

    ...
  }

  ...
  超级多的处理😁，全部省略。
  ...
  
  function reconcileChildFibers(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    newChild: any,
    lanes: Lanes,
  ): Fiber | null {

    const isUnkeyedTopLevelFragment =
      typeof newChild === 'object' &&
      newChild !== null &&
      newChild.type === REACT_FRAGMENT_TYPE &&
      newChild.key === null;
    if (isUnkeyedTopLevelFragment) {
      newChild = newChild.props.children;
    }

    const isObject = typeof newChild === 'object' && newChild !== null;

    if (isObject) {
      switch (newChild.$$typeof) {
        // 如果是react.element节点，对这个进行处理
        case REACT_ELEMENT_TYPE:
          return placeSingleChild(
            reconcileSingleElement(
              returnFiber,
              currentFirstChild,
              newChild,
              lanes,
            ),
          );
          ...
          ...
      }
    }

    // 文本节点
    if (typeof newChild === 'string' || typeof newChild === 'number') {
      return placeSingleChild(
        reconcileSingleTextNode(
          returnFiber,
          currentFirstChild,
          '' + newChild,
          lanes,
        ),
      );
    }

    // 数组类型，比如说一个element包裹多个element，虽然是数组类型，但是生成的还是当前第一个fiber节点。
    if (isArray(newChild)) {
      return reconcileChildrenArray(
        returnFiber,
        currentFirstChild,
        newChild,
        lanes,
      );
    }

    ...
    省略
    ...
  }

  return reconcileChildFibers;
}
```

completeUnitOfWork 省略部分代码

```js

function completeUnitOfWork(unitOfWork: Fiber): void {

  let completedWork = unitOfWork;
  do {

    const current = completedWork.alternate;
    // 保存父级fiber节点
    const returnFiber = completedWork.return;

    if ((completedWork.flags & Incomplete) === NoFlags) {

      let next;

      if (
        !enableProfilerTimer ||
        (completedWork.mode & ProfileMode) === NoMode
      ) {
        next = completeWork(current, completedWork, subtreeRenderLanes);
      } else {
        // 记录开始时间
        startProfilerTimer(completedWork);
        // 构建dom节点
        next = completeWork(current, completedWork, subtreeRenderLanes);
        // 更新渲染持续时间
        stopProfilerTimerIfRunningAndRecordDelta(completedWork, false);
      }

      if (next !== null) {
        // completeWork过程产生了新的Fiber节点, 退出循环, 回到performUnitOfWork阶段
        workInProgress = next;
        return;
      }

      if (
        returnFiber !== null &&
        // 兄弟节点未完成
        (returnFiber.flags & Incomplete) === NoFlags
      ) {
        // 收集当前Fiber节点以及其子树的副作用effects
        // 把所有子树的effects和当前Fiber的effects添加到父节点的effect队列当中去
        if (returnFiber.firstEffect === null) {
          returnFiber.firstEffect = completedWork.firstEffect;
        }
        if (completedWork.lastEffect !== null) {
          if (returnFiber.lastEffect !== null) {
            returnFiber.lastEffect.nextEffect = completedWork.firstEffect;
          }
          returnFiber.lastEffect = completedWork.lastEffect;
        }

        // 如果当前Fiber有side-effects, 将其添加到子节点的side-effects之后.
        const flags = completedWork.flags;

        if (flags > PerformedWork) {
          if (returnFiber.lastEffect !== null) {
            returnFiber.lastEffect.nextEffect = completedWork;
          } else {
            returnFiber.firstEffect = completedWork;
          }
          returnFiber.lastEffect = completedWork;
        }
      }
    } else {
    }

    // 切换下一个需要处理的Fiber节点
    // 同级节点
    const siblingFiber = completedWork.sibling;
    if (siblingFiber !== null) {
      workInProgress = siblingFiber;
      return;
    }
    // 父级节点
    completedWork = returnFiber;
    workInProgress = completedWork;
  } while (completedWork !== null);

  // 所有Fiber处理完成, 设置全局状态为RootCompleted, workLoopSync结束
  if (workInProgressRootExitStatus === RootIncomplete) {
    workInProgressRootExitStatus = RootCompleted;
  }
}
```

completeWork 省略部分代码
```js
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps;

  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case FunctionComponent:
    case ForwardRef:
    case Fragment:
    case Mode:
    case Profiler:
    case ContextConsumer:
    case MemoComponent:
      return null;
    case ClassComponent: {
      const Component = workInProgress.type;
      if (isLegacyContextProvider(Component)) {
        popLegacyContext(workInProgress);
      }
      return null;
    }
    // tag为3的根节点
    // 此时workInProress.child.child.stateNode指向已经构建完的整颗dom树
    case HostRoot: {
      popHostContainer(workInProgress);
      popTopLevelLegacyContextObject(workInProgress);
      resetMutableSourceWorkInProgressVersions();

      const fiberRoot = (workInProgress.stateNode: FiberRoot);
      if (fiberRoot.pendingContext) {
      };
      // 首次挂载只有根fiber节点存在current，current.child === null
      if (current === null || current.child === null) {

        const wasHydrated = popHydrationState(workInProgress);
        if (wasHydrated) {
        } else if (!fiberRoot.hydrate) {
          workInProgress.flags |= Snapshot;
        }
      }
      updateHostContainer(workInProgress);
      // 至此，初次挂载的Reconiler阶段完成
      return null;
    }
    // 其余类型全删了😄，代码太多
    case HostComponent: {
      popHostContext(workInProgress);
      const rootContainerInstance = getRootHostContainer();
      const type = workInProgress.type;
      // 首次挂载 current除了根节点以外都为空
      if (current !== null && workInProgress.stateNode != null) {
      } else {
        const currentHostContext = getHostContext();

        const wasHydrated = popHydrationState(workInProgress);
        if (wasHydrated) {
        } else {
          // 创建DOM对象
          const instance = createInstance(
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
            workInProgress,
          );

          // 把子树中的DOM对象append到本节点的instance之中
          appendAllChildren(instance, workInProgress, false, false);

          // 设置stateNode，指向DOM对象
          workInProgress.stateNode = instance;

          if (
            // 设置DOM对象的属性, 绑定事件等
            finalizeInitialChildren(
              instance,
              type,
              newProps,
              rootContainerInstance,
              currentHostContext,
            )
          ) {
            markUpdate(workInProgress);
          }
        }

        // 存在ref属性 处理回调
        if (workInProgress.ref !== null) {
          markRef(workInProgress);
        }
      }
      return null;
    }
  }
}
```

存在一个这样的`<App />`组件
```tsx
ReactDOM.render(
    <App />,
  document.getElementById('root')
);

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={''} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.tsx</code> and save to reload.
        </p>
      </header>
    </div>
  );
}

```
工作循环流程如下：

- beginWork：第一次执行`beginWork`，`workInProgress`指向根`Fiber`对象

- beginWork：第二次执行`beginWork`，`workInProgress`指向`Fiber`对象的子节点(`<App />`)

- beginWork：第三次执行`beginWork`，`workInProgress`指向`<App />`的子节点`<div/>`

- beginWork：第四次执行`beginWork`，`workInProgress`指向`<div />`的子节点`<header />`

- beginWork：第五次执行`beginWork`，`workInProgress`指向`<header />`的子节点`<img />`

- beginWork：第六次执行`beginWork`，`<img />`没有子节点，`workInProgress`为空，执行`completeUnitWork`

- completeWork：由于`<img />`节点下不存在相应的字节点，此时对`<img />`执行`completeWork`。`Fiber`节点的`stateNode`属性指向该节点对应的DOM对象。

- beginWork：上一步`completeWork`执行完之后，`workInProgress`指针移动指向`sibling`节点，由于`<p />`节点未执行过`beginWork`阶段，所以先执行`beginWork`

- beginWork：执行对`Edit`文本节点的`beginWork`

- completeWork：执行对`Edit`文本节点的`completeWork`，更新其`stateNode`属性，指向对应的`DOM`文本节点

- beginWork：执行对`<code />`节点的`beginWork`，由于`<code />`节点下就只有存在一个纯文本节点，跳过对该节点的`beginWork`，直接对`<code />`节点进行`completeWork`阶段。

- completeWork：执行对`<code />`节点的`completeWork`

- beginWork：执行对文本节点`and save to reload`的`beginWork`

- completeWork：执行对`<p />`节点的`compltetWork`，`<p />`节点所有子节点都执行完`completeWork`之后。workInProgress指向`Fiber(p)`节点，更新其`stateNode`属性，指向对应的`DOM`对象

- completeWork：执行对`<header />`节点的`completeWork`，`<header />`节点所有子节点都执行完`completeWork`之后。workInProgress指向`Fiber(header)`节点，更新其`stateNode`属性，指向对应的`DOM`对象

- completeWork：执行对`<div/>`节点的`completeWork`，`<div />`节点所有子节点都执行完`completeWork`之后。workInProgress指向`Fiber(div)`节点，更新其`stateNode`属性，指向对应的`DOM`对象

- completeWork：`workInProgress`指针指向`<App/>`节点

- completeWork：`workInProgress`指向根fiber节点。`firstEffect`和`lastEffect`属性分别指向`effects`队列的开始（根fiber节点）和末尾（根fiber节点）

![beginWork - completeWork](../assets/img.Render/%20beginWork%20-%20completeWork.jpg)

### commit阶段

```js
function commitRoot(root) {
  const renderPriorityLevel = getCurrentPriorityLevel();

  // Scheduler调度器提供的方法
  // 参数一为调度的优先级
  // 参数二为调度的回调函数
  runWithPriority(
    ImmediateSchedulerPriority,
    commitRootImpl.bind(null, root, renderPriorityLevel),
  );
  return null;
}
```
* commit阶段开始于commitRoot这个方法。在这个方法中会执行runWithPriority()，该方法是由Scheduler，也就是调度器提供的这个方法。该方法接收两个参数，第一个参数是一个调度的优先级，第二个参数是一个调度的回调函数。在这个回调函数中触发的任何调度都会以第一个参数作为优先级。所以commitRoot实际要执行的方法是commitRootImpl。

```js
function commitRootImpl(root, renderPriorityLevel) {

  // 将effectList赋值给firstEffect
  // 由于每个fiber的effectList只包含他的子孙节点，根节点如果有effectTag则不会被包含进来
  // 这里将有effectTag的根节点插入到effectList尾部
  // 这样才能保证有effect的fiber都在effectList中
  let firstEffect;
  if (finishedWork.flags > PerformedWork) {
    if (finishedWork.lastEffect !== null) {
      finishedWork.lastEffect.nextEffect = finishedWork;
      firstEffect = finishedWork.firstEffect;
    } else {
    }
  } else {
    firstEffect = finishedWork.firstEffect;
  }

  if (firstEffect !== null) {

    // commit before mutation阶段
    nextEffect = firstEffect;
    do {
      if (__DEV__) {
      } else {
        try {
          commitBeforeMutationEffects();
        } catch (error) {
          captureCommitPhaseError(nextEffect, error);
          nextEffect = nextEffect.nextEffect;
        }
      }
    } while (nextEffect !== null);

    focusedInstanceHandle = null;

    if (enableProfilerTimer) {
      recordCommitTime();
    }

    // mutation阶段
    nextEffect = firstEffect;
    do {
      if (__DEV__) {
      } else {
        try {
          commitMutationEffects(root, renderPriorityLevel);
        } catch (error) {
          captureCommitPhaseError(nextEffect, error);
          nextEffect = nextEffect.nextEffect;
        }
      }
    } while (nextEffect !== null);

    if (shouldFireAfterActiveInstanceBlur) {
      afterActiveInstanceBlur();
    }
    resetAfterCommit(root.containerInfo);

    // FiberRoot的current指向finishedWork
    // 在layout阶段之前, current指向的是HostRootFiber.
    // 在layout阶段, current指向的是RootFiber.alternate.
    root.current = finishedWork;

    // layout阶段
    nextEffect = firstEffect;
    do {
      if (__DEV__) {
      } else {
        try {
          commitLayoutEffects(root, lanes);
        } catch (error) {
          captureCommitPhaseError(nextEffect, error);
          nextEffect = nextEffect.nextEffect;
        }
      }
    } while (nextEffect !== null);

    nextEffect = null;
  } else {
  }


  // 检测无限循环的同步任务
  if (remainingLanes === SyncLane) {
    if (root === rootWithNestedUpdates) {
      nestedUpdateCount++;
    } else {
      nestedUpdateCount = 0;
      rootWithNestedUpdates = root;
    }
  } else {
    nestedUpdateCount = 0;
  }

  // 在离开commitRoot函数前调用，触发一次新的调度，确保任何附加的任务被调度
  ensureRootIsScheduled(root, now());

  // 执行同步任务，这样同步任务不需要等到下次事件循环再执行
  // 比如在 componentDidMount 中执行 setState 创建的更新会在这里被同步执行
  // useLayoutEffect  
  // 我们目前项目中使用的useEffect版本还未用到异步更新，所以都会执行该方法，可以采用React的concurrent模式，尝试新功能。。。
  flushSyncCallbackQueue();

  return  null;
}
```
commitRootIml方法主要执行力before mutation、mutation和layout阶段

before muatation阶段：
* 遍历effectlist
* 处理DOM节点渲染/删除后的 autoFocus、blur逻辑
* 调用getSnapshotBeforeUpdate生命周期钩子
* 调度useEffect

mutation阶段：
* 遍历effectlist
* 是否需要重置文本节点、更新ref
* 根据不同的effectTag执行不同的工作。Placement -> 插入DOM、Update -> 更新属性、Deletion -> 删除DOM节点、 Hydrating -> SSR相关

layout阶段：
* commitLayoutEffectOnFiber（调用生命周期钩子和hook相关操作）
* commitAttachRef（赋值 ref）


学习到了什么？
---
* 位运算
```js
 // 没有任何操作
const NoLane = 0b000000;
// 更新操作
const Update = 0b000001;
// 插入操作
const Insert = 0b000010;

let current = 0b000000;

// 当前节点同时需要更新与插入操作 ？
current = current | Update | Insert

console.log((current & Update) !== NoLane);
console.log((current & Insert) !== NoLane);
```

* <strong>为什么要设计React16之后改成Fiber架构</strong>
  <br>
  为了实现异步可中断渲染。

* <strong>异步可中断渲染</strong>
  <br>
  什么是异步可中断渲染？为什么需要它？<br><br>
  react设计理念为快速响应，而制约快速响应有两种场景：
  <br>1、计算能力（CPU瓶颈） 
  <br>2、网络请求（IO瓶颈）。
  <br>网络延迟是我们前端开发暂时没办法解决的。而CPU的瓶颈，当项目变得庞大、组件数量众多，就容易达到CPU的瓶颈。主流浏览器刷新频率为60Hz，即1000ms/60Hz = 16.6ms 浏览器刷新一次。JS是可以操作DOM的，GUI渲染线程与js线程是互斥的。所以js脚本执行和浏览器布局、绘制不能同时执行。在这个过程中需要 JS脚本执行 -->样式布局 ---> 样式绘制。当js执行过程过长，超过一帧时，这次刷新就没有时间执行样式布局和样式绘制了，就会造成掉帧，给使用者一种卡顿的感觉。在浏览器每一帧的时间中，预留一些时间给JS线程，React利用这部分时间更新组件（[在源码中，预留的初始时间是5ms](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/scheduler/src/forks/SchedulerHostConfig.default.js#L119)）。当预留的时间不够用时，React将线程控制权交还给浏览器使其有时间渲染UI，React则等待下一帧时间继续被中断的工作。
  <br><br>
  [在react中](https://github1s.com/facebook/react/blob/17.0.2/packages/react-reconciler/src/ReactFiberWorkLoop.old.js)，在进行每一次工作循环时会检查是否存在空余时间.

```js
  /** @noinline */
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
 ```
但是在上述的legacy模式中，默认执行的是`workLoopSync`函数，是没有开启这个功能的，在concurrent模式中是开启的。
  
* <strong>JSX与Fiber节点之间的关系</strong>
  <br>
  JSX是一种描述当前组件内容的数据结构.，不包含组件在更新中的优先级、组件的state、组件需要被调用的tag。

* <strong>如何debug源码</strong>
  ```shell
  <!-- 个人建议寻找React16.8版本以后的包 -->

  # 拉取代码
  git clone https://github.com/facebook/react.git

  # 切入到react源码所在文件夹
  cd react

  # 安装依赖
  yarn

  # 执行打包命令
  yarn build react/index,react/jsx,react-dom/index,scheduler --type=NODE

  # 声明react指向
  cd build/node_modules/react
  yarn link

  # 声明react-dom指向
  cd build/node_modules/react-dom
  yarn link

  # 将项目内的react react-dom指向之前声明的包
  yarn link react react-dom
  ```
* <strong>react为什么不采用Generator？</strong>
  <br><br>
  看React这个[issuse](https://github.com/facebook/react/issues/7942#issuecomment-254987818)

  全文翻译：
  
  类似async，Generator也是传染性的，使用了Generator则上下文的其他函数也需要作出改变。这样心智负担比较重。

  Generator执行的中间状态是上下文关联的。

  ```jsx
  function* doWork(A, B, C) {
    var x = doExpensiveWorkA(A);
    yield;
    var y = x + doExpensiveWorkB(B);
    yield;
    var z = y + doExpensiveWorkC(C);
    return z;
  }
  ```
  每当浏览器有空闲时间都会依次执行其中一个doExpensiveWork，当时间用尽则会中断，当再次恢复时会从中断位置继续执行。只考虑“单一优先级任务的中断与继续”情况下Generator可以很好的实现异步可中断更新。

  但是当我们考虑“高优先级任务插队”的情况，如果此时已经完成doExpensiveWorkA与doExpensiveWorkB计算出x与y。此时B组件接收到一个高优更新，由于Generator执行的中间状态是上下文关联的，所以计算y时无法复用之前已经计算出的x，需要重新计算。如果通过全局变量保存之前执行的中间状态，又会引入新的复杂度。

  基于这些原因，React没有采用Generator实现协调器。