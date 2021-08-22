## 在脚手架泛滥成灾的如今，怎么能够不理解React Web开发的基础知识之一的React-Router呢，下面记录了[react-router-dom源码中暴露出的方法和组件](https://github.com/remix-run/react-router/blob/main/packages/react-router-dom/modules/index.js)

<br>

>_源码解析后面会出__

<br>

### MemoryRouter
-----
### Prompt
-----
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
### Redirect

渲染 Redirect 将导航到新位置。 新位置将覆盖历史堆栈中的当前位置，就像服务器端重定向（HTTP 3xx）一样

| 属性      | 说明 | 类型 | 默认值 |
| ----------- | ----------- | ----------- | ----------- |
| to      | 要重定向到的 URL       | string、object       | 无       |
| push   | 当为 true 时，重定向会将新条目推送到历史记录中，而不是替换当前条目。        | boolean       | false       |
| from      | 要重定向的路径名。理解的任何有效 URL 路径。 所有匹配的 URL 参数都提供给 to 中的模式。 必须包含在 to 中使用的所有参数。 to 未使用的其他参数将被忽略。注意：这只能用于在 Switch 内部渲染 Redirect 时匹配位置。       | string       | 无       |
| exact      | 完全匹配；相当于Route.exact.注意：这只能与 from 结合使用以在 Switch 内部渲染 Redirect 时精确匹配位置       | boolean       | 无       |
| strict      | 严格匹配； 等效于 Route.strict.注意：这只能与 from 结合使用以在 Switch 内部渲染 Redirect 时严格匹配位置。       | boolean       | 无       |
| sensitive      | 区分大小写匹配； 相当于 Route.sensitive。      | boolean       | 无       |

```jsx
import React, { useContext, createContext, useState } from "react";
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link,
  Redirect,
  useHistory,
  useLocation
} from "react-router-dom";

export default function AuthExample() {
  return (
    <ProvideAuth>
      <Router>
        <div>
          <AuthButton />

          <ul>
            <li>
              <Link to="/public">Public Page</Link>
            </li>
            <li>
              <Link to="/protected">Protected Page</Link>
            </li>
          </ul>

          <Switch>
            <Route path="/public">
              <PublicPage />
            </Route>
            <Route path="/login">
              <LoginPage />
            </Route>
            <PrivateRoute path="/protected">
              <ProtectedPage />
            </PrivateRoute>
          </Switch>
        </div>
      </Router>
    </ProvideAuth>
  );
}

const fakeAuth = {
  isAuthenticated: false,
  signin(cb) {
    fakeAuth.isAuthenticated = true;
    setTimeout(cb, 100); // fake async
  },
  signout(cb) {
    fakeAuth.isAuthenticated = false;
    setTimeout(cb, 100);
  }
};

const authContext = createContext();

function ProvideAuth({ children }) {
  const auth = useProvideAuth();
  return (
    <authContext.Provider value={auth}>
      {children}
    </authContext.Provider>
  );
}

function useAuth() {
  return useContext(authContext);
}

function useProvideAuth() {
  const [user, setUser] = useState(null);

  const signin = cb => {
    return fakeAuth.signin(() => {
      setUser("user");
      cb();
    });
  };

  const signout = cb => {
    return fakeAuth.signout(() => {
      setUser(null);
      cb();
    });
  };

  return {
    user,
    signin,
    signout
  };
}

function AuthButton() {
  let history = useHistory();
  let auth = useAuth();

  return auth.user ? (
    <p>
      Welcome!{" "}
      <button
        onClick={() => {
          auth.signout(() => history.push("/"));
        }}
      >
        Sign out
      </button>
    </p>
  ) : (
    <p>You are not logged in.</p>
  );
}

function PrivateRoute({ children, ...rest }) {
  let auth = useAuth();
  return (
    <Route
      {...rest}
      render={({ location }) =>
        auth.user ? (
          children
        ) : (
          <Redirect
            to={{
              pathname: "/login",
              state: { from: location }
            }}
          />
        )
      }
    />
  );
}

function PublicPage() {
  return <h3>Public</h3>;
}

function ProtectedPage() {
  return <h3>Protected</h3>;
}

function LoginPage() {
  let history = useHistory();
  let location = useLocation();
  let auth = useAuth();

  let { from } = location.state || { from: { pathname: "/" } };
  let login = () => {
    auth.signin(() => {
      history.replace(from);
    });
  };

  return (
    <div>
      <p>You must log in to view the page at {from.pathname}</p>
      <button onClick={login}>Log in</button>
    </div>
  );
}
```
官网的例子就是，在触发这个路径的时候 --> ```path: "/protected"``` 调用鉴权检查是否登录了，登录了就直接展示登录后的界面，未登录就重定向至login的页面。

```jsx
// 该路由匹配展示的组件就是来验证是否已登录来确认展示哪个组件的。
<PrivateRoute 
  ...
/>
```

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