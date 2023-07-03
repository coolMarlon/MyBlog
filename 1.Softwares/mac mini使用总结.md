

# mac mini使用总结

## 一、快捷键

### 2.1 系统快捷键

参考：[MAC快捷键](https://support.apple.com/zh-cn/HT201236)

PS：我的键盘CTRL是CTRL键，Windows键是Command键



| 快捷键               | 描述                                                         | 备注               |
| -------------------- | ------------------------------------------------------------ | ------------------ |
|                      | ALT+ ↑↓                                                      |                    |
| 光标定位到文字开头   | **CTRL+← →**                                                 |                    |
| 光标按照单词左右移动 | ALT+← →                                                      |                    |
| 视图左右切换         | Comand +← →                                                  |                    |
|                      | Comand +↑↓                                                   |                    |
| 中英文切换           | Shift                                                        | 需要安装搜狗输入法 |
| 屏幕角               | 左上：快速回到桌面<br />左下：锁屏<br />右上：打开所有应用程序<br />右下：打开启动台 | 需要自行设置       |
| 切换应用程序         | CTRL+TAB                                                     |                    |
| 跳到行首行尾         | CTRL+A 行首，CTRL+E行尾                                      |                    |
| 打开访达             | CTRL+ALT+空格                                                |                    |
| 关闭当前应用程序     | CTRL+W                                                       |                    |
| 退出                 | CTRL+Q                                                       |                    |
| 区域截屏到剪切板     | CTLR+Command+Shift+4                                         |                    |
| 切换输入法           | Command+空格                                                 |                    |
| 打开隐藏扩展坞       | CTRL+ALT+D                                                   |                    |
| 移动焦点到窗口       | Comand+F4                                                    |                    |



###  2.2 typora快捷键

| 描述     | **快捷键** |
| -------- | ---------- |
| 插入表格 | CTRL+ALT+T |
|          |            |
|          |            |



### 2.3 chrome 快捷键

| 描述         | 快捷键        |
| ------------ | ------------- |
| 定位到地址栏 | CTRL+L        |
| 标签栏切换   | Commnad + TAB |
|              |               |



## 二、终端



### 2.1 终端 brew命令 文件找不到

其实是因为没有配置环境变量

```bash
opt/homebrew/bin
```



### 2.2 mac的zsh不会去执行`/etc/profile`,而且去执行

~/.zshrc与/etc/zshenv与/etc/zshrc



### 2.3 ls命令报错

参考：[Fix Terminal “Operation not permitted” Error in macOS Big Sur, Catalina, Mojave](https://qastack.cn/superuser/310160/what-is-the-keyboard-shortcut-to-focus-to-address-bar-in-mac-chrome)



### 2.4 history 命令显示不全

```bash
# 默认显示15条，如果接数字n的话，表示从第n条历史记录开始显示
history 0
```



### 2.5 终端显示颜色

参考：[macOS 修改终端Terminal的颜色设置](https://blog.csdn.net/u010391437/article/details/75126310)

```bash
export CLICOLOR='Yes'                       # 是否输出颜色
export LSCOLORS='ExGxFxdaCxDaDahbadacec'    # 指定颜色
```





## 三、git 

### 3.1 fatal: unable to access 'https://github.com/coolMarlon/blogs.git/': Failed to connect to github.com port 443: Operation timed out

参考：[知乎:Failed to connect to raw.githubusercontent.com:443](https://zhuanlan.zhihu.com/p/115450863) 

我有代理，所以这个方法就可以了，也有其他方法：如何解决类似 [curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused 的问题](https://github.com/hawtim/blog/issues/10) 

```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```



###  3.2 终端下git status 命令中文无法显示



```bash
git config --global core.quotepath false

# tree 命令也会有乱码
tree -N
```



### 3.3 git 配置公钥，免密上传代码



### 3.4 为git设置代理

参考资料：https://www.jianshu.com/p/648af07cfdf9

```bash
# 走 ss 代理，其中 socks5 的默认本地端口为 1080
$ git config --global http.proxy socks5://127.0.0.1:1080

# 走 http 代理
$ git config --global http.proxy http://<ip>:<port>

取消代理：
$ git config --global --unset http.proxy
```



## 四、brew

```bash
brew cask install upic
brew install node
brew install vscode
```



## 五、Typora

###  5.1 Typora 配置图床

 参考资料：

* [PicGo+GitHub图床，让Markdown飞](https://juejin.cn/post/6844903768782290957)
* [下载安装pic go](https://picgo.github.io/PicGo-Doc/zh/guide/#%E7%89%B9%E8%89%B2%E5%8A%9F%E8%83%BD)
* [Typora+PicGo设置GitHub图床](https://blog.csdn.net/weixin_45965432/article/details/108911937)

![DF](https://raw.githubusercontent.com/coolMarlon/image/main/pic/20210801180111.png)



## 九十九、其他

