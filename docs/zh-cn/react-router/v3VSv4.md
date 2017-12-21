> 本文主要说下v4和v3版本的区别 以例子的形式分析  ps: React Router 的早期版本是將 router 和 layout components 分开

```
import React from "react";
import { render } from "react-dom";
import { Router, Route, IndexRoute, Link, browserHistory } from "react-router";

const PrimaryLayout = props =>
  <div className="primary-layout">
    <header>Our React Router 3 App</header>
    <ul>
      <li>
        <Link to="/">Home</Link>
      </li>
      <li>
        <Link to="/user">User</Link>
      </li>
    </ul>
    <main>
      {props.children}
    </main>
  </div>;

const HomePage = () => <h1>Home Page</h1>;
const UsersPage = () => <h1>User Page</h1>;

const App = () =>
  <Router history={browserHistory}>
    <Route path="/" component={PrimaryLayout}>
      <IndexRoute component={HomePage} />
      <Route path="/user" component={UsersPage} />
    </Route>
  </Router>;

render(<App />, document.getElementById("root"));

```
>上面代码中有几个关键的点在 V4 中就不复存在了 我们使用 V4 来实现相同的应用程序对比一下

- 集中式 router
- 通过 <Route> 嵌套，实现 Layout 和 page 嵌套
- Layout 和 page 组件 是作为 router 的一部分

```
import React from "react";
import { render } from "react-dom";
import { BrowserRouter, Route, Link } from "react-router-dom";

const PrimaryLayout = () =>
  <div className="primary-layout">
    <header>Our React Router 4 App</header>
    <ul>
      <li>
        <Link to="/">Home</Link>
      </li>
      <li>
        <Link to="/User">User</Link>
      </li>
    </ul>
    <main>
      <Route path="/" exact component={HomePage} />
      <Route path="/user" component={UsersPage} />
    </main>
  </div>;

const HomePage = () => <h1>Home Page</h1>;
const UsersPage = () => <h1>User Page</h1>;

const App = () =>
  <BrowserRouter>
    <PrimaryLayout />
  </BrowserRouter>;

render(<App />, document.getElementById("root"));

```
> 注意，我们现在 import 的是 BrowserRouter，而且是从 react-router-dom 引入，而不是 react-router
v3中的 router不存在了    另外，V4 中，我们不再使用 {props.children} 来嵌套组件了，替代的 <Route>，
当 route 匹配时，子组件会被渲染到 <Route> 书写的地方 <
> 在上面的 example 中，读者可能注意到 V4 中有 exact 这么一个 props，那么，这个 props 有什么用呢？
 V3 中的 routing 规则是 exclusive， 意思就是最终只获取一个 route，
 而 V4 中的 routes 默认是 inclusive 的，这就意味着多个 <Route>可以同时匹配和呈现
 
 ##### 为了演示 inclusive routing 的作用，我们新增一个 UserMenu 组件如下.
 

```
const PrimaryLayout = () =>
  <div className="primary-layout">
    <header>
      Our React Router 4 App
      <Route path="/user" component={UsersMenu} />
    </header>
    <main>
      <Route path="/" exact component={HomePage} />
      <Route path="/user" component={UsersPage} />
    </main>
  </div>;
  
  现在，当访问 /user 时，两个组价都会被渲染，
  在 V3 中存在一些模式也可以实现，但过程实在是复杂，在 V4 中，是不是感觉轻松了很多
```

#### Exclusive Routing
> 如果你只想匹配一个 route，那么你也可以使用 <Switch> 来 exclusive routing

```
const PrimaryLayout = () =>
  <div className="primary-layout">
    <PrimaryHeader />
    <main>
      <Switch>
        <Route path="/" exact component={HomePage} />
        <Route path="/user/add" component={UserAddPage} />
        <Route path="/user" component={UsersPage} />
        <Redirect to="/" />
      </Switch>
    </main>
  </div>;


在 <Switch> 中只有一个  <Route>  会被渲染，
另外，我们还是要给 HomePage 所在 <Route> 添加 exact，否则，在访问 /user 或 /user/add 的时候还是会匹配到 /，从而，只渲染 HomePage。
同理，不知有没同学注意到，我们将 /user/add 放在 /user 前面是保证正确匹配的很有策略性的一步，
因为，/user/add 会同时匹配 /user 和 /user/add，如果不这么做，大家可以尝试交换它们两个的位置，看下会发生什么
当然，如果我们给每一个 <Route> 都添加一个 exact，那就不用考虑上面的 策略 了，但不管怎样，现在至少知道了我们还有其它选择
```

#### "Index Routes" 和 "Not Found"

```
V4 中也没有 <IndexRoute>，
但 <Route exact> 可以实现相同的功能，或者 <Switch> 和 <Redirect> 重定向到默认的有效路径，甚至一个找不到的页面

```
#### 嵌套布局

