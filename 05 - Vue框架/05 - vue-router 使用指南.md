# vue-router

[toc]

## 如何安装

**方式一**

通过`JavaScript` 或者 `CDN`引入

```js
https://unpkg.com/vue-router/dist/vue-router.js
```

**方式二**

模块化安装：

```js
npm install vue-router
```

> 注意：如果在模块化工程中使用vue，必须使用Vue.use()来明确的安装路由功能

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```



## vue-router是什么

vue-router 是 vue 官方的路由管理器，主要用于单页面之间的路由跳转。

**vue-router主要包含的功能有哪些？**

- 可嵌套的路由/视图
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 vue.js过渡系统的页面过渡效果
- 细粒度的导航控制
- 激活状态下的css class 的配置
- 支持hash模式 和 history 模式
- 自定义的滚动条行为

以上功能都可在运行实例中查看具体的实现效果。



## 核心概念

### 路由实例

通过 new 一个路由对象从而得到路由实例。

当我们在项目中已经注入路由之后就可以通过`this.$router`的方式访问到路由实例

通过 `this.$route`访问当前路由。

```js
const router = new VueRouter({
    ...
})
```



### 路由标签

**router-link 组件 和 router-view 组件**

router-link 组件 和 router-view 组件是路由系统的基础标签，主要用以路由跳转及视图展示。



### 动态路由匹配

**什么是动态路由匹配**

我们通常把不同模式匹配到的路由，全部映射到同一个组件上。即：不同的路由地址匹配相同的组件。

**如何实现动态匹配**

动态匹配路由：由 `：`开始标记一个动态路由匹配。

当成功匹配到一个路由时，动态值会作为参数被配置到 `this.$route.params`中，可以在任何组件中获取到该值。

例如：我们要实现一个 从 `user/Tom` 跳转到 `user/Join`的路由跳转，但是两个路由匹配的组件都是 `UserInfo`组件。

```js
const router = new VueRouter({
	routes:[{
		path:"/user/:id",
		component:UserInfo
	}]
})
// 在组件中去使用该参数值
const UserInfo = {
    template:`<div>{{ this.$route.params.id }}</div>`
}
```

**多段参数路由匹配**

如果路由有多个参数，需要进行多段路由匹配，在动态路由配置中也是完全支持的。例如：`user/:id/age/:num`可以匹配到 `user/12/age/18` ， 此时路由下的参数为：

```js
this.$route.params = {
	id:'12',
	num:'18'
}
```

**使用路由参数要注意些什么？**

在使用路由参数跳转时，虽然发生了路由跳转，但是因为映射到了同一个组件，所以组件会被复用，导致声明周期的钩子不再执行，如果在声明周期中有数据刷新或其他操作都将不在执行。

**如何解决组件复用问题 - 监听路由参数变化**

- 通过 watch 监听路由变化

  ```js
  const UserInfo = {
  	watch:{
  		$route(to , from){
  			// 路由发生了变化，此时进行一些操作
  		}
  	}
  }
  ```

- 通过路由守卫 beforeRouterUpdate 来监听路由变化

  ```js
  const UserInfo = {
  	beforeRouteUpdate(to , from , next){
  		// 执行路由变化进行的操作
          // 在手动执行守卫时，一定要执行next()函数。具体原因会在后面说道
  	}
  }
  ```

**路由匹配 - 通配符***

如果我们想要匹配到任意路由，或以指定前缀开头的任意路由，就可以使用到通配符*

```js
// 匹配到所有路由
{
    path:*
}
    
// 如果想要匹配到 以 user-作为前缀的所有路由
{
    path:'user-*'
}
```

>  注意：在使用通配符时，要注意使用顺序，一般我们使用通配符是为了匹配404界面，所以此时通配符一定是放在最后一项的，当所有的理由匹配都没有结果时，匹配404界面；



**动态路由匹配的优先级**

如果我们在route中使用同一个路径匹配了多个路由，则谁先定义的谁的优先级最高。



### 嵌套路由

**什么是嵌套路由**

一个被渲染的视图组件 `router-view` 的模板里可以包含自己嵌套的视图组件 `router-view`，即嵌套路由。

例如在User组件中嵌套视图：

app 中对应的视图是顶层出口，在User中的视图属于嵌套出口。

```vue
<div id="app">
  <router-view></router-view>
</div>

