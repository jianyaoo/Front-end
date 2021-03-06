## js 部分

[toc]

#### 和渲染无关的数据
vue中data的数据默认便会进行双向数据绑定，若是将大量的和渲染无关的数据直接放置在data中，将会浪费双向数据绑定时所消耗的性能，将这些和渲染无关的数据进行抽离并配合Object.freeze进行处理。

table中columns数据可以单独提取一个外部js文件作为配置文件，也可以在当前.vue文件中定义一个常量定义columns数据，因为无论如何都是固定且不会修改的数据，应该使用Object.freeze进行包裹，既可以提高性能还可以将固定的数据抽离，一些下拉框前端固定的数据也建议此操作

```js
const columnList = Object.freeze([
  { title: '姓名', key: 'name', align: 'center' },
  { title: '性别', key: 'gender', align: 'center' }
])
```

> 需要注意的是 Object.freeze() 冻结的是值，这时仍然可以将变量的引用替换掉，还有确保数据不会变才可以使用这个语法，如果要对数据进行修改和交互，就不适合使用冻结了。



#### `Modal`框的控制

一个页面种通常会存在很多个不同功能的弹框，若是每一个弹框都设置一个对应的变量来控制其显示，则会导致变量数量比较冗余和命名困难，可以使用一个变量来控制同一页面中的所有`Modal`弹框的展示

比如某个页面中存在三个`Modal`弹框

```js
// bad
// 每一个数据控制对应的Modal展示与隐藏
new Vue({
    data() {
        return {
            modal1: false,
            modal2: false,
            modal3: false,
        }
    }
})

// good
// 当modalType为对应的值时 展示其对应的弹框
new Vue({
    data() {
        return {
            modalType: '' // modalType值为 modal1，modal2，modal3
        }
    }
})
```

#### `debounce`使用

例如远程搜索时需要通过接口动态的获取数据，若是每次用户输入都接口请求，是浪费带宽和性能的

当一个按钮多次点击时会导致多次触发事件，可以结合场景是否立即执行`immediate`

```js
<Select :remote-method="remoteMethod">
    <Option v-for="item in temoteList" :value="item.value" :key="item.id">{{item.label}}	</Option>
</Select>
```

```js
import {debounce} from 'lodash'

methods：{
    remoteMethod：debounce(function (query) {
        // to do ...
       // this 的指向没有问题
    }, 200),
}
```

#### 图片

功能的开发过程中，图片的处理往往是比较容易被忽略的环节，也会在一定程度影响开发的效率和页面的性能

- 图片压缩问题，除非特别要求图片必须高质量的显示，否则都应该进行对应的压缩处理

- 不同业务场景进行图片格式的选型

- - JPG 适用于呈现色彩丰富的图片，JPG 图片经常作为大的背景图、轮播图或 Banner 图出现等
  - Logo、颜色简单且对比强烈的图片或背景、需要透明度等
  - 将常用且变动频率很低的小图片进行合并成雪碧图，对于变动比较频繁和小于`6KB`的图片进行`base64`处理
  - 根据项目图片数量和项目的用户机型分布等，考虑采取`webp`进行图片的处理



#### 路由组件传参

> 在组件中使用 `$route` 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性。

使用 `props` 将组件和路由解耦：

**取代与 `$route` 的耦合**

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

**通过 `props` 解耦**

这样你便可以在任何地方使用该组件，使得该组件更易于重用和测试。

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

#### Vue生命周期

在父子组件中，掌握父子组件对应的生命周期钩子加载顺序可以让开发者在更合适的时候做适合的事情**父组件**

```vue
<template>
  <div>
    <h3>home</h3>
    <list @hook:mounted="listMounted" />
  </div>
</template>

<script>
import List from './list'

export default {
  name: "home",
  components: {
    List
  },
  methods: {
    listMounted(){
      console.log('------------ listMounted');
    }
  },
  beforeCreate() {
    console.log("home beforeCreate");
  },
  created() {
    console.log("home created");
  },
  beforeMount() {
    console.log("home beforeMount");
  },
  mounted() {
    console.log("home mounted");
  },
  beforeDestroy() {
    console.log("home beforeDestroy");
  },
  destroyed() {
    console.log("home destroyed");
  }
}
</script>
```

**子组件**

