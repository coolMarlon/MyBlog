# [解决PIP下载安装速度慢【Python】 ](https://www.cnblogs.com/shengwang/p/9979764.html)

目录

- [解决PIP下载安装速度慢](https://www.cnblogs.com/shengwang/p/9979764.html#解决pip下载安装速度慢)
- [pip install 安装指定版本的包](https://www.cnblogs.com/shengwang/p/9979764.html#pip-install-安装指定版本的包)



# 解决PIP下载安装速度慢

于Python开发用户来讲，PIP安装软件包是家常便饭。但国外的源下载速度实在太慢，浪费时间。而且经常出现下载后安装出错问题。所以把PIP安装源替换成国内镜像，可以大幅提升下载速度，还可以提高安装成功率。

国内源：
新版ubuntu要求使用https源，要注意。

```bash
清华：https://pypi.tuna.tsinghua.edu.cn/simple

阿里云：http://mirrors.aliyun.com/pypi/simple/

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

华中理工大学：http://pypi.hustunique.com/

山东理工大学：http://pypi.sdutlinux.org/ 

豆瓣：http://pypi.douban.com/simple/
```

临时使用：
可以在使用pip的时候加参数`-i https://pypi.tuna.tsinghua.edu.cn/simple`

例如：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyspider`，这样就会从清华这边的镜像去安装pyspider库。

永久修改，一劳永逸：
Linux下，修改 **~/.pip/pip.conf **(没有就创建一个文件夹及文件。文件夹要加“.”，表示是隐藏文件夹)

内容如下：

> [global]
> index-url = https://pypi.tuna.tsinghua.edu.cn/simple
> [install]
> trusted-host=mirrors.aliyun.com

windows下，直接在user目录中创建一个pip目录，如：**C:\Users\xx\pip**，新建文件**pip.ini**。内容同上。

原网页：http://www.cnblogs.com/microman/p/6107879.html

# pip install 安装指定版本的包

要用 pip 安装指定版本的 Python 包，只需通过 == 操作符 指定

> pip install scrapy==1.3.0
> 将安装scrapy 1.3.0 版本。