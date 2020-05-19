[toc]

React 组件

# 1. React 中的组件
## 1.1 定义
在React 中, 组件是最小的页面组成单位

## 1.2 组件的重要组成部分
1. state
2. props
3. ref
4. 生命周期方法
5. static 方法
6. React 元素树

# 2. state
## 2.1 使用State 注意点
1. 不要直接修改 State, 直接修改State 的值, 会将值修改, 但是不会触发页面渲染
2. State 的更新可能是异步的
3. State 的更新会被合并, 已经存在的内容会被替换, 不存在的内容会被添加

## 2.2 数据流
React 中数据流是单向的, 数据只能从父组件传递到子组件, 如果子组件要实现与父组件通信, 那么只能调用父组件传递过来的函数(回调函数的方式) 

# 3. props
## 3.1 函数组件
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

## 3.2 class 组件
```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

## 3.4 props
props 表示从父组件向子组件传递的参数, 特别注意所有 React 组件都必须像纯函数一样保护它们的 props 不被更改(虽然React 没有手段阻止开发者显示修改, 但是显示修改一定会造成莫名其妙的BUG) 

## 3.5 props.children
props 表示的是父类元素的对象, 可以访问父类元素的所有内容
```
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}

function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
``` 

# 4. 组件创建的方法
## 4.1 function
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

## 4.2 class
```
class Greeting extends React.Component {
    // 定义构造函数
    constructor(props) {
        // 给props 定义默认值
        super(props)
        // 绑定函数的this
        this.handleClick = this.handleClick.bind(this)
        // this.state 定义初始值
        this.state = {
            count: this.props.initialCount
        }
    }
    
    handleClick: function() {
        alert(this.state.message);
    },
    
    render: function() {
        return(
            <button onClick={this.handleClick}>
                Say hello
            </button>
        )
    }
}

// 声明默认的defaultProps 属性
Greeting.defaultProps = {
    name: 'Mary'
};
```

## 4.3 create-react-class
不使用ES6 创建组件时, 不需要绑定this, create-react-class 自动对this 进行绑定
```
var createReactClass = require('create-react-class');
var Greeting = createReactClass({
    // getInitialState函数的返回值会用来初始化组件的this.state
    getInitialState: function() {
        return {count: this.props.initialCount};
    },
    
    // getDefaultProps 函数的返回值可以作为props 的初始值
    getDefaultProps: function() {
        return {name: 'oulae'}
    }
    
    // 使用create-react-class 创建的组件不需要进行绑定this
    handleClick: function() {
        alert(this.state.message);
    },

    render: function() {
        return(
            <button onClick={this.handleClick}>
                Say hello
            </button>
        )
    }
});

// 声明默认的defaultProps 属性
Greeting.defaultProps = {
    name: 'Mary'
};
```

# 5. 组件复用
## 5.1 页面复用
1. 页面复用
2. 继承(React 官方不推荐组件继承)

## 5.2 逻辑复用
1. 高阶组件
2. render props
3. hook
4. render props
