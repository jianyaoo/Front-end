## Vue代码风格指南整理
![](http://10.10.2.24:8181/uploads/202001/share/attach_15e69154addb0650.png)

------------

### A级：必要的

> 使用以下代码规则会规避一些不必要的错误

#### 1、组件名为多个单词
- **好处**
避免和现有的以及未来的HTML产生冲突，因为html元素名称为单元素

- **举例**
```javascript
// 反例1，以todo为组件名
Vue.component("todo" , {
    ...
})
// 反例2，以todo为组件名
export default{
    name:"Todo"
}
```
```javascript
// 好例子1，以'-'连接
Vue.component("todo-item" , {
    ...
})
// 好例子2，单词首字母大写
export default{
    name:"TodoItem"
}
```

- **在项目中的应用 - 以天眼为例**
![](http://10.10.2.24:8181/uploads/202001/share/attach_15e686c48df90156.png)


#### 2、组件的data必须是一个函数

- **目的**
实现每个组件实例都能够管理自己的数据，以便于组件的重用。

- **举例**
```javascript
// 反例1:以对象作为组件的data
Vue.component('some-comp', {
  data: {
    foo: 'bar'
  }
})
// 反例2:以对象作为组件的data
export default {
  data: {
    foo: 'bar'
  }
}
```
```javascript
// 好例子：以函数作为组件的data
Vue.component('some-comp', {
  data: function () {
    return {
      foo: 'bar'
    }
  }
})
export default{
    data() {
        return {
          foo: 'bar'
        }
      }
}
```

- **项目中的应用**
项目中均使用函数返回

#### 3、prop的定义应该尽可能的详细

- **好处**
① 写明了组件的API，增加组件的易用性
② 在开发环境下，如果向一个组件提供格式不正确的 prop，Vue 将会告警，以帮助你捕获潜在的错误来源。

- **举例**
```javascript
// 反例一：只写了属性名的代码
props: ['status']；
```
```javascript
// 好例子：在定义prop中，至少应该指出该prop值得类型
props: {
  status: String
}
// 进阶，更详细的说明
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```

- **在项目中的使用 - 以天眼为例**
![](http://10.10.2.24:8181/uploads/202001/share/attach_15e686d46caed549.png)

#### 4、总是使用key配合v-for

> 更高效的复用dom节点

- **作用**
① 对比组件自身新旧Dom进行更新，跟踪节点身份
② 辅助判断新旧dom节点在逻辑上是否是同一对象
https://www.jianshu.com/p/a634eb3c19c2


 - **注意**
尽量始终添加 **唯一** 的键值

- **示例**
```javascript
// 反例：直接使用循环
<ul>
      <li v-for="todo in todos">
        {{ todo.text }}
      </li>
</ul>
```
```javascript
// 使用唯一的键值
<ul>
      <li
        v-for="todo in todos"
        :key="todo.id"
      >
        {{ todo.text }}
      </li>
</ul>
```

- **项目中的使用 - 以天眼为例**
![](http://10.10.2.24:8181/uploads/202001/share/attach_15e686d6e939dfdf.png)

> 疑问：使用index是否会产生影响-对于值更新后的index值时相同的

#### 5、避免 v-if 和 v-for 用在一起
 - **目的**
 `v-for`比`v-if`具有更高的优先级，会先执行一遍`v-for`


- **我们一般会在什么情况下采用写在一起的方式**
  - 为了过滤一个列表中的项目
  - 为了避免渲染本应该被隐藏的列表

- **代码实例**
```javascript
// 反例 - 直接将v-for 和 v-if直接使用在一起
<ul>
  <li
    v-for="user in users"
    v-if="shouldShowUsers"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```
```javascript
// 好例子
<ul v-if="shouldShowUsers">
  <li
    v-for="user in users"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```

- **项目中的应用**
在实际开发过程中一般不会讲两者写在一起

#### 6、为组件样式设置作用域
- **好处**
确保样式只会运用在它们想要作用的作用域上

------------

### B级：超级推荐

> 能够极大的改善工程的可读性及开发体验，主要分为命名规范和代码格式规范两个部分

#### 1、单文件组件文件的大小写

- **重点**
一个工程中的单文件组件的文件名应始终保持一致

- **可选方式**
 - 单词大写字符开头（MyGroup）
 - 横线连接的方式（my-group）


- **项目相关 - 以天眼为例**
![](http://10.10.2.24:8181/uploads/202001/share/attach_15e686d9b3e36303.png)


#### 2、基础组件的命名方式

- **什么是基础组件**
主要用于展示类的、无状态的、无逻辑的组件

- **命名方式**
采用特定的前缀开头，例如 `App`、`Base`、`V`等

- **在项目中哪些属于基础组件**
类似按钮、表格、卡片、表单、输入框等等


#### 3、单例组件的命名方式

- **什么是单例组件**
在页面中只使用一次，且不接受任何prop，为当前项目工程定制的组件。例如项目中的`menu`

- **命名方式**
采用特定的The前缀开头


#### 4、紧密耦合的组件命名方式
- **紧密耦合的方式**
如果一个组件只在某个父组件的场景下有意义，这层关系应该体现在其名字上。

- **命名方式**
采用父级组件相关的单词作为前缀

#### 5、组件名中的单词顺序
- **命名方式**
一般以高级别的单词作为开头，描述性的单词作为结尾

- **举例**
```js
// 好例子：名词在前，形容词动词在后
components/
|- SearchButtonClear.vue
|- SearchButtonRun.vue
|- SearchInputQuery.vue
|- SearchInputExcludeGlob.vue
|- SettingsCheckboxTerms.vue
|- SettingsCheckboxLaunchOnStartup.vue
```

#### 7、组件命名使用单词全拼
一般组件的命名更倾向于使用完整的单词而并不是缩写，以便于代码的维护。

#### 8、prop的命名方式
- **官方命名方式**
在声明prop时，使用camelCase的方式
在模板中使用时，使用camel-case的方式

#### 9、标签含有多个特性的书写方式
当一个元素含有多个属性时，应该分行写，每个属性单独一行

------------


### D级：应避免的

#### 1、避免在使用v-if/v-else中不使用key值
如果在一组 `v-if/v-else` 中，两者的元素相同，为了避免将不同的元素识别为相同的元素而发生意外的情况，最好添加不同的key值使用

#### 2、避免在scoped样式中，使用元素选择器
在 `scoped`中，使用元素样式 + 自定义选择器样式，会降低界面的性能问题

#### 3、尽量减少使用隐形的父子组件通信
官方推荐，尽量使用prop和父子组件之间的通信