[toc]

Hook

# 1. Hook 简介
## 1.1 定义
Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性

Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数。Hook 不能在 class 组件中使用 —— 这使得你不使用 class 也能使用 React

Hook 是一个特殊的函数，它可以让你“钩入” React 的特性

hook 只会在组件首次渲染时被创建

# 2. useState
## 2.1 定义
useState 是一个Hook, 通过在函数组件中调用它来给组件添加一些内部的state.
useState 唯一的参数就是初始 state, 它可以是一个值, 同时也可以是一个对象
useState 会返回一对值：当前状态和一个让你更新它的函数，你可以在事件处理函数中或其他一些地方调用这个函数

## 2.2 demo
```
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 “count” 的 state 变量。
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

## 2.3 注意点
1. Hook 返回的更新函数有点类似于this.setState, 但是它不会将旧的state 和新的state合并, 而是直接使用新的state 替换就的state
2. useState的初始值，只在第一次有效

## 2.3 常见问题
问题描述: 当我点击按钮修改name的值的时候，我发现在Child组件， 是收到了，但是并没有通过useState赋值给name！
原因: useState 可以理解为构造函数, 只会在组件第一次构建的时候执行, 后续的更新操作并不会执行useState
```
const Child = memo(({data}) =>{
    console.log('child render...', data)
    const [name, setName] = useState(data)
    return (
        <div>
            <div>child</div>
            <div>{name} --- {data}</div>
        </div>
    );
})

const Hook =()=>{
    console.log('Hook render...')
    const [count, setCount] = useState(0)
    const [name, setName] = useState('rose')

    return(
        <div>
            <div>
                {count}
            </div>
            <button onClick={()=>setCount(count+1)}>update count </button>
            <button onClick={()=>setName('jack')}>update name </button>
            <Child data={name}/>
        </div>
    )
}
```

# 3. useEffect
## 3.1 副作用
副作用:你之前可能已经在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”，或者简称为“作用”

## 3.2 定义
useEffect 就是一个 Effect Hook，给函数组件增加了操作副作用的能力, 
可以把 useEffect Hook 看做 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合
它在第一次渲染之后和每次更新之后都会执行, 在卸载和重新渲染的时候会执行Effect Hook 返回的函数
清除函数可以是一个匿名函数

PS: 与 componentDidMount 或 componentDidUpdate 不同，在浏览器完成布局与绘制之后，传给 useEffect 的函数会延迟调用, 使用 useEffect 调度的 effect 不会阻塞浏览器更新屏幕

```
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 使用浏览器的 API 更新页面标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

<<<<<<< HEAD
## 3.3 不需要清除的 effect
=======
## 3.3不需要清除的 effect
>>>>>>> ff8a7f02f28d96ec96d077e6c2c81d5bcd21f2d3
```
useEffect(() => {
    document.title = `You clicked ${count} times`;
});
```

## 3.4 需要清除的 effect
这是 effect 可选的清除机制。每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分
effect hook 返回的函数表示清除机制
```
useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
});
```

## 3.5 effect 知识点集合
### 1. 只在第一次使用的componentDidMount，可以用来请求异步数据
> useEffect最后，加了[]就表示只第一次执行
```
useEffect(()=>{
    const users = 获取全国人民的信息()
},[])
```

### 2. 用来替代willUpdate等每次渲染都会执行的生命函数
> useEffect最后，不加[]就表示每一次渲染都执行
```
useEffect(()=>{
    const users = 每次都获取全国人民的信息()
})
```

### 3. effect 的shouldComponentUpdate
> useEffect最后，加[]，并且[]里面加的字段就表示，这个字段更改了，我这个effect才执行
```
useEffect(() => {
    const users = （name改变了我才获取全国人民的信息()）
},[name])
```

### 4. 多熟悉的effect
> 可以写多个useEffect
```
useEffect(() => {
    const users = （name改变了我才获取全国人民的name信息()）
},[name])

useEffect(() => {
    const users = （name改变了我才获取全国人民的age信息()）
},[age])
```

