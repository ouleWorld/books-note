[toc]

DOM 元素

# 1. React 中的DOM 
React 实现了一套独立于浏览器的 DOM 系统，兼顾了性能和跨浏览器的兼容性

在 React 中，所有的 DOM 特性和属性（包括事件处理）都应该是小驼峰命名的方式(data- 属性除外)

# 2. 属性差异
## 2.1 checked
checked 用于受控组件, defaultChecked  用于非受控组件

## 2.2 className
表示属性class

## 2.3 dangerouslySetInnerHTML
dangerouslySetInnerHTML 是 React 为浏览器 DOM 提供 innerHTML 的替换方案

用法如下:
```
function createMarkup() {
    return {__html: 'First &middot; Second'};
}

function MyComponent() {
    // 但当你想设置 dangerouslySetInnerHTML 时，需要向其传递包含 key 为 __html 的对象，以此来警示你
    return <div dangerouslySetInnerHTML={createMarkup()} />;
}
``` 

## 2.4 htmlFor
表示属性for

## 2.5 onChange
在HTML 中绑定事件使用的属性是onchange, React 中的写法是onChange

## 2.6 selected
<option> 组件支持 selected 属性, 该属性常用与受控组件

## 2.8 value
value 常用与受控组件, defaultValue 常用与非受控组件

# 3. React DOM 属性
```
accept acceptCharset accessKey action allowFullScreen alt async autoComplete
autoFocus autoPlay capture cellPadding cellSpacing challenge charSet checked
cite classID className colSpan cols content contentEditable contextMenu controls
controlsList coords crossOrigin data dateTime default defer dir disabled
download draggable encType form formAction formEncType formMethod formNoValidate
formTarget frameBorder headers height hidden high href hrefLang htmlFor
httpEquiv icon id inputMode integrity is keyParams keyType kind label lang list
loop low manifest marginHeight marginWidth max maxLength media mediaGroup method
min minLength multiple muted name noValidate nonce open optimum pattern
placeholder poster preload profile radioGroup readOnly rel required reversed
role rowSpan rows sandbox scope scoped scrolling seamless selected shape size
sizes span spellCheck src srcDoc srcLang srcSet start step style summary
tabIndex target title type useMap value width wmode wrap
```

# 4. 注意点
1. 你也可以使用自定义属性，但要注意属性名全都为小写
2. React 中, 所有的 SVG 属性也完全得到了支持
