## Vue事件总线 - EventBus 使用详解

[toc]

### 事件总线产生的背景

在vue中最核心的部分就是组件，组件间的通信就是重中之重。通信分为父子组件通信、兄弟组件通信、不相干的组件通信等等多种方式。父级向子级通过`props`传递数据，子级通过 `$emit` 向父组件通知事件。但是当两个毫不相关的组件互相通信时应该通过什么来实现呢？

### EventBus的简介

EventBus 又称时间总线 ，理解上来讲 EventBus 机制是通知的概念，EventBus作为所有组件共享的事件中心，既可以发送事件也可以接受事件，所有组件都可以平行的接到到相对应的数据。

### 开始

#### 第一步：初始化

在项目中使用事件总线时，需要初始化一条总线，初始化的方式有两种

- 通过 .js 文件将总线作为模块化导入

```js
// EventBus.js
import Vue from 'vue'
export defalut new Vue()
```

- 通过 prototype 将总线注册为全局总线，直接在main.js中初始化以下代码

```js
vue.prototype.$bus = new Vue();
```

#### 第二步：发送事件

- 在组件中引入总线文件 ，使用 `EventBus.$emit(‘事件名称’，数据)`进行发送数据

例如：在A组件中向兄弟组件B发送一条消息：

```js
import EventBus from '../assets/EventBus'
export default {
    methods:{
        sendMessageToB(){
            EventBus.$emit("sendMessage" , "这是从A组件中传递到兄弟组件的内容")
        }
    }
}
```

#### 第三步：接受事件

- 在组件中引入总线文件 ，使用 `EventBus.$on(‘emit事件名’，callback(payload1...))`进行接受事件
- 在一个组件中某事件如果只监听一次的话，可以使用 `EventBus.$once('事件名称',callBack(payload1...))`;

例如：从A组件接收到的数据并处理。

```js
import EventBus from '../assets/EventBus'
export default {
    name: "ComponentB.vue",
    data(){
        return{
            message:""
        }
    },
    mounted(){
        let _this = this;
        EventBus.$on("sendMessage" , (msg) => {
             _this.message = msg
        })
    },
}
```

#### 第四步：移除事件

- `EventBus.$off('事件名', callback)`，只移除这个回调的监听器。
- `EventBus.$off('事件名')`，移除该事件所有的监听器。
- `EventBus.$off()`， 移除所有的事件监听器，注意不需要添加任何参数。

```js
import EventBus from '../assets/EventBus'
EventBus.$off('sendMessage', {}) // 移除指定事件
EventBus.$off('sendMessage') // 移除应用内所有对此事件的监听
EventBus.$off() // 移除应用内所有事件的监听
```

### 全局EventBus

#### 第一步：注册，在main.js中

```js
Vue.prototype.$bus = new Vue();
```

#### 第二步：在组件中使用

```js
this.$bus.$on("sendMessage" , (msg) => {
    _this.message = msg
})
this.$bus.$emit("sendMessage" , "这是从D组件中传递到兄弟组件的内容")
```

**关于调研EventBus的优缺点**

**缺点**

- vue作为单页面应用，如果在某一个页面刷新了之后，与之相关的EventBus会被移除。
- 如果业务有反复操作的页面，EventBus在监听的时候就会触发很多次，也是一个非常大的隐患。这时候我们就需要好好处理EventBus在项目中的关系。通常会用到，在vue页面销毁时，同时移除EventBus事件监听。
- 由于是都使用一个Vue实例，所以容易出现重复触发的情景，两个页面都定义了同一个事件名，并且没有用$off销毁（常出现在路由切换时）。

**优点**

- 解决了多层组件之间繁琐的事件传播。
- 使用原理十分简单，代码量少。

### 参考链接

- [参考链接1](https://blog.csdn.net/i168wintop/article/details/95107935?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)
- [参考链接2](https://segmentfault.com/a/1190000021707081)