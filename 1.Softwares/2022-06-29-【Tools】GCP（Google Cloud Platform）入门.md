# 2022-06-29-GCP（Google Cloud Platform）入门

**原文地址：** https://zhuanlan.zhihu.com/p/40983101



好久没更专栏了，都快荒废了，这段时间发生了很多事，往事不提，开始更正文：

众所周知，GCP 的中国台湾节点对大陆十分友好，但小白在新建、配置实例的时候会碰到很多困难，我在第一次使用的时候，也遭遇不少坑，有许多设定是极其反人类的。

下面分享一些经验，给后面的同学做参考。

阅读本文的前提：

1. 你已经有了一个 Google 账号；

\2. 能够正常访问 [http://google.com](http://link.zhihu.com/?target=http%3A//google.com) 域名的爱国工具；

\3. 已注册 GCP；

\4. 已绑定信用卡并获得新注册用户 300 美金的赠送余额。

## **一、创建实例**

登陆 GCP 平台的控制面板：

[https://console.cloud.google.comconsole.cloud.google.com](http://link.zhihu.com/?target=https%3A//console.cloud.google.com/)![img](https://pic3.zhimg.com/v2-9f0c48fe918672135549c76c4358a6d2_b.jpg)![img](https://pic3.zhimg.com/v2-9f0c48fe918672135549c76c4358a6d2_r.jpg)

打开侧边栏，找到「Compute Engine」，在弹出的二级列表中，单击「VM 实例」：

![img](https://pic3.zhimg.com/v2-4b46ef426acafda5a8a416c35c626772_b.jpg)![img](https://pic3.zhimg.com/v2-4b46ef426acafda5a8a416c35c626772_r.jpg)

GCP 平台的实例，都是依附于对应的项目下的，所以你需要先新建一个项目，项目名 ID 保持默认，项目名称可按照自己的喜好自定，单击「创建」即可：

![img](https://pic4.zhimg.com/v2-2c26ede2d7d30e3d12e561acf42e354f_b.jpg)![img](https://pic4.zhimg.com/80/v2-2c26ede2d7d30e3d12e561acf42e354f_hd.jpg)![img](https://pic2.zhimg.com/v2-11ba91f73273fab97550d9bd495b1e0d_b.jpg)![img](https://pic2.zhimg.com/80/v2-11ba91f73273fab97550d9bd495b1e0d_hd.jpg)

首次创建项目，需要一段时间，请耐心等待：

![img](https://pic4.zhimg.com/v2-392f15957c5f3b1be4e803282e23d42f_b.jpg)![img](https://pic4.zhimg.com/80/v2-392f15957c5f3b1be4e803282e23d42f_hd.jpg)

继续等，如果时间过长，请刷新浏览器。

当「创建」按钮由灰蓝变蓝，就可以创建实例了：

![img](https://pic2.zhimg.com/v2-6e8e9173425abb1cae888cbe46103df5_b.jpg)![img](https://pic2.zhimg.com/80/v2-6e8e9173425abb1cae888cbe46103df5_hd.jpg)

挨个讲一下创建实例时的选项：

\1. 名称自定，使用默认的也行。

\2. 区域选择「asia-east1(中国台湾)」，地区都是中国台湾彰化县，有 a、b、c 可选，我选的是 c，据说网络质量更好一些。

\3. 机器类型选择「微型」，租用机器本身按时付费，月计 5，流量费0.23，流量费0.23/GB，不包含在机器价格内，额外计算，也是用多少算多少，挺金贵的。

![img](https://pic2.zhimg.com/v2-8fbcba90a178e4264de21dabbb35edad_b.jpg)![img](https://pic2.zhimg.com/80/v2-8fbcba90a178e4264de21dabbb35edad_hd.jpg)

\4. 「容器」功能相当于链接一个远程镜像，当做系统安装盘，用 GCP 自带的镜像就好了，都是最新版的系统，不用选。

![img](https://pic4.zhimg.com/v2-fd62a95436a3776bdd2bed6cf4359c97_b.jpg)![img](https://pic4.zhimg.com/80/v2-fd62a95436a3776bdd2bed6cf4359c97_hd.jpg)

\5. 点击「启动磁盘」的「更改」按钮，可以在 GCP 自带的系统安装镜像里选一个，我习惯用 Debian 系 Linux，选择 Debian 9 或是 Ubuntu 16.04 都是可以的。默认磁盘容量 10GB，富强上网足够，选择更大容量的磁盘需额外计费。

![img](https://pic4.zhimg.com/v2-65e2898357a7d43f20cec0874c1f8b87_b.jpg)![img](https://pic4.zhimg.com/80/v2-65e2898357a7d43f20cec0874c1f8b87_hd.jpg)

\6. 「身份和默认 API 权限」及关联选项保持默认

![img](https://pic1.zhimg.com/v2-bcdadb14712068620d6f0cb1c3ad24f8_b.jpg)![img](https://pic1.zhimg.com/80/v2-bcdadb14712068620d6f0cb1c3ad24f8_hd.jpg)

\7. 「防火墙」选项下的 http 和 https 必须开启，否则你将无法通过这两个协议访问到机器，比如在 GCP 实例上建设网站，访客无法访问

![img](https://pic3.zhimg.com/v2-6d9b037cc2304de9173d38a3e7c1f4d6_b.jpg)![img](https://pic3.zhimg.com/80/v2-6d9b037cc2304de9173d38a3e7c1f4d6_hd.jpg)

\8. 不用单独展开「管理、安全、磁盘、网络、单独租用」做额外设置，点击「创建」即可新建实例：

![img](https://pic3.zhimg.com/v2-c8b506e7d2341acea881783069d2250a_b.jpg)![img](https://pic3.zhimg.com/80/v2-c8b506e7d2341acea881783069d2250a_hd.jpg)![img](https://pic1.zhimg.com/v2-386f0a3bc71b5dc36f251ed3dd03857c_b.jpg)![img](https://pic1.zhimg.com/80/v2-386f0a3bc71b5dc36f251ed3dd03857c_hd.jpg)

新创建的实例如下：

![img](https://pic2.zhimg.com/v2-e1319729c2e7bfe557c61277ece81169_b.jpg)![img](https://pic2.zhimg.com/80/v2-e1319729c2e7bfe557c61277ece81169_hd.jpg)

## **二、配置实例（用户管理、网络设置）**

下面我们可以在浏览器里远程连接服务器，在命令行里直接设置 root 用户密码，使用 root 用户登录等。

在「连接」功能的下拉菜单里，点击「在浏览器窗口中打开」：

![img](https://pic2.zhimg.com/v2-4d95399e03c4a11bd34e490f12316971_b.jpg)![img](https://pic2.zhimg.com/80/v2-4d95399e03c4a11bd34e490f12316971_hd.jpg)![img](https://pic3.zhimg.com/v2-f2aba90e5356dc8ebfd1cc841b85a29a_b.jpg)![img](https://pic3.zhimg.com/80/v2-f2aba90e5356dc8ebfd1cc841b85a29a_hd.jpg)

稍后出现命令行操作界面：

![img](https://pic3.zhimg.com/v2-94cef0fd8ad6b70e23858f1b088eb8b6_b.jpg)![img](https://pic3.zhimg.com/80/v2-94cef0fd8ad6b70e23858f1b088eb8b6_hd.jpg)

输入：

```undefined
sudo passwd root
```

为 root 用户设置密码。

建议在[在线随机密码生成器](http://link.zhihu.com/?target=https%3A//suijimimashengcheng.51240.com/) 生成一个包含大小写字母、特殊符号、数字的 30 位以上随机密码，将密码复制下来粘贴（Ctrl+C）到命令行中，按回车，再输入一次密码，按回车，使密码生效：

![img](https://pic3.zhimg.com/v2-83fa5b4eea2ccdd5617e436aff2b0882_b.jpg)![img](https://pic3.zhimg.com/80/v2-83fa5b4eea2ccdd5617e436aff2b0882_hd.jpg)

使当前用户具备 root 权限：

```undefined
su root
```

再次输入密码，授权成功：

![img](https://pic4.zhimg.com/v2-907787bef9cd9c50a6ec8deef613ca03_b.jpg)![img](https://pic4.zhimg.com/80/v2-907787bef9cd9c50a6ec8deef613ca03_hd.jpg)

编辑 ssh 服务配置文件：

```bash
vim /etc/ssh/sshd_config
```

找到这一行：

```undefined
PermitRootLogin prohibit-password
```

![img](https://pic3.zhimg.com/v2-c13fa49fb8eceaa1651dfa23f08973c2_b.jpg)![img](https://pic3.zhimg.com/80/v2-c13fa49fb8eceaa1651dfa23f08973c2_hd.jpg)

将「**prohibit-password**」改成「**yes**」，使允许 root 用户登录：

![img](https://pic4.zhimg.com/v2-5aa59b4824a9a5922057c6f6d7f136df_b.jpg)![img](https://pic4.zhimg.com/80/v2-5aa59b4824a9a5922057c6f6d7f136df_hd.jpg)

并且将「PasswordAuthentication」从默认的「no」改成「yes」：

![img](https://pic2.zhimg.com/v2-910b2a87c37ca022864cb1b67942faf1_b.jpg)![img](https://pic2.zhimg.com/80/v2-910b2a87c37ca022864cb1b67942faf1_hd.jpg)

最后是这样的：

![img](https://pic2.zhimg.com/v2-b1e59eb739d3e3e55ffcd6f31b7d47dd_b.jpg)![img](https://pic2.zhimg.com/80/v2-b1e59eb739d3e3e55ffcd6f31b7d47dd_hd.jpg)

顺便把 ssh 端口号（Port）改成高端口（范围 10000~65535），避免密码在默认端口被暴力破解：

![img](https://pic2.zhimg.com/v2-7182523b93beccfd535b4306d5b637f5_b.jpg)![img](https://pic2.zhimg.com/80/v2-7182523b93beccfd535b4306d5b637f5_hd.jpg)

按 Esc，输入：

```ruby
:wq
```

确认保存并退出。

重新启动 ssh 服务，使更改生效：

```bash
/etc/init.d/ssh restart
```

![img](https://pic4.zhimg.com/v2-fcdca6d260715533d73391a114f48c7f_b.jpg)![img](https://pic4.zhimg.com/80/v2-fcdca6d260715533d73391a114f48c7f_hd.jpg)

特别地，GCP 配置了自己的防火墙（类似阿里云的安全组规则），默认只允许 22、80、443、3306 等常用端口传入，这就意味着如果我们设置了高端口用本地 ssh 工具连接，以及使用带有动态端口切换功能的敬业代理时，防火墙规则会造成许多麻烦，所以我们要把所有端口的访问都打开。

![img](https://pic3.zhimg.com/v2-2e49ce4c3a3ac5190d553525cdbe83f6_b.jpg)![img](https://pic3.zhimg.com/80/v2-2e49ce4c3a3ac5190d553525cdbe83f6_hd.jpg)![img](https://pic4.zhimg.com/v2-8e74c44a3f1575e2dfdcebc689daaa4f_b.jpg)![img](https://pic4.zhimg.com/80/v2-8e74c44a3f1575e2dfdcebc689daaa4f_hd.jpg)![img](https://pic3.zhimg.com/v2-e462a385bc751c9a38f8a00e612a128a_b.jpg)![img](https://pic3.zhimg.com/80/v2-e462a385bc751c9a38f8a00e612a128a_hd.jpg)

设置非 22 端口用 GCP 自带工具做 ssh 连接，如果未在防火墙里允许放行对应的端口，即使指定对应端口做登陆也会登录失败（疯起来我寄几都咬）。

返回实例管理列表，点击最右边的竖排省略号，在展开的列表点击「查看网络详情」：

![img](https://pic2.zhimg.com/v2-1ec47dcbe71b37e856e7e79b2c7523b9_b.jpg)![img](https://pic2.zhimg.com/80/v2-1ec47dcbe71b37e856e7e79b2c7523b9_hd.jpg)

鼠标移到左侧列表并展开：

![img](https://pic4.zhimg.com/v2-d482703aa3e898847e7b478a2893dd6f_b.jpg)![img](https://pic4.zhimg.com/80/v2-d482703aa3e898847e7b478a2893dd6f_hd.jpg)

点击「防火墙规则」：

![img](https://pic1.zhimg.com/v2-c185de684d33d83d373977a893807034_b.jpg)![img](https://pic1.zhimg.com/80/v2-c185de684d33d83d373977a893807034_hd.jpg)

默认处于「入站」选项卡，点击「创建防火墙规则」：

![img](https://pic2.zhimg.com/v2-4907d2161c7039536239abf291fd3e81_b.jpg)![img](https://pic2.zhimg.com/80/v2-4907d2161c7039536239abf291fd3e81_hd.jpg)

设置好防火墙规则名称后，「流量方向」选择「入站」：

![img](https://pic4.zhimg.com/v2-921e3f25cce968be822b9ff349f8a46b_b.jpg)![img](https://pic4.zhimg.com/80/v2-921e3f25cce968be822b9ff349f8a46b_hd.jpg)

「对匹配项执行的操作」保持默认「允许」。

「目标」选择「网络中的所有实例」，「来源 IP 地址范围」输入「0.0.0.0/0」（允许所有 IP）。

「次要来源过滤条件」保持默认「无」，「协议和端口」选择「全部允许」，最后点击「创建」：

![img](https://pic4.zhimg.com/v2-ec982ca19b9374150e6ccdd27a8cc613_b.jpg)![img](https://pic4.zhimg.com/80/v2-ec982ca19b9374150e6ccdd27a8cc613_hd.jpg)

等待创建完成：

![img](https://pic2.zhimg.com/v2-28e85b1fd51f8e8ce8fedc837bae6289_b.jpg)![img](https://pic2.zhimg.com/80/v2-28e85b1fd51f8e8ce8fedc837bae6289_hd.jpg)

同样给出站规则设置放行所有 IP、协议和端口，因为法治上网的代理需要允许主机去访问目标网站，把获得的流量转发给客户端，方法如法炮制，直接上图：

![img](https://pic1.zhimg.com/v2-a546a90c1b1ee967ed765663207d9dec_b.jpg)![img](https://pic1.zhimg.com/80/v2-a546a90c1b1ee967ed765663207d9dec_hd.jpg)

特别地，「来源过滤条件」除了可以设置允许全部内网机器访问外网，你还可以指定「子网」规则之一，如勾选「10.140.0.0/20」（亚洲东部 1 区，即中国台湾机房），不同地域的实例处于不同的子网，从逻辑上，它们从属于同一 GCP 账户下的同一项目中，彼此之间构成了一个局域网。

![img](https://pic4.zhimg.com/v2-100228b08bfe15c8d2f3a44153382123_b.jpg)![img](https://pic4.zhimg.com/80/v2-100228b08bfe15c8d2f3a44153382123_hd.jpg) 在这里「全选」也是可以的![img](https://pic2.zhimg.com/v2-e0051d6509c5e83ff0b287074ecc48cd_b.jpg)![img](https://pic2.zhimg.com/80/v2-e0051d6509c5e83ff0b287074ecc48cd_hd.jpg) 不同地域的机房拥有不同的子网 IP 地址![img](https://pic4.zhimg.com/v2-c385a4bf652aa649006a2c72af7e2033_b.jpg)![img](https://pic4.zhimg.com/80/v2-c385a4bf652aa649006a2c72af7e2033_hd.jpg)GCP 管理面板首页可直接查看当前实例所在子网的 IP 地址

创建：

![img](https://pic2.zhimg.com/v2-b9a6263e76c05b51d2c02e6aca1293c1_b.jpg)![img](https://pic2.zhimg.com/80/v2-b9a6263e76c05b51d2c02e6aca1293c1_hd.jpg)

至此，所有的设置就完成了。凭服务器 IP、端口、root 用户名、root 用户密码，就可以在 XShell 里连接到机器了。

![img](https://pic4.zhimg.com/v2-afe26c0f5a55db5d623f2d8a0d54db07_b.jpg)![img](https://pic4.zhimg.com/80/v2-afe26c0f5a55db5d623f2d8a0d54db07_hd.jpg)![img](https://pic1.zhimg.com/v2-270bcfc2617c43318cc8baac5d306dbc_b.jpg)![img](https://pic1.zhimg.com/80/v2-270bcfc2617c43318cc8baac5d306dbc_hd.jpg)![img](https://pic3.zhimg.com/v2-439ce93e011e8b1ced77ce0ffade1b2a_b.jpg)![img](https://pic3.zhimg.com/80/v2-439ce93e011e8b1ced77ce0ffade1b2a_hd.jpg)

## **三、意外处理（实例新建后无系统 & 设置新的用户并采用密钥登录）**

**3.1**

在创建实例时，偶尔会发现「启动磁盘」项没有「更改」按钮，也没有标注系统名称，这意味着实例创建后是没有系统的，更无从远程连接，使用后续功能。碰到这种情况，请返回 VM 实例控制面板首页，先将当前磁盘完全为空的机器删除：

![img](https://pic3.zhimg.com/v2-a7665b82b0d2c840945e1eaba77e6c0a_b.jpg)![img](https://pic3.zhimg.com/80/v2-a7665b82b0d2c840945e1eaba77e6c0a_hd.jpg)

接着，将左侧列表展开，点击「映像」：

![img](https://pic1.zhimg.com/v2-d1e8cb0566da286025dad6fb29778cdc_b.jpg)![img](https://pic1.zhimg.com/80/v2-d1e8cb0566da286025dad6fb29778cdc_hd.jpg)

在列表中点击需要安装的系统：

![img](https://pic4.zhimg.com/v2-92eca461fdb7540f4061a957a2f27e07_b.jpg)![img](https://pic4.zhimg.com/80/v2-92eca461fdb7540f4061a957a2f27e07_hd.jpg)

点击「创建实例」：

![img](https://pic3.zhimg.com/v2-ba39aaaebec76f06322662ce9ad30c06_b.jpg)![img](https://pic3.zhimg.com/80/v2-ba39aaaebec76f06322662ce9ad30c06_hd.jpg)

注意事项和第一章「创建实例」的要求相同：

![img](https://pic1.zhimg.com/v2-52c3f1d619d822d8e6fdc38e22e81618_b.jpg)![img](https://pic1.zhimg.com/80/v2-52c3f1d619d822d8e6fdc38e22e81618_hd.jpg)

**3.2**

由于安全方面的考虑，你仍然可以选择自定义用户名，并用对应的密钥登录，你可以先在 Xshell 中生成一个新的密钥文件：

工具（T）→新建用户密钥生成向导（W）：

![img](https://pic4.zhimg.com/v2-0c9afcebdd02e4f73d42d95e3d9ce097_b.jpg)![img](https://pic4.zhimg.com/80/v2-0c9afcebdd02e4f73d42d95e3d9ce097_hd.jpg)

选项默认，下一步：

![img](https://pic3.zhimg.com/v2-3eb0aa5309fae4ff2e21fefc4368ff5e_b.jpg)![img](https://pic3.zhimg.com/80/v2-3eb0aa5309fae4ff2e21fefc4368ff5e_hd.jpg)

继续：

![img](https://pic2.zhimg.com/v2-a80378e07756a4077e1e8423f5136a85_b.jpg)![img](https://pic2.zhimg.com/80/v2-a80378e07756a4077e1e8423f5136a85_hd.jpg)

密钥名称自定，设置什么样的密钥名称，就意味着你以什么样的用户名登录系统（不可设置为 root），用户密钥加密密码属于可选项，可不填，意思就是在这个密钥的外壳又添加一个密码，使用密钥的时候需要输入密码：

![img](https://pic2.zhimg.com/v2-387bae641f2d94a6a3112afb00a7f9e1_b.jpg)![img](https://pic2.zhimg.com/80/v2-387bae641f2d94a6a3112afb00a7f9e1_hd.jpg)

是（Y）：

![img](https://pic3.zhimg.com/v2-5688d87cf457fe61ce684e3a862c8eda_b.jpg)![img](https://pic3.zhimg.com/80/v2-5688d87cf457fe61ce684e3a862c8eda_hd.jpg)

「用户密钥」里，可看到我们刚才创建的，名称叫「exp」的密钥：

![img](https://pic2.zhimg.com/v2-d2e30050ef89f02c9e3fc9e33ba1d3ed_b.jpg)![img](https://pic2.zhimg.com/80/v2-d2e30050ef89f02c9e3fc9e33ba1d3ed_hd.jpg)

接着，选中要导入到 GCP 钥匙串的密钥，点击右边的「属性（P）」，并切换到「公钥」选项卡：

![img](https://pic1.zhimg.com/v2-b48bf7d03864dc13ef2d66069a6d78a4_b.jpg)![img](https://pic1.zhimg.com/80/v2-b48bf7d03864dc13ef2d66069a6d78a4_hd.jpg)

接着，将公钥里的内容（以 ssh-rsa 开头的），将到末尾两个等于号（=）处的内容，复制出来：

![img](https://pic3.zhimg.com/v2-ce848cde0a4f9171d633ddbd57f5b4d6_b.jpg)![img](https://pic3.zhimg.com/80/v2-ce848cde0a4f9171d633ddbd57f5b4d6_hd.jpg)

然后，登陆 GCP 的控制面板，点击侧边栏的「元数据」：

![img](https://pic4.zhimg.com/v2-27a23b251deb4d92f67f556b8db1e497_b.jpg)![img](https://pic4.zhimg.com/80/v2-27a23b251deb4d92f67f556b8db1e497_hd.jpg)

点击第二个选项卡——「SSH 密钥」：

![img](https://pic4.zhimg.com/v2-894623aa4d8f936b2199c6da4200d443_b.jpg)![img](https://pic4.zhimg.com/80/v2-894623aa4d8f936b2199c6da4200d443_hd.jpg)

修改：

![img](https://pic2.zhimg.com/v2-9133f010679d6abf78013948f975c951_b.jpg)![img](https://pic2.zhimg.com/80/v2-9133f010679d6abf78013948f975c951_hd.jpg)

添加一项：

![img](https://pic1.zhimg.com/v2-aae5209cfd19eadfcb5fc73c4644ca8c_b.jpg)![img](https://pic1.zhimg.com/80/v2-aae5209cfd19eadfcb5fc73c4644ca8c_hd.jpg)

把刚才复制来的公钥粘贴进来，此时会提示密钥格式无效：

![img](https://pic4.zhimg.com/v2-2b225236fad73c6d6c024c50ce75c25b_b.jpg)![img](https://pic4.zhimg.com/80/v2-2b225236fad73c6d6c024c50ce75c25b_hd.jpg)

不要着急，在密钥末尾的两个等于号之后，打一个空格，再输入任意字母 + 英文数字，ssh 密钥就创建成功了，同时空格后面输入的内容，就是你登录 GCP 实例的用户名：

![img](https://pic1.zhimg.com/v2-64203d2bccc45cfca6166a3f54b0e458_b.jpg)![img](https://pic1.zhimg.com/80/v2-64203d2bccc45cfca6166a3f54b0e458_hd.jpg)

保存生效：

![img](https://pic4.zhimg.com/v2-86b23cf81ed0d0f36d279a74ed0130a3_b.jpg)![img](https://pic4.zhimg.com/80/v2-86b23cf81ed0d0f36d279a74ed0130a3_hd.jpg)

然后，在 Xshell 里建立新连接：

名称（N）自定，主机（H）填 IP，端口号（O）默认 22，如果在 sshd 文件里更改了，请自行用新的端口号：

![img](https://pic4.zhimg.com/v2-a6d88f69eb963f7314077eee8770cf7b_b.jpg)![img](https://pic4.zhimg.com/80/v2-a6d88f69eb963f7314077eee8770cf7b_hd.jpg)

点击侧边栏的「用户身份验证」，方法（M）选择「PublicKey」，用户名填你刚才在 GCP 后面板→元数据→ssh 密钥里，在「== 」后面设置过的用户名。

![img](https://pic2.zhimg.com/v2-e0dd03c82b501cff8dd1a679e9fbd6dd_b.jpg)![img](https://pic2.zhimg.com/80/v2-e0dd03c82b501cff8dd1a679e9fbd6dd_hd.jpg)

「用户密钥」选择你刚才用 Xshell 密钥生成工具生成的那个，点击下方「确定」保存。

![img](https://pic1.zhimg.com/v2-981a7aef66a3f0efe88a12b26dee61c4_b.jpg)![img](https://pic1.zhimg.com/80/v2-981a7aef66a3f0efe88a12b26dee61c4_hd.jpg)

快捷键「Alt+O」快速启动连接，选中刚才设置好的那台机器，点击连接（C），就可以成功连上 GCP 实例了，用户名就是我们刚才设置的「exp」：

![img](https://pic1.zhimg.com/v2-99c5d485b995f475b3cbdc593fea410c_b.jpg)![img](https://pic1.zhimg.com/80/v2-99c5d485b995f475b3cbdc593fea410c_hd.jpg)![img](https://pic2.zhimg.com/v2-b88914428b02cdb644a1b6593a23f041_b.jpg)![img](https://pic2.zhimg.com/80/v2-b88914428b02cdb644a1b6593a23f041_hd.jpg)
写下你的评论...

还有哪些靠谱加密方式？

密钥 + 任意高端口登录就很安全啦，比密码登录安全，注意密钥文件不要随便暴露给别人

不赞，但是分享到朋友圈。

您的转发是支持我的动力

非常详细，感谢
还有哪些靠谱加密方式？

密钥 + 任意高端口登录就很安全啦，比密码登录安全，注意密钥文件不要随便暴露给别人

抱歉我没说清楚，我说的不是 ssh 登录，我是说科学上网酸酸乳的靠谱加密方式。貌似 aes 加密已经 GG 了。。。
唉，太贵了，最便宜的都要 4.28 刀，还不包括流量费用。流量费用 0.23 刀一个 G。用不起用不起。快到期了，正发愁以后用什么呢。

第一年有送 300 刀给你免费用的呀，刨掉最低配机器钱，分摊每个月折合流量也有 80G 左右可用，你一个人爱国上网用肯定够，先占一年便宜再说呗

已经用了 11 个月了，还剩 1 个月到期。不过找到了 AWS 了，又能蹭一年了

图文非常详细，太赞了，竟然还是个妹纸~

尝试了一下感觉延迟还是比较严重... 但是好像也没有很好的解决方法 orz

协议和端口全部允许。。。看得我很慌

手机党操作: wc 那步后如何保存呢，我只有自己关了，然后重启 SSH 显示失败

就是重启 SSH 使更改生效显示失败，另外 SSR 手机端已经可以上网，但是我的谷歌软件都无法升级，没有速度，不知道怎么破

想知道这东西理论上会不会被 q 掉，比如直接禁止访问台湾的 GCP 服务器