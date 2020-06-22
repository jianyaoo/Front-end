## Vue 错误集

[toc]

### 路由跳转相关

#### 子路由不显示默认视图

**问题重现**

```js
let router = new VueRouter({
	routes:[
		{
			  path: '/parent',
              component: Parent,
              name:"parent",
              children:[
              	{ path:'' , component:Foo },
              	{ path:"Foo" , component:Foo }
              ]
		}
	]
})
```

```vue
// 路由跳转
<router-link :to="{name:'parent'}"></router-link>
```

此时跳转到 `parent` 路由后，默认子视图Foo不显示，刷新界面后显示

**原因**

使用 父级的name属性进行跳转，不会触发父级默认子视图，从父级路由移除name值或将name值设置为默认子路由即可。

![img](https://img-blog.csdnimg.cn/20190604215538471.png)

**解决方法**

```js
let router = new VueRouter({
	routes:[
		{
			  path: '/parent',
              component: Parent,
              children:[
              	{ path:'' , component:Foo },
              	{ path:"Foo" , component:Foo }
              ]
		}
	]
})
```

```vue
// 路由跳转
<router-link :to="{path:'/parent'}"></router-link>
```

**参考链接**

- https://www.cnblogs.com/fqh123/p/10892807.html 
- https://blog.csdn.net/liuhailong2014/article/details/90814162