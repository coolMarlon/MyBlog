传送门:
[Magic Bitstrings](http://acm.hdu.edu.cn/showproblem.php?pid=1153)

# 思路
菜逼终究是菜逼，首先表示题目没读懂。看了别人的翻译之后，总算读懂题了。
然后公式推不出来，菜哭了。

把矩阵列出来
a[1%n], a[2%n], a[3%n], ..., a[n-1]                  (1)
a[2%n], a[4%n], a[6%n], ..., a[2(n-1)%n]       (2)
a[3%n], a[6%n], a[9%n], ..., a[3(n-1)%n]       (3)
...

比较 (1), (2)
发现：
如果 a[1%n] != a[2%n]，那么 a[2%n] != a[4%n]，那么 a[1%n] == a[4%n]；
如果 a[1%n] == a[2%n]，那么 a[2%n] == a[4%n]，那么 a[1%n] == a[4%n]。
所以 a[1%n] == a[4%n]
同样的方法得到：
a[1%n] == a[9%n],
a[1%n] == a[16%n],
....
所有下标是 i 平方 mod n 都相等。
下标不是 i 平方 mod n 都相等。

为了字典顺序最小，并且避免整个数列都是同一个数（题目要求 non-constant），令 a[1%n] = a[4%n] = a[9%n] = ... = 0，其他数都是 1，这样符合题意。

# My AC Code Relying Network
参考 https://blog.csdn.net/chengouxuan/article/details/6877054
```
/*
核心：推导+找规律
solution:令 a[1%n] = a[4%n] = a[9%n] = ... = 0，其他数都是 1
*/
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=a;i<=b;i++)
using namespace std;
const int maxn=1e5+5;

int A[maxn];

int main()
{
	int n;
	while(scanf("%d",&n))
	{
		if(n==0)
			break;
		if(n==2)
		{
			printf("Impossible\n");
			continue;
		}
		for(int i=1;i<=n-1;i++)
		{
			A[i]=1;
		}
		//这里要用long long 10^10 会溢出
		for(long long i=1;i<=n-1;i++)
		{
			A[(i*i)%n]=0;
		}
		for(int i=1;i<=n-1;i++)
			printf("%d",A[i]);
		printf("\n");
	}		
    return 0;
}
```