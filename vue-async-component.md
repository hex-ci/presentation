# Vue 子组件的延迟加载及其生命周期控制

前端开发社区的繁荣，造就了很多优秀的基于 MVVM 设计模式的框架，而组件化开发思想也越来越深入人心。这其中不得不提到 Vue.js 这个专注于 VM 层的框架。

本文主要对 Vue 组件化开发中子组件的延迟加载和其生命周期进行一些探讨。阅读本文需要对 Vue 有一定的了解。

## 异步组件

讨论延迟加载，需要先了解下异步组件。Vue 的异步组件是把组件定义为一个工厂函数，在组件需要渲染时触发工厂函数，并且把结果缓存起来，用于后面的再次渲染。例如注册一个全局异步组件：

```javascript
Vue.component('async-demo', function(resolve, reject) {
  setTimeout(function() {
    // 将组件定义传入 resolve 回调函数
    resolve({
      template: '<div>I am async!</div>'
      // 组件的其他选项
    })
  }, 1000)
})
```

异步子组件和全局注册很类似：

```javascript
Vue.component('parent-demo', {
  // 父组件的其他选项
  components: {
    'async-demo': function(resolve, reject) {
      setTimeout(function() {
        // 将组件定义传入 resolve 回调函数
        resolve({
          template: '<div>I am async!</div>'
          // 子组件的其他选项
        })
      }, 1000)
    }
  }
})
```


## 延迟加载

## 生命周期
