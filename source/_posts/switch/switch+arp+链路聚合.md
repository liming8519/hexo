---
title: switch+arp+链路聚合
date: 2019-10-22 23:00:00
tags: 
- 5700 
categories: 
- switch
---


>switch+arp+链路聚合

<!-- more -->




## 链路聚合

```
<HUAWEI> system-view
[HUAWEI] sysname SA
[SA] interface Eth-Trunk 1
[SA-Eth-Trunk1]mode manual load-balance
[SA-Eth-Trunk1]trunkport g 0/0/1 to 0/0/3
[SA-Eth-Trunk1]quit
[SA]vlan batch 10 20
[SA]interface g 0/0/4
[SA-GigabitEthernet0/0/4] port link-type access
[SA-GigabitEthernet0/0/4] port default vlan 10
[SA-GigabitEthernet0/0/4] quit
[SA] interface g 0/0/5
[SA-GigabitEthernet0/0/5] port link-type access
[SA-GigabitEthernet0/0/5] port defalut vlan 20
[SA-GigabitEthernet0/0/5] quit
[SA] interface Eth-Trunk 1
[SA-Eth-Trunk1]port link-type trunk
[SA-Eth-Trunk1]port trunk allow-pass vlan 10 20
[SA-Eth-Trunk1]load-balance src-dst-mac
[SA-Eth-Trunk1]quit
[SA] display eth-trunk 1
[SA]port-group 1
[SA-port-group-1]group-member g 0/0/1 to g 0/0/3


switchb 配置同switcha




```

## 链路聚合LACP

```
<Huawei>system-view
Enter system view, return user view with Ctrl+Z.
[Huawei]sysname SA
[SA]interface eth-trunk 1
[SA-Eth-Trunk1]mode lacp
[SA-Eth-Trunk1]quit
[SA]interface g 0/0/1
[SA-GigabitEthernet0/0/1]eth-trunk 1
[SA-GigabitEthernet0/0/1]quit
[SA]interface g 0/0/2
[SA-GigabitEthernet0/0/2]eth-trunk 1
[SA-GigabitEthernet0/0/2]quit
[SA]interface g 0/0/3
[SA-GigabitEthernet0/0/3]eth-trunk 1
[SA-GigabitEthernet0/0/3]quit
[SA]lacp priority 100
[SA]interface eth-trunk 1
[SA-Eth-Trunk1]max active-linknumber 2
[SA-Eth-Trunk1]quit
[SA]interface g 0/0/1
[SA-GigabitEthernet0/0/1]lacp priority 100
[SA-GigabitEthernet0/0/1]quit
[SA]interface g 0/0/2
[SA-GigabitEthernet0/0/2]lacp priority 100
[SA-GigabitEthernet0/0/2]quit
[SB]dis eth-trunk 1
Eth-Trunk1's state information is:
Local:
LAG ID: 1                   WorkingMode: STATIC                               
Preempt Delay: Disabled     Hash arithmetic: According to SIP-XOR-DIP         
System Priority: 32768      System ID: 4c1f-ccf4-4162                         
Least Active-linknumber: 1  Max Active-linknumber: 2                          
Operate status: up          Number Of Up Port In Trunk: 2                     
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
GigabitEthernet0/0/1   Selected 1000TG   100     2      401     10111100  1     
GigabitEthernet0/0/2   Selected 1000TG   100     3      401     10111100  1     
GigabitEthernet0/0/3   Unselect 1000TG   32768   4      401     10100000  1     

Partner:
--------------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri PortNo PortKey PortState
GigabitEthernet0/0/1   100      4c1f-cce3-2653  100     2      401     10111100
GigabitEthernet0/0/2   100      4c1f-cce3-2653  100     3      401     10111100
GigabitEthernet0/0/3   100      4c1f-cce3-2653  32768   4      401     10100000
    

[SA]display eth-trunk 1
Eth-Trunk1's state information is:
Local:
LAG ID: 1                   WorkingMode: STATIC                               
Preempt Delay: Disabled     Hash arithmetic: According to SIP-XOR-DIP         
System Priority: 100        System ID: 4c1f-cce3-2653                         
Least Active-linknumber: 1  Max Active-linknumber: 2                          
Operate status: up          Number Of Up Port In Trunk: 2                     
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
GigabitEthernet0/0/1   Selected 1000TG   100     2      401     10111100  1     
GigabitEthernet0/0/2   Selected 1000TG   100     3      401     10111100  1     
GigabitEthernet0/0/3   Unselect 1000TG   32768   4      401     10100000  1     

Partner:
--------------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri PortNo PortKey PortState
GigabitEthernet0/0/1   32768    4c1f-ccf4-4162  100     2      401     10111100
GigabitEthernet0/0/2   32768    4c1f-ccf4-4162  100     3      401     10111100
GigabitEthernet0/0/3   32768    4c1f-ccf4-4162  32768   4      401     10100000
    
    
```


