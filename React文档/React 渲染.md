[toc]

React 渲染

# 1. React 渲染
## 1.1 元素渲染
在react 中, 所有的内容都是由根节点渲染的, 所有的内容都是由React DOM 管理
```
ReactDom.render(<App />, document.getElementById('root'))
``` 

## 1.2 组件更新之后的渲染
React 中, render 方法会将JSX 语法转换成为virtual DOM
当第一次渲染完成之后, 如果组件再发生更新, React DOM 会将新virtual DOM 与旧的virtual DOM 比较, 得到需要更新页面的操作, 因此, React 应用的页面更新只会更新需要改变的内容

# 2. 条件渲染
## 2.1 条件渲染的方式
1. 与运算符 &&
2. 三目运算符

## 2.2 阻止事件渲染
在极少数情况下，你可能希望能隐藏组件，即使它已经被其他组件渲染。若要完成此操作，你可以让 render 方法直接返回 null，而不进行任何渲染

在组件的 render 方法中返回 null 并不会影响组件的生命周期。例如，上面这个示例中，componentDidUpdate 依然会被调用

# 3. 列表渲染
## 3.1 列表渲染demo
列表渲染就是使用map 方法渲染具有规律的元素, 不过需要注意的是列表渲染需要指定key
```
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

## 3.2 key
key 帮助 React 识别哪些元素改变了，比如被添加或删除。因此你应当给数组中的每一个元素赋予一个确定的标识, 如果你选择不指定显式的 key 值，那么 React 将默认使用索引用作为列表项目的 key 值。

一个元素的 key 最好是这个元素在列表中拥有的一个独一无二的字符串, 如果元素没有确定id 的时候, 此时可以使用index 作为 key

如果列表项目的顺序可能会变化，我们不建议使用索引来用作 key 值，因为这样做会导致性能变差，还可能引起组件状态的问题

一个好的经验法则是：在 map() 方法中的元素需要设置 key 属性 