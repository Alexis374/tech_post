####Ajax
---

+ create xhr object:

```js
var request;
if(window.XMLHttpRequest){
    request = new XMLHttpRequest();
}
else{
    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}
```

+ open(method,url,async),true为异步，默认为true。
+ send(string)，string为请求体，post的请求体不能为空（为空意义不大）。get请求体为空，string可以不填。
```js
request.open('post',http://baidu.com,true);
request.setRequestHeader("Content-type","application/x-www-form-urlencoded");//表单
request.send("name='fuck'&password='123'");
```
+ 获得响应：`response.responseText`,`responseXML`,`status`,`statusText`,`getResponseHeader()`,`getAllResponseHeader()`
+ readyState，代表服务器响应的状态：0 未初始化，opne未调用；1，服务器连接已经建立，open已经调用；2，请求已接受，接收响应头信息 3接收响应主体 4响应已就绪，响应完成了
+ readyStatechange事件：
```js
request.onreadystatechange = function(){
    if(request.readyState === 4 && request.status===200){
        doSth();
    }
}
```
+ 通过fiddler的conposer模块可以模拟表单提交等动作
+ post提交时open和send函数中间的`setRequestHeader`不可少

+ json 长度短，速度快，可以直接转化为js对象，已经替代了html
+ json在js中解析 eval 和
```js
var jsondata = '{"name":"fuck"}';
var jsonobj = eval(jsondata);
var jsonobj2 = JSON.parse(jsondata);
```
+ eval中若字符串中有alert(1)这样的，会直接执行；建议用JSON.parse
+ 服务端设置`Content-type:application/json
+ jquery 中的ajax：
```js
jQuery.ajax({type:"GET",
    url:"service.php?name=100",
    //data:{};  若方法为POST，此参数为POST数据
    dataType:"json",//预期返回的MIME类型
    success:function(data){
        //成功时调用的函数   status==200时
    }，
    error:function(jqXHR){
        //失败时调用的函数
    }
});
```
+ 同源策略，跨域： 协议，子域名，主域名，端口之中有一个不同时，则为不同域。localhost 和 127.0.0.1也是不同域。js不允许跨域操作/调用其他域的对象。返回status为0不是200
+ 处理跨域
    1. 代理，与js相同域的服务器端代理请求，之后再返回请求的结果
    2. JSONP 处理主流浏览器**GET**请求跨域数据访问问题。利用了`<script>`标签进行跨域。简单来说就是`<script>`可以跨域访问脚本，所以在不同的域上设置与本域上相同的格式，来返回数据
    ```js
    //在 a.com 中:
    function jsonp(){
        alert(json["name"]);
    }
    <script src="http://b.com/jsonp.js">
    //在b.com jsonp.js中
    //jsonp('{"name":"fuck"}');//这里函数名称要与a.com上的一样。当a.com的js加载了b.com/json.js，就相当于调用了jsonp('{"name":"fuck"}')这个函数，实现了数据传递
    ```
    jquery中，给ajax函数参数改为`dataType=jsonp`，并且增加一个 jsonp= "callback"//这个名字是随便取的，但要在b.com中，从querystring中获得这个名称GET['callback']
    3. html5 中XMLHttpRequest Level2标准。ie10一下版本不支持。在服务端加上header("Access-Control-Allow-Oringin':*或指定某个域"),header("Access-Control-Allow-Methods：POST，GET")