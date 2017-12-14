# Vue.js 子组件的异步加载及其生命周期控制

前端开发社区的繁荣，造就了很多优秀的基于 MVVM 设计模式的框架，而组件化开发思想也越来越深入人心。这其中不得不提到 Vue.js 这个专注于 VM 层的框架。

本文主要对 Vue.js 组件化开发中子组件的异步加载和其生命周期进行一些探讨。阅读本文需要对 Vue.js 有一定的了解。

> 注意：本文中的一些例子代码，是以 vue-cli 采用 webpack 模板初始化的项目为基础。

## 异步组件

讨论异步加载，需要先了解下异步组件。Vue.js 的异步组件是把组件定义为一个工厂函数，在组件需要渲染时触发工厂函数，并且把结果缓存起来，用于后面的再次渲染。例如注册一个全局异步组件：

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

这里使用 `setTimeout` 只是为了模拟异步，在实际项目中，应该配合 webpack 的代码分离功能来实现异步加载。

## 异步加载

在实际的项目中，如果不使用异步加载，则 Vue.js 组件的 JS、CSS 和模板都会打包到一个 .js 文件中，这个文件可能达到几 MB 甚至更多，严重影响首屏加载时间。所以在项目中我们需要启用组件的异步加载。

### webpack 代码分离

webpack 的代码分离有两种，第一种，也是优先选择的方式是，使用符合 ECMAScript 提案的 `import()` 语法。第二种，则是使用 webpack 特定的 `require.ensure`。让我们先看看第一种：

> import() 调用会在内部用到 promises。如果在旧有版本浏览器中使用 import()，记得使用一个 polyfill 库（例如 es6-promise 或 promise-polyfill），来 shim Promise。

```javascript
Vue.component(
  'async-demo',
  // 该 import 函数返回一个 Promise 对象。
  () => import('./async-demo')
)
```

上面的例子中，前文提到的工厂函数支持返回一个 Promise 对象，所以可以使用 `import()` 这种代码分离方式。

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
  }, function(error) {
    // 加载出错执行这里
  })
})
```

看起来比较繁琐，如果你使用 webpack 2 及以上版本，则**不建议**使用第二种方式。

## 生命周期控制

在使用子组件（或者叫局部注册）时，我们可能需要在子组件实例化（或者叫创建完毕）后做某些事情。在非异步的子组件中，我们很容易做这件事：

```html
<template>
  <div>
    <my-demo ref="demo"></my-demo>
  </div>
</template>

<script>
import Demo from './Demo'

export default {
  mounted() {
    // 在这里可以通过组件的 $refs 获取到子组件的实例
    // 可以认为，在这里子组件实例化完毕
    console.log(this.$refs.demo)
  },
  components: {
    MyDemo: Demo
  }
}
</script>
```

上例中使用了 Vue.js 的子组件引用，所以可以在生命周期函数 `mounted` 中很方便的获取到子组件的实例，这样就可以在这个函数中处理一些子组件实例化后要做的事情。

但是在异步子组件中，`mounted` 函数中是无法获取到子组件的实例的，所以我们需要一些技巧来实现这个功能。

```html
<template>
  <div>
    <my-demo ref="demo"></my-demo>
  </div>
</template>

<script>
export default {
  components: {
    MyDemo: () => import('./Demo').then(component => {
      // 清理已缓存的组件定义
      component.default._Ctor = {}

      if (!component.default.attached) {
        // 保存原组件中的 created 生命周期函数
        component.default.backupCreated = component.default.created
      }

      // 注入一个特殊的 created 生命周期函数
      component.default.created = function() {
        // 子组件已经实例化完毕

        // this 即为子组件 vm 实例
        console.log(this)

        if (component.default.backupCreated) {
          // 执行原组件中的 created 生命周期函数
          component.default.backupCreated.call(this)
        }
      }

      // 表示已经注入过了 
      component.default.attached = true

      return component
    })
  }
}
</script>
```

上例中，可以看到我们对组件异步加载做了一些特殊的控制，其中 `import().then()` 则是在加载完子组件的 .js 文件后，实例化子组件之前的回调，如果需要处理出错的情况，则 `import().then().catch()` 即可。

以上代码只是注入了一个 `created` 函数，如果要注入其他生命周期函数，例如 `mounted`，也是类似的：

```html
<template>
  <div>
    <my-demo ref="demo"></my-demo>
  </div>
</template>

<script>
export default {
  components: {
    MyDemo: () => import('./Demo').then(component => {
      component.default._Ctor = {}

      if (!component.default.attached) {
        component.default.backupMounted = component.default.mounted
      }

      component.default.mounted = function() {
        if (component.default.backupMounted) {
          component.default.backupMounted.call(this)
        }
      }

      component.default.attached = true

      return component
    })
  }
}
</script>
```

通过上面的讨论，我们可以做到完全控制 Vue.js 组件的异步加载的全过程，这对于需要精确控制子组件加载的组件，会有很大的帮助。

## 演示项目

根据上面的思路，写了一个基于 Bootstrap 的异步弹窗演示项目：

https://github.com/hex-ci/vue-async-bootstrap-modal-demo
