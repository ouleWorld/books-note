[toc]

profiler

# 1. 定义
Profiler 测量渲染一个 React 应用多久渲染一次以及渲染一次的“代价”。 它的目的是识别出应用中渲染较慢的部分
这是一个测试使用的组件, 一个为了性能优化而存在的组件

PS: 
1. Profiling 增加了额外的开支，所以它在生产构建中会被禁用
2. 尽管 Profiler 是一个轻量级组件，我们依然应该在需要时才去使用它。对一个应用来说，每添加一些都会给 CPU 和内存带来一些负担

# 12. demo
```
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Main {...props} />
  </App>
);
```
参数:
1. id: 表示当前profiler 的id(可以理解成为的一般组件的ID)
2. onRender: 表示当树组件提交更新时被React 调用的函数

# 3. onRender
参数解释的链接: https://react.docschina.org/docs/profiler.html

# 4. 个人评价
这个组件是一个为了测试而存在的组件, 用于某些内容的优化查询

暂时没有见过很好的使用方式, 如果以性能优化为目的, 个人可能还不会使用该API