const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}
```

**如何配置嵌套路由**

在定义路由的时候，使用 `children` 属性来配置嵌套路由。

```js
const router = new VueRouter({
  routes: [
    { path: '/user/:id',
      component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

> 注意：所以使用 `/`开头的嵌套都会被映射到根路径上。所以在子路径上只使用路径名即可。

**在嵌套路由中设置默认组件**

以上的配置中，如果输入 `user/foo`，在嵌套视图中是不会显示任何组件的，如果想在嵌套路由中设置一个默认组件，可以添加以下配置：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id', 
      component: User,
      children: [
        // 当 /user/:id 匹配成功，
        // UserHome 会被渲染在 User 的 <router-view> 中
        { path: '', component: UserHome },
        // ...其他子路由
      ]
    }
  ]
})
```



### 编程式导航

**什么是编程式导航**

我们除了使用 `router-link` 组件进行导航外，还可以调用 router 的实例方法进行导航，这种通过调用方法的导航形式被称为编程式导航。而使用 `router-link`的方式叫声明式导航。

**常用的实例方法**

- push（）路由跳转
- replace（）路由替换
- go（）路由回退

**push方法**

- 作用： 进行路由跳转，导航到不同的url。

- 特性：通过push跳转的路由，会向history中添加一条记录，当用户点击浏览器的回退按钮时会回退到之前的url中。而我们在使用 `router-link`时，在内部调用了push方法，所以调用push等同于点击了 `router-link`

- 参数：接受一个字符串或者一个对象

  ```js
  // 字符串
  router.push('home')
  
  // 对象
  router.push({ path: 'home' })
  
  // 命名的路由
  router.push({ name: 'user', params: { userId: '123' }})
  
  // 带查询参数，变成 /register?plan=private
  router.push({ path: 'register', query: { plan: 'private' }})
  ```

  > 使用对象参数，一定要注意：如果定义了path属性，则params属性会被忽略，如果想要使用params属性，前面的值只能是name
  >
  > ```js
  > const userId = '123'
  > router.push({ name: 'user', params: { userId }}) // -> /user/123
  > router.push({ path: `/user/${userId}` }) // -> /user/123
  > // 这里的 params 不生效
  > router.push({ path: '/user', params: { userId }}) // -> /user
  > ```

**replace方法**

- 作用：和 push 方法类似，用于路径跳转
- 特性：不会向 history中添加新的记录，而是替换掉当前的history记录
- 参数：和 push 方法一致。
- 声明式：`<router-link :to='...' replace>`

**go方法**

- 作用：路由回退。在history中回退指定的步数

- 参数：接受一个整数

  ```js
  // 在浏览器记录中前进一步，等同于 history.forward()
  router.go(1)
  
  // 后退一步记录，等同于 history.back()
  router.go(-1)
  
  // 前进 3 步记录
  router.go(3)
  
  // 如果 history 记录不够用，那就默默地失败呗
  router.go(-100)
  router.go(100)
  ```



### 命名路由

**什么是命名路由**

即在定义路由的时候，给路由设置一个name属性用于标志当前路由。尤其在具有路由参数的时候。

**如何定义个命名路由**

```js
const router = new VueRouter({
    routes:[{
        path:"user/:id",
        name:"user",
        component:User
    }]
})
```

**如何链接到一个命名路由**

- 使用 `router-link 组件声明式导航`

  ```vue
  <router-link :to='{name:"user",params:{id:123}}'>user</router-link>
  ```

- 使用 push方法 编程式导航

  ```js
  this.$router.push({
  	name:"user",
  	params:{
  		id:"123"
  	}
  })
  ```



### 命名视图

**什么是命名视图**

即 给 路由视图 `router-view` 添加name属性。没有name属性的视图默认为 default

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

**一般什么时候会用到命名视图呢**

当我们想要在一个组件中同时同级的展示多个 路由视图，此时就需要使用命名视图来控制每个路由视图匹配的组件

**使用命名视图时的路由配置**

```js
const router = new VueRouter({
    routes:[{
        path:"user",
        // 注意：如果是多个视图使用的是 components ， 如果是一个的话使用的是component
        components:{
            default:Foo,
            a:Bar,
            b:Baz
        }
    }]
})
```



### 重定向和别名

**重定向**

- 通过 `routes` 配置重定向

  ```js
  const router = new VueRouter({
  	routes:[{
  		path:"/a",
  		redirect:"/b"
  	}]
  })
  ```

- 通过 `routes` 重定向具名路由

  ```js
  const router = new VueRouter({
  	routes:[{
  		path:"/a",
  		redirect:{
  			name:"/b"
  		}
  	}]
  })
  ```

- 动态设置重定向的目标

  ```js
  const router = new VueRouter({
  	routes:[{
  		path:"/a",
  		redirect:to => {
  			// 接收目标路由作为参数
  			// 返回重定向的目标或者路径对象
  		}
  	}]
  })
  ```

**别名**

别名的含义是指给当前路由添加一个其他名称。当访问这个名称时，和访问path的效果是一样的。

例如：当访问/b时，url会保持/b ,但是路由匹配规则会匹配/a。一般用于想要简化路由或者嵌套路由时。

```js
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

