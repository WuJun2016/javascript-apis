# HTTP

> Hello, HTTP.

## HTTP 的发展历程

`HTTP/0.9` 标准于 1990 年问世，因为当是的 HTTP 没有作为正式的标准被确立，因此该版本含有 HTTP/1.0 之前版本的意味。

`HTTP/1.0` 标准于 1996 年 5 月作为第一份标准被公布，它被记载于 [RFC1945 - Hypertext Transfer Protocol -- HTTP/1.0](http://www.ietf.org/rfc/rfc1945.txt)

`HTTP/1.1` 标准于 1999 年 6 月被公布，截止到目前它应该是最主流的 HTTP 协议版本，它被记载于 [RFC2616 - Hypertext Transfer Protocol -- HTTP/1.1](http://www.ietf.org/rfc/rfc2616.txt)

`HTTP/2` 标准于 2015 年 5 月被正式发布，它被记载于 [RFC7540 - Hypertext Transfer Protocol -- HTTP/2](http://www.ietf.org/rfc/rfc7540.txt)，它的特点是 ① 采用二进制而非明文来打包，② 多路复用，③ 修复队头堵塞，④ 允许设置设定请求优先级，⑤ 服务器推送，⑥ WebSocket 等等，后面的章节会一一详细解释。

据 [w3techs](https://w3techs.com/technologies/details/ce-http2/all/all) 统计，截止到 2019/04/22，HTTP/2 的全球占有率为 36%，普及之路任重道远啊。我的 [个人博客](https://yanceyleo.com) 在上线之初就支持了 HTTP/2 (卧槽，我什么时候还配了 IPV6？)

![My Website](https://yancey-assets.oss-cn-beijing.aliyuncs.com/Jietu20190422-104131%402x.jpg)

## TCP/IP 通信传输流

在讲解 TCP/IP 通信传输流之前，首先复习一下 TCP/IP 的五层协议。

**应用层**：决定向用户提供应用服务时通信的活动。TCP/IP 协议族内预存了各类通用的应用服务。比如：FTP、DNS、HTTP 协议。

**传输层**：传输层对上层应用层，提供处于网络连接中的两台计算机之间的数据传输。在传输层有两个性质不同的协议，分别是 TCP (Transmission Control Protocol，传输控制协议) 和 UDP (User Data Protocol，用户数据报协议)

**网络层**：网络层用来处理在网络上流动的数据包。数据包是网络传输的最小数据单位。该层规定了通过怎样的路径到达对方计算机，并把数据包传送给对方。与对方计算机通过多台计算机或网络设备进行传输时，网络层所起的作用就是在众多的选项内选择一条传输路线。

**数据链路层**: 在物理层提供比特流服务的基础上，建立相邻结点之间的数据链路，通过差错控制提供数据帧（Frame）在信道上无差错的传输，并进行各电路上的动作系列。数据的单位称为帧（frame）

**物理层**：物理层建立在物理通信介质的基础上，作为系统和通信介质的接口，用来实现数据链路实体间透明的比特 (bit) 流传输。只有该层为真实物理通信，其它各层为虚拟通信。

TCP/IP 通信传输流如下图所示：

![Jietu20190422-142841@2x.jpg](https://yancey-assets.oss-cn-beijing.aliyuncs.com/Jietu20190422-142841%402x.jpg)

客户端在应用层 (HTTP 协议) 发出一个 HTTP 请求。

为了方便传输，在传输层 (TCP 协议) 把从应用层处收到的数据 (HTTP 请求报文) 进行分割，并在各个报文上打上标记序号及端口号后转发给网络层。

在网络层 (IP 协议)，增加作为通信目的地的 MAC 地址后转发给数据链路层。这样，发送给服务端的请求就准备齐全了。

当服务端在链路层接收到数据时，按序往上层发送，一直到应用层。**当传输到应用层时，才算真正的接收到由客户端发送过来的请求**。

## 什么是 MAC 地址？

MAC 地址 (Media Access Control Address)，直译为媒体访问控制地址，也称为局域网地址
(LAN Address)，以太网地址 (Ethernet Address) 或物理地址 (Physical Address)，它是一个用来确认网上设备位置的地址。ARP (Address Resolution Protocol) 是一种用来解析地址的协议，它可以根据 IP 地址反查出对应的 MAC 地址。

下图展示了一台电脑内网 IP 和 MAC 地址。在终端 (MAC OS 环境) 输入 `ifconfig`，找到 `en0`，便可查找本地以太网的信息。

![内网IP / MAC地址](https://yancey-assets.oss-cn-beijing.aliyuncs.com/%3AUsers%3Ayanceyleo%3ADownloads%3AJietu20190422-141257.jpg)

那么什么是 MAC 地址呢？我们知道 IP 地址是可变的，可以通过各种方式分配 IP 地址给一个设备，比如 DHCP， PPP，静态 IP 等。而 MAC 地址一般来讲却是不会变的，设备在生产时就被“烙”上了 `唯一的标识`，这个 `唯一的标识` 就是 MAC 地址。

逼乎上有个很有趣的例子：你中午在公司点了份外卖，收货地址一定是写公司的地址；晚上回到家，再点外卖时就得把地址写成家 (IP 是动态的)。但无论在哪儿点外卖，订单上的姓名和手机号一定是不变的 (MAC 地址)。

外卖小哥把午餐送到公司，但在门口等着收外卖的人肯定不止你一个 (多台设备在同一个 broadcast 网络里)，于是他就会通过姓名和手机号来精确的将午餐送到你手上。

## TCP 协议

TCP (Transmission Control Protocol, 传输控制协议) 是一种面向连接的、可靠的、基于字节流服务的传输层通信协议，由 IETF 的 RFC 793 定义。

所谓字节流服务 (Byte Stream Service) 是指为了方便传输，将大块数据分割成以报文段 (segment) 为单位的数据包进行管理。而可靠的传输服务是指 **能够把数据准确可靠的传给对方**。简言之，TCP 协议为了更容易的传送大数据而把数据分割，而且 TCP 协议可以确认数据最终是否能送达对方。

### 三次握手

- 我可以连你嘛？
- 可以。
- 那我连了。

emmmmm，单身久了，看三次握手都这么眉清目秀，哭瞎。

![images.jpeg](https://yancey-assets.oss-cn-beijing.aliyuncs.com/images.jpeg)

TCP 协议通过三次握手 (three-way handshaking) 策略来保障将数据送达目标处。TCP 协议的特点是：**在客户端发送数据包之后会向服务端确认是否送达**。

客户端首先发送一个带 SYN (synchronize) 标志的数据包给服务端。服务端收到后，回传一个带有 SYN/ACK (acknowledgement) 的数据包以示传达确认信息。最后，客户端再发送一个带 ACK 标志的数据包。至此，三次握手结束。若在握手过程中某个阶段莫名中断，TCP 协议会再次以相同的顺序发送相同的数据包。

### 为什么是三次握手？

按一般思维，我们觉得两次握手足够了，第三步看起来有些多余。那么 TCP 协议为什么要加上这一次呢？这是因为在网络请求中，我们应该时刻牢记：**网络是不可靠的，数据包是可能丢失的**。

假设没有 `第三次握手`，客户端向服务端发送一个带 SYN 标志的数据包，请求建立连接，但由于网络延迟，服务端没能及时收到这个包。上面说到如果握手过程某个阶段中断，客户端会重新发数据包。因此客户端重新发送了一个 SYN 包。

假设服务端能正常收到第二个 SYN 包，并建立了通信，一段时间后通信结束，连接被关闭。恰好第一个 SYN 包抵达了服务端，此时服务端就会发送一次 SYN/ACK 确认。

假设没有第三次确认，客户端向服务端发送了 SYN，请求建立连接。由于延迟，服务端没有及时收到这个包。于是客户端重新发送一个 SYN 包。回忆一下介绍 TCP 首部时提到的序列号，这两个包的序列号显然是相同的。
假设服务端接收到了第二个 SYN 包，建立了通信，一段时间后通信结束，连接被关闭。这时候最初被发送的 SYN 包刚刚抵达服务端，服务端又会发送一次 ACK 确认。由于两次握手就建立了连接，此时的服务端就会建立一个新的连接，然而客户端觉得自己并没有请求建立连接，所以就不会向服务端发送数据。从而导致服务端建立了一个空的连接，白白浪费资源。
在三次握手的情况下，服务端直到收到客户端的应答后才会建立连接。因此在上述情况下，客户端会接受到一个相同的 ACK 包，这时候它会抛弃这个数据包，不会和服务端进行第三次握手，因此避免了服务端建立空的连接。

![1604b4917725c794.jpg](https://yancey-assets.oss-cn-beijing.aliyuncs.com/1604b4917725c794.jpg)


## URI 和 URL 的区别

## 参考

《图解 HTTP》 -- 上野宣

[TCP 和 UDP](https://juejin.im/post/583d2d6a67f356006bb7d535)

[有了 IP 地址，为什么还要用 MAC 地址？](https://www.zhihu.com/question/21546408)

[[面试∙网络] TCP/IP（四）：TCP 与 UDP 协议简介](https://juejin.im/post/5a2ff1f36fb9a04500030771)
