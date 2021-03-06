# 从输入URL到页面展示发生了什么

## 总体过程
* 在浏览器中输入url
* DNS域名解析
* 建立TCP连接
* 浏览器发送一个HTTP请求
* 服务器响应请求
* 浏览器解析渲染页面
* 断开连接

## 在浏览器中输入url
URL（Uniform Resource Locator,统一资源定位符）。由一串简单的文本字符组成，他的作用是用于定位互联网上资源的地址。URL通常由以下几部分组成：传输协议、域名、端口、文件路径。

当我们开始在浏览器中输入网址的时候，浏览器其实就已经在智能的匹配可能得 url 了，他会从历史记录，书签等地方，找到已经输入的字符串可能对应的 url，然后给出智能提示，让你可以补全url地址。

## DNS域名解析
使用域名是为了方便人们记忆，实际上对应服务器的是IP地址，当我们在浏览器中输入域名后，需要将域名转换成IP地址，查找顺序：
> 浏览器缓存 -> 操作系统缓存 -> 本地host文件 -> 路由器缓存 -> ISP DNS缓存 -> 顶级DNS服务器/根DNS服务器

在根域名收到请求后，会判别这个域名(.com)是授权给哪台服务器管理，并返回这个顶级DNS服务器的IP。请求者收到这台顶级DNS的服务器IP后，会向该服务器发起查询，如果该服务器无法解析，该服务器就会返回下一级的DNS服务器IP（example.com），本机继续查找，直到服务器找到(www.example.com)的主机。

## 建立TCP连接
拿到IP地址后，浏览器会向服务器发起TCP的连接请求，三次握手以建立TCP连接。
1. 第一次握手: 建立连接。客户端发送连接请求，发送SYN报文，将seq设置为0。然后，客户端进入SYN_SEND状态，等待服务器的确认。SYN seq=0。
2. 第二次握手: 服务器收到客户端的SYN报文段。需要对这个SYN报文段进行确认，发送ACK报文，将ack设置为1。同时，自己还要发送SYN请求信息，将seq为0。服务器端将上述所有信息一并发送给客户端，此时服务器进入SYN_RECV状态。SYN ACK seq=0，ack=1。
3. 第三次握手: 客户端收到服务器的ACK和SYN报文后，进行确认，然后将ack设置为1，seq设置为1，向服务器发送ACK报文段，这个报文段发送完毕以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。ACK ack=1，seq=1。

## 浏览器发送一个HTTP请求
建立了TCP连接之后，发起一个http请求。一个典型的 http request header 一般需要包括请求的方法，例如 GET 或者 POST 等，不常用的还有 PUT 和 DELETE 、HEAD、OPTION以及 TRACE 方法。客户端向服务器发起http请求的时候，会有一些请求信息，请求信息包含三个部分：
* 请求方法URI协议/版本
* 请求头
* 请求正文

