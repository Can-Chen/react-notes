React如何把我们编写的JSX文件转化成DOM？

### [渲染器（Renderer）](https://zh-hans.reactjs.org/docs/codebase-overview.html#renderers)
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
createElement做了什么工作？
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
      ....
      省略
      ....
  }

  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
  //  处理children
      ....
      省略
      ....
  }

  // Resolve default props
  if (type && type.defaultProps) {
  //  处理default props
      ....
      省略
      ....
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
看这React源码，很明显处理了传入的type，config，children后，最后调用ReactElement方法，返回一个包含组件数据的对象

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

  ....
    省略
  ....

  return element;
};
```

ReactDOM.render做了什么？ 

> ReactDOM.render() 会控制你传入容器节点里的内容。当首次调用时，容器节点里的所有 DOM 元素都会被替换，后续的调用则会使用 React 的 DOM 差分算法（DOM diffing algorithm）进行高效的更新。<br><br>
> 不会修改容器节点（只会修改容器的子节点）。可以在不覆盖现有子节点的情况下，将组件插入已有的 DOM 节点中。

ReactDOM.render()中的，render方法很简洁，执行了一个叫做legacyRenderSubtreeIntoContainer方法。
```js
export function render(
  element: React$Element<any>,
  container: Container,
  callback: ?Function,
) {
  invariant(
    isValidContainer(container),
    'Target container is not a DOM element.',
  );

  ...
  省略
  ...

  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
}
