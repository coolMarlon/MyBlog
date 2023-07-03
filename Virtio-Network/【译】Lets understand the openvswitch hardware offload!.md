 💡 一点感受： 1、这篇文章不太行，看完没学到啥，可能是我太菜，很多东西没理解。 2、另外改用notion写BLOG了，不得不说，真好看！

原文地址：https://hareshkhandelwal.blog/2020/03/11/lets-understand-the-openvswitch-hardware-offload/

本博客的目的是提供一个参考指南，解释与 Ovs 硬件卸载 + Openstack + Mellanox 智能 网卡相关的配置/故障排除问题的所有基础知识

让我们开始。

## 构建模块

下图描绘了使用 Openstack 部署 ovs 硬件卸载时涉及的构建块。 自 Redhat OSP13（社区版 Queen）以来，此功能可用作技术预览版。

![img](https://pic1.zhimg.com/80/v2-a89a38338c11b7344ad08915decd6c43_720w.png?source=d16d100b)



编辑

添加图片注释，不超过 140 字（可选）

让我们理解他们中的每一个

## Nova

Nova 调度程序在处理卸载功能时没有任何变化，其行为与 SR-IOV 传递相同，例如生成。 Nova 调度程序应配置为使用 PciPassthroughFilter，Nova 计算应配置为 passthrough_whitelist。 Os-vif 会找出映射到插入 VM 的 VF 端口的代表端口名称，并将代表端口插入 br-int。

## Neutron

启用卸载后，NIC e-swithc模式更改为“switchdev”。 这将在 NIC 上创建代表端口并映射/对应于每个创建的 VF。 Representator 端口被插入主机，并将数据包从 VF 接收到和从 VF 到内核交换。

为了从启用 switchdev 的 NIC 分配端口，需要使用绑定配置文件创建 neutron 端口，其中提到的功能如下。

```
$ openstack port create –network private –vnic-type=direct –binding-profile ‘{“capabilities”: [“switchdev”]}’ direct_port1
```

管理员在创建实例时传递此端口信息。代表端口与实例 VF 接口相关联，并插入到 ovs 桥 br-int 进行一次 ovs 数据路径处理。

## OVS

在卸载环境中，第一个数据包遍历 ovs 内核数据路径。 有关图形表示，请参阅“数据包旅程”部分。 第一次遍历使 ml2 ovs 能够为实例流量找出传入/传出数据包的规则。 一旦形成与特定流量流有关的所有流，ovs 将使用 TC 花实用程序在 NIC 硬件上推送和编程这些流。

我们需要在 ovs 上应用以下配置才能启用硬件卸载。

```
$ sudo ovs-vsctl set Open_vSwitch . other_config:hw-offload=true
```

## TC 子系统

当启用硬件卸载标志时，Ovs 使用 TC 数据路径。 TC Flower 是一个 iproute2 实用程序，用于在硬件上写入数据路径流。

我们需要在 ovs 上应用以下配置。 如果未明确配置 tc-policy，这也是默认选项。

```
$ sudo ovs-vsctl set Open_vSwitch . other_config:tc-policy=both
```

这将确保当流程被编程时，它将在硬件和软件数据路径上被编程。 如果硬件程序失败，流量仍然可以通过 tc 数据路径进行引导，但会产生性能成本。

## 网卡驱动(PF 和 VF)

Mellanox ConnectedX-5 使用 mlx5_core 作为其 PF 和 VF 驱动程序。 该驱动程序通常负责硬件上的表创建、流处理、switchdev 配置、块设备创建等。 devlink 工具用于设置 pci 设备的模式，如下所示。

```
$ sudo devlink dev eswitch set pci/0000:03:00.0 mode switchdev
```

## 网卡防火墙

NIC 固件是一种非易失性内存软件，并在重新启动后保持其配置。 但是，它是运行时可修改的。 它通常负责维护表/规则、修复表管道、硬件资源管理、VF 创建。 除了驱动程序外，我们还需要固件支持才能使任何功能顺利运行。

我们需要在接口上应用以下配置来启用端口级别的tc花推流编程。

```
$ sudo ethtool -K enp3s0f0 hw-tc-offload on
```

## 包的旅程

![img](https://picx.zhimg.com/80/v2-684c49c7877d5925655ba5022625e79e_720w.png?source=d16d100b)



编辑切换为居中

添加图片注释，不超过 140 字（可选）

注意：

- 这里说明的流程只是一个例子。 实际流将包含更多匹配和操作字段，如 vxlan 隧道、vlan encap/decap 等。
- 一旦在 NIC 上对流进行编程，VLAN 和 VxLAN 的数据包流将保持不变。 在 VxLAN 的情况下，br-tun 将处理 VxLAN 传输层报头的封装/去封装以进行第 1 个数据包处理。、、
- 对于每个交通方向，都会对流量进行编程。 IE。 流量发送/接收实例将为两个方向编程 2 个流。 如果不支持该操作，则无法卸载该规则。
- 类似地，当无法识别字段时，无法卸载规则。 有关支持的分类器和操作的列表，请访问此处。

从下面的流剖析部分显示的实例中捕获 ping 流量的示例输出。

## 基本故障排除：

### 功能支持的版本

Linux Kernel >= 4.13

Open vSwitch >= 2.8

openstack >= Pike version

iproute >= 4.12

Mellanox NICs FW

FW ConnectX-5: >= 16.21.0338

FW ConnectX-4 Lx: >= 14.21.0338

注意：Mellanox ConnectX-4 NIC 仅支持 VLAN 卸载。 Mellanox ConnectX-4 Lx/ConnectX-5 NIC 支持 VLAN/VXLAN 卸载。

[当前支持的分类器（Ovs 2.11）](https://www.notion.so/a7e7403abb5e474f8df257d0dbf8118e)

![img](https://picx.zhimg.com/80/v2-335362629f2aee9c7e91b0f0280a611d_720w.png?source=d16d100b)



编辑切换为居中

添加图片注释，不超过 140 字（可选）

## 流解剖

卸载流功能与硬件交换机/路由器使用 ASIC 芯片组所做的非常相似。 一旦在 ASIC 上做出路由/转发决策并写入，ASIC 就会处理传入数据包的匹配和操作，并提供线速处理。 路由器/交换机可以在需要进行此类调试时进入 ASIC 外壳并找出其表条目。

看看下面从 cumulus linux 运行交换机中获取的 Broadcom 芯片组输出。

```
root@dni-7448-26:~# cl-bcmcmd l2 show
mac=00:02:00:00:00:08 vlan=2000 GPORT=0x2 modid=0 port=2/xe1
mac=00:02:00:00:00:09 vlan=2000 GPORT=0x2 modid=0 port=2/xe1 Hit
```

但是，NIC 供应商不提供此类访问，并让疑难解答人员猜测在未卸载流量时 NIC 硬件可能出现什么问题。

有了这个限制，运营商就信任 ovs 和 tc 来确保流程制定符合前向政策并推送到硬件。

让我们剖析取自 ovs 数据路径表的示例流程并尝试理解它。

下面是一对交易流。 来自外部网络的数据包到达绑定接口（PFs 绑定）并推送到代表端口（最终进入 VM）的第一个流，而来自代表端口（来自 VM 的出口）的数据包通过绑定离开的第二个流。

```
ufid:6ef8818a-17df-4714-9010-1203cbc163c9, skb_priority(0/0),skb_mark(0/0),in_port(bond-pf),packet_type(ns=0/0,id=0/0),eth(src=fa:16:3e:51:c3:7c,dst=f8:f2:1e:03:bf:e4),eth_type(0x8100),vlan(vid=398,pcp=0),encap(eth_type(0x0800),ipv4(src=0.0.0.0/0.0.0.0,dst=0.0.0.0/0.0.0.0,proto=0/0,tos=0/0,ttl=0/0,frag=no)), packets:11, bytes:1078, used:0.080s, offloaded:yes, dp:tc, actions:pop_vlan,eth3
ufid:724ad8e3-2f49-4aa9-8ea7-b5331fba2a1f, skb_priority(0/0),skb_mark(0/0),in_port(eth3),packet_type(ns=0/0,id=0/0),eth(src=f8:f2:1e:03:bf:e4,dst=fa:16:3e:51:c3:7c),eth_type(0x0800),ipv4(src=0.0.0.0/0.0.0.0,dst=0.0.0.0/0.0.0.0,proto=0/0,tos=0/0,ttl=0/0,frag=no), packets:11, bytes:1122, used:0.080s, offloaded:yes, dp:tc, actions:push_vlan(vid=398,pcp=0),bond-pf
```

- Ufid：唯一流标识符
- skb_priority：这个 qdisc 字段不是写在硬件上的。 它有一个掩码值 /0，这使它在处理数据包时可以忽略。
- skb_mark：这个 qdisc 字段不是写在硬件上的。 它有一个掩码值 /0，这使它在处理数据包时可以忽略。
- in_port: 传入数据包的端口
- Packet_type：ovs支持的包类型有很多。 然而，虽然卸载这不是写在硬件上，它的掩码值也使它在处理数据包时具有相关性。
- eth_type：offload 支持所有 eth_type 并**写入硬件**。 然而，除了 vlan 和 mpls 之外，所有其他 eth_type 的内部字段都没有用于匹配数据包。 转发仅由 eth_type 而不是其他字段完成。 IE。 如果 eth_type = 0x0806，则 arp 标头不用于匹配操作。
- Eth (src,dst)：二层以太网源和mac地址
- vlan：vlan id 和 pcp（priority code points）
- ipv4(src/dst)：因为这里的掩码是/0，这些被忽略，任何ipv4 src、dst包被这个流处理。
- ipv4(proto,tos,tt,frag)：下一个头部的协议，Tos 字节，生存时间和片段偏移量。
- Statistics：这里的数字是从接口中提取的。
- Used：卸载后的时间
- Offloaded：ovs端确认
- dp: tc: – 这里使用的数据路径是 tc

让我们看一条ARP流

```
ufid:31ec10fa-ffdc-454b-867f-f673dae77ea6, skb_priority(0/0),skb_mark(0/0),in_port(bond-pf),packet_type(ns=0/0,id=0/0),eth(src=fa:16:3e:51:c3:7c,dst=f8:f2:1e:03:bf:e4),eth_type(0x8100),vlan(vid=398,pcp=0),encap(eth_type(0x0806),arp(sip=0.0.0.0/0.0.0.0,tip=0.0.0.0/0.0.0.0,op=0/0,sha=00:00:00:00:00:00/00:00:00:00:00:00,tha=00:00:00:00:00:00/00:00:00:00:00:00)), packets:1, bytes:56, used:4.950s, offloaded:yes, dp:tc, actions:pop_vlan,eth3
ufid:7329344d-b5f9-4b03-b26a-a9c7318b24b7, skb_priority(0/0),skb_mark(0/0),in_port(eth3),packet_type(ns=0/0,id=0/0),eth(src=f8:f2:1e:03:bf:e4,dst=fa:16:3e:51:c3:7c),eth_type(0x0806),arp(sip=0.0.0.0/0.0.0.0,tip=0.0.0.0/0.0.0.0,op=0/0,sha=00:00:00:00:00:00/00:00:00:00:00:00,tha=00:00:00:00:00:00/00:00:00:00:00:00), packets:0, bytes:0, used:5.150s, offloaded:yes, dp:tc, actions:push_vlan(vid=398,pcp=0),bond-pf
```

在这里，整个 **arp** 头（发送方 ip/硬件地址、目标 ip/硬件地址）都用 0 掩码，因此只应使用 eth_type=0x086 来做出决策并卸载到硬件。

以下是 VxLAN 的卸载流。

```
ufid:d56153f3-f8c7-4e8a-a6ce-3ddeb73cf27a, skb_priority(0/0),tunnel(tun_id=0x3d,src=10.10.147.101,dst=10.10.147.104,ttl=0/0,tp_dst=4789,flags(+key)),skb_mark(0/0),in_port(vxlan_sys_4789),packet_type(ns=0/0,id=0/0),eth(src=fa:16:3e:45:a9:e2,dst=f8:f2:1e:03:bf:e6),eth_type(0x0800),ipv4(src=0.0.0.0/0.0.0.0,dst=0.0.0.0/0.0.0.0,proto=0/0,tos=0/0,ttl=0/0,frag=no), packets:19, bytes:1862, used:0.080s, offloaded:yes, dp:tc, actions:eth0
ufid:3bcda9c2-2d36-4692-82c2-aa631f015560, skb_priority(0/0),skb_mark(0/0),in_port(eth0),packet_type(ns=0/0,id=0/0),eth(src=f8:f2:1e:03:bf:e6,dst=fa:16:3e:45:a9:e2),eth_type(0x0800),ipv4(src=0.0.0.0/0.0.0.0,dst=0.0.0.0/0.0.0.0,proto=0/0,tos=0/0x3,ttl=0/0,frag=no), packets:18, bytes:2736, used:0.090s, offloaded:yes, dp:tc, actions:set(tunnel(tun_id=0x3d,src=10.10.147.104,dst=10.10.147.101,ttl=64,tp_dst=4789,flags(key))),vxlan_sys_4789
```

- 隧道类型取决于接口封装的类型。 它支持 VXLAN / NVGRE / RAW 和 GENEVE。
- flags(key) 被忽略并且不会卸载到硬件

### 系统验证

- 确保在系统上启用 SR-IOV 和 VT-d。
- 在 Linux 中通过将 intel_iommu=on 添加到内核参数来启用 IOMMU，例如，使用 GRUB。

### Debugs

Ovs 调试目前没有帮助。 增强流编程的可调试性，但已在最新版本中修复。 虽然我们可以在 ovs 上启用调试，如下所示。

```
$ ovs-appctl vlog/set dpif_netlink:file:DBG;
```

日志打印在 **/var/log/openvswitch/ovs-vswitchd.log**.

例如，以下是失败的流编程的捕获。

```
less /var/log/openvswitch/ovs-vswitchd.log | grep -Ei “ERR|syndrome”
2019-02-19T12:48:46.126Z|00001|dpif_netlink(handler1)|ERR|failed to offload flow: Operation not supported
```

并行检查 ovsdb 日志也会有所帮助。 它们位于 /var/log/openvswitch/ovsdb-server.log

Mellanox 提供了一个系统信息脚本，它通常从系统中捕获所有必需的信息。 它可以在

https://github.com/Mellanox/linux-sysinfo-snapshot/blob/master/sysinfo-snapshot.py

并且可以作为

```
# ./sysinfo-snapshot.py –asap –asap_tc –ibdiagnet –openstack
```

注意：如果操作不受支持，则无法卸载规则。 类似地，当无法识别字段时，无法卸载规则。

### CLIs:

```
$ ovs-dpctl dump-flows -m type=offloaded
$ tc filter show dev ens1_0 ingress
$ tc -s filter show dev ens1_0 ingress
$ tc monitor
```

Ethtool 提供更多细节 Tov view number of channels:  **ethtool -l <uplink representor>** To check Statistics: **ethtool -I <uplink/VFs>** To see driver Information: **ethtool -i <uplink rep>** To check ring sizes: **ethtool -g <uplink rep>** Enabled features: **ethtool -k <uplink/VFs>**

同样，我们可以在代表端口和 PF 端口使用 tcpdump 检查流量。

### Notes

- VF 链接状态的管理操作由 VF 代表链接状态控制。 这意味着，对代表端口链接状态的任何更改都会影响 VF 的链接状态。
- Representator 端口统计数据也提供了 VF 的统计数据。

### 限制

“openvswitch”防火墙驱动程序不能与卸载一起使用，因为卸载路径中不支持流的连接跟踪属性。 支持默认的“iptables_hybrid”或“none”。 补丁已合并到 Ovs 2.13 以支持连接跟踪。