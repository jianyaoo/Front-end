# 一篇文章了解WebSocket

[toc]

##  WebSocket 产生背景

在我们开发过程中使用最多的就是 `HTTP`协议，当我们想要获取某些数据时由客户端发起请求，服务端接受请求并返回相对应的数据。

但是这种单项请求方式存在一定的弊端，那就是只能由客户端发起请求，服务端响应，服务端不能够直接给客户端发送数据。而在实际开发中我们往往会遇到这些情况：数据在不断的发生变化，需要实时从服务器请求数据。例如：系统的实时运行日志、实时通信消息、系统的实时状态等等。

如果我们继续使用 `http`协议就需要定时请求，即常说的 ‘ 轮询 ’。轮询的效率较低，而且对于实时性并不是很友好，此时我们便需要使用到 `WebSocket`。

## 什么是WebSoket？

#### 定义

`WebSocket` 是 `html5` 新增的一种协议，是一种网络通信协议。`WebSocket`协议不是一种全新的协议，而是利用 `HTTP`协议来建立的一种协议。

#### `WebSocket`的连接过程

`WebSocket`的连接必须是客户端发起，因为请求协议是一个标准的 `http` 协议，请求过程如下：

```javascript
GET ws://localhost:3000/ws/chat HTTP/1.1
Host: localhost
Upgrade: websocket
Connection: Upgrade
Origin: http://localhost:3000
Sec-WebSocket-Key: client-random-string
Sec-WebSocket-Version: 13
```

`websocket`  的请求与普通的 `http` 的请求存在以下几种区别点：

- GET请求的地址不是类似`/path/`，而是以`ws://`开头的地址；
- 请求头`Upgrade: websocket`和`Connection: Upgrade`表示这个连接将要被转换为`WebSocket`连接；
- `Sec-WebSocket-Key`是用于标识这个连接，并非用于加密数据；
- `Sec-WebSocket-Version`指定了`WebSocket`的协议版本。

当请求成功后，服务器会返回以下响应：

```JavaScript
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: server-random-string
```

服务器返回的响应中，应该注意以下几点：

- 响应代码：101 表示本次连接的http协议即将被更改，更改后的协议就是 `upgrade：websocket` 指定的 `websocket`协议

此时，一个完成的 `websocket` 连接已经建立成功，浏览器和服务器可以随时通信，发送消息。



#### websocket连接的特点

`websocket`连接最大的特点就是 **服务器 可以主动向 客户端发送消息，客户端也可以向服务器端发送消息**，是真正平等的对话。

其他的特点如下：

1. 建立在 `tcp` 协议之上，服务器端的实现比较容易
2. 与 `http`具有良好的兼容性，能够通过各种 `http` 代理服务器
3. 数据格式比较轻，性能开销小
4. 可以发送文本，也可以发送二进制数据
5. 没有同源策略的限制，客户端可以向任意服务器通信
6. 协议标识是 `ws` , 如果加密就是 `wss`

![image-20200423151357603](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423151357603.png)

## 客户端的示例

```javascript
// 建立websocket连接
var ws = new WebSocket("wss://echo.websocket.org");

// websocket建立成功之后的回调函数
ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

// websocket接受到数据后的回调函数
ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

// websocket关闭后的回调函数
ws.onclose = function(evt) {
  console.log("Connection closed.");
};   
```



## 从 API 的角度理解websocket

#### 4-1 构造函数

`**WebSocket()**`构造函器会返回一个 [`WebSocket`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket) 对象

**语法**

```javascript
var ws = new WebSocket(url [, protocols]);
```

**示例**

```javascript
var ws = new WebSocket('ws://localhost:8080');
```

**执行结果**

执行构造函数后，客户端会返回一个websocket对象的实例，并且客户端和服务器端建立连接



#### 4-2 属性

##### （1）binaryType

**作用**

`WebSocket.binaryType` 返回websocket连接所传输二进制数据的类型。

**语法**

```JavaScript
var binaryType = aWebSocket.binaryType;
```

**返回值**

1） blob：传输的类型为 `Blob`

2）`arraybuffer`：传输类型为 `arraybuffer`

##### （2）bufferedAmount【只读】

**作用**