## ARP配置三则

```
VLAN间

<HUAWEI> system-view
[HUAWEI] sysname Switch
[Switch] vlan batch 2
[Switch] interface gigabitethernet 0/0/1
[Switch-GigabitEthernet0/0/1] port link-type access
[Switch-GigabitEthernet0/0/1] port default vlan 2
[Switch-GigabitEthernet0/0/1] quit
[Switch] interface gigabitethernet 0/0/2
[Switch-GigabitEthernet0/0/2] port link-type access
[Switch-GigabitEthernet0/0/2] port default vlan 2
[Switch-GigabitEthernet0/0/2] quit




[Switch] vlan batch 3
[Switch] interface gigabitethernet 0/0/3
[Switch-GigabitEthernet0/0/3] port link-type access
[Switch-GigabitEthernet0/0/3] port default vlan 3
[Switch-GigabitEthernet0/0/3] quit
[Switch] interface gigabitethernet 0/0/4
[Switch-GigabitEthernet0/0/4] port link-type access
[Switch-GigabitEthernet0/0/4] port default vlan 3
[Switch-GigabitEthernet0/0/4] quit


[Switch] vlan 4
[Switch-vlan4] aggregate-vlan
[Switch-vlan4] access-vlan 2
[Switch-vlan4] access-vlan 3
[Switch-vlan4] quit
[Switch] interface vlanif 4
[Switch-Vlanif4] ip address 10.10.10.1 24
[Switch-Vlanif4] arp-proxy inter-sub-vlan-proxy enable
[Switch-Vlanif4] quit
#配置Host_1的IP地址为10.10.10.2/24。
#配置Host_2的IP地址为10.10.10.3/24。
#配置Host_3的IP地址为10.10.10.4/24。
#配置Host_4的IP地址为10.10.10.5/24。



[Switch] display arp interface vlanif 4
IP ADDRESS      MAC ADDRESS     EXPIRE(M) TYPE        INTERFACE   VPN-INSTANCE                                                      
                                          VLAN/CEVLAN                                                                               
------------------------------------------------------------------------------                                                      
10.10.10.1      101b-5441-5bf6            I -         Vlanif4                                                                     
------------------------------------------------------------------------------                                                      
Total:1         Dynamic:0       Static:0     Interface:1

C:\Documents and Settings\Administrator> ping 10.10.10.4
Pinging 10.10.10.4 with 32 bytes of data:
Reply from 10.10.10.4: bytes=32 time<1ms TTL=128
Reply from 10.10.10.4: bytes=32 time<1ms TTL=128
Reply from 10.10.10.4: bytes=32 time<1ms TTL=128
Reply from 10.10.10.4: bytes=32 time<1ms TTL=128

Ping statistics for 10.10.10.4:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
    
    
    
    



VLAN内


<HUAWEI> system-view
[HUAWEI] sysname Switch
[Switch] vlan batch 2
[Switch] interface gigabitethernet 0/0/1
[Switch-GigabitEthernet0/0/1] port-isolate enable
[Switch-GigabitEthernet0/0/1] port link-type access
[Switch-GigabitEthernet0/0/1] port default vlan 2
[Switch-GigabitEthernet0/0/1] quit
[Switch] interface gigabitethernet 0/0/2
[Switch-GigabitEthernet0/0/2] port-isolate enable
[Switch-GigabitEthernet0/0/2] port link-type access
[Switch-GigabitEthernet0/0/2] port default vlan 2
[Switch-GigabitEthernet0/0/2] quit



[Switch] vlan 3
[Switch-vlan3] aggregate-vlan
[Switch-vlan3] access-vlan 2
[Switch-vlan3] quit




[Switch] interface vlanif 3
[Switch-Vlanif3] ip address 10.10.10.1 24
[Switch-Vlanif3] arp-proxy inner-sub-vlan-proxy enable
[Switch-Vlanif3] quit



#配置Host_1的IP地址为10.10.10.3/24。
#配置Host_2的IP地址为10.10.10.2/24。



[Switch] display arp interface vlanif 3
IP ADDRESS      MAC ADDRESS     EXPIRE(M) TYPE        INTERFACE   VPN-INSTANCE                                                      
                                          VLAN/CEVLAN                                                                               
------------------------------------------------------------------------------                                                      
10.10.10.1      101b-5441-5bf6            I -         Vlanif3                                                                     
------------------------------------------------------------------------------                                                      
Total:1         Dynamic:0       Static:0     Interface:1  




C:\Documents and Settings\Administrator> ping 10.10.10.2
Pinging 10.10.10.2 with 32 bytes of data:
Reply from 10.10.10.2: bytes=32 time<1ms TTL=128
Reply from 10.10.10.2: bytes=32 time<1ms TTL=128
Reply from 10.10.10.2: bytes=32 time<1ms TTL=128
Reply from 10.10.10.2: bytes=32 time<1ms TTL=128

Ping statistics for 10.10.10.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
    
    
    



路由式 ， 路由可达，如果开启arp-proxy数据包就会送到三层去处理，子网掩码会在计算机系统中起作用，子网掩码需要仔细理解他只会限制自己的网卡的意思。


1.创建VLAN，将接口加入VLAN，并配置接口的IP地址 

# 配置Switch_1。
<HUAWEI> system-view
[HUAWEI] sysname Switch_1
[Switch_1] vlan batch 10
[Switch_1] interface gigabitethernet 0/0/1
[Switch_1-GigabitEthernet0/0/1] port link-type access
[Switch_1-GigabitEthernet0/0/1] port default vlan 10
[Switch_1-GigabitEthernet0/0/1] quit
[Switch_1] interface vlanif 10
[Switch_1-Vlanif10] ip address 172.16.1.1 24


# 配置Switch_2。
<HUAWEI> system-view
[HUAWEI] sysname Switch_2
[Switch_2] vlan batch 20
[Switch_2] interface gigabitethernet 0/0/1
[Switch_2-GigabitEthernet0/0/1] port link-type access
[Switch_2-GigabitEthernet0/0/1] port default vlan 20
[Switch_2-GigabitEthernet0/0/1] quit
[Switch_2] interface vlanif 20
[Switch_2-Vlanif20] ip address 172.16.2.1 24



2.配置路由式Proxy ARP 

# 配置Switch_1。
[Switch_1-Vlanif10] arp-proxy enable
[Switch_1-Vlanif10] quit

# 配置Switch_2。
[Switch_2-Vlanif20] arp-proxy enable
[Switch_2-Vlanif20] quit


3.验证配置结果 

# 在Switch_1上查看VLANIF10接口的ARP表项，可以看到VLANIF10接口的IP地址对应的MAC地址。
[Switch_1] display arp interface vlanif 10
IP ADDRESS      MAC ADDRESS     EXPIRE(M) TYPE        INTERFACE   VPN-INSTANCE                                                      
                                          VLAN/CEVLAN                                                                               
------------------------------------------------------------------------------                                                      
172.16.1.1      101b-5441-5bf6            I -         Vlanif10                                                                     
------------------------------------------------------------------------------                                                      
Total:1         Dynamic:0       Static:0     Interface:1  

# 在子公司A选取一台主机Host_1（IP地址：172.16.1.2/16，操作系统以Windows 7为例），在子公司B选取一台主机Host_2（IP地址：172.16.2.2/16）。在主机Host_1上Ping主机Host_2的IP地址，可以Ping通。
C:\Documents and Settings\Administrator> ping 172.16.2.2
Pinging 172.16.2.2 with 32 bytes of data:
Reply from 172.16.2.2: bytes=32 time<1ms TTL=128
Reply from 172.16.2.2: bytes=32 time<1ms TTL=128
Reply from 172.16.2.2: bytes=32 time<1ms TTL=128
Reply from 172.16.2.2: bytes=32 time<1ms TTL=128

Ping statistics for 172.16.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

# 在Host_1上查看ARP表，可以看到主机Host_2的IP地址对应的MAC地址是Switch_1的VLANIF10接口的MAC地址，可见Host_1和Host_2之间是通过ARP代理实现互通的。
C:\Documents and Settings\Administrator> arp -a
Interface: 172.16.1.2 --- 0xd
  Internet Address      Physical Address      Type
  172.16.2.2            101b-5441-5bf6        dynamic
...



```