.......
#### match.path vs match.url
最开始，可能觉得这两者的区别并不明显，控制台经常出现相同的输出，比如，访问 /user

```
const UserSubLayout = ({ match }) => {
  console.log(match.url)   // output: "/user"
  console.log(match.path)  // output: "/user"
  return (
    <div className="user-sub-layout">
      <aside>
        <UserNav />
      </aside>
      <div className="primary-content">
        <Switch>
          <Route path={match.path} exact component={BrowseUsersPage} />
          <Route path={`${match.path}/:userId`} component={UserProfilePage} />
        </Switch>
      </div>
    </div>
  )
}
```
>match 在组件的参数中被解构，意思就是我们可以使用 match.path 代替 props.match.path
虽然我们看不到什么明显的差异，但需要明白的是 match.url 是浏览器 URL 的一部分，match.path 是我们为 router 书写的路径

##### 如何选择
如果我们是构建 route 路径，那么肯定使用 match.path
为了说明问题，我们创建两个子组件，一个 route 路径来自 match.url，一个 route 路径来自 match.path
```
const UserComments = ({ match }) =>
  <div>
    UserId: {match.params.userId}
  </div>;

const UserSettings = ({ match }) =>
  <div>
    UserId: {match.params.userId}
  </div>;

const UserProfilePage = ({ match }) =>
  <div>
    User Profile:
    <Route path={`${match.url}/comments`} component={UserComments} />
    <Route path={`${match.path}/settings`} component={UserSettings} />
  </div>;
然后，我们按下面方式来访问

/user/5/comments
/user/5/settings
实践后，我们发现，访问 comments 返回 undefined，访问 settings 返回 5

正如 API 所述
 match:
 path - (string) The path pattern used to match. Useful for building nested <Route>s
 url - (string) The matched portion of the URL. Useful for building nested <Link>s


```



####避免 Match Collisions

```
假设我们的 App 是一个仪表盘，我们希望访问 /user/add 和 /user/5/edit 添加和编辑 user。使
用上面的实例，user/:userId 已经指向 UserProfilePage，
我们这是需要在 UserProfilePage 中再添加一层 routes 么？显示不是这样的

const UserSubLayou = ({ match }) =>
  <div className="user-sub-layout">
    <aside>
      <UserNav />
    </aside>
    <div className="primary-content">
      <Switch>
        <Route exact path={match.path} component={BrowseUsersPage} />
        <Route path={`${match.path}/add`} component={AddUserPage} />
        <Route path={`${match.path}/:userId/edit`} component={EditUserPage} />
        <Route path={`${match.path}/:userId`} component={UserProfilePage} />
      </Switch>
    </div>
  </div>;
现在，看清楚这个策略了么

另外，我们使用 ${match.path}/:userId(\\d+) 作为 UserProfilePage 对应的 path，保证 :userId 是一个数字，
可以避免与 /users/add 的冲突，这样，将其所在的 <Route> 丢到最前面去也能正常访问 add 页面，
这一招，就是我在 path-to-regexp 学的

```

##### Authorized Route

```
在应用程序中限制未登录的用户访问某些路由是非常常见的，还有对于授权和未授权的用户 UI 也可能大不一样，为了解决这样的需求，我们可以考虑为应用程序设置一个主入口

class App extends React.Component {
  render() {
return (
      <Provider store={store}>
        <BrowserRouter>
          <Switch>
            <Route path="/auth" component={UnauthorizedLayout} />
            <AuthorizedRoute path="/app" component={PrimaryLayout} />
          </Switch>
        </BrowserRouter>
      </Provider>
    )
  }
}
现在，我们首先会去选择应用程序在哪个顶级布局中，比如，/auth/login 和 /auth/forgot-password 肯定在 UnauthorizedLayout 中，另外，当用户登陆时，我们将判断所有的路径都有一个 /app 前缀以确保是否登录。如果用户访问 /app 开头的页面但并没有登录，我们将会重定向到登录页面

下面就是我写的 AuthorizedRoute 组件，这也是 V4 中一个惊奇的特性，可以为了满足某种需要而书写自己的路由

class AuthorizedRoute extends React.Component {
  componentWillMount() {
    getLoggedUser();
  }

  render() {
    const { component: Component, pending, logged, ...rest } = this.props;
return (
      <Route
        {...rest}
        render={props => {
if (pending) return <div>Loading...</div>;
return logged
            ? <Component {...this.props} />
            : <Redirect to="/auth/login" />;
        }}
      />
    );
  }
}

const stateToProps = ({ loggedUserState }) => ({
  pending: loggedUserState.pending,
  logged: loggedUserState.logged
});

export default connect(stateToProps)(AuthorizedRoute);

```
点击 [这里](https://zhuanlan.zhihu.com/p/28585911) 可以查看的我的整个 Authentication


#### 总结
React Router 4 相比 V3，变化很大，若是之前的项目使用的 V3，不建议立即升级，但 V4 比 V3 确实存在较大的优势

