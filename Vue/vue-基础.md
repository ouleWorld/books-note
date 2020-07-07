[toc]

# 1. 特点
1. 渐进式框架
2. SPA
3. Vue 也提供一个强大的过渡效果系统，可以在 Vue 插入/更新/移除元素时自动应用过渡效果

# 2. VUE 指令
1. v-bind: 绑定一个attribute, 动态计算它的值
2. v-if: 
3. v-for: 列表渲染
4. v-on: 事件监听
5. v-model: 双向绑定
6. v-once
7. v-html


# 3. 注册组件
```
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
```