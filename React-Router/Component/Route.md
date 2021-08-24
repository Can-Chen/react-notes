### Route
-----
路径与当前url匹配时正确的呈现ui
>notes:如果在组件树的同一点使用同一个组件作为多个 Route 的子组件，React 会看到这个 因为相同的组件实例和组件的状态将在路由更改之间保留。 如果不希望这样，添加到每个路由组件的唯一 key prop 将导致 React 在路由更改时重新创建组件实例

传入Route的组件形式可以是
Component
```jsx
function User() {
  return <div>user</div>
}

ReactDOM.render(
  <Router>
    <Route path="/user" component={User}  />
  </Router> 
)
```
render
```jsx
ReactDOM.redner(
  <Router>
    <Route path="/user" render={() => <div>user</div>} />
  </Router>
)
```
children
```jsx
function ListItemLink({ to, ...props }) {
  return <Route 
    path={to}
    children={({ match }) => (
      <li className={match ? 'active' : ''}>
        <Link to={to} {...props}>
      </li>
    )}
  />
};

ReactDOM.redner(
  <Router>
    <ul>
      <ListItemLink to="/user" />
    </ul>
  </Router>
)
```
这三种渲染方法都会船体相同的三个路由参数match、location、history
<br />

| 属性      | 说明 | 类型 | 默认值 |
| ----------- | ----------- | ----------- | ----------- |
| path      | 有效的URL路径或路径数组。没有路径的路由总是匹配       | string、string[]       | 无       |
| exact   | 当为 true 时，仅当路径与 location.pathname 完全匹配时才匹配。        | boolean       | false       |
| strict   | 当为 true 时，具有尾部斜杠的路径将仅匹配带有尾部斜杠的 location.pathname。 当 location.pathname 中有额外的 URL 段时，这不起作用。 notes: 如果为true，路径为/path/，匹配路径为/path/12，会成功匹配，需要配合exact使用      | boolean       | false       |
| location   |  Route元素尝试将其路径与当前历史位置（通常是当前浏览器URL）匹配。也可以传递具有不同路径名的位置进行匹配        | object       | 无       |
| sensitive   | 当为 true 时，路径区分大小写。    | boolean       | false       |

傻瓜式例子
```jsx
import React from "react";
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link
} from "react-router-dom";

export default function BasicExample() {
  return (
    <Router>
      <div>
        <ul>
          <li>
            <Link to="/basic">基础例子</Link>
          </li>
          <li>
            <Link to="/exact-false/1">否严格匹配例子</Link>
          </li>
          <li>
            <Link to="/exact-true/2">是严格匹配例子</Link>
          </li>
          <li>
            <Link to="/strict-false/">否尾部斜杠匹配例子</Link>
          </li>
          <li>
            <Link to="/strict-true">是尾部斜杠匹配例子</Link>
          </li>
          <li>
            <Link to="/SensiTive-fAlse">否区分大小写</Link>
          </li>
           <li>
            <Link to="/SensiTive-tRue">是区分大小写</Link>
          </li>
        </ul>

        <hr />

        <Switch>
          <Route  path="/basic">
            <Basic />
          </Route>
          <Route path="/exact-false">
            <Exact />
          </Route>
          <Route exact path="/exact-true">
            <Exact />
          </Route>
          <Route  path="/strict-false/">
            <Strict />
          </Route>
          <Route strict path="/strict-true/">
            <Strict />
          </Route>
          <Route path="/sensitive-false">
            <Sensitive />
          </Route>
          <Route sensitive path="/sensitive-true">
            <Sensitive />
          </Route>
        </Switch>
      </div>
    </Router>
  );
}

function Basic() {
  return (
    <div>
      <h2>基础例子</h2>
    </div>
  );
}

function Exact() {
  return (
    <div>
      <h2>是否可以严格匹配</h2>
    </div>
  );
}

function Strict() {
  return (
    <div>
      <h2>尾部斜杠是否匹配</h2>
    </div>
  );
}

function Sensitive() {
  return (
    <div>
      <h2>是否区分大小写</h2>
    </div>
  );
}


```