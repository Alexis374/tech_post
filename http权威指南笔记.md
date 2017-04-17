##### chapter 1
---
+ http使用可靠的tcp传输，这保证了从服务器到客户端的内容是一样的。
+ 资源有很多类型，可以是静态文件，也可以是动态生成的内容
+ mimetype，服务器给所有返回的http对象附加一个媒体类型。浏览器根据mimetype选择如何处理这个对象。
+ URI 统一资源标识符，分为URL统一资源定位符和URN统一资源名。URN处于试验阶段，基本上在http中URL=URI
+ 一个http事务由请求和响应组成
+ HTTP报文都是纯文本。包括起始行，首部字段headers，主体body。
+ 因http基于tcp，且是纯文本的。很容易与服务器对话。可以使用telnet 主机名 80。建立一条到服务器80端口的tcp连接；
然后输入http请求报文，即可得到服务器返回的响应报文。
+ web组件：
	+ 代理 proxy
	+ 缓存 cache
	+ 网关 gateway：特殊的服务器，通常用于将http流量转换成其他协议，如http/ftp网关
	+ 隧道 tunel：通常在一条或多条http连接上转发非http数据。
	+ agent代理：自动发起请求的半智能web客户端，如浏览器