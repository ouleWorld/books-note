[toc]

# 1. Vue 实例
1. 一个 Vue 应用由一个通过 new Vue 创建的根 Vue 实例
2. 所有的 Vue 组件都是 Vue 实例，并且接受相同的选项对象 (一些根实例特有的选项除外)

# 2. Vue 的响应式系统
说明对象的赋值的时候属于浅复制
```
// 我们的数据对象
var data = { a: 1 }

// 该对象被加入到一个 Vue 实例中
var vm = new Vue({
  data: data
})

// 获得这个实例上的 property
// 返回源数据中对应的字段
vm.a == data.a // => true

// 设置 property 也会影响到原始数据
vm.a = 2
data.a // => 2

// ……反之亦然
data.a = 3
vm.a // => 3
```

注意点: 只有存在的属性才会触发响应式, 后续添加的属性将不会触发响应式(这个点其实很好理解)

# 3. 阻止响应式更新: Object.freeze()
Object.freeze(): 冻结一个对象。一个被冻结的对象再也不能被修改
```
var obj = {
  foo: 'bar'
}

Object.freeze(obj)

obj.foo = 'here' // 数据不会被更新
```

# 4. Vue 实例的属性和方法
```
let vm = new Vue()
```
1. vm.$data: 表示实例的data 属性
2. vm.$el: 表示实例的根元素
3. vm.$watch: 监视器函数


# 5. 实例的生命周期
生命周期钩子的 this 上下文指向调用它的 Vue 实例
这个内容之后再来补充吧


# 6. 注意点
1. 不要在选项 property 使用箭头函数, 因为箭头本身没有this
2. 不要声明周期的回调函数中使用箭头函数, 因为箭头函数本身没有this