```vue
<template>
  <div>
    list
  </div>
</template>

<script>
export default {
  naem: "list",
  beforeCreate() {
    console.log("list beforeCreate");
  },
  created() {
    console.log("list created");
  },
  beforeMount() {
    console.log("list beforeMount");
  },
  mounted() {
    console.log("list mounted");
  },
  beforeDestroy() {
    console.log("list beforeDestroy");
  },
  destroyed() {
    console.log("list destroyed");
  }
}
</script>
```

**加载父组件时的加载顺序**

```
home beforeCreate --> home created --> home beforeMount --> list created --> list beforeMount --> list mounted
```

实际开发过程中会遇到当子组件某个生命周期完成之后通知父组件，然后在父组件做对应的处理。

**emit**

```vue
// 子组件在对应的钩子中发布事件
created(){
  this.$emit('done')
}

// 父组件订阅其方发
<list @done="childDone">
```

#### `Select`优化

下拉框遍历时，需要注意`options`标签保持同一行，若是存在换行，会导致选中时的值存在多余的空白

```vue
<!-- bad -->
<Select :remote-method="remoteMethod">
    <Option v-for="item in temoteList" :value="item.value" :key="item.id">
        {{item.label}}
    </Option>
</Select>
<!-- good -->
<Select :remote-method="remoteMethod">
    <Option v-for="item in temoteList" :value="item.value" :key="item.id">{{item.label}}</Option>
</Select>
```

#### `data`数据层级

`data`数据具有数据层级结构，切勿过度扁平化或者嵌套层级过深，若是过度扁平化会导致数据命名空间冲突，参数传递和处理，若是层级嵌套过深也会导致`vue`数据劫持的时候递归层级过深，若是嵌套层级丧心病狂那种的，小心递归爆栈的问题。而且层级过深会导致数据操作和处理不便，获取数据做容错处理也比较繁琐。一般层级保持2-3层最好。

若是只有一层数据，过于扁平,导致处理不方便

```vue
{
    name: '',
    age: '',
    gender: ''
}
```

适当的层级结构不仅增加代码的维护和阅读性，还可以增加操作和处理的便捷性

```vue
{
    person: { // 个人信息
        name: '',
        age: '',
        gender: ''
    }
}
```

#### 策略模式

策略模式的使用，避免过多的`if else`判断，也可以替代简单逻辑的`switch`

```js
const formatDemandItemType = (value) => {
    switch (value) {
        case 1:
            return '基础'
        case 2:
            return '高级'
        case 3:
            return 'VIP'
    }
}

// 策略模式
const formatDemandItemType2 = (value) => {
    const obj = {
        1: '基础',
        2: '高级',
        3: 'VIP',
    }
    
    return obj[value]
}
```



#### 解构

解构赋值以及默认值，当解构的数量小于多少时适合直接解构并赋值默认值，数据是否进行相关的聚合处理

```js
const {
  naem = '',
  age = 10,
  gender = 'man'
} = res.data

// bad
this.name = name
this.age = age
this.gender = gender

// good
this.person = {
  naem,
  age,
  gender
}
```

#### 职责单一

任何时候尽量是的一个函数就做一件事情，而不是将各种逻辑全部耦合在一起，提高单个函数的复用性和可读性

每个页面都会在加载完成时进行数据的请求并展示到页面

```vue
created() {
  this.init();
},
methods: {
  // 将全部的请求行为聚合在init函数中
  // 将每个请求单独拆分
  init() {
    this.getList1()
    this.getList2()
  },
  getList1() {
    // to do ...
  },
  getList2() {
    // to do ...
  }
}
```



## html部分

#### html书写

编写`template`模板时，属性过多时，是否换行

```vue
<template>
  <!-- 不换行 -->
  <VueButton class="icon-button go-up" icon-left="keyboard_arrow_up" v-tooltip="$t('org.vue.components.folder-explorer.toolbar.tooltips.parent-folder')" @click="openParentFolder" />

  <!-- 换行 -->
  <VueButton
    class="icon-button go-up"
    icon-left="keyboard_arrow_up"
    v-tooltip="$t('org.vue.components.folder-explorer.toolbar.tooltips.parent-folder')"
    @click="openParentFolder"
  />
</template>
```

#### 实体使用

实体使用一些如`<`，`>`,`&`等字符时，使用字符实体代替

```js
<!-- bad -->
<div>
  > 1 & < 12
</div>
  
<!-- bad -->
<div>
  &gt; 1 &amp; &lt; 12
</div>
```

