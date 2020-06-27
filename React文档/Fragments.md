[toc]

Fragments.md

# 1. 定义
React 中的一个常见模式是一个组件返回多个元素。Fragments 允许你将子列表分组，而无需向 DOM 添加额外节点
```
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

# 2. 动机
Fragments 创造的原因是因为, 在组件中只允许组件返回一个根节点, 该性质可能会造成页面中大量的节点剩余, 消耗性能;
同时组件只允许返回一个节点的性质造成了 table ul 这种标签不适合在react 作组件写法;
以上就是创造Fragments 的原因
https://react.docschina.org/docs/fragments.html#short-syntax

# 3. 应用
## 3.1 React.Fragment
```
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```

## 3.2 短语法
```
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```

# 4. React.Fragment 属性
key 是唯一可以传递给 Fragment 的属性。未来我们可能会添加对其他属性的支持，例如事件。
```
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // 没有`key`，React 会发出一个关键警告
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```