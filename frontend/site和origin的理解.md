# site和origin的理解

翻译自：https://jub0bs.com/posts/2021-01-29-great-samesite-confusion/

本文将剖析一个普遍的关于cookie属性`sameSite`的误解，已经分享它在web安全中一些潜在的影响。

### samesite的到来

你毫无疑问听说过cookie的属性`sameSite`。在2020年2月，Chrome开始推出对`sameSite`默认行为的改变，此时，它也成为了头条新闻。几个月后FireFox也[随之效仿](https://hacks.mozilla.org/2020/08/changes-to-samesite-cookie-behavior/)。

从2016年`samesite`被创建以来，它一直潜伏在浏览器的中心，作为一个纵深防御机制来低于跨站攻击，比如跨站点请求伪造（CSRF）和跨站点脚本包含（XSSI）。

在2020年初，samesite的激活需要一些网站进行繁琐的适配以保持第三方访问权限，但却被广泛地认为是浏览器防御中的一种受欢迎的补充。基于其在浏览器上所达到的效果，很多博客中的帖子都在传播这种“新”的Cookie属性机制。

### 术语

一些帖子的描述非常准确，但不幸的是，关于samesite的帖子中，很少有人致力于澄清site的概念。当然，同站点请求和跨站点请求的概念都是从当中衍生出来的。此外，许多帖子，包括infosec社区有影响力的成员写的帖子，似乎都在互换使用site和origin这两个术语。Domain、host、origin、site...在信息交流中不严谨地表述这些术语都是常事。

### site和origin是否可互换？

“site”在samesite中有非常技术性的含义，但却经常被忽视。site和origin的区别也非常重要，尽管这两个概念经常被混淆。在chrome激活samesite后几个月，Google的开发者觉得有必要写一篇[文章](https://web.dev/same-site-same-origin/)来区别开origin和site。

### origin的含义

如果你使用web技术，至少应该熟悉[同源策略](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)（SOP） ，它可以说是web安全的主要支柱之一。URI中origin的概念是SOP的核心，因此它是个相对容易理解的概念。[RFC6454的3.2章](https://tools.ietf.org/html/rfc6454#section-3.2)将origin定义为三元组：

> Roughly speaking, two URIs are part of the same origin (i.e., represent the same principal) if they have the same **scheme**, **host**, and **port**.

port端口是可选的；如果没有特殊指定，scheme协议关联的端口是默认的（如http为80，https为443）。[MDN](https://developer.mozilla.org/en-US/docs/Glossary/Origin)上有一些示例。

### site的含义

site有比origin更难理解的含义。首先，它比SOP早，在跨站脚本等攻击出现时就已经被广泛使用了。此外，现在站点的概念充满了技术难点。它与主机的可注册域紧密联系在一起，[URL标准](https://url.spec.whatwg.org/#host-registrable-domain)将其定义为：

> a domain formed by the most specific public suffix, along with the domain label immediately preceeding it, if any.

一个主机的可注册域也被称之为`eTLD+1`，即“effective top-level domain plus one”。

在最简单的例子中，源的站点是与源的主机可注册域相对应的。两个例子：

- `https://www.example.org`的站点是`example.org`，因为`org`是主机最具体的公共后缀，所以`example.org`是主机的eTLD+1。
- `https://jub0bs.github.io`的站点是`jub0bs.github.io`，因为`github.io`是主机最具体的公共后缀，所以`jub0bs.github.io`是主机的eTLD+1。

是的，令人惊讶的是，`github.io`是一个公共后缀。值得注意的是，可注册域是一个不稳定的概念，因为它依赖于[公共后缀列表](https://publicsuffix.org/list/)，这个列表是随时间的推移而[变化](https://github.com/publicsuffix/list/blob/master/public_suffix_list.dat)的。更不用说，不同的浏览器不一定能够以相同的速度更上列表的更新。

### same-site和cross-site请求

了解了站点的概念，现在可以来讨论同站请求和跨站请求。一个请求要么是同站的，要么是跨站的，这取决于请求来源站点和目标站点的比较：

- 如果两个站点是完全相同的，那请求就是同站点的
- 如果两个站点是不同的，那请求就是跨站点的

这里有三个例子：

- 一个请求从`https://foo.exmaple.org`发送至`https://bar.example.org`是同站点的，因为他们的站点都是`example.org`。
- 一个请求从`https://foo.github.io`发送至`https://bar.github.io`是跨站的，因为来源站点是`foo.github.io`，目标站点是`bar.github.io`。
- 一个请求从`https://foo.bar.example.org`发送至`https://bar.example.org`是同站的，因为他们的站点都是`example.org`。

### cross-origin和same-site请求

正如上面第一个和第三个例子所示，所有的跨站请求都是跨源点请求，但不是所有的跨源点请求都是跨站请求。

cookie参数`SameSite`只关注跨站点请求，它对同站点的跨源请求不起作用。这就是为什么区分origin和site这么重要的原因。