是一个只读属性，用于返回当前`websocket`对象已经执行`send`方法后，还未发送到服务器的字节数。如果已经全部发送给服务器，则该值会被重置为0 ， 如果连接断开后还一直执行`send`方法，该值便会一直增加

**语法**

```javascript
var bufferedAmount = ws.bufferedAmount;
```

**示例**

```javascript
var data = new ArrayBuffer(10000000);
ws.send(data);

if (ws.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```

##### （3）extensions【只读】

**作用**

返回服务器已选择的扩展值，目前链接支持的扩展值只有空字符串或者一个协议列表

**语法**

```JavaScript
var extensions = ws.extensions;
```

##### （4）protocol【只读】

**作用**

返回服务器端选中的子协议的名称，这是在创建`websocket`连接时，在参数中指定的字符串

**语法**

```javascript
var protocol = ws.protocol;
```

##### （5）readState【只读】

**作用**

返回当前连接的链接状态

**语法**

```JavaScript
var readyState = ws.readyState;
```

**返回值**

- CONNECTING：值为0，表示正在连接。
- OPEN：值为1，表示连接成功，可以通信了。
- CLOSING：值为2，表示连接正在关闭。
- CLOSED：值为3，表示连接已经关闭，或者打开连接失败。

##### （6）onopen

**作用**

创建连接成功后的回调函数

**语法**

```
ws.onopen = function(event) {
  console.log("WebSocket is open now.");
};
```

**说明**

如果一个实例对象想要创建多个open的回调事件，可使用 `addEventListen`绑定事件

```
ws.addEventListen('open' , funciton(event){
	console.log("创建监听事件");
})
```

##### （7）onclose

**作用**

连接关闭后的回调函数

**语法**

```javascript
ws.onclose = function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
};

ws.addEventListener("close", function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
});
```

##### （8）onmessage

**作用**

连接接收到数据之后的回调函数

**语法**

```
ws.onmessage = function(event) {
  console.debug("WebSocket message received:", event.data);
};
```

##### （9）onerror

**作用**

连接发生错误之后的回调函数

**语法**

```
ws.onerror = function(event) {
  console.error("WebSocket error observed:", event);
};
```

##### （10）url【只读】

**作用**

返回创建连接时url的绝对路径



#### 4-3 方法

##### （1）close（）

**作用**

关闭 `websocket` 连接或连接尝试，如果`websocket`连接已经关闭，该方法没有任何作用

**语法**

```javascript
WebSocket.close();
```

##### （2）send（）

**作用**

send方法将需要通过 `WebSocket` 链接传输至服务器的数据排入队列

**语法**

```
WebSocket.send("Hello server!");
```

**可传输的数据类型**

