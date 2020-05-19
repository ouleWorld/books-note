[toc]

Portals

# 1. portals
## 1.1 定义
Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案

## 1.2 语法
```
ReactDOM.createPortal(child, container)
```
参数:
1. child: 表示任意可渲染的React 的子元素(需要渲染的元素)
2. container: 一个DOM 元素, 表示child 的父节点(需要将child 渲染的目标节点)

# 2. 应用
现在有这样的一个应用场景: ComponentA, ComponentB 表示两个相连的兄弟组件, 现在的需求是需要在组件ComponentA 中控制组件ComponentB 中内容的显示状态

## 2.1 props 组件通信
使用props 解决上述场景, 只能将ComponentA, ComponentB 两个组件包装在同一个组件下, 然后使用单向数据流 + callback 的方式实现需求:

具体做法如下: 当点了ComponentA 中的按键, 那么调用回调函数, 在父组件中改变this.state.number 状态, 之后更新页面UI
```
<React.Fragment>
    <ComponentA onClick={this.onClick}></ComponentA>
    <ComponentB number={this.state.number}></ComponentB>
</React.Fragment>
```

这种方式是最常见的解决方式, 他利用React props 数据流的特点
PS: 由于这种方式太常见, 故没有写demo 演示

## 2.2 portal 渲染组件
使用React 的portals 解决上述问题
demo演示地址:  D:\code\githubCode\studyDemo\react\Portals

```
<div id="root"></div>
<div class="ComponentB"></div>
```
这是portal 演示内容的html 文件, 可以看到最外层的出了React 常见的root 组件, 还存在了一个ComponentB 组件; 对于这种html 结构, 传统props 是不能够实现两个组件

# 3. portals 的特点
## 3.1 portals 中的事件, 不仅会沿着被插入的节点冒泡, 同时也会沿着定义的节点冒泡
在portal 演示demo 中, 点击了portals 组件, 在控制台输出了以下内容:
```
here
Click Event!
```

可以看到, 事件同时被root, ComponentB 两个节点捕获

## 3.2 一种全新的兄弟组件通信的方式
情况1: 在传统的React 项目中, html 的结构一般是以下这种形式:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>React demo</title>
</head>
<body>
<div id="root"></div>
<script src="./build/a.js"></script>
</body>
</html>
```

情况2: 而如果有些项目需要如下这种布局: 拥有两个或以上的最外层div 节点
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>React demo</title>
</head>
<body>
<div id="root"></div>
<div class="ComponentB"></div>
<script src="./build/a.js"></script>
</body>
</html>
```

针对于情况2 这种情况, 传统的props 通信方式是不能进行root, ComponentB 两个组件之间的通信的; 因为props 进行兄弟组件之间的通信必须满足拥有同一个能够定义操作的父组件

在这种情况下, root, ComponentB 虽然拥有同一个父组件 body, 但是body 在React 框架中是不能够定义操作的, 因此props 通信的局限性就显示了出来. 而portals 就拓展了这种局限性.

PS: 该特点是portals 与props 最大的不同点, 因为大部分情况下都能使用props 代替 portals; 唯独这种情况, 只能使用props 处理

# 4. 应用场景
1. 弹出框, 弹出框是portals 一个非常好的应用场景.

# 5. 参考链接
1. [demo](D:\code\githubCode\studyDemo\react\Portals)



 