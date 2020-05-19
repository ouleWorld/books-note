[toc]

# 1. JSX
## 1.1 什么是JSX?
所谓的JSX, 就是JavaScript 语法的扩展, 在JSX 中, 使用的元素不局限于HTML 中的元素, 可以是任何一个React 组件

## 1.2 传统onclick 写法的弊端
1. onclick 添加的事件处理函数是在全局环境下执行的, 这污染了全局环境, 很容易造成意料不到的结果
2. 给很多DOM 元素添加onclick 事件, 可能会影响网页的性能, 毕竟, 网页需要的事件处理函数越多, 性能就越低
3. 对于使用onclick 的DOM 元素, 如果要动态地从DOM 树中删掉的话, 需要把对应的事件处理器注销掉, 假如忘了注销, 就可能造成内存泄漏, 这样的BUG 很难被发现

## 1.3 JSX 语法优势
相较于传统的onclick 写法, JSX 具有如下的优势:
1. JSX 中onClick 处理事件的本质是事件委托; 无论页面中出现多少个onClick, 其实最后都只在DOM 树上添加了一个事件处理函数, 挂在最顶层的DOM 节点上; 而onclick 本质是事件绑定, 事件委托的性能当然比事件绑定的性能要好
2. 因为React 控制了组件的生命周期, 在unmount 的时候自然能够去除相关的所有事件处理函数, 内存泄漏将不再是一个问题

## 1.3 JSX 的组件封装
JSX 可以实现一种真正的组件封装, 他可以将JavaScript CSS HTML 等内容都定义在一个组件内
```
const button = {
    background: 'black',
    color: '#fff'
}

class App extends React.Component{
    constructor(props) {
        super(props)
        this.onClickButton = this.onClickButton.bind(this)
        this.state = {}
    }

    onClickButton() {
        console.log('click Event')
    }

    render() {
        return (
            <div className="box">
                <span style={ button }>hello world</span>
                <div onClick={ this.onClickButton }>click me!</div>
            </div>
        )
    }
}
```

# 2. virtual DOM
virtual DOM 是React 优化渲染的方式, 当组件进行更新时, virtual DOM 能够使浏览器使用最少的步骤去更新页面(不是理论意义上的最少, 这里的最少考虑的时间的因素)
每次自上而下地渲染组件时, 会对比这一次产生的virtual DOM 和上一次产生的virtual DOM, 对比就会发现差别, 然后修改真正的DOM 树时就只需要触及差别中的部分即可

virtual DOM 更新页面的步骤
1. 根据数据变化, 执行render
2. 生成新的virtual DOM
3. 将新的virtual DOM 和旧的virtual DOM 比较, 获取两个virtual DOM 之间的差别
4. 根据virtual DOM 的差别, 计算出需要更新页面的步骤
5. 更新页面
6. 更新virtual DOM

# 3. React 注意点
1. React 非常适合构建用户交互组件
2. React 判断一个元素是HTML 元素还是React 组件的原则就是第一个字母是否大写
3. react的理念:UI=render(data), 即数据发生改变, 页面就发生改变