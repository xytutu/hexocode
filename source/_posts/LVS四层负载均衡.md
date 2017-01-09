---
title: LVS四层负载均衡
date: 2016-12-28 17:47:26
tags: Net
---
>## 四层负载均衡
四层负载均衡指的是在即在OSI第4层工作，就是TCP层，一般使用的LVS的IP负载均衡；每个LVS中的负载均衡服务器都有一个IP一般称为VIP，而用户对服务请求必须基于对此VIP进行访问。

>#### 当用户的请求到达VIP如何分配到RealServer上的，此处有3种实现分别是NAT、TUN和DR。
>### NAT： 即（Virtual Server via Network Address Translation）
网络翻译技术实现虚拟服务器，当用户的请求到达VIP Server的时候，VIP Server根据一定的负载均衡策略，修改报文的目的IP和端口为Real Server，而Real Server做出相应之后，将报文返回VIP Server，VIP 修改报文的源地址从Real Server改为VIP Server 发给Client。从上面过程中流量的进入和方位全部需要经过VIP Server重写，所以存在VIP Server的瓶颈。
>### TUN ：即（Virtual Server via IP Tunneling）
IP隧道技术实现虚拟服务器， 和NAT技术相似流量进入并转发给Real Server链路是相同的，不同的是Real Server返回直接范围Client不经过前端VIP Server，此方式VIP Server将只处理进入的流量，集群系统的吞吐量大大提高。
>### DR： 即（Virtual Server via Direct Routing） 
直接路由实现虚拟服务器，它的思路和TUN技术比较相似，当时在用户进入的流量转发到Real Server的时候不是采用IP进行转发的，而是根据mac地址，直接修改数据包的mac的地址，相应报文也是直接返回给Client。(DR模式下需要LVS和绑定同一个VIP,通过loopback实现)这种方式是3种方式中效率最高的，但是它是基于mac转发的，限制VIP Server与Real Server都有一块网卡连在同一物理网段上。

>### 总括
负载均衡系统必须建立在面对网络连接的基础上，而不是面对数据包的基础上。也就是说网络连接是有状态，A数据包和后续B数据报很可能是有关联，比如TCP的三次握手，软负载面向的对象应该是一个已经建立连接的用户，而不是一个孤零零的IP包。所以说在第一次对某Client的流量选定了后端服务器之后，一定记录出先转发的路由表，在一定时间的范围内保证保持Client和后端服务器连接关联性。为了确保这一点，LVS内部维护着一个Session的Hash表，通过客户端的某些信息可以找到应该转发到哪一台Real Server上。