### 路由组件传参

**为什么会使用路由组件传参**

一般我们获取路由中的参数会使用 `$route.params.id` 获取到我们需要的参数。但是`$route`于目前的路由行程高度耦合，使得组件只能在特定的路由下使用。

**如何使用路由组件传参**

使用 props 属性，将路由组件解耦。

- 首先，在组件中使用 props 属性接受路由参数
- 然后，在定义路由对象的时候设置当前路由的props属性为true

```js
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },

    // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```



## 使用进阶

### 导航守卫

**什么是导航守卫**

导航守卫类似于导航的一个生命周期，主要从进入导航、导航跳转、取消导航等方面进行导航守卫

**导航守卫的种类**

- 全局前置守卫
- 全局解析守卫
- 全局后置守卫
- 路由独享守卫
- 组件内守卫

**全局前置守卫**

- 注册使用 - beforeEach：

  ```js
  const router = new VueRouter({...})
  router.beforeEach( (to , from , next) => {...} )
  ```

- 参数说明

  ```js
  to:Router: // 表示将要跳转的路由对象
  from:Router:// 表示跳转来源的路由对象
  next：Function：// 一定要调用改方法来解析当前钩子。执行效果依赖于next的参数
  	next();不传参数表示直接进入管道中的下一个钩子，如果所有钩子都已经执行结束，则导航的状态就是confirm的状态。
      next( false ):传入false表示中断路由跳转，如果url的导航改变了，则会重置到 from 对应的地址。
      next( '/' )： 进入到next指定的路由对象中，支持传入一个路由对象，允许 replace、name等等路由灯箱支持的属性.
  ```

  > 重点说明：next 函数一定要确保在任何一个导航守卫中被严格执行一次，否则钩子不会被解析或者解析报错。



**全局解析守卫**

- 注册使用 - beforeResolve 
- 调用时间：在导航被确认之前，且同时在所有组件内的守卫和异步路由组件被解析之后执行



**全局后置守卫**

- 注册使用 - afterEach
- 调用事件：路由导航解析之后
- 用途：一般用于在路由解析成功后统一执行某些函数



**路由独享守卫**

- 注册使用 - beforeEnter

- 用途：针对单独某个路由的前置守卫，使用方法和全局前置守卫一致，只是 类似于作用域不同

- 使用实例：

  ```js
  const router = new VueRouter({
    routes: [
      {
        path: '/foo',
        component: Foo,
        beforeEnter: (to, from, next) => {
          // ...
        }
      }
    ]
  })
  ```



**组件内守卫**

直接在组件内部定义的路由守卫，被称之为组件内守卫。

- beforeRouteEnter
- beforeRouteUpdate
- beforeRouterLeave

```js
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
    // 一般用于在用户离开前进行进行一些提示  
  }
}
```

在 beforeRouteEnter 中我们无法获取到 this 实例，但是又想使用实例怎么办？在 beforeRouteEnter 钩子中支持给next 函数传递 实例作为参数，然后在next的回调函数中进行一些操作,

```js
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
  })
}
```

**导航解析流程**

![image-20200611153558298](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200611153558298.png)



### 路由元信息

**什么是路由元信息**

`meta`属性，我们在定义路由的时候可以使用`mate`字段，`meta`字段接受任意值

**如何访问到meta字段**

- 路由记录：在routes 配置中的每一个路由对象都被称之为路由记录

- $router.matched 数组：一个路由匹配到的所有路由记录都会暴露给matched数组

- 访问meta字段：

  ```js
  // 如果只能匹配到一个路由：to.meta.requiresAuth;
  if (to.matched.some(record => record.meta.requiresAuth)) {
      // this route requires auth, check if logged in
      // if not, redirect to login page.
      if (!auth.loggedIn()) {
        next({
          path: '/login',
          query: { redirect: to.fullPath }
        })
      } else {
        next()
      }
    } else {
      next() // 确保一定要调用 next()
    }
  ```

  

### 过渡效果

**如何使用过渡效果**

```vue
<transition>
  <router-view></router-view>
</transition>
```

**单个路由的过渡效果**

- 单个路由的过渡效果：即针对每一个组件都有自己对对应的不同的过渡效果

- 实现方式：在路由对应的组件上添加过渡效果，并设置不同的nama值

  ```vue
  <transition name='slider'>
  	<div>
          foo
      </div>
  </transition>
  ```

**基于路由的动态过渡效果**

- 基于路由的动态过渡：针对路由的跳转不同，设置的路由效果不同

