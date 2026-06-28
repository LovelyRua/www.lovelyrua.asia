---
title: Cisco IOS 速查表 - Switching and Routing Quick Reference
description: 面向 CCNA 与日常实验环境的 Cisco IOS 配置与排障命令速查表
date: 2026-06-28 13:55:00+0800
# image: cover.jpg
categories:
    - IT-Techs
tags:
    - Cisco
    - CCNA
    - Network
    - CheatSheet
---

这是一份偏 CCNA 语境的 Cisco IOS 速查表，覆盖基础管理、二层交换、三层交换、路由器、常见网络服务与排障检查命令。  
约定：尖括号 `<...>` 表示需要替换的变量；命令以典型 IOS 语法为主，不同平台或版本可能略有差异。

---

#### **基础模式与编辑**

```ios
enable
configure terminal
```

```text
Ctrl+A      移动到行首
Ctrl+E      移动到行尾
```

```ios
show running-config | include <string>
default interface <interface-id>
do write
```

`do` 可在配置模式下执行 EXEC 命令，例如 `do show ip interface brief`。

---

#### **安全管理基线**

```ios
hostname R1
enable secret <password>
service password-encryption
ip domain-name CCNA.local
```

```ios
username <username> privilege 15 secret <password>
ip ssh version 2
crypto key generate rsa modulus 2048
```

限制远程管理来源：

```ios
access-list 1 permit host <management-ip>

line vty 0 15
 login local
 exec-timeout 5 0
 transport input ssh
 access-class 1 in
```

Console 口基础保护：

```ios
line console 0
 password <password>
 login
 exec-timeout 5 0
 logging synchronous
```

检查：

```ios
show ssh
show running-config | include username|enable|transport|access-class
```

---

#### **日志与时间戳**

```ios
logging console <severity>
logging monitor <severity>
logging buffered <size> <severity>
logging host <syslog-server-ip>
logging trap <severity>
```

```ios
service timestamps log datetime
service sequence-numbers
terminal monitor
logging synchronous
```

说明：

- `logging buffered`：保存在本机内存中；
- `logging host` + `logging trap`：发送到远程 Syslog；
- `terminal monitor`：远程终端实时查看日志；
- `logging synchronous`：避免日志打断正在输入的命令行。

---

#### **SNMP 基础**

```ios
snmp-server contact <contact>
snmp-server location <location>
snmp-server community <community> ro
snmp-server host <ip-address> version 2c <community>
snmp-server enable traps
```

生产环境建议使用 SNMPv3；实验与 CCNA 语境中常见 `v2c + community`。

---

#### **IOS 升级流程**

```ios
ip ftp username <username>
ip ftp password <password>
copy ftp: flash:
```

指定启动镜像：

```ios
boot system flash:<ios-image.bin>
write
reload
```

确认与清理：

```ios
show version
show file systems
show flash
delete flash:<old-ios-image.bin>
```

---

#### **接入交换机基础配置**

创建 VLAN：

```ios
vlan 10
 name ENGINEERING
vlan 11
 name VOICE
```

配置接入口：

```ios
interface range f0/1-12
 description -- Engineering Department Ports --
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 11
```

禁用未使用端口：

```ios
interface range f0/25-48
 description -- Not in Use - Disabled --
 shutdown
```

配置上联 Trunk：

```ios
interface g0/1
 description -- Trunk to Upstream Device --
 switchport mode trunk
 switchport nonegotiate
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 1001
```

交换机管理地址：

```ios
interface vlan 1
 ip address 192.168.123.253 255.255.255.0
 no shutdown

ip default-gateway 192.168.123.1
```

检查：

```ios
show ip interface brief
show interfaces status
show vlan brief
show interfaces trunk
show interfaces <interface-id> switchport
```

---

#### **STP：生成树基础**

```ios
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default
```

指定根桥：

```ios
spanning-tree vlan <vlan-id> root primary
spanning-tree vlan <vlan-id> root secondary
```

或手动指定优先级：

```ios
spanning-tree vlan <vlan-id> priority <priority>
```

检查：

```ios
show spanning-tree
show spanning-tree vlan <vlan-id>
show errdisable recovery
```

