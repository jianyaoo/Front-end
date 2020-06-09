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



## 使用进阶

## 从 API 的角度认识vue-router

## 运行实例





