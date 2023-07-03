传送门： https://www.nowcoder.com/acm/contest/16/A

在找关于 CDQ 分治道题，找到下面这条博客。
[二维偏序问题](https://blog.csdn.net/qq_35914587/article/details/79061277)

# 题意：
N个对象，每个对象有2个值，求有多少对象的2个值均小于其他某一个对象

# 思路：
等价于s,w为二维坐标x,y。
首先把s按升序排列。 
离散化w。 
倒着枚举s，然后查询w的后缀和不为0即为一个答案。 

# AC Code
```
#include <cstdio>
#include <iostream>
#include <algorithm>
#define lowbit(x) x&-x
#define il inline
using namespace std;
const int maxm=1e5+100;
int sum[maxm];
struct node
{
	int x,y;
} a[maxm];
int w[maxm];
il void ins(int x)
{
	for(int i=x; i<=maxm; i+=lowbit(i))
		sum[i]++;
}
il int ask(int x)
{
	int ans=0;
	for(int i=x; i; i-=lowbit(i))
		ans+=sum[i];
	return ans;
}
il bool comp(node x,node y)
{
	return x.x<y.x;
}
int main()
{
	int n;
	scanf("%d",&n);
	for(int i=1; i<=n; i++)
		scanf("%d%d",&a[i].x,&a[i].y),w[i]=a[i].y;
	sort(w+1,w+n+1);
	int t=unique(w+1,w+n+1)-w-1;

	// cout<<t<<endl;	
	for(int i=1; i<=n; i++)
		a[i].y=lower_bound(w+1,w+t+1,a[i].y)-w;
	/*
	for(int i=1;i<=n;i++)
		cout<<a[i].y<<" ";
	cout<<endl;
	*/
	sort(a+1,a+n+1,comp);
	int ans=0;
	for(int i=n; i>=1; i--)
	{
		int d=ask(a[i].y);
		//cout<<d<<endl;
		// n-i 表示可能的最多的完虐的数的数量
		// d 表示当前有几个数比他小
		if(n-i-d!=0) ans++;
		ins(a[i].y);
	}
	return printf("%d",ans)*0;
}

```