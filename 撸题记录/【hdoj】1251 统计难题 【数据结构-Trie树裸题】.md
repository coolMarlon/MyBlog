传送门：[统计难题](http://acm.hdu.edu.cn/showproblem.php?pid=1251)

# 题意：
字典树裸题。

# 分析
字典树板子，但是这题需要注意一点。
关于字典树的只是可以参考[hihocoder hiho一下 第二周](https://hihocoder.com/contest/hiho2/problem/1)

用G++提交会爆内存(Memory Limit Exceeded),用c++提交可以AC。

## G++ 与 C++提交的区别
参考：[OJ中的语言选项里G++ 与 C++的区别](https://blog.csdn.net/FeBr2/article/details/52068357)

C++是一门计算机编程语言，而G++则是C++的编译器。

选择C++意味着你将使用C++最标准的编译方式，也就是ANSI C++编译。

选择G++则意味这你使用GNU项目中适用人群最多的编译器（其实也就是我们熟悉的Code::Blocks 自带的编译器）。

类似的还有选择C和GCC，前者是标准的C编译器，后者则是用GCC来编译。


# My AC Code
```
#include <iostream>
#include<cstdlib>
#include<cstring>

#define rep(i,a,b) for(int i=a;i<=b;i++)
using namespace std;
const int maxn=26;

struct TrieNode
{
	// 当前结点前缀的数量
	int prefix;
	TrieNode* Next[maxn];
};

typedef TrieNode Trie;

// 根不代表任何字母
void insert(Trie *root,char *s)
{
	Trie*  p=root;	
	while(*s!='\0')
	{
		if(p->Next[*s-'a']==NULL)
		{
			Trie* temp=new Trie();
			for(int i=0;i<maxn;i++)
			{
				temp->Next[i]=NULL;
			}
			temp->prefix=1;
			p->Next[*s-'a']=temp;
			p=p->Next[*s-'a'];
		}
		else
		{			
			p=p->Next[*s-'a'];
			p->prefix++;
		}
		s++;
	}
}

int count(Trie *root,char *pre)
{
	Trie* p=root;
	while(*pre!='\0')
	{
		if(p->Next[*pre-'a']==NULL)
			return 0;
		else
		{
			p=p->Next[*pre-'a'];
			pre++;
		}
	}
	return p->prefix;
}

void del(Trie *root)
{
	for(int i=0;i<maxn;i++)
	{
		if(root->Next[i]!=NULL)
			del(root->Next[i]);
	}
	free(root);
}

int main()
{
	char s[16];
	Trie *root=new Trie();
	for(int i=0;i<maxn;i++)
		root->Next[i]=NULL;
	root->prefix=0;
	while(1)
	{
		gets(s);
		if(strcmp(s,"")==0)
		{
			break;
		}
		insert(root,s);
	}
	while(gets(s))
	{
		printf("%d\n",count(root,s));
	}
	del(root);
    return 0;
}

```