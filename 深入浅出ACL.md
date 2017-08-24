---
title: 深入浅出ACL
date: 2016-08-11 00:33:21
tags: 网络协议
---

## 什么是ACL
ACL，访问控制列表，用于控制网络流量的访问，同时可以用作某些感兴趣流的定义，如定义需要NAT转换的数据流，或被VPN加密的数据流

## ACL类别
标准ACL、扩展ACL、带时间的ACL、动态ACL、自反ACL

## 标准ACL
1.使用ACL的列表号范围为**1~99**，只能控制基于**源地址**的流量访问，因为如此，所有应该应用于距离目标（流量发出方）最近的位置。

2.有一条隐身拒绝ACL，拒绝一切流量。

3.用反码来匹配IP，ACL语句逐条匹配

```
	R1(config)# access-list 1 deny 192.168.1.0 0.0.0.255
```
access-list 1 表示ACL的列表号为1（1~99)，拒绝192.168.1.0 0.0.0.255对应的子网，基于**源地址过滤**

```
	R1(config)# interface ethernet 1/0
	R1(cofig-if)# ip access-group 1 in
```
将这个ACL配置到e1/0的入方向，出方向为out

这里提一下数据转发和ACL执行的过程：
1.判断进入路由器的数据包是否可路由，不可路由的数据包被丢弃

2.数据包可路由，判断数据包在进入路由器的接口的入方向有无ACL

3.没有就执行路由表查询，有ACL，则根据ACL逐条匹配，如果ACL的条目拒绝转发，数据包将被丢弃，反之，执行路由查表

4.数据包执行完路由查表后，被送达到路由器出接口，判断出接口的出方向有无配置ACL，如果ACL的条目允许了，数据包就转发出去，反之，数据包被丢弃

## 扩展ACL
1.ACL的列表号100~199，可以同时基于**源地址和目标地址**来控制流量，还可以**通过不同协议的端口号**来控制网络流量，应该**距离源（要访问的IP）最近**

2.逐条匹配，具有一个隐式全部拒绝ACL

```
	R1(config)# acces-list 101 permit tcp 192.168.1.0 0.0.0.255 host 192.168.5.2 eq www
```

101位列表号，允许192.168.1.0 0.0.0.255上的主机对192.168.5.2的tcp 80端口访问，host表示全匹配，如：0.0.0.0

ACL设计要注意的东西：
1.标准ACL只关心源，所有应用的距离控制目标最近的位置

2.扩展ACL及关心源又关心目标地址，应运用到距离控制源最近的接口位置，这样可以减少主干网上无用的流量

3.**同一接口、同一协议、同一方向只能应用一个访问控制列表**

4.ACL只能过滤路由器的流量，不能对ACL本身应用的路由器的流量进行过滤

## 带时间的ACL
它能控制ACL策略在不同时间段生效

首先，要明白，配置带时间ACL必须保证网络上的时钟同步，可以去路由器上手动设置时间，但最好用NTP（网络时钟协议）来同步时钟

然后要明白，建立时间表，才可配置带时间ACL

最后，配置ACL控制语句时，将建立的时间表关联到ACL上

配置R1的时钟源
```
 R1# clock set 01:01:01 12 jun 2016//配置R1时间
 R1(config)# ntp master//将R1设为NTP服务器，其他路由器的时间从R1上进行同步
 R1(config)# ntp source ethernet 1/1//配置NTP的更新源接口，同步时间数据包从该接口发出
```

配置R2通过NTP与R1同步
```
 	R2(config)# ntp server 192.168.1.1//R1的IP
```

在R1上配置带时间的ACL
```
	R1(config)# time-range workday
	R1(config-time-range)# periodic weekdays 08:30 to 17:30//指定一个关键字weekdays，给它设置一个时间范围
	R1(config-time-range)# periodic weekend 17:31 to 23:59 

	R1(config)# ip access-list extended ayu//建立一个名为ayu的扩展ACL

	R1(config-ext-nacl)# permit tcp host 192.168.1.2 host 192.168.1.3 eq www time-range workday//带上时间

	R1(config)# int e1/0
	R1(config-if)# ip access-group ayu in//将带时间的ACL配置到相应接口的入方向
```
## 动态ACL
动态ACL要求非安全域的主机访问受保护的网络资源之前，首先要经过身份认证，然后它根据自身配置的规则自动创建动态ACL来允许会话传入

工作过程：
1.客户机A要访问服务器U，在中间的路由器Y上配置了动态ACL，那么A要先尝试telnet到路由器Y，telnet进程会要求A输入用户名和密码，动态ACL根据这个用户名和密码来完成对远程主机身份的认证，**这里的telnet只是一种假象，真正的目的在于身份认证**

2.身份认证完后，配置了动态ACL的Y会添加一条临时的允许访问内部网络的ACL，这个ACL是有绝对时间限制的，还有一个时间限制就是这条ACL没有数据流通过时，经过一段时间，就算没有到绝对时间限制，也会失效

3.当到达绝对时间限制时，这条临时建立的ACL会被删除

