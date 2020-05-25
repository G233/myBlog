# [翻译] 理解Nuxt 2.12中的 fetch

> 在研究 Nuxt 源码的过程中发现 fetch 的运行过程竟然和官网上说的完全不同，于是我灵机一动去看了英文官网...  果然 Nuxt 2.12 中更新了 fetch，可以说完全变了个样，更加强大好用了，但是中文官网的文档还是 2.11.X ，并没有更新。看了英语文档之后在官网发现了这篇[博文](https://zh.nuxtjs.org/blog/understanding-how-fetch-works-in-nuxt-2-12)，写得很不错，想翻译过来给学习 Nuxt 的同学一个参考，毕竟现在中文资料基本没有，限于水平，有错误还请指出。

<a name="A8vH3"></a>
# 介绍
Nuxt在最新发布的2.12版本中引入了一个新的方法-- `fetch`。Fetch 提供了一种全新的方式将数据引入Nuxt应用程序中。在这篇文章中，我们将探讨 `fetch` 钩子的不同功能，并尝试了解它的工作原理。
<a name="FOxf7"></a>
# Fetch 钩子函数 & Nuxt 生命周期
Nuxt 的钩子函数 `fetch` 运行在 Vue 的 `create` 钩子函数之后，正如我们所知，所有的 Vue 生命周期钩子函数都是在他们对应的 `this` 上下文中被调用，`fetch` 也不例外。


![生命周期](https://img.liuxiaogu.com/blog-img/2020-5-25-new-fetch-lifecycle-hooks.png)
Fetch 钩子函数是在服务器端组件实例化后调用的，这使得 fetch 函数可以通过 `this` 来引用组件的实例对象。
```javascript
export default {
  fetch() {
    console.log(this);
  }
};
```
让我来看看这对页面组件来说意味着什么。
<a name="QEiYD"></a>
# 对页面组件而言
在 this 上下文的帮助下，fetch 可以直接更改组件的 data ，这意味着我们可以直接设置组件的本地数据，而不需要使用类似 Vuex 这种外部储存来储存数据。<br />
<br />这样一来，Vuex 就变成一个可选项了。当然如果你需要使用的话，还是能够使用 `this.$store` 来访问。
<a name="z0lc5"></a>
# Fetch 钩子函数的可用性
通过 fetch 函数我们可以在任何 Vue 组件中预取异步数据，这意味着之前只能在 page 目录下的 Vue 组件才能使用的限制没有了，所有的 Vue 组件都可以使用 fetch 函数。比如 /layouts 和 /components 目录下的 vue 组件。<br />我们来看看这对 layouts 和 building-block 组件来说意味着什么
<a name="qomnS"></a>
## Layout 组件
通过使用新的 fetch 函数，我们可以直接从 layout 组件中进行 API 调用。这在 v2.12 发布之前是不可能的。<br />**一些示例的用法：**

- 从后端获取配置数据，然后动态的生成页脚和导航栏
- 获取用户相关信息，比如用户资料，购物车商品数量
- 可以在 `layouts/error.vue` 获取站点相关数据
<a name="MXrIy"></a>
## building-block（子、嵌套）组件
因为可以在子组件中使用钩子函数了，所以我们可以把之前全部在页面组件中进行的数据获取任务交由子组件 fetch 函数来进行。<br />这很大程度的降低了路由级（page）组件的任务。<br />我们仍然可以使用 props 来向子组件传递数据，但是如果子组件需要自己的数据抓取逻辑，现在是可行的了！
<a name="T7euT"></a>
# 多个钩子函数的运行顺序
既然每个组件都可以有自己的数据获取逻辑，你可能会问，每个组件的调用顺序是什么？<br />Fetch钩子在服务器端被调用一次（在Nuxt 应用程序的第一次请求时，然后在客户端导航到其他路径时被调用。但由于我们可以为每个组件定义一个fetch钩子，所以fetch钩子是按其层次结构的顺序调用的。（译：page 的钩子函数先于 components 的钩子函数运行）
<a name="CG04z"></a>
# 在服务端禁用 fetch
如果你需要的话，可以很方便的在服务端禁用 fetch 函数。
```javascript
export default {
  fetchOnServer: false
};
```
<a name="BQ2dq"></a>
# 错误处理
新的 fetch 的错误处理机制是组件级别的，来看看是如何实现的。<br />因为我们是异步获取数据，所以新的 `fetch``()`提供了一个对象用来检查请求是否结束以及是否成功。<br />这是这个对象的样子 --`$fetchState`
```javascript
$fetchState = {
  pending: true | false,
  error: null | {},
  timestamp: Integer
};
```
有三个属性：

1. **Peding** - 让你在客户端调用 fetch 时显示一个占位符（是否正在请求）。
1. **Error** - 让你可以显示错误信息
1. **Timestamp **- 显示最后一次调用 fetch 的时间戳，这对通过 keep-alive使用缓存的组件来说很有用


<br />可以直接在组件的模版中使用这些值，用来显示数据获取过程中的 Loding。
```html
<template>
  <div>
    <p v-if="$fetchState.pending">Fetching posts...</p>
    <p v-else-if="$fetchState.error">Error while fetching posts</p>
    <ul v-else>
      …
    </ul>
  </div>
</template>
```
当在组件级别发生错误时，我们可以通过在fetch钩子中判断 `process.server` 来在服务器端设置HTTP状态代码，并使用 `throw new Error()`语句进行后续处理。
```javascript
async fetch() {
  const post = await fetch(`https://jsonplaceholder.typicode.com/posts/${this.$route.params.id}`)
                     .then((res) => res.json())

  if (post.id === this.$route.params.id) {
      this.post = post
    } else {
      // 在服务端设置状态码
      if (process.server) {
        this.$nuxt.context.res.statusCode = 404
      }
      // 然后抛出错误
      throw new Error('Post not found')
    }
}
```
这样设置HTTP状态码对正确的SEO是有帮助的。
<a name="fN0vZ"></a>
# Fetch 也可以通过 methods 来调用
新的fetch钩子也作为一个方法，可以在用户交互时被调用，也可以在 methods 中调用。
```html
<!-- from template in template  -->
<button @click="$fetch">Refresh Data</button>
```
```javascript
// from component methods in script section
export default {
  methods: {
    refresh() {
      this.$fetch();
    }
  }
};
```
<a name="T3rqJ"></a>
# 增强 Nuxt 页面的性能
我们可以利用 `:keep-alive-props` prop 和 activated  钩子函数来使用 fetch 使得页面性能更强<br />Nuxt允许在内存中缓存一定数量的页面及其获取的数据。并且还可以设置这些缓存的过期时间。<br />如果想使用上面提到的方法，我们需要在 <nuxt> 和 <nuxt-child> 组件中添加 keep-alive prop 。
```html
<!-- layouts/default.vue -->
<template>
  <div>
    <nuxt keep-alive />
  </div>
</template>
```
此外我们可以通过添加 :keep-alive-props 来控制缓存的页面数量。
```html
<nuxt keep-alive :keep-alive-props="{ max: 10 }" />
```
上面是一种比较高级和通用的提高页面性能的方法，我们还可以通过使用 $fetchState.timestamp 来优化 fetch 请求的调用，主要是通过对比上次请求的时间戳，可以实现更加自由的缓存策略
```javascript
export default {
  activated() {
    // 如果上次请求超过一分钟了，就再次发起请求
    if (this.$fetchState.timestamp <= Date.now() - 60000) {
      this.$fetch();
    }
  }
};
```
<a name="wFgIP"></a>
# asyncData vs Fetch
对于页面组件而言，新的 fetch 似乎和 asyncData 太相似了。因为他们都负责本地数据的处理，但是仍然有一些关键的区别值得注意，如下：
<a name="5yZ3b"></a>
## AsyncData

1. 只有在页面组件中（路由级别组件）可以使用 asyncData 方法
1. asyncData 是在组件实例化之前被调用的，所以无法访问 this
1. 通过 retrun 向组件中添加数据 
```javascript
export default {
  async asyncData(context) {
    const data = await context.$axios.$get(
      `https://jsonplaceholder.typicode.com/todos`
    );
    // `todos` 无需在 data 中进行声明
    return { todos: data.Item };
    // `todos` 会与本地 data 进行合并
  }
};
```
<a name="KDjll"></a>
## New Fetch

1. fetch 在所有 vue 组件中都可以使用
1. fetch 是在组件实例化之后被调用的，可以访问到 this
1. 可以非常简单的直接修改本地数据
```javascript
export default {
  data() {
    return {
      todos: []
    };
  },
  async fetch() {
    const { data } = await axios.get(
      `https://jsonplaceholder.typicode.com/todos`
    );
    // `todos` has to be declared in data()
    this.todos = data;
  }
};
```
<a name="CTsPp"></a>
# Nuxt 2.12 之前的 Fetch
如果你使用 nuxt 有一阵了，你会发现新的 fetch 和之前版本中是完全不同的。
<a name="00q1X"></a>
## 这是一个破坏性更新吗（是否会导致之前的 fetch 不可用）
实际上并不是，我们仍然可以向 fetch 中传入 context 参数来使用之前版本中的 fetch，也就是说不会对你的应用造成任何破坏性的更改。<br />这里有一些新旧版本中 fetch 的一些对比
<a name="d99OL"></a>
## 1.调用 fetch 钩子的时机
**旧：**在组件实例化之前被调用，所以无法访问到 this<br />**新：**当访问路由时在组件实例化之后被调用，可以访问到 this
<a name="Ypk2O"></a>
## 2. this VS context
**旧：**我们可以访问页面级组件上的 Nuxt 上下文，前提是上下文作为第一个参数传递。
```javascript
export default {
  fetch(context) {
    // …
  }
};
```
**新：**我们可以像访问Vue客户端钩子函数一样访问 this 上下文，而用不传递任何参数。
```javascript
export default {
  fetch() {
    console.log(this.data);
  }
};
```
<a name="oUQLh"></a>
## 3. 可用性
**旧：**只能在页面组件（路由级组件）中在服务端被调用。<br />**新：**可以在任何 vue 组件中使用来获取异步数据。
<a name="uzBGK"></a>
## 4. fetch 钩子调用顺序
**旧：**fetch 可以在服务端被调用一次（第一次请求到达 nuxt 服务端时），而在客户端中每当导航到包含 fetch 的组件之前被调用<br />**新：**新 fetch 和旧 fetch 其实是一样的调用，但是由于新 fetch 可以在子组件中设置，所以 fetch 的调用顺序是按照他们的结构层次来进行的。
<a name="9Momw"></a>
## 5. 错误处理
旧：我们使用了 `context.error` 函数，当 API 调用过程中发生错误时，显示一个自定义的错误页面。<br />新：新的 fetch 使用 `$fetchState` 对象来处理 API 调用过程中模板区域的错误。其错误处理是组件级别的。

**这是否意味着在 Nuxt 2.12 中我们不能向用户显示自定义错误页面呢？**<br />**<br />当然不是，我们仍然可以在页面级别的组件中使用 asyncData()，调用  `this.$nuxt.error({ statusCode: 404, message: 'Data not found' }) ` 来展示自定义错误页面。
<a name="nAJRG"></a>
# 结论
新的fetch钩子函数带来了很多改进，并以全新的方式提供了更多的灵活性来获取数据和组织路由级和构建块组件!<br />当你在规划和设计新的Nuxt项目且其需要在一个路由中调用多个 API的时候，它肯定会让你的思维方式有一些改变。<br />我希望这篇文章能帮助你熟悉新的 fetch 功能。我很期待你能够用它来干点什么。
