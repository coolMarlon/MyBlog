原文地址:     [https://www.redhat.com/en/blog/journey-vhost-users-realm?page=3&f%5B0%5D=taxonomy_blog_post_category_tid%3A121](https://www.redhat.com/en/blog/journey-vhost-users-realm?page=3&f[0]=taxonomy_blog_post_category_tid%3A121)

注：这篇文章内容较长，概念较多，理解起来较为困难，需要反复研究

这篇文章是对使用 DPDK 的针对高性能用户空间网络的 vhost-user/virtio-pmd 架构的技术深入探讨，建立在[解决方案概述文章提](https://www.redhat.com/en/blog/how-vhost-user-came-being-virtio-networking-and-dpdk)供的介绍的基础上。 它适用于有兴趣了解该架构的具体细节的架构师和开发人员，随后将提供一个补充动手博客，以第一手探索这些概念。

## **介绍**

在之前的[深入探讨博客](https://www.redhat.com/en/blog/deep-dive-virtio-networking-and-vhost-net)中，我们展示了使用 vhost-net 协议将网络处理从 qemu 移到内核驱动程序中的好处。 在这篇文章中，我们将更进一步，展示如何使用 D[PDK：数据平面开发套件](https://www.dpdk.org/) 将数据平面从内核移动到来宾和主机中的用户空间来实现更好的网络性能。 为了实现这一点，我们还将详细研究 vhost 协议的新实现：vhost-user 库。 在本文结束时，您应该对基于 vhost-user/virtio-pmd 的体系结构中涉及的所有组件有深入的了解，并了解它提供的显着性能改进背后的原因。

## **DPDK和它的优势**

您可能已经听说过 DPDK。这个用户空间快速数据包处理库是许多网络功能虚拟化 (NFV) 应用程序的核心，允许它们完全在用户空间中实现，绕过内核的网络堆栈。 DPDK 是一组用户空间库，使用户能够创建优化的、高性能的数据包处理应用程序。它带来了许多优势，使其在开发人员中非常受欢迎，并且是一个强大的工具。以下是其中一些：

- CPU亲核性 - DPDK 将每个不同的线程固定到特定的逻辑核心，以最大限度地提高并行性。
- 巨页 - DPDK 具有多层内存管理（例如 [Mempool ](https://doc.dpdk.org/guides/prog_guide/mempool_lib.html)库或 [Mbuf ](https://doc.dpdk.org/guides/prog_guide/mbuf_lib.html)库）。然而，在引擎盖下，所有内存都是使用 Hugetlbfs 中的 mmap 分配的。使用 2MB 甚至 1GB 页面，DPDK 减少了缓存未命中和 TLB 查找。
- 无锁环形缓冲区- DPDKs 数据包处理基于 [Ring ](https://doc.dpdk.org/guides/prog_guide/ring_lib.html?highlight=lockless)库，它提供了一个高效、无锁的环形队列，支持突发入队和出队操作。
- 轮询模式驱动 - 为避免中断开销，DPDK 中提供了轮询模式驱动程序 ([PMD](https://doc.dpdk.org/guides/prog_guide/poll_mode_drv.html)) 抽象。
- VFIO 支持 - [VFIO（Virtual Function I/O）](https://www.kernel.org/doc/Documentation/vfio.txt)提供了一个用户空间驱动程序开发框架，允许用户空间应用程序通过将 I/O 空间直接映射到应用程序的内存来直接与硬件设备交互。

除了这些特性之外，DPDK 还支持另外两种技术，它们为我们提供了极大地增强云环境中网络应用程序性能的工具。这些是：

- Vhost-user 库 - 实现 vhost 协议的用户空间库（在“[virtio-networking vhost-net](https://www.redhat.com/en/blog/introduction-virtio-networking-and-vhost-net)”帖子中进行了描述）。
- Virtio-PMD - 建立在 DPDK 的 PMD 抽象之上，virtio-pmd 驱动程序

## **DPDK和OVS:一个完美的组合**

DPDK 优势的一个很好的例子是为已经流行的 Open vSwitch ([OVS-DPDK](https://docs.openvswitch.org/en/latest/intro/install/dpdk/)) 带来的性能提升。 Open vSwitch 是一个功能丰富的多层分布式虚拟交换机，广泛用作虚拟环境和其他 SDN 应用程序的主要网络层。 传统上，它被分为由流表组成的基于内核的快速数据路径 (fastpath) 和处理与快速路径中的任何流都不匹配的数据包的较慢用户空间数据路径 (slowpath)。 通过将 OVS 与 DPDK 集成，快速路径也在用户态，最大限度地减少内核与用户态的交互，并利用 DPDK 提供的高性能。 结果是使用 DPDK 的 OVS 性能比原生 OVS 提高了大约 10 倍。

那么，我们如何将 OVS-DPDK 的特性和性能与基于 virtio 的架构相结合？ 在接下来的部分中，我们将一一引导您了解所需的组件。

## **DPDK中的VHost-user 库**

vhost 协议是一组消息和机制，旨在将 virtio 数据路径处理从 QEMU（主要想要卸载数据包处理）卸载到外部元素（处理程序，它配置 virtio 环并执行实际的数据包处理）。最相关的机制是：

- 一组消息，允许主节点将 virtqueue 的内存布局和配置发送到处理程序。
- 一对 eventfd 类型的文件描述符，允许guest绕过master并直接向/从处理程序发送和接收通知：可用缓冲区通知（从guest发送到处理程序以指示有缓冲区准备好处理）和已用缓冲区通知（从处理程序发送到guest以指示它已完成处理缓冲区）

在“[virtio-networking vhost-net](https://www.redhat.com/en/blog/deep-dive-virtio-networking-and-vhost-net)”一文中，我们描述了该协议的具体实现（vhost-net 内核模块）以及它如何允许 qemu 将网络处理卸载到主机内核。现在我们介绍 vhost-user 库。该库内置于 DPDK，是 vhost 协议的用户空间实现，允许 qemu 将 virtio 设备数据包处理卸载到任何 DPDK 应用程序（例如 Open vSwitch）。 vhost-user 库和 vhost-net 内核驱动程序之间的主要区别在于通信通道。虽然 vhost-net 内核驱动程序使用 ioctls 实现此通道，但 vhost-user 库[定义了通过 unix 套接字发送的消息的结构](https://github.com/qemu/qemu/blob/master/docs/interop/vhost-user.rst)。 DPDK 应用程序可以配置为提供 unix 套接字（服务器模式）并让 QEMU 连接到它（客户端模式）。然而，相反的情况也是可能的，它允许在不重启 VM 的情况下重启 DPDK。

在这个套接字上，所有请求都由master（即：QEMU）发起，其中一些请求需要响应，例如 GET_FEATURES 请求或任何设置了 REPLY_ACK 位的请求。 与 vhost-net 内核模块的情况一样，vhost-user 库允许master通过执行以下重要操作来配置数据平面卸载：

- 特性协商：virtio 特性和 vhost-user-specific 特性都以类似的方式协商：首先，主要“获取”处理程序支持的特性位掩码，然后“设置”它也支持的子集。
- 内存区域配置：主机设置内存映射区域，以便处理程序可以 mmap() 它们。
- Vring 配置：primary 设置要使用的虚拟队列数量及其在内存区域内的地址。请注意，vhost-user 支持多队列，因此可以配置多个 virtqueue 以提高性能。
- Kick 和 Call 文件描述符发送：通常使用 irqfd 和 ioeventfd 机制。有关这些机制的更多信息，请参阅“[Deep dive Virtio-networking 和 vhost-net](https://www.redhat.com/en/blog/deep-dive-virtio-networking-and-vhost-net)”，有关 virtio 队列机制的更多信息，请参阅即将发布的 virtio dataplane 帖子。

由于这种机制，DPDK 应用程序现在可以通过与guest共享内存区域来处理数据包，并且可以直接向guest发送和接收通知，而无需 qemu 干预。 将所有内容粘合在一起的最后一个元素是 QEMU 的 virtio 设备模型。它有两个主要任务：

- 它模拟一个 virtio 设备，该设备显示在来宾中的特定 PCI 端口中，可以由来宾无缝探测和配置。此外，它将 ioeventfd 映射到模拟设备的内存映射 I/O 空间，将 irqfd 映射到它的全局系统中断 (GSI)。结果是来宾不知道通知和中断都在没有 QEMU 干预的情况下转发到 vhost-user 库和从 vhost-user 库转发的事实
- 它不是实现实际的 virtio 数据路径，而是充当 vhost-user 协议中的主控，以将此处理卸载到 DPDK 进程中的 vhost-user 库
- 它处理来自控制虚拟队列的请求，在某些情况下，这些请求被转换为 vhost-user 请求

该图显示了作为 DPDK-APP 一部分运行的 vhost-user-library 如何使用 virtio-device-model 和 virtio-pci 设备与 qemu 和来宾交互：

![img](https://picx.zhimg.com/80/v2-35569ad2d5a0b7fe5f9bc3000c3ca20b_720w.png?source=d16d100b)



编辑切换为居中

添加图片注释，不超过 140 字（可选）

作为 DPDK-APP 一部分运行的 vhost-user-library 如何使用 virtio-device-model 和 virtio-pci 设备与 qemu 和guest交互 这张图有几点需要说明：

- virtio 内存区域最初由guest分配。
- 相应的 virtio 驱动程序通过 virtio 规范中定义的 PCI BARs 配置接口与 virtio 设备正常交互。
- virtio-device-model（在 QEMU 内部）使用 vhost-user 协议来配置 vhost-user 库，以及设置 irqfd 和 ioeventfd 文件描述符。
- 由guest分配的 virtio 内存区域由 vhost-user 库（即 DPDK 应用程序）映射（使用 mmap 系统调用）。
- 结果是DPDK应用程序可以直接从guest内存读取和写入数据包，并使用irqfd和ioeventfd机制直接通知guest。

## **guest中的用户空间网络**

我们已经介绍了 DPDK 的 vhost-user 实现如何允许我们将数据路径处理从主机内核 (vhost-net) 卸载到专用的 DPDK 用户空间应用程序（例如 Open vSwitch），从而显着提高我们的网络性能。现在，我们将看到如何通过在来宾的用户空间中运行高性能网络应用程序（例如 NFV 服务）来代替 virtio-net 内核方法来对来宾执行相同的操作。 为了能够直接在设备上运行用户空间网络应用程序，我们需要三个组件：

- VFIO：VFIO 是一个用户空间驱动程序开发框架，它允许用户空间应用程序直接与设备交互（绕过内核）
- Virtio-pmd 驱动程序：是一个 DPDK 驱动程序，建立在 Poll Mode Driver 抽象之上，实现了 virtio 协议
- IOMMU 驱动程序：IOMMU 驱动程序管理虚拟 IOMMU（I/O 内存管理单元），这是一个模拟设备，为支持 DMA 的设备执行 I/O 地址映射

让我们一一进行，更详细地描述这些组件

## **VFIO**

VFIO 代表虚拟功能 I/O。但是，vfio-pci 内核驱动程序的维护者 Alex Williamson 建议将其称为“用于用户空间 I/O 的多功能框架”，这可能更准确。 VFIO 基本上是一个用于构建用户空间驱动程序的框架，它提供：

- 设备配置和 I/O 内存区域到用户内存的映射
- 基于 IOMMU 组的 DMA 和中断重映射和隔离。我们将在本文后面深入探讨 IOMMU 是什么以及它是如何工作的。现在，假设它允许创建映射到物理内存的虚拟 I/O 内存空间（类似于普通 MMU 如何映射非 IO 虚拟内存），因此当设备想要 DMA 到虚拟 I/O 时地址，IOMMU 将重新映射该地址并可能应用隔离和其他安全策略。
- 基于 Eventfd 和 irqfd（还记得那些吗？）的信号机制来支持来自和到用户空间应用程序的事件和中断。

引用[内核文档](https://www.kernel.org/doc/Documentation/vfio.txt)：“如果您想在 VFIO 之前编写驱动程序，您必须经历完整的开发周期才能成为合适的上游驱动程序，在树外维护或使用没有IOMMU 保护的概念、有限的中断支持以及需要 root 权限才能访问 PCI 配置空间等内容。”

VFIO 公开了一个用户友好的 API，用于创建字符设备（在 /dev/vfio/ 中），该 API 支持用于描述设备的 ioctl 调用、I/O 区域及其在设备描述符上的读/写/mmap 偏移量以及用于描述和注册中断通知。 **Virtio-pmd** DPDK 提供了一个称为轮询模式驱动程序 (PMD) 的驱动程序抽象，它位于设备驱动程序和用户应用程序之间。它为用户应用程序提供了很大的灵活性，同时保持了可扩展性，即为新设备实现驱动程序的能力。 下面列出了它的一些更显着的功能：

- 一组 API 允许特定驱动程序实现特定于设备的接收和传输功能。
- 每个端口和每个队列的硬件卸载可以静态和动态配置
- 一个用于统计的可扩展 API 可用，允许驱动程序定义自己的特定于驱动程序的统计信息和应用程序来探测可用的统计信息并检索它们。

virtio 轮询模式驱动程序 (virtio-pmd) 是使用 PMD API 的众多驱动程序之一，并为使用 DPDK 编写的应用程序提供对 virtio 设备的快速无锁访问，使用 virtio 的 virtqueues 提供数据包接收和传输的基本功能. 除了任何 PMD 具有的所有功能外，virtio-pmd 驱动程序实现还支持：

- 接收时每包灵活的可合并缓冲区，发送时每包分散缓冲区
- 多播和混杂模式
- MAC/VLAN过滤

结果是一个高性能的用户空间 virtio 驱动程序，它允许任何 DPDK 应用程序无缝地使用 virtio 的标准接口。 **介绍 IOMMU** IOMMU 几乎相当于 I/O 空间的 MMU（设备直接使用 DMA 访问内存）。它位于主内存和设备之间，为每个设备创建一个虚拟 I/O 空间，并提供一种机制将虚拟 I/O 内存动态映射到物理内存。因此，当驱动程序配置设备的 DMA（例如，网卡）时，它会配置虚拟地址，当设备尝试访问它们时，它们会被 IOMMU 重新映射。 引入 IOMMU 它提供了许多优点，例如：

- 可以分配大的连续虚拟内存区域，而无需在物理内存中连续。
- 有些设备不支持足够长的地址来访问整个物理内存，IOMMU 解决了这个问题。
- 内存受到保护，免受恶意我们的故障设备执行的 [DMA 攻击](https://en.wikipedia.org/wiki/DMA_attack)，这些设备试图访问未明确分配给它们的内存。设备只“看到”虚拟地址空间，运行的操作系统专门控制 IOMMU 的映射。
- 在某些架构中，支持中断重映射，这可以允许中断隔离和迁移。

像往常一样，一切都是有代价的，在 IOMMU 的情况下，缺点是：

- 与页面翻译服务相关的性能下降
- 添加的页转换表会消耗物理内存

最后，IOMMU 提供 [PCIe 地址转换服务接口](https://composter.com.ua/documents/ats_r1.1_26Jan09.pdf)，设备可以使用该接口在其内部设备表后备缓冲区 (TLB) 中查询和缓存地址转换。

**vIOMMU - guest的IOMMU** 当然，如果有物理 IOMMU（例如 Intel VT-d 和 AMD-VI），那么 qemu 内部就有一个虚拟 IOMMU。 具体来说，QEMU 的 vIOMMU 有以下特点

- 它将guest I/O 虚拟地址 (IOVA) 转换为访客物理地址 (GPA)，后者可以通过 QEMU 的内存管理系统转换为 QEMU 的主机虚拟地址 (HVA)。
- 执行设备隔离。
- 实现 I/O TLB API，以便可以从 qemu 外部查询映射。

因此，为了让虚拟设备与虚拟 IOMMU 一起工作，我们必须：

1.使用可用的 API 之一创建必要的 IOVA 映射到 vIOMMU。 目前这些 API 是：

- [内核驱动程序的内核 DMA API](https://github.com/torvalds/linux/blob/master/Documentation/DMA-API-HOWTO.txt)
- 用于用户空间驱动程序的 VFIO
- 使用虚拟 I/O 地址配置设备的 DMA

**vIOMMU and DPDK integration** 虚拟 IOMMU 和任何用户空间网络应用程序之间的集成通常是通过 VFIO 驱动程序完成的。 正如我们已经提到的，该驱动程序将执行设备隔离并自动将 iova-to-gpa 映射添加到 IOMMU。 除了支持 VFIO 设置网络设备外，使用 DPDK 对 IOMMU 来说还有一个非常重要的好处。 多亏了 DPDK 使用的内存管理机制，它分配了一个静态内存池，然后用于存储数据包缓冲区和虚拟队列，设备 TLB 同步消息的数量急剧下降，与它们相关的性能损失也是如此。 大页面的使用进一步有助于优化 TLB 查找，因为较少数量的内存页面可以覆盖相同数量的内存。

**vIOMMU and vhost-user 集成** 当在 QEMU 中模拟的设备尝试通过 DMA 访问guest的 virtio I/O 空间时，它将使用 vIOMMU TLB 查找页面映射并执行安全的 DMA 访问。问题是，如果实际 DMA 被卸载到外部进程（例如使用 vhost-user 库的 DPDK 应用程序）会发生什么？ 当 vhost-user 库尝试直接访问共享内存时，它必须将所有地址（I/O 虚拟地址）转换为自己的内存。它通过设备 TLB API 请求 QEMU 的 vIOMMU 进行转换来实现这一点。 Vhost-user 库（以及 vhost-kernel 驱动程序）使用 [PCIe 的地址转换服务标准消息集](https://composter.com.ua/documents/ats_r1.1_26Jan09.pdf)，使用在配置 IOMMU 支持时创建的辅助通信通道（另一个 unix 套接字）请求将页面转换为 QEMU。 总的来说，必须进行 3 个地址转换：

1. QEMU 的 vIOMMU 将 IOVA（I/O 虚拟地址）转换为 GPA（访客物理地址）。
2. Qemu 的内存管理将 GPA（访客物理地址）转换为 HVA（qemu 进程地址空间内的主机虚拟地址）。
3. Vhost-user 库将（QEMU 的）HVA 转换为它自己的 HVA。这通常就像在 vhost-user 库映射 QEMU 内存时将 QEMU 的 HVA 添加到 [mmap(2)](http://man7.org/linux/man-pages/man2/mmap.2.html) 返回的地址一样简单。

显然，所有这些转换都可能对性能产生重要影响，尤其是在使用动态映射的情况下。然而，静态的大页分配（这正是 DPDK 所做的）可以最大限度地减少这种性能损失。 下图增强了之前的 vhost-user 架构以包含 IOMMU 组件： 包含 IOMMU 组件的 Vhost-user 架构

![img](https://picx.zhimg.com/80/v2-e183fea1e94642d66ad080c63a3dcd47_720w.png?source=d16d100b)



编辑切换为居中

添加图片注释，不超过 140 字（可选）

关于这个相当复杂的图表，有几点需要提及：

- guest的物理内存空间是guest认为是物理的内存，但显然，它位于 QEMU 的进程（主机）虚拟地址内。分配 virtqueue 内存区域后，它最终会位于guest物理内存空间的某个位置。
- 当 I/O 虚拟地址被分配到包含 virtqueue 的内存范围时，一个条目及其相关的客户物理地址 (GPA) 被添加到 vIOMMU 的 TLB 表中。
- 另一方面，QEMU 的内存管理系统知道guest物理内存空间驻留在它自己的内存空间中的哪个位置。因此，它能够将客户物理地址转换为主机（QEMU）虚拟地址。
- 当 vhost-user 库尝试访问它没有翻译的 IOVA 时，它会通过辅助 unix 套接字发送 IOTLB 未命中消息。
- IOTLB API 接收请求并查找地址，首先将 IOVA 转换为 GPA，最后将 GPA 转换为 HVA。然后它通过主 unix 套接字将转换后的地址发送回 vhost-user 库。
- 最后，vhost-user 库必须进行最终翻译。由于它已经将 qemu 的内存映射到自己的内存中，因此它必须将 qemu 的 HVA 转换为自己的 HVA 并访问共享内存。

## **把所有东西放在一起**

在这篇文章中，我们介绍了许多组件，包括 DPDK、virtio-pmd、VFIO、IOMMU 等…… 下图显示了这些构建块如何协同工作以实现 vhost-user/virtio-pmd 架构：

![img](https://picx.zhimg.com/80/v2-172951c8ce6beb861e20fff2fdbc7929_720w.png?source=d16d100b)



编辑切换为居中

添加图片注释，不超过 140 字（可选）

这张图有几点需要说明：

- 将此图与前一个图进行比较，我们添加了使用硬件 IOMMU、VFIO 和供应商特定的 PMD 驱动程序将主机的 OVS-DPDK 应用程序连接到物理网卡所需的组件。 到目前为止，这不应该引起任何注意，因为它等同于为guest所做的事情。

## **示例流程**

以下流程图展示了设置 virtio 高性能数据平面所需的步骤：

![img](https://pic1.zhimg.com/80/v2-77cc357626949a581665e3febfa279a2_720w.png?source=d16d100b)



编辑切换为居中

添加图片注释，不超过 140 字（可选）

### **控制面**

以下是设置控制平面的必要步骤：

1. 当主机 (OvS) 中的 DPDK 应用程序启动时，它会创建一个用于（在服务器模式下）与 qemu 进行 virtio 相关协商的套接字。
2. 当 qemu 启动时，它连接到主套接字，如果 vhost-user 提供 VHOST_USER_PROTOCOL_F_SLAVE_REQ 功能，它会创建第二个套接字并将其传递给 vhost-user 以连接到并发送 iotlb 同步消息。
3. 当 QEMU <-> vhost-library 协商结束时，它们之间共享两个套接字。一个用于 virtio 配置，另一个用于 iotlb 消息交换。
4. guest启动并且 vfio 驱动程序绑定到 PCI 设备。它创建对 iommu 组的访问（取决于硬件拓
5. 当 dpdk 应用程序在guest中启动时，它执行以下初始化步骤：
6. 初始化 PCI-vfio 设备。此外，vfio 驱动程序将 PCI 配置空间映射到用户内存。
7. virtqueues 被分配。
8. 使用vfio，执行virtqueue内存空间的DMA映射请求，通过IOMMU内核驱动将dma映射添加到vIOMMU设备中。
9. 然后，进行 virtio 功能协商。在这种情况下，用作 virtqueue 基地址的地址是 IOVA（在 I/O 虚拟地址空间中）。设置 eventfd 和 irqfd 的映 射，以便中断和通知可以直接在来宾和 vhost-user 库之间路由，而无需 QEMU 的干预。
10. 最后，dpdk 应用程序为网络缓冲区分配一大块连续内存。此内存区域的映射也通过 VFIO 和 IOMMU 驱动程序添加到 vIOMMU。

此时配置已完成，数据平面（virtqueues 和通知机制）已准备好使用。 数据平面

为了传输数据包，需要执行以下步骤:

1. guest的 DPDK 应用程序命令 virtio-pmd 发送数据包。它写入缓冲区并将其相应的描述符添加到可用的描述符环中。
2. 主机中的 vhost-user PMD 正在轮询 virtqueue，因此它会立即检测到新的描述符可用并开始处理它们。
3. 对于每个描述符，vhost-user PMD 映射其缓冲区（即：将其 IOVA 转换为 HVA）。在极少数情况下，缓冲内存是尚未映射到 vhost-user 的 IOTLB 中的页面，将向 QEMU 发送请求。然而，来宾中的 DPDK 应用程序分配静态大页面这一事实使对 QEMU 的 IOTLB 请求保持在最低限度。
4. vhost-user PMD 将缓冲区复制到 mbufs（DPDK 应用程序使用的消息缓冲区）。
5. 描述符被添加到使用的描述符环中。这些会被来宾中的 DPDK 应用程序立即检测到，该应用程序也在轮询 virtqueue。
6. mbuf 由主机中的 DPDK 应用程序处理。

## **总结和结论**

DPDK 是一项很有前途的技术，因其为用户领域的高性能网络带来的好处而受到广泛关注。这项技术与 Open vSwitch 相结合，不仅可以提供现代虚拟环境所需的灵活性和性能，而且还将在 NFV 部署中发挥关键作用。 为了充分利用这项技术，作为数据中心交换数据路径和guest中 NFV 应用程序的推动者，有必要在主机和来宾之间安全地创建高效的数据路径。这就是 virtio-net 技术发挥作用的地方。 Vhost-user 提供了一种可靠且安全的机制来将网络处理卸载到基于 DPDK 的应用程序。它与 vIOMMU 集成以提供隔离和内存保护，同时将 QEMU 从处理所有数据包的繁重工作中解放出来。 在来宾中，DPDK (virtio-pmd) 中 virtio 规范的实现能够在来宾中创建快速数据路径，利用高效的内存管理和 DPDK 轮询模式驱动程序提供的高速。 如果您想了解有关 virtio 技术、vhost-user 和 DPDK 的更多信息，并且不害怕弄脏自己的手，请不要错过我们关于 vhost-user 的动手操作帖子！