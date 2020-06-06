## 初始vuex

[toc]

### 第一步：安装vuex

- 直接下载或cdn连接

  ```html
  <script src="/path/to/vue.js"></script>
  <script src="/path/to/vuex.js"></script>
  ```

- 通过脚手架直接安装

  ```sh
  vue create projectName
  ```

- 创建项目后通过 `npm` 安装

  ```shell
  npm install vuex --save
  ```

- 创建项目后使用 `yarn` 安装

  ```sh
  yarn add vuex
  ```

- 通过vuex项目自行构建

  ```sh
  git clone https://github.com/vuejs/vuex.git node_modules/vuex
  cd node_modules/vuex
  npm install
  npm run build
  ```



### 第二步：了解vuex是什么

> vuex 是 专门为了vue应用程序开发的状态管理模式
>
> 本质：集中式的存储管理应用到的所有组件的状态，并让相应的状态以可预测的形式发生变化

**设计思想：**

把项目中全部的组件共享状态抽取出来，进行一个单例模式管理。

项目中的所有组件树构成了一个巨大的视图，无论组件在树的哪个位置，都能够获取到共享状态的值并触发相对应的事件。

**和定义一个全局变量的区别：**

1. vuex中存储的状态是响应式的，任何一个组件修改了状态值，在其他组件中都会相应的得到高效更新。
2. vuex中储存的状态值是不能够直接修改的。只能够通过提交mutation进行修改，即不能在组件中随便定义一个函数对状态值进行修改。所有能够使状态值发生变化的事件都是被定义在store中的。

### 第三步：开始使用vuex

#### 1、计数器

**计数器实例**

```js
import Vue from 'vue'
import Vuex from 'vuex'

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

**获取当前状态的值**

```js
store.state.count; 
```

**触发事件变更**

```js
store.commit("increment");
```

#### 2、在vue中注入vuex

通过在根实例中注册 `store` 选项，该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 `this.$store` 访问到。

**注入vuex**

```
Vue.use(Vuex);
new vue({
	el:"#app",
	store,
})
```

**在子组件中获取vuex状态值**

```js
this.$store.state.count
```

### 第四步：了解vuex的核心概念

#### State

**state是什么**

state 是vuex中唯一的一个状态树，即单一树，包含了项目中全部的应用层级的状态。

state 在vuex中作为唯一的状态源而存在

**在子组件中如何获取状态值**

假设你已经通过以上方式将vuex注入到了vue中

```js
this.$store.state.count
```

**在子组件中应该如何展示状态值**

一般情况下，如果我们在子组件中使用状态值，最简单的方式是通过计算属性返回需要的某个状态，这样如果状态发生改变，计算属性也会相应的发生改变。

**学会使用mapState()辅助函数**

在开发过程中，如果一个子组件需要使用到多个共享状态值，为了减少计算属性的声明，可以使用mapState辅助函数进行声明。

```
import { mapState } from 'vuex'
```

- 如果需要给计算属性重新命名，则mapState接受一个对象

  ```javascript
  const Counter = {
  	template:`<div>{{count}}</div>`,
  	computed:mapState({
  		myCount:'count'
  	})
  }
  ```

- 如果计算属性名和状态值的名称一致，则mapState可以接受一个数组

  ```javascript
  const Counter = {
  	template:`<div>{{count}}</div>`,
  	computed:mapState([count])
  }
  ```

- 如果除了状态值外，该子组件还有其他的计算属性，则推荐使用对象展开运算符

  ```javascript
  const Counter = {
  	template:`<div>{{count}}</div>`,
  	computed:{
  		childData(){ return '计算属性' },
  		...mapState({
  			//...
  		})
  	}
  }
  ```



#### Getter

**getter 的使用背景**

在多个子组件中，一个计算属性值依赖相同的多个状态，或需要对状态进行相同的处理。类似于以下场景：

```javascript
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

为此，vuex支持getter属性，也可以理解为store中的计算属性。当getter的依赖值发生改变时，getter就会发生改变。

**如何在store中声明一个计算属性？**

Getter 接受 state 作为自己的第一个参数

