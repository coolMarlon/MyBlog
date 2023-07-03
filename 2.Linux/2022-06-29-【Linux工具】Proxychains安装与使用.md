# 2022-06-29-【Linux工具】Proxychains安装与使用

> 原文地址 https://cloud.tencent.com/developer/article/1157554

## Ubuntu安装与配置

Proxychains 是 Linux 上一款全局代理工具，通过 Hook Socket 函数实现透明代理，这和 Windows 上的 Proxifier 有点类似。 在 Ubuntu 上安装 Proxychains 的方法是：

> apt-get install proxychains

安装的是 3.1 版本，配置文件的路径是：/etc/proxychains.conf，内容如下：

```
# proxychains.conf  VER 3.1
#
#        HTTP, SOCKS4, SOCKS5 tunneling proxifier with DNS.
#

# The option below identifies how the ProxyList is treated.
# only one option should be uncommented at time,
# otherwise the last appearing option will be accepted
#
#dynamic_chain
#
# Dynamic - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped)
# otherwise EINTR is returned to the app
#
strict_chain
#
# Strict - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# all proxies must be online to play in chain
# otherwise EINTR is returned to the app
#
#random_chain
#
# Random - Each connection will be done via random proxy
# (or proxy chain, see  chain_len) from the list.
# this option is good to test your IDS :)

# Make sense only if random_chain
#chain_len = 2

# Quiet mode (no output from library)
#quiet_mode

# Proxy DNS requests - no leak for DNS data
proxy_dns 

# Some timeouts in milliseconds
tcp_read_time_out 15000
tcp_connect_time_out 8000

# ProxyList format
#       type  host  port [user pass]
#       (values separated by 'tab' or 'blank')
#
#
#        Examples:
#
#               socks5  192.168.67.78   1080    lamer   secret
#               http    192.168.89.3    8080    justu   hidden
#               socks4  192.168.1.49    1080
#               http    192.168.39.93   8080
#
#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4         127.0.0.1 9050
```

Proxychains 支持 HTTP（HTTP-Connect）、SOCKS4 和 SOCKS5 三种类型的代理，需要注意的是：配置代理服务器只能使用 ip 地址，不能使用域名，否则会连不上。

Proxychains 支持 3 种模式：

1. 动态模式 按照配置的代理顺序连接，不存活的代理服务器会被跳过
2. 严格模式 按照配置的代理顺序连接，必须保证所有代理服务器都是存活的，否则会连接失败
3. 随机模式 随机选择一台代理服务器连接，也可以使用代理链

如果不需要代理 DNS 的话，可以注释掉 proxy_dns 这行。

使用的时候在命令行前加上 proxychains 即可。

> root@ubuntu-pc:~# proxychains telnet www.baidu.com 80 ProxyChains-3.1 ([http://proxychains.sf.net](http://proxychains.sf.net/)) Trying 14.215.177.37… |R-chain|-<>-10.0.0.10:8080-<><>-14.215.177.37:80-<><>-OK Connected to www.a.shifen.com. Escape character is ‘^]’.

proxychains 命令其实是个脚本文件，内容如下：

```
#!/bin/sh
echo "ProxyChains-3.1 (http://proxychains.sf.net)"
if [ $# = 0 ] ; then
        echo "  usage:"
        echo "          proxychains  [args]"
        exit
fi
export LD_PRELOAD=libproxychains.so.3
exec "$@"
```

它的目的是设置 LD_PRELOAD 环境变量，以便创建的新进程会加载 libproxychains.so.3，这个 so 的作用是 Hook Socket 函数。因此，也可以在当前 shell 中执行：

```
export LD_PRELOAD=libproxychains.so.3
```

这样之后执行的命令都会使用代理访问。

不过这个版本有个问题，配置代理后所有的连接都会走代理，包括对回环地址的访问。这并不是我们所期望的，幸好有个版本提供了解决方案。

> git clone https://github.com/rofl0r/proxychains cd proxychains ./configure make make install

安装后在配置文件中加入：

> localnet 127.0.0.0/255.0.0.0

安装后的命令是 proxychains4，因此可以和旧版本命令并存。这样对于回环地址就可以绕过代理，使用直连了。

相对于 Proxifier 而言，这种方式还是弱了一点，毕竟有时候我们还是需要根据不同的情况使用不同的代理服务器。