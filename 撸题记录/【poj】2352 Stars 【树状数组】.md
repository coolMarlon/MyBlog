传送门:[Stars](http://poj.org/problem?id=2352)
# 题意：
给你星星的坐标（y递增，若y相等，x递增），每个星星都有一个等级，规定它的等级就是在它左下方的星星的个数。输入所有星星后，依次输出等级为0到n-1的星星的个数。

# 思路：
就是统计x前面比它小的星星的个数，符合树状数组最基本的应用。

注意的是：树状数组下标为0的位置不可用，所以我们需要在输入x坐标时+1.
另外  X 坐标的范围是32000 ，所以树状数组要开到32000 而不是节点数15000。

# My AC Code
```
#include<cstdio>

const int maxn=32005;

int Tree[maxn];
inline int lowbit(int x)
{
	return (x&-x);
}

void add(int x,int value)
{
	//std::cout<<value<<" : ";
	for(int i=x;i<=maxn;i+=lowbit(i))
	{
		Tree[i]+=value;
		//std::cout<<i<<" ";
		
	}
	//std::cout<<std::endl;
}

int get(int x)
{
	int sum=0;
	for(int i=x;i;i-=lowbit(i))
		sum+=Tree[i];
	return sum;
}

int level[maxn];

int main()
{
	int n;
	scanf("%d",&n);
	for(int i=0;i<n;i++)
	{
		int a,b;
		scanf("%d%d",&a,&b);
		a++;
		level[get(a)]++;
		add(a,1);
	}
	for(int i=0;i<n;i++)
		printf("%d\n",level[i]);
	return 0;
}
```