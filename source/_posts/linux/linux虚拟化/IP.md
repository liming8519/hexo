---
title: IP报文
date: 2019-10-15 02:00:00
tags: 
- 虚拟化 
categories: 
- linux
---


>IP报文

<!-- more -->

## IP报文

```

		|**4*b***|***4b***|******8b********|**3b**|*******13b*********|
		---------------------------------------------------------------
		|*Version|**IHL***|*Type*of*Service|**********Total*Lengeth***|
		|*******Identification*************|Flags*|FragmentationOffset|
		|**Time*To*Live***|*Protocol*******|***Header*Checksum********|
		|*******************Source*Address****************************|
		|***********************Destionation*Address******************|
		|**********Options*************************|**Padding*********|
		---------------------------------------------------------------
		|****************data*****************************************|
		---------------------------------------------------------------


```

## Version 
* 0100 ipv4
* 0110 ipV6
## IHL(Internet Header Length ip表头长度)
* IHL最大是60（字节），IHL最小是20（字节）

## Type of Service服务类型

* O:3位优先权字段(现已被忽略)
* O:3位优先权字段(现已被忽略)
* O:3位优先权字段(现已被忽略)
* D：若为 0 表示一般延迟(delay)，若为 1 表示为低延迟；
* T：若为 0 表示为一般传输量 (throughput)，若为 1 表示为高传输量；
* R：若为 0 表示为一般可靠度(reliability)，若为 1 表示高可靠度。
* M: 传输成本：0：普通，1：成本尽量小
* O: 最后一位被保留，恒定为 0

## Total Length(总长度)

* 指这个IP 封包的总容量，包括表头与内容 (Data) 部分。最大可达 65535 bytes
## Identification(标识符)
* 唯一的标识主机发送的报文。IP软件在储存器中维护一个计数器，每产生一个数据报，计数器就+1，并将此值赋值给标识字段。当数据报长度超过网络的MTU而必须分片时，这个标识字段的值就被复制到所有的数据报片的标识字段中。

## Flag(分片标志)

0： 不可用
D（DF）：若为 0 表示可以分片，若为 1 表示不可分片，不让路由器做分片处理。
M（MF）：若为 0 表示此 IP 为最后分片，若为 1 表示非最后分片。

## Fragment Offset(片偏移)

* 较长的报文在分片之后，某片在原报文中的相对位置。片偏移以8字节为单位，这就是说除了最后一个分片，每个分片的长度是8字节的整数倍。
## Time To Live(生存周期)

* 表示这个 IP 封包的存活时间，范围为 0-255。当这个 IP 封包通过一个路由器时， TTL 就会减一，当 TTL 为 0 时，这个封包将会被直接丢弃。说实在的，要让 IP 封包通过 255 个路由器，还挺难的。
## Protocol

* 通过协议号标识上层使用的是哪一层协议，常用的协议号有：TCP=6，UDP=17，ICMP=1，IGMP=2，IP=4，EGP=8，IPV6=41
## Header Checksum
* ☻
## Source Address

* ☻
## Destination Adress

* ☻
## Options (其他参数)

* ☻
## Padding(补齐项目)

* ☻

## ETHERTYPE

* 6B 源MAC
* 6B 目标MAC
* 2B 协议如下

```	

	0x0000 - 0x05DC   IEEE 802.3 长度
	0x0101 – 0x01FF   实验
	0x0600   XEROX NS IDP
	0x0661   DLOG
	0x0800   网际协议（IP）
	0x0801   X.75 Internet
	0x0802   NBS Internet
	0x0803   ECMA Internet
	0x0804   Chaosnet
	0x0805   X.25 Level 3
	0x0806   地址解析协议（ARP ： Address Resolution Protocol）
	0x0808   帧中继 ARP （Frame Relay ARP） [RFC1701]
	0x6559   原始帧中继（Raw Frame Relay） [RFC1701]
	0x8035   动态 DARP （DRARP：Dynamic RARP）反向地址解析协议（RARP：Reverse Address Resolution Protocol）
	0x8037   Novell Netware IPX
	0x809B   EtherTalk
	0x80D5   IBM SNA Services over Ethernet
	0x80F3   AppleTalk 地址解析协议（AARP：AppleTalk Address Resolution Protocol）
	0x8100   以太网自动保护开关（EAPS：Ethernet Automatic Protection Switching）
	0x8137   因特网包交换（IPX：Internet Packet Exchange）
	0x814C   简单网络管理协议（SNMP：Simple Network Management Protocol）
	0x86DD   网际协议v6 （IPv6，Internet Protocol version 6）
	0x880B   点对点协议（PPP：Point-to-Point Protocol）
	0x880C   通用交换管理协议（GSMP：General Switch Management Protocol）
	0x8847   多协议标签交换（单播） MPLS：Multi-Protocol Label Switching <unicast>）
	0x8848   多协议标签交换（组播）（MPLS, Multi-Protocol Label Switching <multicast>）
	0x8863   以太网上的 PPP（发现阶段）（PPPoE：PPP Over Ethernet <Discovery Stage>）
	0x8864   以太网上的 PPP（PPP 会话阶段）（PPPoE，PPP Over Ethernet<PPP Session Stage>）
	
	0X888E   认证
	0X88C7   预认证
	0x88BB   轻量级访问点协议（LWAPP：Light Weight Access Point Protocol）
	0x88CC   链接层发现协议（LLDP：Link Layer Discovery Protocol）
	0x8E88   局域网上的 EAP（EAPOL：EAP over LAN）
	0x9000   配置测试协议（Loopback）
	0x9100   VLAN 标签协议标识符（VLAN Tag Protocol Identifier）
	0x9200   VLAN 标签协议标识符（VLAN Tag Protocol Identifier）
	0xFFFF   保留

```