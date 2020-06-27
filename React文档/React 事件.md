[toc]

React 事件

# 1. React 中的事件
## 1.1 定义
SyntheticEvent 实例将被传递给你的事件处理函数，它是浏览器的原生事件的跨浏览器包装器。除兼容所有浏览器外，它还拥有和浏览器原生事件相同的接口，包括 stopPropagation() 和 preventDefault()。

不能通过返回 false 的方式阻止默认行为。你必须显式的使用 preventDefault
你一般不需要使用 addEventListener 为已创建的 DOM 元素添加监听器。事实上，你只需要在该元素初始渲染的时候添加监听器即可

stopPropagation(): 终止事件的冒泡
preventDefault(): 取消事件的默认行为

## 1.2 属性
每个SyntheticEvent 对象都包含以下属性
```
boolean bubbles
boolean cancelable
DOMEventTarget currentTarget
boolean defaultPrevented
number eventPhase
boolean isTrusted
DOMEvent nativeEvent
void preventDefault()
boolean isDefaultPrevented()
void stopPropagation()
boolean isPropagationStopped()
void persist()
DOMEventTarget target
number timeStamp
string type
```

# 2. 事件池
SyntheticEvent 是合并而来。这意味着 SyntheticEvent 对象可能会被重用，而且在事件回调函数被调用后，所有的属性都会无效。出于性能考虑，你不能通过异步访问事件

```
function onClick(event) {
  console.log(event); // => nullified object.
  console.log(event.type); // => "click"
  const eventType = event.type; // => "click"

  setTimeout(function() {
    console.log(event.type); // => null
    console.log(eventType); // => "click"
  }, 0);

  // 不起作用，this.state.clickEvent 的值将会只包含 null
  this.setState({clickEvent: event});

  // 你仍然可以导出事件属性
  this.setState({eventType: event.type});
}
```

# 3 绑定this
## 3.1 手动绑定
```
constructor(props) {
    super(props)
    this.jsxTest = this.jsxTest.bind(this)
    this.state = {}
}

jsxTest() {
    return <div>jsx test</div>
}
```

## 3.2 使用public class fields 语法
```
 // 此语法确保 `handleClick` 内的 `this` 已被绑定。
  // 注意: 这是 *实验性* 语法。
  handleClick = () => {
    console.log('this is:', this);
  }
```

## 3.3 在回调中使用箭头函数
如果该回调函数作为 prop 传入子组件时，这些组件可能会进行额外的重新渲染(不推荐使用)
```
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    return (
      <button onClick={() => this.handleClick()}>
        Click me
      </button>
    );
  }
}
```

# 4 向事件处理程序传递参数
```
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

# 5. 事件参考
https://react.docschina.org/docs/events.html#clipboard-events


# 注意点
1. onClick 表示在事件冒泡阶段触发回调; onClickCapture 表示在事件捕获阶段触发回调