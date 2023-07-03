传送门：[后缀数组一·重复旋律](https://hihocoder.com/contest/hiho120/problem/1)

[最长不可重叠重复子串问题](https://hihocoder.com/contest/hiho121/problem/1)
二分答案，转化成判定问题
```
bool check(int K)
{
    for(int i=1;i<=n;i++)
    if(height[i]< K)
    {
        minsa=sa[i];
        maxsa=sa[i];
    }
    else 
    {
        minsa=min(minsa,sa[i]);
        maxsa=max(maxsa,sa[i]);
        if(maxsa-minsa>=K)return true;
    }
    return false;
}
```

[最长公共子串问题](https://hihocoder.com/contest/hiho122/problem/1)
求排名相邻，原来不在同一个字符串的 height 值的最大值

[重复次数最多的连续字串](https://hihocoder.com/contest/hiho123/problem/1)
逃。。。


# 题意：最长可重叠重复K次子串问题

# 分析：
难~，求sa、rank、height数组跪了，好难理解啊。

suffix(i)表示从原字符串第i个字符开始到字符串结尾的后缀。我们把它所有的后缀拿出来按字典序排序
把排好序的数组记做sa。比如sa[1]=7,sa[4]=2
名次数组 Rank， Rank[i] 保存的是后缀 i 在所有后缀中从小到大排列的“名次”
height[i] 是 suffix(sa[i-1]) 和 suffix(sa[i]) 的最长公共前缀长度，即排名相邻的两个后缀的最长公共前缀长度。


有几个结论：

**有了height，求最长可重叠重复K次子串就方便了。重复子串即两后缀的公共前缀，最长重复子串，等价于两后缀的最长公共前缀的最大值。问题就转化成了，求height 数组中最大的长度为 K的子序列的最小值。**


后缀数组的求法有很多，最有名的是两种**倍增算法**和**DC算法**。时间复杂度上DC算法更优，但是很复杂。我们这里只介绍相对容易的倍增算法。

有空再仔细研究这个算法，先溜了。

代码参看 《ACM-ICPC 算法与实现》

# AC Code
```
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=a;i<=b;i++)
using namespace std;
const int maxn=1e6+5;

void  radis(int *str,int *a,int *b,int n,int m)
{
	static int count[200000];
	memset(count,0,sizeof(count));
	for(int i=0;i<n;++i) ++count[str[a[i]]];
	for(int i=1;i<=m;++i) count[i]+=count[i-1];
	for(int i=n-1;i>=0;--i) b[--count[str[a[i]]]]=a[i];
}

void suffix_array(int *str,int *sa,int n,int m)
{
	static int rank[200000],a[200000],b[200000];
	for(int i=0;i<n;++i)rank[i]=i;
	radis(str,rank,sa,n,m);
	
	rank[sa[0]]=0;
	for(int i=1;i<n;++i)
		rank[sa[i]]=rank[sa[i-1]]+(str[sa[i]]!=str[sa[i-1]]);
	for(int i=0;1<<i<n;++i)
	{
		for(int j=0;j<n;++j)
		{
			a[j]=rank[j]+1;
			b[j]=j+(1<<i) >=n?0:rank[j+(1<<i)]+1;
			sa[j]=j;
		}
		radis(b,sa,rank,n,n);
		radis(a,rank,sa,n,n);
		rank[sa[0]]=0;
		for(int j=1;j<n;++j)
		{
			rank[sa[j]]=rank[sa[j-1]]+(a[sa[j-1]]!=a[sa[j]] || b[sa[j-1]] != b[sa[j]]);
		}		
	}	
}

void calc_height(int *str,int *sa,int *h,int n)
{
	static int rank[200000];
	int k=0;
	h[0]=0;
	for(int i=0;i<n;++i)
		rank[sa[i]]=i;
	for(int i=0;i<n;++i)
	{
		k=k==0?0:k-1;
		if(rank[i]!=0)
			while(str[i+k] == str[sa[rank[i]-1]+k]) ++k;
		h[rank[i]]=k;
	}
}

int sa[20005 ];
int h[20005 ];
int  str[20005 ];
int main()
{
	int n,k;
	scanf("%d%d",&n,&k);
	//n=8,k=2;
	int max_num=0;
	for(int i=0;i<n;i++)
	{
		scanf("%d",&str[i]);	
		max_num=max(str[i],max_num);
	}
	suffix_array(str,sa,n,max_num);
	calc_height(str,sa,h,n);
	/*
	for(int i=0;i<n;i++)		
		cout<<sa[i]<<" ";
	cout<<endl;
	for(int i=0;i<n;i++)		
		cout<<h[i]<<" ";
	cout<<endl;		
	*/
	
	int left=0,right=n-1;
	int res=0;
	//cout<<666<<endl;
	while(left<=right)
	{
		int mid=(left+right)>>1;
		//cout<<mid<<endl;
		int cnt=1;
		for(int i=0;i<n;i++)
		{
			if(h[i]>=mid)
				cnt++;
			else cnt = 1;
			if(cnt >=k) break;
		}
		if(cnt>=k)
		{
			res=max(res,mid);
			left=mid+1;			
		}
		else
			right=mid-1;
	}
	cout << res << endl;
	
    return 0;
}
```