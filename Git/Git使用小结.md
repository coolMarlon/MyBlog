# Git使用小结

合并commit 点，保留一个pick，剩下的填f

```bash
# 此处的3表示合并前3次提交
git rebase -i HEAD~3
```

**配置代理：**

```bash
git config --global http.proxy 'socks5://127.0.0.1:7890'
git config --global https.proxy 'socks5://127.0.0.1:7890'

git config --global --unset http.proxy
git config --global --unset https.proxy
```

