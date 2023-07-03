传送门：[无间道之并查集](https://hihocoder.com/contest/hiho14/problem/1)
# 分析
并查集的分析可以看上面的传送门，写的挺好的了。
其实在我看来并查集就是一种方便的维护集合的一种技巧，提出了代表元素这一概念。

# My AC Code
```
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=a;i<=b;i++)
using namespace std;
const int maxn=1e5+5;

int represent[maxn];

int find_represent(int x)
{
	if(x == represent[x])
	{
		return x;
	}
	else
	{
		represent[x]=find_represent(represent[x]);
		return represent[x];
	}
	
}
int main()
{
	int n;
	scanf("%d",&n);
	int temp;
	int index=1;
	map<string,int> map_str;
	for(int i=1;i<maxn;i++)
	{
		represent[i]=i;
	}
	for(int i=0;i<n;i++)
	{
		scanf("tmep");
		string sa,sb;
		scanf("%d",&temp);
		cin>>sa>>sb;
		if(map_str[sa]==0)
			map_str[sa]=index++;
		if(map_str[sb]==0)
			map_str[sb]=index++;
		int ia=map_str[sa];
		int ib=map_str[sb];
		
		int ra=find_represent(ia);
		int rb=find_represent(ib);
		if(temp)
		{
			if(ra==rb)
			{
				printf("yes\n");
			}
			else
			{
				printf("no\n");
			}
		}
		else
		{		
			represent[ra]=rb;
		}
		
	}
    return 0;
}
```