- 字符串
- ArrayBuffer
- Blob
- [`ArrayBufferView`](https://developer.mozilla.org/zh-CN/docs/Web/API/ArrayBufferView)



### 从 实例 角度理解websocket - 发送消息通信

##### 5-1 准备dom元素

```html
<div id="box">
    <div class="messageWrap">
        <input class="form-control urlInput" id="url" type="text" placeholder='请输入连接地址'>
        <button class="btn btn-primary" id="open" onclick="openWS()">连接</button>
        <button class="btn btn-danger" onclick="closeWS()">关闭</button>
        <textarea placeholder="请输入发送的数据" class="form-control msg" name="" id="message" cols="30" rows="10"></textarea>
        <button id="send" class="form-control btn btn-success" onclick="sendMessage()">发送</button>
    </div>
    <div class="resultWrap">

    </div>
</div>
```

##### 5-2 创建websocket对象

```javascript
let myWebsocket = {
    ws: null ,
    init(url , event){
        // 判断浏览器是否支持ws
        if ('WebSocket' in window){
            this.ws = new WebSocket( url );
            this.listenerEvent( event ) ;
            this.listenClose();
        }else {
            alert("当前浏览器不支持 websocket 通信");
        }
    },
    listenerEvent(event) {
        this.ws.onopen = function (e) {
            event.open(e);
        };
        this.ws.onmessage = function (e) {
            event.message(e);
        };
        this.ws.onclose = function (e) {
            event.close(e);
        };
        this.ws.onerror = function (e) {
            event.error(e);
        }
    },
    listenClose(){
        window.onbeforeunload = function () {
            this.ws.close();
        }
        window.onbeforeload = function () {
            this.ws.close();
        }
    }
};
```

##### 5-3 使用websocket对象

```javascript
// 事件配置
function open( event ){
    let a = `<p class="alert alert-success">已经成功建立连接啦，可以发送数据呦！</p>`;
    $(".resultWrap").append(a);
}
function close( event ){
    let a = `<p class="alert  alert-secondary ">已关闭连接</p>`;
    $(".resultWrap").append(a);
}
function error( event ){
    let a = `<p class="alert  alert-danger ">连接失败</p>`;
    $(".resultWrap").append(a);
}
function message( event ){
    let data = event.data;
    let a = `<b>从服务器接收到的数据：</b>`+data;
    $(".resultWrap").append(a);

}
```

```javascript
// 创建连接
function openWS(){
    myWebsocket.init(
         $("#url").val(),
        { open:open , close:close , error:error , message:message  }
    )
}

// 关闭连接
function closeWS() {
    myWebsocket.ws.close();
}

// 发送数据
function sendMessage(){
    myWebsocket.ws.send(document.querySelector("#message").value)
    let a = `<b style="color: #1e7e34">你向服务端发送：</b>`+ document.querySelector("#message").value;
    $(".resultWrap").append(a);
    $("#message").val("");
}
```

##### 5-4 运行效果

![image-20200424101540832](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424101540832.png)



### 高级版 - websocket 心跳机制



**使用背景**

如果在连接后需要长时间保持连接，但又不是一直都在通信，可以在连接中添加心跳机制，每隔一段时间就从客户端发送一个数据给后台，后台再返回一个数据给客户端保持通信。

##### 6-1 准备元素

同 5-1

##### 6-2 创建含有心跳机制的websocket对象

```javascript
// 心跳机制
let heartCheck = {
    timeout:600000,
    timeoutObj:null,
    serverTimeoutObj:null,
    reset(){
        clearTimeout(this.timeoutObj);
        clearTimeout(this.serverTimeoutObj);
        return this;
    },
    start: function(){
        let self = this;
        this.timeoutObj = setTimeout(function(){
            myWebsocket.ws.send("ping");
            let a = `<b style="color: #1e7e34">检测心跳：ping</b>`;
            $(".resultWrap").append(a);
            self.serverTimeoutObj = setTimeout(function(){
              ·  myWebsocket.ws.close();
            }, self.timeout)
        }, this.timeout)
    }
}
```

```javascript
// 含有心跳的websocket对象
let myWebsocket = {
    ws: null ,
    url:"",
    event: null ,
    init(url , event){
        try{
            if('WebSocket' in window){
                this.ws = new WebSocket( url );
                let a = `<b style="color: #1e7e34">再次连接</b>`;
                $(".resultWrap").append(a);
                this.url = url ;
                this.event = event;
                this.listenerEvent( event ) ;
                this.listenClose();
            }else{
                alert("您的浏览器不支持websocket协议,建议使用新版谷歌、火狐等浏览器，请勿使用IE10以下浏览器，360浏览器请使用极速模式，不要使用兼容模式!");
            }
        }catch(e){
            this.reconnect(url);
            console.log(e);
        }
    },
    listenerEvent(event) {
        let $this = this;
        $this.ws.onopen = function (e) {
            heartCheck.reset();
            heartCheck.start();
            event.open(e);
        };
        $this.ws.onmessage = function (e) {
            heartCheck.reset();
            heartCheck.start();
            event.message(e);
        };
        $this.ws.onclose = function (e) {
            event.close(e);
            $this.reconnect();
        };
        $this.ws.onerror = function (e) {
            $this.reconnect( $this.url );
            event.error(e);
        }
    },
    listenClose(){
        window.onbeforeunload = function () {
            this.ws.close();
        };
        window.onbeforeload = function () {
            this.ws.close();
        }
    },
    reconnect(){
        let $this = this;
        setTimeout(function () {
            $this.init($this.url , $this.event);
        }, 3000);
    },
};
```

##### 6-3 使用

同 5-3

### 参考链接

- 【廖雪峰官方网站】https://www.liaoxuefeng.com/wiki/1022910821149312/1103332447876608
- 【阮一峰教程】http://www.ruanyifeng.com/blog/2017/05/websocket.html
- 【MDN API】https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket
- https://www.cnblogs.com/fuqiang88/p/5956363.html

