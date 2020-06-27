[toc]

# 1. 定义
在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但这种做法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题）Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法

# 2. context Demo
```
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');
class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // React 会往上找到最近的 theme Provider，然后使用它的值。
  // 在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

# 3. React 值传递的方式
1. props, 一般使用props 传递值
2. 组件组合, 组件组合本质上还是使用props 的传递方式, 此时props 传递的内容为JSX(this.props.children)
3. context

# 4. API
## 4.1 React.createContext
```
const MyContext = React.createContext(defaultValue);
```
创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 Provider 中读取到当前的 context 值

只有在组件所在树中没有匹配到provider 时(即MyContext.Provider 没有显示提供值时), defaultValue 参数才会生效

PS: 将 undefined 传递给 Provider 的 value 时，consumer 组件的 defaultValue 不会生效

## 4.2 Context.Provider
```
<MyContext.Provider value={/* 某个值 */}>
```

定义Context 对象的有效范围, 在Context 有效范围中的组件叫consumer 组件;

一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据

特别注意: 当 Provider 的 value 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 shouldComponentUpdate 函数，因此只要Provider 的value 值发生变化, 其consumer 组件一定会发生改变

## 4.3 Class.contextType
```
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* 基于 MyContext 组件的值进行渲染 */
  }
}
MyClass.contextType = MyContext;
```

挂载在 class 上的 contextType 属性会被重赋值为一个由 React.createContext() 创建的 Context 对象(定义组件的Context 对象)
相当于: static contextType = MyContext;

context 能在任意的生命周期中被访问到

## 4.4 static contextType = MyContext
给一个组件绑定context, 相当于Class.contextType
```
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* 基于这个值进行渲染工作 */
  }
}

```

## 4.5 Context.Consumer
```
import {ThemeContext} from './theme-context';

function ThemeTogglerButton() {
  // Theme Toggler 按钮不仅仅只获取 theme 值，它也从 context 中获取到一个 toggleTheme 函数
  return (
    <ThemeContext.Consumer>
      {({theme, toggleTheme}) => (
        <button onClick={toggleTheme}
          style={{backgroundColor: theme.background}}>

          Toggle Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
}

export default ThemeTogglerButton;
```
在函数式组件中完成订阅Context (由于函数式组件没有constructor, 不会调用React.component 的构造函数来绑定props, 因此需要这种特殊写法显示订阅context)

传递给函数的 value 值等同于往上组件树离这个 context 最近的 Provider 提供的 value 值。如果没有对应的 Provider，value 参数等同于传递给 createContext() 的 defaultValue。

PS: 注意在函数式组件中访问context 的形式

## 4.6 Context.displayName
不明白这个属性到底有啥用

# 5. 多个Context
```
import React from 'react'

// Theme context，默认的 theme 是 “light” 值
const ThemeContext = React.createContext('light');

// 用户登录 context
const UserContext = React.createContext({
    name: 'Guest',
});

class App extends React.Component {
    render() {
        // const {signedInUser, theme} = this.props;
        // 提供初始 context 值的 App 组件
        return (
            <ThemeContext.Provider value='dark'>
                <UserContext.Provider value='oulae'>
                    <Layout />
                </UserContext.Provider>
            </ThemeContext.Provider>
        );
    }
}

function Layout() {
    return (
        <div>
            {/*<Sidebar />*/}
            <Content />
        </div>
    );
}

// 一个组件可能会消费多个 context
// function Content() {
    // return (
        // <ThemeContext.Consumer>
            // {theme => (
                // <UserContext.Consumer>
                    // {user => (
                        // <div>{user}{theme}123</div>
                    // )}
                // </UserContext.Consumer>
                // )}
        // </ThemeContext.Consumer>
    // );
// }

class Content extends React.Component{
    render() {
        return (
            <ThemeContext.Consumer>
                {theme => (
                    <UserContext.Consumer>
                        {user => (
                            <div>{user}{theme}123</div>
                        )}
                    </UserContext.Consumer>
                )}
            </ThemeContext.Consumer>
        )
    }
}

export default App

```

# 6. Context 的性能提升
```
class App extends React.Component {
  render() {
    return (
      <MyContext.Provider value={{something: 'something'}}>
        <Toolbar />
      </MyContext.Provider>
    );
  }
}
```
以上代码, 每当App 组件重新渲染一次时, Toolbar 组件都会无条件再次重新渲染, 而且是强制重新渲染, 因为value 在每次App 重新渲染时都被赋值给了新的值(因为每次都被赋予了一个新的对象, 注意理解这句话)

解决这个问题需要将value 的值的状态提升到父节点
```
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: {something: 'something'},
    };
  }

  render() {
    return (
      <Provider value={this.state.value}>
        <Toolbar />
      </Provider>
    );
  }
}
```

# 7. context 通信的优势
Context 解决了props 通信必须层层传递的问题, 能有效较少代码的嵌套

# 8. 参考链接
1. [contextDemo](D:\code\githubCode\studyDemo\react\contextDemo)