---

#### **EtherChannel**

配置 LACP：

```ios
port-channel load-balance src-dst-ip

interface range f0/1-4
 channel-group 1 mode active
```

配置聚合接口：

```ios
interface port-channel 1
 description -- Uplink EtherChannel --
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

检查：

```ios
show etherchannel summary
show etherchannel port-channel
show interfaces port-channel 1
```

---

#### **端口安全**

```ios
interface f0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum <number>
 switchport port-security mac-address sticky
 switchport port-security violation <shutdown|restrict|protect>
```

老化策略：

```ios
switchport port-security aging time <minutes>
switchport port-security aging type <absolute|inactivity>
```

Errdisable 自动恢复：

```ios
errdisable recovery cause psecure-violation
errdisable recovery interval <seconds>
```

检查：

```ios
show port-security
show port-security interface <interface-id>
show mac address-table secure
show errdisable recovery
```

---

#### **DHCP Snooping 与 ARP Inspection**

启用 DHCP Snooping：

```ios
ip dhcp snooping
ip dhcp snooping vlan <vlan-id>
no ip dhcp snooping information option
```

信任上联端口：

```ios
interface g0/1
 ip dhcp snooping trust
```

限制接入口速率：

```ios
interface f0/1
 ip dhcp snooping limit rate <packets-per-second>
```

启用 Dynamic ARP Inspection：

```ios
ip arp inspection vlan <vlan-id>
ip arp inspection validate src-mac dst-mac ip
```

信任上联：

```ios
interface g0/1
 ip arp inspection trust
```

静态 ARP ACL：

```ios
arp access-list <name>
 permit ip host <ip-address> mac host <mac-address>

ip arp inspection filter <name> vlan <vlan-id>
```

检查：

```ios
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection
show ip arp inspection interfaces
```

---

#### **三层交换机**

三层上联口：

```ios
interface g0/1
 description -- L3 Link to Upstream --
 no switchport
 ip address 10.0.0.253 255.255.255.252
 no shutdown
```

SVI 网关：

```ios
vlan 10
 name ENGINEERING

interface vlan 10
 description -- Gateway for Engineering VLAN --
 ip address 10.0.0.62 255.255.255.192
 no shutdown
```

开启三层转发与默认路由：

```ios
ip routing
ip route 0.0.0.0 0.0.0.0 10.0.0.254
```

检查：

```ios
show ip interface brief
show vlan brief
show interfaces trunk
show ip route
```

---

#### **路由器接口与静态路由**

接口地址：

```ios
interface g0/1
 ip address <ip-address> <subnet-mask>
 no shutdown
```

默认路由：

```ios
ip route 0.0.0.0 0.0.0.0 <next-hop-ip>
```

主机路由：

```ios
ip route 1.1.1.1 255.255.255.255 <next-hop-ip>
```

Router-on-a-stick：

```ios
interface g0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0

interface g0/0.99
 encapsulation dot1q 99 native
 ip address 192.168.99.1 255.255.255.0
```

---

#### **OSPF 与动态路由**

进程配置：

```ios
router ospf 1
 passive-interface <interface-id>
 network <network-address> <wildcard-mask> area 0
 auto-cost reference-bandwidth 100000
```

接口配置：

```ios
interface g0/0
 ip ospf 1 area 0
```

检查：

```ios
show ip protocols
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
```

---

#### **FHRP：默认网关冗余**

HSRP 示例：

```ios
interface g0/0
 standby 1 ip <virtual-ip>
 standby 1 priority 150
 standby 1 preempt
```

检查：

```ios
show standby
show standby brief
```

---

#### **IPv6 基础**

开启 IPv6 转发：

```ios
ipv6 unicast-routing
```

检查：

```ios
show ipv6 interface brief
show ipv6 route
```

---

#### **ACL**

标准 ACL：

```ios
access-list 1 permit <source-ip> <wildcard-mask>
access-list 1 deny any
```

命名 ACL：

```ios
ip access-list standard <name>
 permit <source-ip> <wildcard-mask>
 deny any
```

扩展 ACL：

```ios
ip access-list extended <name>
 permit tcp <source> <source-wildcard> <destination> <destination-wildcard> eq 443
 deny ip any any
