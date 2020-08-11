### 关于vue登录鉴权

#### 原实现方式

使用store保存数据

```js
 data() {
     return {
         activeIndex2: sessionStorage.getItem("setActionIndex") || '',
         bgColor: window.location.pathname.indexOf("report") == -1 ? '#267CE1' : '#000',
         login: localStorage.getItem("loginToken") ? false : true,
         userName:(localStorage.getItem('loginToken') && 			localStorage.getItem("userName")) ? localStorage.getItem("userName") : '登录'
     }
 },
```

当  `token` 过期后，提示token过期并点击跳转到登录界面。

```js
this.$message.error("token失效，请重新登录")
this.$router.replace({
    path: 'login',
    // query: {redirect: this.$router.history.base},  //重定向地址需测试
})
```

**存在的问题**

当用户登录后，在token不过期的前提下关闭浏览器，浏览器中依旧保存着token值，导致再次打开浏览器时，直接提示用户token过去，请再次登录。

**想要达到的效果**

如果用户打开浏览器时，之前token已经过期，则清空掉storage，默认不登录用户，使用非鉴权接口请求数据，减少用户必须登录的操作。

****

#### 更新方式

**实现思路**

- 使用 `vuex` 配合 `storage`，监听 `storage` 的变化，当本地存储的用户信息失效后，实时更新界面显示效果
- 在初始进入系统时，请求接口判断当前`token`是否过期
- 如果过期则删除`storage`，`commit`状态修改
- 没有过期默认使用已登录账号访问系统

**具体实现**

- `vuex` 配合 `storage` ： 修改store文件，添加用户登录相关的状态

  ```js
  state: {
          ...
          hasLogin:!!localStorage.getItem("loginToken"),
          userName:(localStorage.getItem('loginToken') && localStorage.getItem("userName")) ? localStorage.getItem("userName") : '登录',
      },
  ```

- 在 menu 组件中使用计算属性接受vuex相关状态

  ```js
  computed:mapState([
      'bgColor',
      'defaultMenuActiveIndex',
      'hasLogin',
      'userName'
  ]),
  ```

- 在 menu 组件中简化commit事件

  ```js
  methouds:{
    ...mapMutations([
                  'changeBackgroundColor',
                  'changeActiveIndex',
                  'changeLoginStatus',
                  'changeUserName'
              ]),
  }
  ```

- 在进入系统时判断token是否过期。（因系统存在一个获取信息的接口，所以使用该接口判断，可添加一个接口单独用来判断）

  ```js
  axios.post(api.api.badgeNumber)
      .then(response => {
      	// 请求成功，表示token有效，继续执行该接口正常操作
      	// 默认以当前账号展示
  	})
      .catch(error => {	
      	// 系统过期，直接移除token
          if (error?.data?.message){
              let isTokenExpired = error.data.message.includes("有效验证信息");
              if (isTokenExpired){
                  this.$store.commit('changeLoginStatus' , {
                      hasLogin:false
                  })
                  this.$store.commit('changeUserName',{
                      userName:'登录'
                  })
                  localStorage.removeItem("loginToken");
                  localStorage.removeItem("loginRefreshToken");
                  localStorage.setItem("userName","登录");
              }
      	}
     })
  ```

****

#### 效果

用户打开系统后，如果未登录或token已过期，都直接是未登录状态，以未鉴权接口访问系统，不强制用户必须登录。

#### 优化建议

- 是否可以在路由中或单独某个文件中执行接口请求？
- 在路由守卫中执行请求是否会影响界面性能问题？































