一些概念：

IA：（Inter Architecture）

Netmap：一个高性能网络I/O 框架

NAPI:Linux 内核的一个机制，为了避免大量数据到来时频繁的中断开销。

**硬件加速器**

- ASIC(Application-Specific Intergrated Circuit) 
- 优点：
- 缺点：扩展性不够，开发周期长

- FPGA 
- 优点：使用灵活
- 缺点：
- 全可编程FPGA

**网络处理器(NPU)**

优点：高性能、高可编程性

缺点：开发人员的成长和生态系统的构建比较困难

**多核处理器**

**传统网络设备数据包处理动作：**

**DPDK的一些技术：**

1. 轮训
2. 用户态驱动
3. 亲和性与独占：特定任务可以被指定在某个核上工作，避免线程在不同核间频繁切换导致的cache miss 和cache write back。
4. 降低访存开销
5. 软件调优：cache line对齐、预取数据、再多核间访问避免跨cache line共享、多元数据批量操作。
6. IA新硬件技术：Intel DDIO 技术、指令
7. 充分挖掘网卡的潜能：分流（RSS、FDIR）和卸载（Chksum，TSO）

**整体框架**

![img](https://picx.zhimg.com/80/v2-840bdff6fe29b6325b0774de4107bc71_720w.png?source=d16d100b)



编辑切换为居中

添加图片注释，不超过 140 字（可选）

DPDK 主要模块分解

还有一些其他特性：

- 运行时频率调整(POWER)
- 与Linux kernel stack 建立快速通道的KNI(Kernel Network Interface)
- Packet Framework
- ❓DISTRIB

**性能优化的天花板**

- 线速
- I/O总线最大带宽
- CPU从cache里加载/存储 cache line
- 内存读写带宽

**数据包处理能力**

越小的数据包，挑战越大。

## 实例

### helloword

[5. Hello World Sample Application](https://doc.dpdk.org/guides/sample_app_ug/hello_world.html)

一个简单的例子

```
/* Launch a function on lcore. 8< */
static int
lcore_hello(__rte_unused void *arg)
{
	unsigned lcore_id;
	lcore_id = rte_lcore_id();
	printf("hello from core %u\\n", lcore_id);
	return 0;
}
/* >8 End of launching function on lcore. */

/* Initialization of Environment Abstraction Layer (EAL). 8< */
int
main(int argc, char **argv)
{
	int ret;
	unsigned lcore_id;

	ret = rte_eal_init(argc, argv);
	if (ret < 0)
		rte_panic("Cannot init EAL\\n");
	/* >8 End of initialization of Environment Abstraction Layer */

	/* Launches the function on each lcore. 8< */
	RTE_LCORE_FOREACH_WORKER(lcore_id) {
		/* Simpler equivalent. 8< */
		rte_eal_remote_launch(lcore_hello, NULL, lcore_id);
		/* >8 End of simpler equivalent. */
	}

	/* call it on main lcore too */
	lcore_hello(NULL);
	/* >8 End of launching the function on each lcore. */

	rte_eal_mp_wait_lcore();

	/* clean up the EAL */
	rte_eal_cleanup();

	return 0;
}
```

examples/helloword/main.c

rte：runtime environment

eal：environment abstraion layer

rte_eal_init

- 配置初始化
- 内存初始化
- 内存池初始化
- 队列初始化
- 告警初始化
- ...

### Skeleton

[6. Basic Forwarding Sample Application](https://doc.dpdk.org/guides/sample_app_ug/skeleton.html?highlight=skeleton)

用于平台的单核报文出入性能测试。

```
int
main(int argc, char *argv[])
{
	struct rte_mempool *mbuf_pool;
	unsigned nb_ports;
	uint16_t portid;

	/* Initializion the Environment Abstraction Layer (EAL). 8< */
	int ret = **rte_eal_init**(argc, argv);
	if (ret < 0)
		rte_exit(EXIT_FAILURE, "Error with EAL initialization\\n");
	/* >8 End of initializion the Environment Abstraction Layer (EAL). */

	argc -= ret;
	argv += ret;

	/* Check that there is an even number of ports to send/receive on. */
	nb_ports = rte_eth_dev_count_avail();
	if (nb_ports < 2 || (nb_ports & 1))
		rte_exit(EXIT_FAILURE, "Error: number of ports must be even\\n");

	/* Creates a new mempool in memory to hold the mbufs. */

	/* Allocates mempool to hold the mbufs. 8< */
	mbuf_pool = rte_pktmbuf_pool_create("MBUF_POOL", NUM_MBUFS * nb_ports,
		MBUF_CACHE_SIZE, 0, RTE_MBUF_DEFAULT_BUF_SIZE, rte_socket_id());
	/* >8 End of allocating mempool to hold mbuf. */

	if (mbuf_pool == NULL)
		rte_exit(EXIT_FAILURE, "Cannot create mbuf pool\\n");

	/* Initializing all ports. 8< */
	RTE_ETH_FOREACH_DEV(portid)
		if (port_init(portid, mbuf_pool) != 0)
			rte_exit(EXIT_FAILURE, "Cannot init port %"PRIu16 "\\n",
					portid);
	/* >8 End of initializing all ports. */

	if (rte_lcore_count() > 1)
		printf("\\nWARNING: Too many lcores enabled. Only 1 used.\\n");

	/* Call lcore_main on the main core only. Called on single lcore. 8< */
	lcore_main();
	/* >8 End of called on single lcore. */

	/* clean up the EAL */
	rte_eal_cleanup();

	return 0;
}
```

### L3fwd

发布DPDK性能测试的例子。

https://doc.dpdk.org/guides/sample_app_ug/l3_forward.html

```
// mian函数处理逻辑
初始化运行环境: rte_eal_init(argc, argv);
分析入参: parse_args(argc, argv)
初始化lcore与port配置
端口与队列初始化，类似Skeleton处理
端口启动，使能混杂模式
启动从线程，令其运行main_loop()

// 从线程执行main_loop()的主要步骤如下：
读取自己的lcore信息完成配置；
读取关联的接收与发送队列信息；
进入循环处理：
{
向指定队列批量发送报文；
从指定队列批量接收报文；
批量转发接收到报文；
}
```