- 实现方式：在组件内监听路由，根据 from 和 to 的路径进行效果判断

  ```vue
  <!-- 使用动态的 transition name -->
  <transition :name="transitionName">
    <router-view></router-view>
  </transition>
  <!-- 监听路由变化 -->
  watch: {
    '$route' (to, from) {
      const toDepth = to.path.split('/').length
      const fromDepth = from.path.split('/').length
      this.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
    }
  }
  ```

  

## 从 API 的角度认识vue-router

### router-link支持的属性 - 9种

**★01 - to**

- 接受参数的类型：字符串 | 目标地址的对象

- 必要性：必须

- 作用：用于目标跳转，当被点击后，内部会立刻把 to 的值传递给 router.push()去执行

- 使用实例（官网API）：

  ```vue
  <!-- 字符串 -->
  <router-link to="home">Home</router-link>
  <!-- 不写 v-bind 也可以，就像绑定别的属性一样 -->
  <router-link :to="'home'">Home</router-link>
  <!-- 同上 -->
  <router-link :to="{ path: 'home' }">Home</router-link>
  <!-- 命名的路由 -->
  <router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
  <!-- 带查询参数，下面的结果为 /register?plan=private -->
  <router-link :to="{ path: 'register', query: { plan: 'private' }}"
    >Register</router-link
  >
  ```

**★02 - replace**

- 参数：boolean

- 必要性：非必要

- 作用：路由替换，类似于to，但是使用replace时， to的值被传到的不是push()而是执行replace，导航不会留下hsitory记录。

- 使用实例

  ```vue
  <router-link :to="{ path: '/abc'}" replace></router-link>
  ```

  

**★03 - append**

- 参数：boolean

- 必要性：非必须

- 作用：为当前跳转路由添加跟路径。如果当前在 /a下，to=“/b”。不使用append时，跳转的是/b，使用append 后，跳转的是 /a/b;

- 使用实例：

  ```vue
  <router-link :to="{ path: 'relative/path'}" append></router-link>
  ```

  

**04 - tag**

- 参数：字符串

- 必要性：非必须

- 作用：替换导航的渲染标签。默认情况下，导航渲染为a标签，但是可以使用tag指定渲染的标签类型，且导航点击事件不变。

- 使用实例：

  ```vue
  <router-link to="/foo" tag="li">foo</router-link>
  <!-- 渲染结果 -->
  <li>foo</li>
  ```

  

**05 - active-class**

- 参数：字符串
- 作用：设置路由导航被激活时对应的class类名
- 默认值：`router-link-active`
- 特别说明：可以使用router的 `linkActiveClass`属性来进行全局配置。



**★06 - exact**

- 参数：boolean

- 作用：在链接上使用精准匹配模式。“是否激活”默认类名的依据是包含匹配

- 使用实例：

  ```html
  <!-- 这个链接只会在地址为 / 的时候被激活 -->
  <router-link to="/" exact></router-link>
  ```



**07 - event**

- 参数：字符串 | 字符串数组
- 作用：用来定义触发导航的事件
- 默认值：`click`



**08 - exact-active-class**

- 参数：字符串

- 作用：用来设置当前路由被精准匹配的时候设置的样式

- 默认值：`router-link-exact-actve`

  

**09 - aria-current-value**

- 类型：'page' | 'step' | 'location' | 'date' | 'time'
- 默认值：’page‘



### router的构建选项

**01 - routes**

```ts
interface RouteConfig = {
  path: string,
  component?: Component,
  name?: string, // 命名路由
  components?: { [name: string]: Component }, // 命名视图组件
  redirect?: string | Location | Function,
  props?: boolean | Object | Function,
  alias?: string | Array<string>,
  children?: Array<RouteConfig>, // 嵌套路由
  beforeEnter?: (to: Route, from: Route, next: Function) => void,
  meta?: any,
  // 2.6.0+
  caseSensitive?: boolean, // 匹配规则是否大小写敏感？(默认值：false)
  pathToRegexpOptions?: Object // 编译正则的选项
}
```

**02 - mode**

- 类型：字符串
- 默认值：hash
- 可选值：hash | history | abstract



**03 - base**

- 类型：字符串
- 默认值：’/‘
- 作用：设置基础路径



**04 - linkActiveClass**

- 类型：字符串
- 默认值：`router-link-active`
- 作用：全局设置 `router-link` 默认的激活的class



**05 - linkExactActiveClass**

- 类型：字符串
- 默认值：`router-link-exact-active`
- 作用：全局设置 `router-link` 默认的精准激活的 class



**06 - scrollBehavior**

- 类型：Function
- 作用：设置滚动行为



**07 - parseQuery/ stringifyQuery**

- 类型：Function

- 作用：提供自定义查询字符串的解析/反解析函数

  

**08 - fallback**

- 类型：Boolean
- 默认值：true









