作者QQ：2338953
# 我的网络日记
为了建设这个大规模的无线网络，费了较大的力气，一开始我只搞了一个VLAN，IP范围采用了/21位子网，结果及其不稳定。
当我研究出此文的办法时，只要我愿意，完全可以给任何一个员工分配一个独立的VLAN，经实际观察，一个VLAN的用户不超过100时比较理想。
然而基于MAC划分VLAN依然有一个缺点，随着MAC数量的增多，交换机的验证机制会导致越来越难以存储更多的记录。
因此对于一些AP，完全可以单独分配一个固定的VLAN ID。从而达到不需要把MAC写入交换机的目的。
## 一 涉及设备
华为接入用交换机2700一台、Unifi交换机一台、Unifi AP若干、UniFi Network Controller。
## 二 华为交换机配置
#### 下级交换机有需要配置固定VLAN ID时，此ID得在华为交换机上配置为tag模式，否则一律使用untagged。
```
vlan batch 2 to 11 31 to 59 64 to 71
vlan 59
 mac-vlan mac-address 802a-a816-eac0 priority 0
 mac-vlan mac-address 802a-a816-e88b priority 0
 mac-vlan mac-address 14cc-2001-d7af priority 0
interface GigabitEthernet0/0/1
 port hybrid pvid vlan 69
 port hybrid tagged vlan 32 to 38
 port hybrid untagged vlan 39 to 59 69
 mac-vlan enable
 unicast-suppression 80
 multicast-suppression 80
 broadcast-suppression 80
 storm-control interval 90
 storm-control action block
 storm-control enable log
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 2 to 71
 bpdu disable
```

## 三 AC控制器配置A
Unifi交换机 采用后，默认管理VLAN为LAN
每个交换机端口默认为 All
### 方案一
如果整个系统采用依据mac划分vlan,就啥也不改，此时Unifi交换机就相当于一个傻瓜换机。 
华为交换机全使用 untagged
AP的管理接口默认即可，
在华为交换机的vlan 59上，配置MAC绑定mac-vlan mac-address 802a-a816-eac0 priority 0

### 方案二
如果不但要依据mac划分vlan,还有一部分AP需要使用固定VLAN
就把Unifi交换机的管理vlan改成 59 华为交换机VLAN 59 使用 tagged

交换机的接口 从默认的All 改成 自己定义接口,
自定义接口 Native 网络 选 vlan 32 此时用户手机接入时会自动获取vlan 32对应的IP。
          Tagged 网络 选 vlan 59
AP的管理接口配置为59，AP自身获取vlan 59对应的IP,便于管理。
