# CORS跨域问题

### 问题

[浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)规定某域下的客户端在没明确授权的情况下，不能读写另一个域的资源。而在实际开发中，前后端常常是相互分离的，并且前后端的项目部署也常常不在一个服务器内或者在一个服务器的不同端口下。前端想要获取后端的数据，就必须发起请求，如果不做一些处理，就会受到浏览器同源策略的约束。后端可以收到请求并返回数据，但是前端无法收到数据。

### 为何浏览器会制定同源策略

之所以有同源策略，其中一个重要原因就是对cookie的保护。cookie 中存着sessionID 。黑客一旦获取了sessionID，并且在有效期内，就可以登录。当我们访问了一个恶意网站 如果没有同源策略 那么这个网站就能通过js 访问document.cookie 得到用户关于的各个网站的sessionID 其中可能有银行网站 等等。通过已经建立好的session连接进行攻击，比如CSRF攻击。
这里需要服务端配合再举个例子，现在我扮演坏人。我通过一个iframe 加载某宝的登录页面，等傻傻的用户登录我的网站的时候，我就把这个页面弹出。用户一看，阿里唉大公司，肯定安全，就屁颠屁颠的输入了密码。注意，如果没有同源策略。我这个恶意网站就能通过dom操作获取到用户输入的值，从而控制该账户。所以同源策略是绝对必要的。
还有需要注意的是同源策略无法完全防御CSRF。

### 多种跨域方法

跨域大致分为两种目的：

- 前后端分离时，前端为获取后端数据而跨域。
- 为不同域下的前端页面通信而跨域。

#### 一、前后端分离跨域

添加一系列请求头和响应头，规范安全地进行跨站数据传输。

请求头

| 请求头                         | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| Origin                         | Origin头在跨域请求或预先请求中，标明发起跨域请求的源域名。   |
| Access-Control-Request-Method  | Access-Control-Request-Method头用于表明跨域请求使用的实际HTTP方法 |
| Access-Control-Request-Headers | Access-Control-Request-Headers用于在预先请求时，告知服务器要发起的跨域请求中会携带的请求头信息 |
| with-credentials               | 跨域请求携带cookie                                           |

响应头

| 响应头                        | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| Access-Control-Allow-Origin   | Access-Control-Allow-Origin头中携带了服务器端验证后的允许的跨域请求域名，可以是一个具体的域名或是一个*（表示任意域名）。 |
| Access-Control-Expose-Headers | Access-Control-Expose-Headers头用于允许返回给跨域请求的响应头列表，在列表中的响应头的内容，才可以被浏览器访问。 |
| Access-Control-Max-Age        | Access-Control-Max-Age用于告知浏览器可以将预先检查请求返回结果缓存的时间，在缓存有效期内，浏览器会使用缓存的预先检查结果判断是否发送跨域请求。 |
| Access-Control-Allow-Methods  | Access-Control-Allow-Methods用于告知浏览器可以在实际发送跨域请求时，可以支持的请求方法，可以是一个具体的方法列表或是一个*（表示任意方法）。 |

客户端只需按规范设置请求头。服务端按规范识别并返回对应响应头，或者安装相应插件，修改相应框架配置文件等。具体视服务端所用的语言和框架而定。

##### SpringBoot解决cors

对单个接口配置`@CrossOrigin`注解，或者配置全局CORS

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
        .allowedOrigins("http://localhost:8081")
        .allowedMethods("*")
        .allowedHeaders("*");
    }
}
```

##### JSONP跨域

jsonp的原理就是借助HTML中的`<script>`标签可以跨域引入资源。所以动态创建一个`<srcipt>`标签，src为目的接口 + get数据包 + 处理数据的函数名。后台收到GET请求后解析并返回`函数名(数据)`给前端，前端`<script>`标签动态执行处理函数。
JSONP既是利用了`<srcipt>`，那么就只能支持GET请求。其他请求无法实现。

##### nginx反向代理

既然浏览器有同源策略限制，那我们把前端项目和前端要请求的api接口地址放在同源下就可以了，再结合web服务器提供的反向代理，便可以在前端和后端都不做配置的情况下解决跨域问题。

#### 二、不同域前端通信

window.name + iframe实现跨域

document.domain + iframe