### 常用请求头：
1. Accept：浏览器可以接受服务器回发的类型。text/html、*/*
2. Accept-Encoding：列出浏览器支持的压缩方法。gzip,deflate 
3. Accept-Language：列出浏览器期望的页面语言。zh-CN,zh;q=0.9
4. Connection：控制当前事务完成后TCP连接是否保持打开状态。 如果发送的值是keep-alive，则连接是持久的并且不会关闭，从而允许对同一服务器的后续请求得以完成。如果是close代表一个请求完成后，客户端和服务器之间用于传输HTTP数据的TCP连接会关闭， 当客户端再次发送Request，需要重新建立TCP连接。
5. Host：请求报头域主要用于指定被请求资源的Internet主机和端口号，它通常从HTTP URL中提取出来的。
6. Referer：当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器我是从哪个页面链接过来的，服务器籍此可以获得一些信息用于处理。https://translate.google.cn/
7. User-Agent：允许服务器标识人们从何处访问它们，并且可以使用该数据进行分析，记录或优化缓存。Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36

## 服务器响应请求
服务器收到了我们的请求，也处理我们的请求，到这一步，它会把它的处理结果返回，也就是返回一个HTTP响应。 HTTP响应与HTTP请求相似，HTTP响应也由3个部分构成，分别是：
* 状态行
* 响应头(Response Header)
* 响应正文

### 常用响应头：
1. Cache-Control：缓存控制。
* no-cache：必须先与服务器确认返回的响应是否被更改，然后才能使用该响应来满足后续对同一个网址的请求。因此，如果存在合适的验证令牌 (ETag)，no-cache 会发起往返通信来验证缓存的响应，如果资源未被更改，可以避免下载。
* no-store：所有内容都不会被缓存到缓存或 Internet 临时文件中。
* max-age=xxx：缓存的内容将在 xxx 秒后失效。
2. Content-Type：告诉客户端，资源文件的类型，还有字符编码，客户端通过utf-8对资源进行解码，然后对资源进行html解析。text/html;charset=UTF-8
3. Content-Encoding：告诉客户端，服务端发送资源的压缩格式。gzip, deflate
4. Date：服务端发送资源时的服务器时间，GMT是格林尼治所在地的标准时间。Mon, 26 Oct 2020 06:46:24 GMT
5. Server：服务器和相对应的版本。Apache-Coyote/1.1
6. Transfer-Encoding：告诉客户端，服务器发送的资源的方式是分块发送的。一般分块发送的资源都是服务器动态生成的，在发送时还不知道发送资源的大小，所以采用分块发送，每一块都是独立的，独立的块都能标示自己的长度，最后一块是0长度的，当客户端读到这个0长度的块时，就可以确定资源已经传输完了。chunked
7. Expires：告诉客户端在这个时间前，可以直接访问缓存副本，很显然这个值会存在问题，因为客户端和服务器的时间不一定会都是相同的，如果时间不同就会导致问题。Sun, 1 Jan 2000 01:00:00 GMT
8. Last-Modified：所请求的对象的最后修改日期。
* If-Modified-Since：下次再次请求时，会把该信息附带在请求报文中一并带给服务器去做检查，若传递的时间值与服务器上该资源最终修改时间是一致的，则说明该资源没有被修改过，直接返回304状态码即可。
* If-Unmodified-Since：只有当资源在指定的时间之后没有进行过修改的情况下，服务器才会返回请求的资源，或是接受 POST 或其他 non-safe 方法的请求。如果所请求的资源在指定的时间之后发生了修改，那么会返回 412 (Precondition Failed) 错误。
9. Connection：这个字段作为回应客户端的Connection：keep-alive，告诉客户端服务器的tcp连接也是一个长连接，客户端可以继续使用这个tcp连接发送http请求。keep-alive
10. Etag：就是一个对象（比如URL）的标志值，就一个对象而言，比如一个html文件，如果被修改了，其Etag也会别修改，所以，ETag的作用跟Last-Modified的作用差不多，主要供WEB服务器判断一个对象是否改变了。比如前一次请求某个html文件时，获得了其 ETag，当这次又请求这个文件时，浏览器就会把先前获得ETag值发送给WEB服务器，然后WEB服务器会把这个ETag跟该文件的当前ETag进行对比，然后就知道这个文件有没有改变了。737060cd8c284d8af7ad3082f209582d
11. Access-Control-Allow-Origin：*号代表所有网站可以跨域资源共享，www.baidu.com 指定哪些网站可以跨域资源共享
12. Access-Control-Allow-Methods：允许哪些方法来访问。GET,POST,PUT,DELETE
13. Access-Control-Allow-Credentials：是否允许发送cookie，如果access-control-allow-origin为*，当前字段就不能为true。true

### 常用状态码：
* 1xx：信息性状态码，表示服务器已接收了客户端请求，客户端可继续发送请求。
* 2xx：成功状态码，表示服务器已成功接收到请求并进行处理。200请求成功；204请求成功但是不返回主体
* 3xx： 重定向状态码，表示服务器要求客户端重定向。301资源永久移除；302资源临时性移除；304内容没有更新可使用浏览器缓存
* 4xx：客户端错误状态码，表示客户端的请求有非法内容。400语法错误；401请求未授权；403收到请求但是拒绝提供服务，正文提供原因；404资源不存在
* 5xx：服务器错误状态码，表示服务器未能正常处理客户端的请求而出现意外错误。500服务器不可预知错误；503服务器再试不能处理客户端请求，过段时间可能恢复正常

## 浏览器解析渲染页面
### 渲染步骤：
1. 处理 HTML 标记并构建 DOM 树。
2. 处理 CSS 标记并构建 CSSOM 树。
3. 将 DOM 与 CSSOM 合并成一个渲染树。
4. 根据渲染树来布局，以计算每个节点的几何信息。
5. 将各个节点绘制到屏幕上。

其中需要注意的是：
* 当浏览器遇到一个 script 标记时，DOM 构建将暂停，直至脚本完成执行。
* JavaScript 可以查询和修改 DOM 与 CSSOM。
* CSSOM 构建时，JavaScript 执行将暂停，直至 CSSOM 就绪。

所以资源的位置很重要，遵循下面个原则可以让页面渲染时间尽量少：
* CSS被视为阻塞渲染的资源，应放到代码的头部尽快加载。
* 同步的JavaScript会暂停DOMTree的构建，应放到代码的尾部最后加载，或者使用async/defer属性异步加载JavaScript。
* 重排和重绘会给浏览器渲染线程造成很大的负担，尽量减少重排和重绘的触发次数。

### 回流
会导致回流的操作：
* 页面首次渲染
* 浏览器窗口大小发生改变
* 元素尺寸或位置发生改变
* 元素内容变化（文字数量或图片大小等等）
* 元素字体大小变化
* 添加或者删除可见的DOM元素
* 激活CSS伪类（例如：:hover）
* 查询某些属性或调用某些方法

一些常用且会导致回流的属性和方法：
* clientWidth、clientHeight、clientTop、clientLeft
* offsetWidth、offsetHeight、offsetTop、offsetLeft
* scrollWidth、scrollHeight、scrollTop、scrollLeft
* scrollIntoView()、scrollIntoViewIfNeeded()
* getComputedStyle()
* getBoundingClientRect()
* scrollTo()

### 重绘
当页面中元素样式的改变并不影响它在文档流中的位置时（例如：color、background-color、visibility等），浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。

### defer：
defer 属性表示延迟执行引入的 JavaScript，即这段 JavaScript 加载时 HTML 并未停止解析，这两个过程是并行的。整个 document 解析完毕且 defer-script 也加载完成之后（这两件事情的顺序无关），会执行所有由 defer-script 加载的 JavaScript 代码，然后触发 DOMContentLoaded 事件。defer 不会改变 script 中代码的执行顺序，示例代码会按照 1、2、3 的顺序执行。所以，defer 与相比普通 script，有两点区别：载入 JavaScript 文件时不阻塞 HTML 的解析，执行阶段被放到 HTML 标签解析完成之后。

### async
async 属性表示异步执行引入的 JavaScript，与 defer 的区别在于，如果已经加载好，就会开始执行——无论此刻是 HTML 解析阶段还是 DOMContentLoaded 触发之后。需要注意的是，这种方式加载的 JavaScript 依然会阻塞 load 事件。换句话说，async-script 可能在 DOMContentLoaded 触发之前或之后执行，但一定在 load 触发之前执行。从上一段也能推出，多个 async-script 的执行顺序是不确定的。值得注意的是，向 document 动态添加 script 标签时，async 属性默认是 true。

## 断开连接git 
1. 第一次挥手：客户端向服务器发送一个FIN报文段，将设置seq为160和ack为112;此时，客户端进入 FIN_WAIT_1状态,这表示客户端没有数据要发送服务器了，请求关闭连接;
2. 第二次挥手：服务器收到了客户端发送的FIN报文段，向客户端回一个ACK报文段，ack设置为161，seq设置为112;服务器进入了CLOSE_WAIT状态，客户端收到服务器返回的ACK报文后，进入FIN_WAIT_2状态;
3. 第三次挥手：服务器会观察自己是否还有数据没有发送给客户端，如果有，先把数据发送给客户端，再发送FIN报文；如果没有，那么服务器直接发送FIN报文给客户端。请求关闭连接，同时服务器进入LAST_ACK状态;ack设置为161，seq设置为112;
4. 第四次挥手：客户端收到服务器发送的FIN报文段，向服务器发送ACK报文段，将seq设置为161，将ack设置为113，然后客户端进入TIME_WAIT状态;服务器收到客户端的ACK报文段以后，就关闭连接;此时，客户端等待2MSL后依然没有收到回复，则证明Server端已正常关闭，客户端也可以关闭连接了。