```

应用到接口：

```ios
interface g0/1
 ip access-group <number|name> <in|out>
```

检查：

```ios
show access-lists
show ip interface <interface-id>
```

---

#### **CDP / LLDP**

全局启用：

```ios
cdp run
lldp run
```

接口启用 LLDP：

```ios
interface range g0/0-1
 lldp transmit
 lldp receive
```

检查：

```ios
show cdp neighbors detail
show lldp neighbors detail
```

---

#### **NTP / DNS**

NTP：

```ios
ntp server <ntp-server> key <key-number>
ntp peer <ip-address>
ntp update-calendar
ntp source <interface-id>
```

NTP 认证：

```ios
ntp authentication-key <key-number> md5 <key>
ntp trusted-key <key-number>
```

时区：

```ios
clock timezone CST 8
clock summer-time CDT recurring
```

DNS：

```ios
ip domain-lookup
ip name-server <server1> <server2>
ip host <hostname> <ip-address>
```

检查：

```ios
show ntp status
show ntp associations
show hosts
```

---

#### **DHCP Server / Relay / Client**

排除地址：

```ios
ip dhcp excluded-address 192.168.1.1 192.168.1.100
```

地址池：

```ios
ip dhcp pool <pool-name>
 network 192.168.1.0 255.255.255.0
 dns-server 8.8.8.8
 domain-name lovelyrua.asia
 default-router 192.168.1.1
 lease <days> <hours> <minutes>
```

DHCP Relay：

```ios
interface g0/1
 ip helper-address <dhcp-server-ip>
```

DHCP Client：

```ios
interface f0/1
 ip address dhcp
```

检查：

```ios
show ip dhcp binding
show ip dhcp pool
show ip dhcp server statistics
```

---

#### **NAT / PAT**

标记 inside / outside：

```ios
interface g0/1
 ip nat inside

interface g0/0
 ip nat outside
```

静态 NAT：

```ios
ip nat inside source static 192.168.0.1 100.0.0.1
```

动态 NAT：

```ios
access-list 1 permit 192.168.0.0 0.0.0.255
ip nat pool POOL1 100.0.0.10 100.0.0.20 netmask 255.255.255.0
ip nat inside source list 1 pool POOL1
```

PAT：

```ios
access-list 1 permit 192.168.0.0 0.0.0.255
ip nat inside source list 1 interface g0/0 overload
```

检查：

```ios
show ip nat translations
show ip nat statistics
```

---

#### **QoS 基础**

```ios
class-map HTTP_MAP
 match protocol http
```

```ios
policy-map G0/0/0_OUT
 class HTTP_MAP
  set ip dscp af31
  priority percent 10
 class class-default
  bandwidth percent 90
```

应用：

```ios
interface g0/0/0
 service-policy output G0/0/0_OUT
```

---

#### **GRE Tunnel**

```ios
interface tunnel 0
 ip address <tunnel-ip> <subnet-mask>
 tunnel source <outside-interface>
 tunnel destination <remote-public-ip>
```

检查：

```ios
show interfaces tunnel 0
show ip route
```

---

#### **VRF**

```ios
ip vrf CUSTOMER1
```

```ios
interface g0/0
 ip vrf forwarding CUSTOMER1
 ip address 192.168.1.1 255.255.255.0
```

检查：

```ios
show ip vrf
show ip route vrf CUSTOMER1
```

---

#### **常用排障命令索引**

接口与 VLAN：

```ios
show ip interface brief
show interfaces status
show interfaces trunk
show vlan brief
```

二层安全：

```ios
show spanning-tree
show etherchannel summary
show port-security
show ip dhcp snooping binding
show ip arp inspection
```

路由与邻居：

```ios
show ip route
show ip protocols
show ip ospf neighbor
show standby brief
show cdp neighbors detail
show lldp neighbors detail
```

服务：

```ios
show ntp status
show hosts
show ip dhcp binding
show ip nat translations
```

终端侧快速检查：

```powershell
ipconfig /all
ipconfig /displaydns
ipconfig /flushdns
ping <ip-address> -n 1
```
