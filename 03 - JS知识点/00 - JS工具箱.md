## JS工具箱

[toc]

### 功能函数

#### 判断浏览器是否允许flash

```js
function hasUsableFlash(){
    let flashObject;
    // 判断浏览器是否支持ActiveXobject对象
    if(typeof  window.ActiveXObject !== 'undefined'){
        // 已允许的话返回一个ActiveXobject对象实例
        flashObject = new ActiveXObject("ShockwaveFlash.ShockwaveFlash");
    }else {
        // 允许返回一个plugins对象，不允许返回undefined
        flashObject = navigator.plugins['Shockwave Flash'];
    }
    // 返回一个Boolean值
    return !!flashObject;
}
```

#### 获取当前url的IP、参数、端口等数据

以页面路径为 `http://192.198.3.66:8080/index?id=141&user=tom`为例

**获取当前url**

```js
// 输出结果：http://192.198.3.66:8080/index?id=141&user=tom"
let url = window.location.href;
```

**获取当前文件名或路径**

```js
// 输出结果：/index
let pathName = window.localtion.pathName;
```

**获取当前url的端口**

```js
// 输出结果：8080
let port = window.localtion.port
```

**获取当前url的协议**

```js
// 输出结果：http:
let protocol = window.localtion.protocol
```

**获取当前url中参数部分**

```js
// 输出结果：?id=14&user=tom
let search = window.localtion.search
```

**获取当前网页的IP**

```js
// 输出结果 192.198.3.66
let damain = document.domain;
```