动态ACL的使用情景：
网络边界路由器只允许某些特点的访问来源进入安全区域访问服务器，但是访问来源可能同拨号形式连接Internet，而通常拨号形式的Interent接入，每次获得的IP都会不同，此时就是动态ACL使用之时。

配置
```
	Y(config)# username ayu password lmwen

	Y(config)# username ayu autocommand access-enable host timeout 10
	//为ayu用户配置动态ACL，空闲时间为10分钟，也就是10分钟没有匹配流量，这条ACL将失效，host关键字很重要，表示全匹配，也即是动态插入的ACL只对激活身份验证的主机生效，如果没有host关键字，一旦某个主机验证成功，其他主机将不需要验证，这是不安全的

	Y(config)# access-list 101 permit tcp any host 192.168.1.3 eq telnet
	//运行任何源IP地址通过telnet访问R1（192.168.1.3），因为要通过telnet来做身份验证

	Y(config)# access-list 101 dynamic ayu timeout 120 permit ip any host 192.168.2.2
	//动态ACL是在扩展ACL 101上构造出来的，所以使用扩展ACL 101所使用的ACL号码，关键字dynamic来声明这个名为ayu的ACL是动态ACL，timeout定义了时间限制，120分钟后，这个临时ACL会被删除，无论当时是否有流量

	Y(config)# line vty 0 3//进入VTY 0、1、2、3线路配置模式，留一个VTY 4来给真正要用telnet登录的设备

	Y(config-line)# login local

	Y(config)# line vty 4

	Y(config-line)# login

	Y(config-line)# password ayu

	Y(config-line)# rotary 1//将telnet端口号移动到3001

	Y(config)# enable password ayu
	Y(config)# access-list 1-1 permit tcp any host 192.168.3.1 eq 3001
	Y(config)# int e1/0

	Y(config-if)# ip access-group 101 in//在R1的e1/0接口入方向应用ACL 101
```
这样动态ACL配置完成了，而且正常的telnet不会受到影响
```
C:\> telnet 192.168.3.1 //动态ACL身份认证，完成后会立即切断telnet会话
C:\> telnet 192.168.3.1 3001//这是正常的Telnet管理，不会切断telnet
```

## 自反ACL
自反ACL可以根据上层的会话信息来过滤IP数据包，它可以使自己的网络发起的会话中的数据返回到自己的网络，但是不会允许外部网络的主机主动访问自己

工作流程：
1.出站流量穿越路由器Y时，自动建立流量审计规则，哪些流量建立审计规则由自反ACL语句决定

2.当流量从外部返回内部时，如ping的回程流量，路由器Y就将使用第一步中建立的审计规则来审计流量是否可以到达内部网络，它审计的条目有状态标记，如，TCP返回连接数据的SYN=1，ACK=1表示从外部返回给内部的网络的流量，如果收到SYN=1，ACK=0，就会拒绝

3.如果外部主机访问内部网络将会被拒绝

关于自反ACL要注意的几点：
1.自反ACL只能由带编号或命令的扩展ACL承载

2.自反ACL是临时的，从会话开始到会话结束，自反ACL将会自动删除

3.自反ACL不可直接应用到具体的接口上，它被嵌套在某个具体的扩展ACl中，然后将这个扩展ACL应用到具体的接口上，自反ACL不存在隐式拒绝deny ip any any，但是它嵌套的扩展ACL存在隐式拒绝

4.自反ACL不能应用在会话过程中端口改变的服务，如主动FTP，主动FTP之前的博文有详细聊过，在主动FTP会话过程中端口号是会发生改变的，此时要么用被动FTP，要么用CBAC（基于上下文的访问控制）

配置：
```
	Y(config)# ip access-list extended ayu//建立一个叫作ayu的扩展ACL

	Y(config-ext-nacl)# permit ip any any reflect lmwen timeout 60
	//建立一个名为lmwen的自反ACL，permit ip any any 指审计任意源地址和目标地址的IP流量，也就是只要内部出去的流量，外部返回都会通过，除了端口号改变的，临时ACL列表将根据这个审计条件来建立，reflect审计建立自反ACL的关键字，timeout 60表示自反ACL空闲超时值
	//这里只是定义了一个自反ACL的一个审计条件，并不会真实的阻挡外部的流量进入内部
 
	Y(config)# ip access-list extended uya//建立一个uya的扩展ACL
	Y(config-ext-nacl)# evaluate lmwen//根据一个名为lmwen的自反ACL来建立一个临时的审计条目来过滤从外部到内部的流量
	//这里定义的ACL会真正的阻挡外部的流量进入内容，虽然lmwen这个自反ACL没有隐式拒绝，但是uya有

	Y(config)# int e1/0
	Y(config-if)# ip access-group ayu//应用审计流量的ayu，不会真实阻止外部流量进入内部
	Y(config-if)# ip access-group uya//应用过滤数据的ACL，会真实阻止外部流量进入内部
```

这里，名为ayu的扩展ACL承载自反ACL的规则，也就是建立流量审计时要根据的规则，名为uya的扩展ACL承载自反ACL建立临时ACL的动作

带时间的ACL、动态ACL、自反ACL都是被扩展ACL承载的

**用代码创造乐趣---ayu**