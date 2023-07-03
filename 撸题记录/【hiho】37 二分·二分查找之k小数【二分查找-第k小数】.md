传送门：[二分·二分查找之k小数](https://hihocoder.com/contest/hiho37/problem/1)

# 分析
分治法牛逼，参考快速排序中的分割函数，把分割的主元放出来，根据主元的下标，小的往左找，大的往右边找。

# 时间复杂度分析
二分查找之k小数的时间复杂度是O(n)

假设有n个数，
第1次划分的时间需要O(n)
第2次划分的时间需要O(n/2)
第3次划分的时间需要O(n/2)
,,,

总的时间需要
$$O(n+\frac{n}{2}+\frac{n}{4}+....+1)= O(\frac{1*2^{log_2^n}}{2-1})=O(n) $$

# My AC Code
```
#include<iostream>
#include<stdlib.h>
 
using namespace std;
const int maxn=1e6+5;
 
int A[maxn];
void swap(int a[],int b[])
{
	int temp=*a;
	*a=*b;
	*b=temp;
}
 
 
int partition_k(int k,int a[],int first,int end)
{
	int i=first;
	int j=end;
	while(i<j)
	{
		while(i<j&&a[i]<a[j]) j--;
		if(i<j)
		{
			swap(a[i],a[j]);
			i++;
		}
		while(i<j&&a[i]<a[j]) i++;
		if(i<j)
		{
			swap(a[i],a[j]);
			j--;
		}
	}
	//return i;改写部分 
	if(k==j-first+1) return a[j];
	else if(k<j-first+1) return partition_k(k,a,first,j-1);
	else return partition_k(k-(j-first+1),a,j+1,end);
}
 
int main()
{
	int n,k;
	scanf("%d%d",&n,&k);
	if(k>n||k<1)
	{
		cout<<-1;
		return 0;
	}
	for(int i=0;i<n;i++)
	{
		scanf("%d",&A[i]);
	}		
	cout<<partition_k(k,A,0,n-1);
	return 0;
}

```