### 5. 取消订阅, 相当于替换componentWillUnMount 生命周期
> 在effect的return里面可以做取消订阅的事
```
useEffect(() => {
    const subscription = 订阅全国人民吃饭的情报！
    return () => {
        取消订阅全国人民吃饭的情报！
    }
},[])
```

### 6. useEffect 中的state 值被固定在useEffect内部， 不会被改变，除非useEffect刷新，重新固定state的值
```
const [count, setCount] = useState(0)
   useEffect(() => {
       console.log('use effect...',count)
       const timer = setInterval(() => {
           console.log('timer...count:', count)
           setCount(count + 1)
       }, 1000)
       return ()=> clearInterval(timer)
   },[])
```

### 7. useEffect 不能被打断
```
const [count, setCount] = useState(0)
useEffect(...)

return // 函数提前结束了

useEffect(...)
}
```

# 4. useRef
## 4.1 定义
useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变

## 4.2 demo
```
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

## 4.3 相当于全局作用域, 一出修改, 其他地方全更新
> const countRef = useRef(0)
```
const [count, setCount] = useState(0)
const countRef = useRef(0)
useEffect(() => {
    console.log('use effect...',count)
    const timer = setInterval(() => {
        console.log('timer...count:', countRef.current)
        setCount(++countRef.current)
    }, 1000)
    return ()=> clearInterval(timer)
},[])

```

## 4.3 普通操作, 用于表示DOM 
> const btnRef = useRef(null)
```
const Hook =()=>{
    const [count, setCount] = useState(0)
    const btnRef = useRef(null)

    useEffect(() => {
        console.log('use effect...')
        const onClick = ()=>{
            setCount(count+1)
        }
        btnRef.current.addEventListener('click',onClick, false)
        return ()=> btnRef.current.removeEventListener('click',onClick, false)
    },[count])

    return(
        <div>
            <div>
                {count}
            </div>
            <button ref={btnRef}>click me </button>
        </div>
    )
}

```

# 5. useMemo
## 5.1 定义
根据指定的函数, 返回一个 memoized 值
useMemo 具有暂存能力

## 5.2 demo
```
const data = useMemo(()=>{
      return {
          name
      }
},[name])
```

## 5.3 确定一个组件是否需要更新
如果没有useMemo 的存在, 那么每次Hook 组件执行render 方法时, Child 组件都会被更新(data 被赋予新的变量时)
而由于useMemo 具有暂存能力, 因此能够比较前后传递的参数变化, 如果传递的参数没有变化(值本身), 那么就不会更新组件
```
const Child = memo(({data}) =>{
    console.log('child render...', data.name)
    return (
        <div>
            <div>child</div>
            <div>{data.name}</div>
        </div>
    );
})

const Hook =()=>{
    console.log('Hook render...')
    const [count, setCount] = useState(0)
    const [name, setName] = useState('rose')

   const data = useMemo(()=>{
        return {
            name
        }
    },[name])
    
    return(
        <div>
            <div>
                {count}
            </div>
            <button onClick={()=>setCount(count+1)}>update count </button>
            <Child data={data}/>
        </div>
    )
}

```

# 6. useCallback
## 6.1 定义
提供函数的暂存能力, 用于比较前后两个函数是否发生变化, 以此来确定是否执行组件的render

## 6.2 demo
```
const onChange = useCallback((e)=>{
     setText(e.target.value)
},[])
```

## 6.3 确定一个组件是否需要更新
```
const Child = memo(({data, onChange}) =>{
    console.log('child render...')
    return (
        <div>
            <div>child</div>
            <div>{data}</div>
            <input type="text" onChange={onChange}/>
        </div>
    );
})

const Hook =()=>{
    console.log('Hook render...')
    const [count, setCount] = useState(0)
    const [name, setName] = useState('rose')
    const [text, setText] = useState('')

   const onChange=(e)=>{
        setText(e.target.value)
   }
    
    return(
        <div>
            <div>count: {count}</div>
            <div>text : {text}</div>
            <button onClick={()=>setCount(count + 1)}>count + 1</button>
            <Child data={name} onChange={onChange}/>
        </div>
    )
}
```
当Child 组件被点击时, 触发onChange 函数, 重新渲染Hook 组件, 此时由于该组件是函数组件, 会重新执行一遍函数体, 此时onChange 变量拥有了新的内存空间, 因此会触发props 不一致而导致的virtual DOM 判断组件不一致, 而导致组件的更新;

但是, 此时这种更新完全是多余的, 因为的函数主体并没有发生改变, 因此useCallback 就是用来解决该问题

# 7. useReducer
## 7.1 定义
useState 的替代方案。它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法

## 7.2 demo
```
const reducer =(state = 0, {type})=>{
    switch (type) {
        case "add":
            return state+1
        case 'delete':
            return state-1
        default:
            return state;
    }
}

