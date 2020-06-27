[toc]

React 概念

# 1. 受控组件/状态组件
受控组件, 即在用于输入和UI显示中添加一层数据处理层, 将用户输入的信息进行处理之后再显示在页面中
可以简单地理解为受控组件即时class 组件
```
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('提交的名字: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          名字:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
```

# 2. 无状态组件
没有使用this.state 定义内容的组件

# 3. 函数组件
使用function 定义的无状态组件中引入了Hook 的组件

# 4. 协调
当组件的 props 或 state 发生变化时，React 通过将最新返回的元素与原先渲染的元素进行比较，来决定是否有必要进行一次实际的 DOM 更新。当它们不相等时，React 才会更新 DOM。这个过程被称为“协调”。

# 5. 复用
React 中, 主张使用组合来实现组件的复用, React 并不推荐使用继承;
因此, React 组件是一项很重要的内容.

# 6. Set Map polyfill
React 16 依赖集合类型 Map 和 Set 。如果你要支持无法原生提供这些能力（例如 IE < 11）或实现不规范（例如 IE 11）的旧浏览器与设备，考虑在你的应用库中包含一个全局的 polyfill ，例如 core-js 或 babel-polyfill 。

# 7. CDN
CDN 代表内容分发网络（Content Delivery Network）。CDN 会通过一个遍布全球的服务器网络来分发缓存的静态内容






