[toc]

设计高质量的React 组件

# 1. 高内聚和低耦合
组件的划分要满足高内聚和低耦合的原则
高内聚: 把逻辑紧密相关的内容放在一个组件中.
低耦合: 不同组件之间的关系尽量弱化, 也就是每个组件要尽量独立, 保持整个系统的低耦合度

# 2. props
1. HTML 组件属性值都是字符串类型, React 组件的props 可以是JavaScript 支持的任意数据类型
2. 当prop 的数据类型不是字符串类型时, 在JSX 中必须用花括号{} 把prop 值包住, 所以style 值有两层花括号, 外层花括号代表是JSX 语法, 内层花括号代表这是一个对象常量
3. super 调用的是父类即React.Component 的构造函数; 如果一个类中构造函数没有调用super(props), 那么组件实例被构造之后, 类实例所有成员的函数就无法通过this.props 访问到父类传递过来的props 值
4. 严格来说, 组件并不能阻止你修改传入的props 值, 但是每个开发者应该把不修改this.props 当做一个规矩, 否则可能会造成意料之外的BUG

# 3. propTypes
propTypes 用于限制props的类型, 但是该功能应该仅仅只用于辅助开发, 不应该在生产环境中使用该功能

babel-react-optimize: 用于将propTypes 去除

# 4. this.state
在一个组件中, this.state 的只能能被直接赋值所改变, 但是这样会造成一个问题:
如果this.state 值通过直接赋值改变, 那么页面将不会重新渲染(页面的重新渲染通过this.setState 触发), 此时页面显示内容和真实的this.state 不是同一个值

# 5. 组件的生命周期
## 5.1 React 组件的生命周期
1. 装载过程(Mount): 组件第一次渲染DOM 树的过程
2. 更新过程(Update): 当组件被重新渲染的过程
3. 卸载过程(Unmount): 组件从DOM 删除的过程

## 5.2 装载过程(Mount)
### 1. 执行流程
1. constructor
2. getInitialState
3. getDefaultProps
4. componentWillMount (即将废弃)
5. render
6. componentDidMount

### 2. constructor
表示组件的构造函数, 构造函数完成了两个内容
1. 初始化state
2. 绑定成员函数的this

### 3. getInitialState/getDefaultProps
getInitialState/getDefaultProps 这两个方法只有在 React.createClass 创造出来的组件类中才会发生作用, 不过遗憾的是React.createClass 已经被Facebook 官方逐渐放弃; 所以这两个API 了解即可
getInitialState: 初始化this.state 值
getDefaultProps: 初始化this.props 值

### 4. render
render 函数是React 组件最重要的函数, 因为所有React 组件的父类React.Component 类对除render 之外的生命周期函数都有默认实现

注意点
1. render 函数返回一个JSX 描述的结构, 最终由React 来操作渲染流程
2. 在render 函数中不要试图去调用this.setState(), 因为这样可能会造成死循环
3. 当render 函数返回一个null 或false, 等于告诉React 这个组件不需要渲染任何DOM 元素

### 5. componentWillMount/componentDidMount
componentWillMount: 在render 函数之前执行, 可以在服务器/浏览器环境中调用;
componentDidMount: 在render 函数之后执行, 只能在浏览器环境中调用; 当该函数被调用时, 组件已经被装载到DOM 上

PS: 一般地我们在componentDidMount 生命周期中进行数据的请求和DOM 操作


## 5.3 更新过程(Update)
### 1. 执行流程
1. componentWillReceiveProps (即将废弃)
2. shouldComponentUpdate
3. componentWillUpdate (即将废弃)
4. render
5. componentDidUpdate

### 2. componentWillReceiveProps
只要父组件render 函数被调用, 在render 函数里被渲染的子组件就会经历更新过程, 不管父组件传给子组件的props 有没有变, 都会触发子组件的componentWillReceiveProps 函数

通过this.setState 方法触发的更新过程不会调用这个函数

### 3. shouldComponentUpdate
除了render 函数, shouldComponentUpdate 可能是React 组件生命周期中最重要的一个函数了, render() 函数决定了组件要渲染什么内容, shouldComponentUpdate() 函数决定了一个组件什么时候不需要渲染

在更新过程, React 库首先会调用shouldComponentUpdate 函数, 如果函数返回true, 那么会继续更新过程; 如果返回false, 那么会立即停止更新

对于React 组件来说, 默认的shouldComponentUpdate() 寿命周期方法在任何情况下均返回true

通过this.setState 函数引发的更新过程, 并不是立刻更新组件的state 值, 在执行到函数shouldComponentUpdate 时, this.state 依然是this.setState 函数执行之前的值
```
shouldComponentUpdate(nextProps, nestState) {
    
}
```

### 4. componentWillMount/componentDidUpdate
componentWillMount: 在render 函数之前执行, 可以在服务器/浏览器环境中调用;
componentDidUpdate: 在render 函数之后执行, 可以在服务器/浏览器环境中调用;


## 5.4 卸载过程(Unmount)
### 1. 执行流程
1. componentWillUnmount()

### 2. componentWillUnmount
该函数适合做一些清理工作, 比如清除定时器, 清除DOM


# 6. 简要点
1. React 应用都是围绕组件设计的
2. 拆分组件最关键的就是确定组件边界, 每个组件都应该是可以独立存在的
3. 每个React 组件都可以通过this.forceUpdate() 函数强行引发一次重新绘制
