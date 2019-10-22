---
title: QINQ配置
date: 2019-10-22 22:00:00
tags:
- 5700
categories:
- switch
---
>QINQ配置
<!-- more -->



## 配置基本QinQ

### 操作步骤
- 1. 创建VLAN 

```
#在SwitchA上创建VLAN100和VLAN200。
<HUAWEI> system-view
[HUAWEI] sysname SwitchA
[SwitchA] vlan batch 100 200


# 在SwitchB上创建VLAN100和VLAN200。
<HUAWEI> system-view
[HUAWEI] sysname SwitchB
[SwitchB] vlan batch 100 200

```

- 2. 配置接口类型为QinQ 

```
#在SwitchA上配置接口GE1/0/1、GE1/0/2的类型为QinQ，GE1/0/1的外层tag为VLAN100，GE1/0/2的外层tag为VLAN200。SwitchB的配置与SwitchA类似，不再赘述。
[SwitchA] interface gigabitethernet 1/0/1
[SwitchA-GigabitEthernet1/0/1] port link-type dot1q-tunnel //配置接口的链路类型为QinQ
[SwitchA-GigabitEthernet1/0/1] port default vlan 100
[SwitchA-GigabitEthernet1/0/1] quit
[SwitchA] interface gigabitethernet 1/0/2
[SwitchA-GigabitEthernet1/0/2] port link-type dot1q-tunnel //配置接口的链路类型为QinQ
[SwitchA-GigabitEthernet1/0/2] port default vlan 200
[SwitchA-GigabitEthernet1/0/2] quit
```


- 3. 配置Switch连接公网侧的接口 

```
#在SwitchA上配置接口GE1/0/3加入VLAN100和VLAN200。SwitchB的配置与SwitchA类似，不再赘述。
[SwitchA] interface gigabitethernet 1/0/3
[SwitchA-GigabitEthernet1/0/3] port link-type trunk
[SwitchA-GigabitEthernet1/0/3] port trunk allow-pass vlan 100 200
[SwitchA-GigabitEthernet1/0/3] quit
```

- 4. 配置外层VLAN tag的TPID值 

```
# 在SwitchA上配置外层VLAN tag的TPID值为0x9100。
[SwitchA] interface gigabitethernet 1/0/3
[SwitchA-GigabitEthernet1/0/3] qinq protocol 9100 //配置QinQ外层VLAN tag的TPID值为0x9100


# 在SwitchB上配置外层VLAN tag的TPID值为0x9100。
[SwitchB] interface gigabitethernet 1/0/3
[SwitchB-GigabitEthernet1/0/3] qinq protocol 9100 //配置QinQ外层VLAN tag的TPID值为0x9100

```


- 5. 验证配置结果 

```
从企业1一处分支内任意VLAN的一台PC ping企业1另外一处分支同一VLAN内的PC，如果可以ping通则表示企业1内部可以互相通信。

从企业2一处分支内任意VLAN的一台PC ping企业2另外一处分支同一VLAN内的PC，如果可以ping通则表示企业2内部可以互相通信。

从企业1一处分支内任意VLAN的一台PC ping企业2任意一处分支同一VLAN内的PC，如果不能ping通则表示企业1和企业2之间相互隔离

```

## 灵活QinQ

基于VLAN ID的灵活QinQ功能可实现接口在接收到数据帧后，依据帧中不同VLAN ID添加不同的外层VLAN Tag。

### 操作步骤
- 1.创建VLAN 

```
# 在SwitchA上创建VLAN2、VLAN3，即叠加后的外层VLAN。
<HUAWEI> system-view
[HUAWEI] sysname SwitchA
[SwitchA] vlan batch 2 3

# 在SwitchB上创建VLAN2、VLAN3，即叠加后的外层VLAN。
<HUAWEI> system-view
[HUAWEI] sysname SwitchB
[SwitchB] vlan batch 2 3

```

- 2.在接口上配置灵活QinQ 

```
 说明： 

对于盒式交换机需要在接口下执行qinq vlan-translation enable命令，使能接口VLAN转换功能。

# 配置SwitchA的接口GE1/0/1。
[SwitchA] interface gigabitethernet 1/0/1
[SwitchA-GigabitEthernet1/0/1] port link-type hybrid
[SwitchA-GigabitEthernet1/0/1] port hybrid untagged vlan 2 3  //配置Hybrid类型接口加入的VLAN，这些VLAN的帧以Untagged方式通过接口
[SwitchA-GigabitEthernet1/0/1] port vlan-stacking vlan 100 stack-vlan 2  //配置内部VLAN Tag为100，并添加外层VLAN Tag为2
[SwitchA-GigabitEthernet1/0/1] port vlan-stacking vlan 300 stack-vlan 3  //配置内部VLAN Tag为300，并添加外层VLAN Tag为3
[SwitchA-GigabitEthernet1/0/1] quit

# 配置SwitchB的接口GE1/0/1。
[SwitchB] interface gigabitethernet 1/0/1
[SwitchB-GigabitEthernet1/0/1] port link-type hybrid
[SwitchB-GigabitEthernet1/0/1] port hybrid untagged vlan 2 3  //配置Hybrid类型接口加入的VLAN，这些VLAN的帧以Untagged方式通过接口
[SwitchB-GigabitEthernet1/0/1] port vlan-stacking vlan 100 stack-vlan 2  //配置内部VLAN为100，并添加外层VLAN为2
[SwitchB-GigabitEthernet1/0/1] port vlan-stacking vlan 300 stack-vlan 3  //配置内部VLAN为300，并添加外层VLAN为3
[SwitchB-GigabitEthernet1/0/1] quit
```

- 3.配置其它接口 

```
# 在SwitchA上配置接口GE1/0/2加入VLAN2、VLAN3。
[SwitchA] interface gigabitethernet 1/0/2
[SwitchA-GigabitEthernet1/0/2] port link-type trunk
[SwitchA-GigabitEthernet1/0/2] port trunk allow-pass vlan 2 3
[SwitchA-GigabitEthernet1/0/2] quit

# 在SwitchB上配置接口GE1/0/2加入VLAN2、VLAN3
[SwitchB] interface gigabitethernet 1/0/2
[SwitchB-GigabitEthernet1/0/2] port link-type trunk
[SwitchB-GigabitEthernet1/0/2] port trunk allow-pass vlan 2 3
[SwitchB-GigabitEthernet1/0/2] quit
```

- 4.验证配置结果 

```
如果SwitchA、SwitchB上配置正确，则：

•PC上网用户可以通过运营商网络互相通信。


•VoIP用户可以通过运营商网络互相通信。

```
