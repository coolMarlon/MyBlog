# 【DPDK】如何编译examples（centos环境）

看了一下官网文档，不是很直观，这里简单整理下。

编译依赖：

* 基本的编译工具：gcc,pkg-config,ninjameson setup -Dexamples

  

- Python 3.6 or later.

* meson/pyelftools/ninja

```shell
pip3 install meson ninja
pip3 install pyelftools
```

* 处理NUMA的库：

```
numactl-devel
```



```
export DPDK_DIR=/root/dpdk_19.11
export DPDK_TARGET=x86_64-native-linuxapp-gcc
export DPDK_BUILD=$DPDK_DIR/$DPDK_TARGET

declare -x RTE_SDK="//root/dpdk_19.11"
declare -x RTE_TARGET="x86_64-native-linuxapp-gcc"
export EXTRA_CFLAGS="-O0 -g"
```

**参考资料：**

https://zhuanlan.zhihu.com/p/457169190
