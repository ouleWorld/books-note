[toc]

API

# 1. React
## 1.1 函数
1. React.createElement(): 创建并返回指定的React 元素, jsx 的本质就是该函数
2. React.cloneElement(): 以 element 元素为样板克隆并返回新的 React 元素。返回元素的 props 是将新的 props 与原始元素的 props 浅层合并后的结果
4. React.isValidElement(): 验证对象是否为 React 元素，返回值为 true 或 false

## 1.2 接口
1. React.Component: React 组件的基类(并未实现shouldComponentUpdate())
2. React.PureComponent: React 组件的基类(以浅层对比 prop 和 state 的方式实现了shouldComponentUpdate())
3. React.memo: 表示高阶组件
5. React.Children: React.Children 提供了用于处理 this.props.children 不透明数据结构的实用方法
6. React.Fragment: 能够在不额外创建 DOM 元素的情况下，让 render() 方法中返回多个元素。
7. React.createRef: 创建一个能够通过 ref 属性附加到 React 元素的 ref
8. React.forwardRef: 会创建一个React组件，这个组件能够将其接受的 ref 属性转发到其组件树下的另一个组件中
9. React.lazy: 允许你定义一个动态加载的组件。这有助于缩减 bundle 的体积，并延迟加载在初次渲染时未用到的组件
10. React.Suspense: 可以指定加载指示器（loading indicator）

## 1.3 React.children
1. React.Children.map(): 在 children 里的每个直接子节点上调用一个函数
2. React.Children.forEach(): 与 React.Children.map() 类似，但它不会返回一个数组。
3. React.Children.count(): 返回 children 中的组件总数量，等同于通过 map 或 forEach 调用回调函数的次数。
4. React.Children.only(): 验证 children 是否只有一个子节点（一个 React 元素），如果有则返回它，否则此方法会抛出错误。


# 2. ReactDom
1. ReactDom.render(): 将一个React 元素渲染到页面中
2. ReactDom.hydrate(): render() 相同，但它用于在 ReactDOMServer 渲染的容器中对 HTML 的内容进行 hydrate 操作
3. ReactDom.unmountComponentAtNode(): 从 DOM 中卸载组件，会将其事件处理器（event handlers）和 state 一并清除。如果指定容器上没有对应已挂载的组件，这个函数什么也不会做。如果组件被移除将会返回 true，如果没有组件可被移除将会返回 false
4. ReactDom.findDOMNode(): findDOMNode 是一个访问底层 DOM 节点的应急方案（escape hatch
5. ReactDom.createPortal(): 创建 portal。Portal 将提供一种将子节点渲染到 DOM 节点中的方式，该节点存在于 DOM 组件的层次结构之外

# 3. ReactDOMServer
```
import ReactDOMServer from 'react-dom/server';

var ReactDOMServer = require('react-dom/server');
```
## 3.1 浏览器 + 服务器
ReactDOMServer.renderToString(element): 将 React 元素渲染为初始 HTML
ReactDOMServer.renderToStaticMarkup(element): 此方法与 renderToString 相似，但此方法不会在 React 内部创建的额外 DOM 属性

## 3.2 服务器
ReactDOMServer.renderToNodeStream(element): 将一个 React 元素渲染成其初始 HTML。返回一个可输出 HTML 字符串的可读流
ReactDOMServer.renderToStaticNodeStream(element): 此方法与 renderToNodeStream 相似，但此方法不会在 React 内部创建的额外 DOM 属性


