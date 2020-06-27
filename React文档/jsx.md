[toc]

jsx

# 1. 基本概念
## 1.1 定义
JSX 是React 一种新的语法结构, 他表示JavaScript 的拓展

## 1.2 JSX 属性值
jsx 的属性值分为两种, 一种是JavaScript 表达式, 另外一种是字符串字面量
```
const element = <img classNames="imgEle" src={user.avatarUrl}></img>;
```

## 1.3 jsx 与函数
在React 组件中, 任何地方均可以使用JSX, 因为React 打包组件时会使用JSX 解析器解析JSX
同时我们可以在函数中直接使用JSX,  不过在函数中, 推荐使用括号包裹JSX 内容, 虽然这样做不是强制要求的，但是这可以避免遇到自动插入分号
```
class App extends React.Component{
    constructor(props) {
        super(props)
        this.jsxTest = this.jsxTest.bind(this)
        this.state = {}
    }

    jsxTest() {
        return <div>jsx test</div>
    }

    render() {
        let button = {
            background: 'black',
            color: '#fff'
        }
        return (
            <div className="box">
                {this.jsxTest()}
            </div>
        )
    }
}
```

## 1.4 JSX 防止注入攻击
JSX 在写入页面之前, 会自动经过转义, 它可以确保在你的应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转换成了字符串。这样可以有效地防止 XSS（cross-site-scripting, 跨站脚本）攻击。

# 2. JSX 的本质
JSX 仅仅只是 React.createElement(component, props, ...children) 函数的语法糖

如下JSX 代码
```
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```
会被编译为
```
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```
这同时也解释了, 为什么所有组件必须使react 在作用域内

# 3. 使用JSX 定义组件
## 3.1 点语法
JSX 中可以直接使用点语法去访问一个组件
```
<MyContext.Provider value={{something: 'something'}}>
    <Toolbar />
</MyContext.Provider>
```

## 3.2 组件名称
大写字母开头的组件会被解析成自定义组件; 小写字母开头的组件会被解析成HTML 内置组件, 因此我们在自定义组件时一定要注意: 组件的名称一定要大写字母开头(其实class 类定义的规则就是大写字母开头)

## 3.3 组件类型
React 组件的名称只能是一个变量, 不能是一个表达式
```
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 错误！JSX 类型不能是一个表达式。
  // return <components[props.storyType] story={props.story} />;
  
  // 正确！JSX 类型可以是大写字母开头的变量。
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

# 4 JSX 中的props
## 4.1 字符串变量
下面两个JSX 表达式是等价的, 因为解析JSX 语法时会默认地进行转义
```
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

## 4.2 props 默认为true 值
下面两个表达式是等价的, 这个性质和HTML 表现一致 
```
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

## 4.3 属性展开
下面两个表达式是等价的, react 解析器就是这么工作的
```
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

# 5. JSX 中的子元素
## 5.1 字符串变量
JSX 会移除行首尾的空格以及空行。与标签相邻的空行均会被删除，文本字符串之间的新行会被压缩为一个空格

因此下面的表达式都是等价的:
```
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

## 5.2 子元素与数组
React 组件也能够返回存储在数组中的一组元素, 这种返回方式允许不设置最外层包裹元素
```
render() {
  // 不需要用额外的元素包裹列表元素！
  return [
    // 不要忘记设置 key :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

## 5.3 JavaScript 表达式作为子元素
渲染HTML 列表
```
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

## 5.4 函数作为子元素
注意这种写法, 如果是我自己, 我可能不会使用这样的写法, 我会把函数定义在子元素的中, 而不是暴露在父元素中
```
// 调用子元素回调 numTimes 次，来重复生成组件
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```

## 5.5 特殊类型
false, null, undefined, true 是合法的子元素。但它们并不会被渲染
但是特别注意, 0 是被会被渲染的