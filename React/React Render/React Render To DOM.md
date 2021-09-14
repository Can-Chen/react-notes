>React@17.0.2

## React如何把我们编写的JSX文件转化成DOM？

#### [渲染器（Renderer）](https://zh-hans.reactjs.org/docs/codebase-overview.html#renderers)
___

>渲染器用于管理一棵 React 树，使其根据底层平台进行不同的调用。

在React开发的项目中，react有对应不同版本的渲染器（Renderer）。有负责浏览器渲染的<b>ReactDOM</b>、渲染APP的<b>ReactNative</b>这两种常见的。还有<b>ReactTest</b>输出纯js对象进行测试、<b>ReactArt</b>渲染到Canvas、svg或VML（IE8）。

在浏览器中环境中所调用对应的eactDOM.render()方法渲染成DOM。render方法接收三个参数分别为所需要渲染的节点信息（element）、容器（一般为document.getElement("id")）、渲染成功或者更新后的回调（callback）。

第一个参数为渲染所需的节点信息element。而在日常中编写的代码都是采用JSX语法的形式进行编码的，那JSX是什么东西呢？
<br><br>
这是[官网的描述](https://react.docschina.org/docs/introducing-jsx.html)。

>是一个 JavaScript 的语法扩展。我们建议在 React 中配合使用 JSX，JSX 可以很好地描述 UI 应该呈现出它应有交互的本质形式。JSX 可能会使人联想到模板语言，但它具有 JavaScript 的全部功能。

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
  {
    className: "Can-Chen"
  },
  React.createElement(
    "a",
    {
      href: "https://github.com/Can-Chen"
    },
    "个人介绍"
  )
)
```
createElement做了什么工作？ 省略部分代码
```js
export function createElement(type, config, children) {
  let propName;

  // Reserved names are extracted
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
```
在React源码中，很明显处理了传入的type，config，children后，最后调用ReactElement方法，返回一个包含组件数据的对象<br><br>
省略部分代码
```js 
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // 标记这是个 React Element
    $$typeof: REACT_ELEMENT_TYPE,

    type: type,
    key: key,
    ref: ref,
    props: props,

    // 记录负责创建此元素的组件。
    _owner: owner,
  };

  return element;
};
```

react应用用三种启动模式，官网上对于[这3种启动模式的介绍](https://zh-hans.reactjs.org/docs/concurrent-mode-adoption.html#why-so-many-modes), 基本说明如下:

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

>注意: 虽然`17.0.2`的源码中有[`createRoot`和`createBlockingRoot`方法](https://github.com/facebook/react/blob/v17.0.2/packages/react-dom/src/client/ReactDOM.js#L202)(如果自行构建, [会默认构建`experimental`版本](https://github.com/facebook/react/blob/v17.0.2/scripts/rollup/build.js#L30-L35)), 但是稳定版的构建入口[排除掉了这两个 api](https://github.com/facebook/react/blob/v17.0.2/packages/react-dom/index.stable.js), 所以实际在`npm i react-dom`安装`17.0.2`稳定版后, 不能使用该 api.如果要想体验非`legacy`模式, 需要[显示安装 alpha 版本](https://github.com/reactwg/react-18/discussions/9)(或自行构建).

在这里我主要介绍经常使用的ReactDOM.render，也就是legacy模式.<br><br> 
ReactDOM.render做了什么？ 

> ReactDOM.render() 会控制你传入容器节点里的内容。当首次调用时，容器节点里的所有 DOM 元素都会被替换，后续的调用则会使用 React 的 DOM 差分算法（DOM diffing algorithm）进行高效的更新。<br><br>
> 不会修改容器节点（只会修改容器的子节点）。可以在不覆盖现有子节点的情况下，将组件插入已有的 DOM 节点中。

在图中我以Green代表绿框、Red代表红框、Blue代表蓝框。
1. Green在第一次执行React.render()方法时调用执行。
2. Red在第一次执行React.render()和更新都会执行。
3. Blue代表着开始把需要挂载的内容渲染到页面上。
![render_process](/React/assets/img.Render/render_process.png)

ReactDOM.render()中的，render方法很简洁，执行了一个叫做legacyRenderSubtreeIntoContainer方法。也没有调用其它的方法或函数。<br><br>
省略部分代码
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
省略部分代码
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

关于root的定义 省略部分代码
```js
export type RootType = {
  render(children: ReactNodeList): void,
  unmount(): void,
  _internalRoot: FiberRoot,
  ...
};
```

但是在legacyRenderSubtreeIntoContainer方法中就会对这次执行进行判断。首次进入到该方法中时，因为在react的容器container中还未初始化react应用的环境，所以`container._reactRootContainer`返回的root字段为undefined，需要对root进行初始化，创建ReactDOMRoot对象，初始化react应用环境。从以下代码可以看出首次加载和更新都有调用`updateContainer`方法，该方法是在首次加载时不需要进行批量更新。
<br><br>
省略部分代码

```js
export function unbatchedUpdates<A, R>(fn: (a: A) => R, a: A): R {
  // 先保存当前值为prevExecutionContext
  const prevExecutionContext = executionContext;
  executionContext &= ~BatchedContext;
  executionContext |= LegacyUnbatchedContext;
  try {
    return fn(a);
  } finally {
    // 回调执行完毕之后，再恢复到以前的值
    executionContext = prevExecutionContext;
    if (executionContext === NoContext) {
      // 刷新此批处理期间计划的即时回调
      resetRenderTimer();
      flushSyncCallbackQueue();
    }
  }
}
```
在上图中标注框框的调用栈中可以看出，调用更新的入口是`updateContainer`函数.先删除一些无关的代码。初次渲染传入的参数`element`React的节点数据、`container`fiberRoot、`parentaComponent`为null、`callback`<br><br>
省略部分代码

```js
function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function,
): Lane {

  // current类型为Fiber，指向的是RootFiber（Fiber树的跟节点）
  const current = container.current;
  // 获取当前时间戳, 计算本次更新的优先级
  const eventTime = requestEventTime();
  // 创建一个优先级变量(车道模型)
  const lane = requestUpdateLane(current);

  // 根据车道优先级, 创建update对象
  const update = createUpdate(eventTime, lane);

  //设置update对象的payload, 这里element需要注意(是tag=HostRoot特有的设置, 指向<App/>) tag=0
  update.payload = {element};

  // 将新建的一个update对象加入的更新队列中（链表结构） fiber.updateQueue.pending队列
  enqueueUpdate(current, update);

  // 4. 进入reconciler运作流程中的`输入`环节
  scheduleUpdateOnFiber(current, lane, eventTime);

  return lane;
}
```
省略部分代码
```js
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