const Hook =()=>{
    const [count, dispatch] = useReducer(reducer, 0)
    return(
        <div>
           count:{count}
           <button onClick={()=> dispatch({type:'add'})}>add</button>
            <button onClick={()=> dispatch({type:'delete'})}>delete</button>
        </div>
    )
}

export default Hook
```

# 8. useContext
## 8.1 定义
相当于class 中的Context

## 8.2 demo
```
import React, {useContext, useReducer} from 'react'

const reducer = (state = 0, {type}) => {
    switch (type) {
        case "add":
            return state + 1
        case 'delete':
            return state - 1
        default:
            return state;
    }
}
const Context = React.createContext(null);

const Child = () => {
    const [count, dispatch] = useContext(Context)
    return (
        <div>
            <div>child...{count}</div>
            <button onClick={() => dispatch({type: 'add'})}>child add</button>
            <button onClick={() => dispatch({type: 'delete'})}>child delete</button>
        </div>

    )
}

const Hook = () => {
    const [count, dispatch] = useReducer(reducer, 10)
    return (
        <Context.Provider value={[count, dispatch]}>
            <div>
                <div>mom ... {count}</div>
                <Child/>
                <button onClick={() => dispatch({type: 'add'})}>mom add</button>
                <button onClick={() => dispatch({type: 'delete'})}>mom delete</button>
            </div>
        </Context.Provider>
    )
}

export default Hook
```

# 9. useImperativeHandle
## 9.1 定义
可以将组件中的ref 指定的元素暴露给父组件

## 9.2 demo
在本例中，渲染 <FancyInput ref={inputRef} /> 的父组件可以调用 inputRef.current.focus()
```
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

# 10. useLayoutEffect
## 10.1 定义
其函数签名与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新

# 11. 自定义Hook
## 11.1 规则
自定义 Hook 是一个函数，其名称以 “use” 开头，函数内部可以调用其他的 Hook; 其他内容和一个正常的函数类似

自定义Hook 在不同的组件中会有不同的状态

## 11.2 demo
> 自定义一个当resize 的时候 监听window的width和height的hook
```
import {useEffect, useState} from "react";

export const useWindowSize = () => {
    const [width, setWidth] = useState()
    const [height, setHeight] = useState()

    useEffect(() => {
        const {clientWidth, clientHeight} = document.documentElement
        setWidth(clientWidth)
        setHeight(clientHeight)
    }, [])

    useEffect(() => {
        const handleWindowSize = () =>{
            const {clientWidth, clientHeight} = document.documentElement
            setWidth(clientWidth)
            setHeight(clientHeight)
        };

        window.addEventListener('resize', handleWindowSize, false)

        return () => {
            window.removeEventListener('resize',handleWindowSize, false)
        }
    })

    return [width, height]
}
```

# 12. Hook 的规则
## 12.1. Hook 就是 JavaScript 函数

## 12.2 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用
这条规则是为了确保Hook 在每一次渲染中都按照同样的顺序被调用, 这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确

## 12.3 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。（还有一个地方可以调用 Hook —— 就是自定义的 Hook 中，我们稍后会学习到。

# 13. ESLint 插件
该插件能够强制执行Hook 规则

下载:
```
npm install eslint-plugin-react-hooks --save-dev
```

配置:
```
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // 检查 Hook 的规则
    "react-hooks/exhaustive-deps": "warn" // 检查 effect 的依赖
  }
}
```


# 参考链接
1. [终于搞懂 React Hooks了](https://juejin.im/post/5e53d9116fb9a07c9070da44#heading-29)


