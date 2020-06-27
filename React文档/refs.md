[toc]

refs

# 1. 定义
Ref 转发是一项将 ref 自动地通过组件传递到其一子组件的技巧, 能够使父组件直接获取子组件的引用

ref属性。它可以声明在DOM Element和Class Component上，无法直接声明在Functional Components上

# 2. string ref
> 别用, 该内容已不被官方所推荐
```
class MyComponent extends React.Component {
  componentDidMount() {
    this.refs.myRef.focus();
  }
  render() {
    return <input ref="myRef" />;
  }
}
```

# 3. function ref
```
class MyComponent extends React.Component {
  componentDidMount() {
    this.myRef.focus();
  }
  render() {
    return <input ref={(ele) => {
      this.myRef = ele;
    }} />;
  }
}
```

# 4. React.createRef
```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  componentDidMount() {
    this.myRef.current.focus();
  }
  render() {
    return <input ref={this.myRef} />;
  }
}
```

# 5. API
## 5.1 React.createRef
> 创建一个能够通过 ref 属性附加到 React 元素的 ref
> 组件被挂载后，回调函数被立即执行，回调函数的参数为该组件的具体实例。
> 组件被卸载或者原有的ref属性本身发生变化时，回调也会被立即执行，此时回调函数参数为null，以确保内存泄露。
```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);

    this.inputRef = React.createRef();
  }

  render() {
    return <input type="text" ref={this.inputRef} />;
  }

  componentDidMount() {
    this.inputRef.current.focus();
  }
}
```

## 5.2 React.forwardRef
创建一个React组件，这个组件能够将其接受的 ref 属性转发到其组件树下的另一个组件中
```
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

# 6. 应用场景
1. 管理焦点，文本选择或媒体播放
2. 触发强制动画
3. 集成第三方 DOM 库

# 7. demo
高阶组件.md

# 8. 参考链接
1. [React ref 的前世今生](https://juejin.im/post/5b59287af265da0f601317e3)
2. [React 顶层 API](https://react.docschina.org/docs/react-api.html#reactcreateref)