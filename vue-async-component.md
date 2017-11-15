# Vue 子组件的延迟加载及其生命周期控制

前端开发社区的繁荣，造就了很多优秀的基于 MVVM 设计模式的框架，而组件化开发思想也越来越深入人心。这其中不得不提到 Vue.js 这个专注于 VM 层的框架。

本文主要对 Vue 组件化开发中子组件的延迟加载和其生命周期进行一些探讨。阅读本文需要对 Vue 有一定的了解。

> 注意：本文中的例子代码，都是以 vue-cli 采用 webpack 模板初始化的项目为基础。

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

工厂函数的第一个参数 `resolve` 为**成功后的回调**，第二个参数 `reject` 为**失败后的回调**，可以在这里提示用户加载失败等。

这里使用 `setTimeout` 只是为了模拟异步，在实际项目中，应该配合 webpack 的代码分离功能来实现延迟加载。

## 延迟加载

在实际的项目中，如果不使用延迟加载，则 Vue 组件的 JS、CSS 和模板都会打包到一个 .js 文件中，这个文件可能达到几 MB 甚至更多，严重影响首屏加载时间。所以在项目中我们需要启用组件的延迟加载。

### webpack 代码分离

webpack 的代码分离有两种，第一种，也是优先选择的方式是，使用符合 ECMAScript 提案的 `import()` 语法。第二种，则是使用 webpack 特定的 `require.ensure`。让我们先看看第一种：

> import() 调用使用会在内部用到 promises。如果在旧有版本浏览器中使用 import()，记得使用一个 polyfill 库（例如 es6-promise 或 promise-polyfill），来 shim Promise。

```javascript
Vue.component(
  'async-demo',
  // 该 import 函数返回一个 Promise 对象。
  () => import('./async-demo')
)
```

上面的例子中，前文提到的工厂函数支持返回一个 Promise 实例，所以可以使用 `import()` 这种代码分离方式。

局部注册也是类似的：

```javascript
Vue.component('parent-demo', {
  // 父组件的其他选项
  components: {
    'async-demo': () => import('./async-demo')
  }
})
```

本质上，`import()` 函数返回一个 Promise 实例，你可以自定义这个过程，下文会有说明。

第二种 webpack 代码分离是这样的：

```javascript
Vue.component('async-demo', function(resolve) {
  require.ensure([], function(require) {
    resolve(require('./async-demo'))
  })
})
```

看起来比较繁琐，如果你使用 webpack 2 及以上版本，则不建议使用第二种方式。

## 生命周期
