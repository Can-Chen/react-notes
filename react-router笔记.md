## 在脚手架泛滥成灾的如今，怎么能够不理解React Web开发的基础知识之一的React-Router呢，下面记录了[react-router-dom源码中暴露出的方法和组件](https://github.com/remix-run/react-router/blob/main/packages/react-router-dom/modules/index.js)

### MemoryRouter

### Prompt
该方法在用户离开当前页面导航时，可以进行阻止。

| 属性      | 说明 | 类型 | 默认值 |
| ----------- | ----------- | ----------- | ----------- |
| when      | 是否开启       | boolean       | true       |
| message   | 提示的消息内容        | string       | 无       |

```jsx
import React, { useState } from "react";
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link,
  Prompt
} from "react-router-dom";

export default function PreventingTransitionsExample() {
  return (
    <Router>
      <ul>
        <li>
          <Link to="/">Form</Link>
        </li>
        <li>
          <Link to="/one">One</Link>
        </li>
        <li>
          <Link to="/two">Two</Link>
        </li>
      </ul>

      <Switch>
        <Route path="/" exact children={<BlockingForm />} />
        <Route path="/one" children={<h3>One</h3>} />
        <Route path="/two" children={<h3>Two</h3>} />
      </Switch>
    </Router>
  );
}

function BlockingForm() {
  let [isBlocking, setIsBlocking] = useState(false);

  return (
    <form
      onSubmit={event => {
        event.preventDefault();
        event.target.reset();
        setIsBlocking(false);
      }}
    >
      <Prompt
        when={isBlocking}
        message={location =>
          `Are you sure you want to go to ${location.pathname}`
        }
      />

      <p>
        Blocking?{" "}
        {isBlocking ? "Yes, click a link or the back button" : "Nope"}
      </p>

      <p>
        <input
          size="50"
          placeholder="type something to block transitions"
          onChange={event => {
            setIsBlocking(event.target.value.length > 0);
          }}
        />
      </p>

      <p>
        <button>Submit to stop blocking</button>
      </p>
    </form>
  );
}
```
例子中如果表单中存在数据，但是触发离开当前导航时，此时因为在对表单中输入了数据，在input输入框中，根据input的事件获取输入数据的长度，改变Prompt组件的flag，监听路由的变化，当存在数据时，触发路由变化就触发Prompt组件，并有对应的提示。
```jsx
<input 
...
onChange={event => {
  setIsBlocking(event.target.value.length > 0)
}}  
/>

<Prompt 
  when={isBlocking}
  message={...}
/>
```
__源码解析稍后面会出__
### Redirect

### Route

### Router

### StaticRouter

### Switch

### generatePath

### matchPath

### withRouter

### useHistory

### useLocation

### useParams

### useRouteMatch

### BrowserRouter

### HashRouter

### Link

### NavLink