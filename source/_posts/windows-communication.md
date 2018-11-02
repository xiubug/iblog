---
title: 跨页面通信的几种方法
date: 2018-04-20 19:35:04
tags:
  - communication
categories:
  - javascript
---

### localStorage
通过监听window对象的“onstorage”事件，其他窗口获取到本窗口发送的消息，注意，必须是同一款浏览器，并且在同一个域名下。

发送消息页面：
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Page A</title>
    <script>
        function setTime() {
          localStorage.setItem('currentTime', (new Date()).getTime());
        }
    </script>
</head>
<body>
    <div>
        <button onClick="setTime()">设置时间</button>
    </div>
</body>
</html>
```
接收消息页面：
``` html
<!DOCTYPE html>
<html>
 
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Page B</title>
</head>
<script>
    document.addEventListener('DOMContentLoaded', function() {
        document.getElementById('message').innerText = localStorage.getItem('currentTime') || '暂无数据';
        /*当存储空间中数据发生变化的时候*/
        window.onstorage = function(ev) {
            /*获取正在变化的数据*/
            document.getElementById('message').innerText = localStorage.getItem(ev.key) || '暂无数据';
        };
    }, false);
</script>
 
<body>
  <div id="message"></div>
</body>
 
</html>
```
注意：
**onload：**当 onload 事件触发时，页面上所有的DOM，样式表，脚本，图片，flash都已经加载完成了。
**DOMContentLoaded：**当 DOMContentLoaded 事件触发时，仅当DOM加载完成，不包括样式表，图片，flash。
开发中我们经常需要给一些元素的事件绑定处理函数，将绑定的函数放在这两个事件的回调中，保证能在页面的某些元素加载完毕之后再绑定事件的函数。相比之下 DOMContentLoaded 机制更加合理，因为我们可以容忍图片，flash延迟加载，却不可以容忍看见内容后页面不可交互。

### Window.postMessage()

#### 定义
window.postMessage 是 HTML5 引入的API，postMessage() 方法允许来自不同源的脚本采用异步方式进行有效的通信，可以实现跨文本文档，多窗口，跨域消息传递。多用于窗口间数据通信，这也使它成为跨域通信的一种有效的解决方案。

#### postMessage发送消息
在需要发送消息的源窗口调用 postMessage 方法即可发送消息。

##### 源窗口
源窗口可以是全局的window对象，也可以是以下类型的窗口:
**文档窗口中的iframe：**
``` js
var iframe = document.getElementById('my-iframe');
var win = iframe.documentWindow;
```
**JavaScript打开的弹窗：**
``` js
var win = window.open();
```
**当前文档窗口的父窗口：**
``` js
var win = window.parent;
```
**打开当前文档的窗口：**
``` js
var win = window.opener();
```
找到源window对象后，即可调用postMessage API向目标窗口发送消息：
``` js
win.postMessage('Hello', 'ttp://jhssdemo.duapp.com/');
```
postMessage函数接收两个参数：第一个为将要发送的消息，第二个为源窗口的源。

注：只有当目标窗口的源与postMessage函数中传入的源参数值匹配时，才能接收到消息。

#### 接收postMessage消息
要想接收到之前源窗口通过postMessage发出的消息，只需要在目标窗口注册message事件并绑定事件监听函数，就可以在函数参数中获取消息。
```js
window.onload = function() {
    var text = document.getElementById('txt');  
    function receiveMsg(e) {
        console.log("Got a message!");
        console.log("nMessage: " + e.data);
        console.log("nOrigin: " + e.origin);
        // console.log("Source: " + e.source);
        text.innerHTML = "Got a message!<br>" +
            "Message: " + e.data +
            "<br>Origin: " + e.origin;
    }
    if (window.addEventListener) {
        //为窗口注册message事件，并绑定监听函数
        window.addEventListener('message', receiveMsg, false);
    }else {
        window.attachEvent('message', receiveMsg);
    }
};
```
message事件监听函数接收一个参数，Event对象实例，该对象有三个属性：
**data；**发送的具体消息。
**origin：**发送消息源。
**source：**发送消息窗口的window对象引用。

#### 说明
**1、**可以将postMessage函数第二个参数设为*，在发送跨域消息时会跳过对发送消息的源的检查。
**2、**postMessage只能发送字符串信息。

#### 实例
**源窗口：**
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Html5 postMessage</title>
    <style>
        #otherWin {
            width: 600px;
            height: 400px;
            background-color: #cccccc;
        }
    </style>
</head>
<body>
    <button id="btn">open</button>
    <button id="send">send</button>
      <!-- 通过 iframe 嵌入子页面(接收消息目标窗口) --> 
      <iframe id="otherWin" src="http://localhost:8080/b.html"></iframe> 
      <br/>
      <br/> 
      <input type="text" id="message" />
      <input type="button" value="Send to child.com" id="sendMessage" /> 
    <script>
        window.onload = function() {
            var btn = document.getElementById('btn');
            var btn_send = document.getElementById('send');
            var sendBtn = document.getElementById('sendMessage');
            var win;
            btn.onclick = function() {
                //通过window.open打开接收消息目标窗口
                win = window.open('http://localhost:8080/b.html', 'popUp');
            }
            btn_send.onclick = function() { 
                // 通过 postMessage 向子窗口发送数据      
                win.postMessage('Hello', 'http://localhost:8080/');
            }
            function sendIt(e){ 
                // 通过 postMessage 向子窗口发送数据
                document.getElementById("otherWin").contentWindow 
                .postMessage( 
                    document.getElementById("message").value, 
                    "http://localhost:8080/"); 
            } 
            sendBtn.onclick = function(e) {
                sendIt(e);
            };
        };
    </script>
</body>
</html>
```
**目标窗口：**
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Html5 postMessage</title>
    <style>
        #txt {
            width: 500px;
            height: 300px;
            background-color: #cccccc;
        }
    </style>
</head>
<body>
    <h1>The New Window</h1>
    <div id="txt"></div>
    <script>        
        window.onload = function() {
            var text = document.getElementById('txt');  
            //监听函数，接收一个参数--Event事件对象
            function receiveMsg(e) {
                console.log("Got a message!");
                console.log("nMessage: " + e.data);
                console.log("nOrigin: " + e.origin);
                text.innerHTML = "Got a message!<br>" +
                    "Message: " + e.data +
                    "<br>Origin: " + e.origin;
            }
            if (window.addEventListener) {
                //为window注册message事件并绑定监听函数
                window.addEventListener('message', receiveMsg, false);
            }else {
                window.attachEvent('message', receiveMsg);
            }
        };
    </script>
</body>
</html>
```

### WebSocket
所有的WebSocket都监听同一个服务器地址，利用send发送消息，利用onmessage获取消息的变化，不仅能窗口，还能跨浏览器，兼容性最佳，只是需要消耗点服务器资源。
``` js
var ws = new WebSocket("ws://www.example.com/socketserver");
ws.onopen = function (event) {
  // 或者把此方法注册到其他事件中，即可与其他服务器通信
  ws.send({username : 'yiifaa', now : Date.now()}); // 通过服务器中转消息
};
ws.onmessage = function (event) {
  // 消费消息
  console.log(event.data);
}
```
