[toc]

Cross-Cutting Concerns

# 1. Cross-Cutting Concerns 是什么?
横切关注点(Cross-Cutting Concerns), 部分关注点「横切」程序代码中的数个模块，即在多个模块中都有出现，它们即被称作横切关注点

具体来说就是, 某一种属性行为贯穿多个不同层级的模块, 这就叫横切关注点; 比如日志功能, 该功能可以需要在项目的大部分组件使用, 该功能就叫横切关注点

# 2. 解决横切关注点的方案?
## 2.1 Render Prop
### 1. 定义
Render Prop 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术

重要的是要记住，render prop 是因为模式才被称为 render prop ，你不一定要用名为 render 的 prop 来使用这种模式。事实上， 任何被用于告知组件需要渲染什么内容的函数 prop 在技术上都可以被称为 “render prop”.


### 2. demo
```
class Cat extends React.Component {
    render() {
        const mouse = this.props.mouse;
        return (
            <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
    }
}

// 定义一个Render Props 组件
class Mouse extends React.Component {
    constructor(props) {
        super(props);
        this.handleMouseMove = this.handleMouseMove.bind(this);
        this.state = { x: 0, y: 0 };
    }

    handleMouseMove(event) {
        this.setState({
            x: event.clientX,
            y: event.clientY
        });
    }

    render() {
        return (
            <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
                {/*需要渲染从父类传递过来的组件*/}
                {this.props.render(this.state)}
            </div>
    );
    }
}

// Render Props 组件的children 属性必须是一个函数, 且该属性是不可省略的
Mouse.propTypes = {
    children: PropTypes.func.isRequired
};

class MouseTracker extends React.Component {
    render() {
        return (
            <div>
                <h1>移动鼠标!</h1>
                <Mouse render={mouse => (
                    <Cat mouse={mouse} />
                )}/>
            </div>
        );
    }
}
```

### 3. 优化写法
下面是Render Prop 一种优化的写法
```
class MouseTracker extends React.Component {
  // 定义为实例方法，`this.renderTheCat`始终
  // 当我们在渲染中使用它时，它指的是相同的函数
  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

如果Mouse 组件使用shouldComponentUpdate 作了优化, 那么demo 中的写法将会造成重复渲染, 因为每次props 都被赋予了新的值, shouldComponentUpdate 均会返回true; 因此可以将render 方法实例化, 消除重复渲染

## 2.2 高阶组件
高阶组件.md

## 2.3 Hook
Hook.md

# 4. 参考链接
1. [Render Props](https://react.docschina.org/docs/render-props.html)