```js
const store = new Vuex.Store({
  state: {
   // ... 状态值
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

如果在Getter中会使用到其他的getter，Getter也支持接受 getter 本身作为自己的第二个参数

```javascript
const store = new Vuex.Store({
  state: {
   // ... 状态值
  },
  getters: {
    doneTodoCount:state => {
         return state.todos.filter(todo => todo.done).length
    }
    doneTodos: ( state , getters ) => {
      return getters.doneTodoCount
    }
  }
})
```

**如何在组件中使用声明好的Getter？**

通过属性访问：getter 在通过属性访问时是作为 Vue 的响应式系统的一部分缓存其中的

```
computed:{
	doneTodoCount(){
		return this.$store.getters.doneTodoCount;
	}
}
```

通过方法调用：该方式的前提是，Getter返回的为一个方法，使用方法调用结果不会作为缓存，每次调用都会重新执行该方法。

```javascript
const store = new Vuex.Store({
	getters:{
		getTodoId:( state ) => (id) => {
			return state.todos.find( todo => todo.id === id )
		}
	}
})
```

```
this.$store.getters.getTodoId(2)
```

**同样被支持的 mapGetters函数**

同 mapState 函数一致，在同一个组件中如果需要获取多个 Getter ，推荐使用 mapGetters 辅助函数，用法同 mapState

#### Mutation

**什么是Mutation**

Mutation 是 vuex 更新状态值唯一可行的地方。即在子组件中通过提交Mutation从而达到修改state的目的。

在store中的Mutation 更像是组件中的方法，它含有一个字符串的事件类型和一个回调函数。回调函数就是我们修改状态值的地方。

**如何声明一个Mutation？**

声明方式 和 Getter 一致，并且同样接受state作为第一个参数

```javascript
const store = {
	state:{
		//...
	},
	getters:{
		//...
	},
	mutations:{
		increment(state){
			state.account++;
		}
	}
}
```

**如何在子组件中提交一个Mutation**

```javascript
this.$store.commit('increment')
```

**如何提交一个带有参数的Mutation**

提交一个带有参数的 Mutation，在vuex中被称之为提交载荷。Mutation接受第二个参数作为额外的参数，一般来说，载荷（即第二个参数）为一个对象，方便传递更多的数据，也方便代码的阅读及维护。

```javascript
const store = {
	state:{
		//...
	},
	getters:{
		//...
	},
	mutations:{
		increment(state ， payLoad){
			state.accout += payLoad.n;
		}
	}
}
```

```javascript
this.$store.commit('increment',{
	n:10
})
```

**使用Mutation时一定要注意的地方**

1. 需要遵守vue的响应规则

   > (1) 建议在state中初始化好所有的默认值
   >
   > (2) 当需要在对象上更新属性时，要遵守vue更新对象的原则

2. Mutation必须是同步函数

**同样被支持的 mapMutations辅助函数**

同 Getter 一样，需要获取多个Mutation时，建议使用 mapMutations辅助函数，支持传入数组和对象

#### Action

**Action的产生背景**

Action的作用类似于Mutation ， 但是又不同于Mutation。因为在Mutation中只能执行同步操作，且是直接修改了state状态值。

- Action同样可以修改状态值，但是是通过提交Mutation从而达到修改状态值的目的。
- Mutation只能执行同步操作，而在Action中可以执行任意异步的操作。

**如何注册Action**

Action 接受一个和store实例具有相同方法和属性的context对象参数，可以直接调用commit方法

```javascript
const store = {
	state:{
		//...
	},
    getters:{
        //...
    },
    mutations:{
        addCount(state){
            state.count++
        }
    },
    actions:{
        increment(context){
            context.commit('addCount');
        }
    }
}
```

**在组件中触发Action**

```javascript
const Count = {
	methods:{
		addCount(){
			this.$store.dispatch('increment');
		}
	}
}
```

**同样支持载荷的Action**

Action支持接受额外的数据作为第二个参数，即载荷数据

```
 actions:{
        increment(context , payLoad){
            context.commit('addCount' , payLoad);
        }
    }
```

```
const Count = {
	methods:{
		addCount(){
			this.$store.dispatch('increment'.{
				count:10
			});
		}
	}
}
```

**一般什么条件下会使用Action**

当我们需要一些异步操作及提交多个Mutation时，我们可能会需要是要Action，尤其是异步操作。

以下为官方购物车案例：

```js
actions: {
  checkout ({ commit, state }, products) {
    // 把当前购物车的物品备份起来
    const savedCartItems = [...state.cart.added]
    // 发出结账请求，然后乐观地清空购物车
    commit(types.CHECKOUT_REQUEST)
    // 购物 API 接受一个成功回调和一个失败回调
    shop.buyProducts(
      products,
      // 成功操作
      () => commit(types.CHECKOUT_SUCCESS),
      // 失败操作
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```

**同同同样被支持的mapActions辅助函数**

```js
import { mapActions } from 'vuex'

export default {
  methods: {
    ...mapActions([
      'increment', 
      'incrementBy' 
    ]),
    ...mapActions({
      add: 'increment' 
    })
  }
}
```

#### Module

