### render过程

#### scheduleUpdateOnFiber 省略部分代码
```js
function scheduleUpdateOnFiber(
  fiber: Fiber,
  lane: Lane,
  eventTime: number,
) {
  // 检查最大更新深度 50次 超过抛出错误
  checkForNestedUpdates();
  // 该方法只作用于Dev环境
  warnAboutRenderPhaseUpdatesInDEV(fiber);

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
      // 初次挂载 传入FiberRoot对象 执行同步更新
      performSyncWorkOnRoot(root);
    } else {
      
    }
  } else {
  }
}
```
#### performSyncWorkOnRoot 省略部分代码
看上方的调用图中，performSyncOnRoot调用栈分为两部分：renderRootSync 和 commitRoot。分别对应着render阶段和commit阶段
```js
function performSyncWorkOnRoot(root) {

  let lanes;
  let exitStatus;

  // 当前初次挂载 workInProgressRoot为空
  // 
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

#### renderRootSync 省略部分代码

```js
function renderRootSync(root: FiberRoot, lanes: Lanes) {
  const prevExecutionContext = executionContext;
  executionContext |= RenderContext;
  const prevDispatcher = pushDispatcher();

  // If the root or lanes have changed, throw out the existing stack
  // and prepare a fresh one. Otherwise we'll continue where we left off.
  if (workInProgressRoot !== root || workInProgressRootRenderLanes !== lanes) {
    prepareFreshStack(root, lanes);
    startWorkOnPendingInteractions(root, lanes);
  }

  const prevInteractions = pushInteractions(root);

  if (enableSchedulingProfiler) {
    markRenderStarted(lanes);
  }

  do {
    try {
      workLoopSync();
      break;
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);
  resetContextDependencies();
  if (enableSchedulerTracing) {
    popInteractions(((prevInteractions: any): Set<Interaction>));
  }

  executionContext = prevExecutionContext;
  popDispatcher(prevDispatcher);

  if (enableSchedulingProfiler) {
    markRenderStopped();
  }

  // Set this to null to indicate there's no in-progress render.
  workInProgressRoot = null;
  workInProgressRootRenderLanes = NoLanes;

  return workInProgressRootExitStatus;
}
```

#### commitRoot 省略部分代码

```js
function commitRoot(root) {
  const renderPriorityLevel = getCurrentPriorityLevel();
  runWithPriority(
    ImmediateSchedulerPriority,
    commitRootImpl.bind(null, root, renderPriorityLevel),
  );
  return null;
}
```