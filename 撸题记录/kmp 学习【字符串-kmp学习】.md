最近学算法，了解了一些好玩的东西。
KMP算法（Knuth-Morris-Pratt）的K的指的是计算机大佬 [高德纳](https://zh.wikipedia.org/wiki/%E9%AB%98%E5%BE%B7%E7%BA%B3)
大神就是大神，人生经历牛逼。
昨天在看舞蹈链的知识（也是高德纳大神发明的୧(๑•̀◡•́๑)૭）

#分析
为了能够理解AC自动机，我又看起了KMP，这个KMP最难理解的还是next数组
就是那一句`k=next[k]`，太难理解了。
写的话还是能把KMP写出来的。
首先写暴力算法，在暴力算法的基础上写出kmp算法，至于`get_next()`结构和kmp算法很想，再此基础上略微改变就行。

kmp算法通过取消了主串的回溯，来加快算法。
因为有一些比较是多余的，可以通过模式串的结构来简化这个过程。
next数组的一个核心概念就是前缀和后缀。
它求的是一个字符串的相同的最大前缀和最大后缀的长度。

至于以前一直困扰我的next数组赋值的问题，最近想明白了，其实他们是一致的，多1少1无非是为了程序好写。

![](https://img2018.cnblogs.com/blog/1484685/201810/1484685-20181012173140134-584778765.png)

# My Code
```
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=a;i<=b;i++)
using namespace std;
const int maxn=1e6+5;

int NEXT[maxn];
int bf(char* source,char *pattern )
{
	int n=strlen(source);
	int m=strlen(pattern);
	int i=0;
	int j=0;
	while(i<n&&j<m)
	{
		if(source[i]==pattern[j])
		{
			i++;
			j++;
		}
		else
		{
			i=i-j+1;
			j=0;			
		}
	}
	if(j==m)
		return i-j;
	else
		return -1;
}

int kmp(char* source,char *pattern )
{
	int n=strlen(source);
	int m=strlen(pattern);
	int i=0;
	int j=0;
	while(i<n&&j<m)
	{
		if(j==-1||source[i]==pattern[j])
		{
			i++;
			j++;
		}
		else
		{
			j=NEXT[j];
		}
	}
	if(j==m)
		return i-j;
	else
		return -1;
}


void GetNext(char* pattern)
{
	int pLen = strlen(p);
	NEXT[0] = -1;
	int k = -1;
	int j = 0;
	while (j < pLen - 1)
	{
		//p[k]表示前缀，p[j]表示后缀
		if (k == -1 || p[j] == p[k]) 
		{
			++k;
			++j;
			NEXT[j] = k;
		}
		else 
		{
			k = NEXT[k];
		}
	}
}



int main()
{
	/*
	// 指针初始化 字符串常量, 内存分配在全局的const内存区，需要加上const
	char* source="BBC ABCDAB ABCDABCDABDE";	
	char* pattern="ABCDABD";
	
	// 正确的初始化操作
	char* source="BBC ABCDAB ABCDABCDABDE";
	char* pattern="ABCDABD";
	
	char source[]="BBC ABCDAB ABCDABCDABDE";
	char pattern[]="ABCDABD";
	*/
	char source[]="BBC ABCDAB ABCDABCDABDE";
	char pattern[]="ABCDABD";	
	printf("%s\n",source);
	printf("%s\n",pattern);
	cout<<bf(source,pattern)<<endl;
	GetNext(pattern);
	cout<<kmp(source,pattern)<<endl;;
    return 0;
}
```