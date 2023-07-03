传送门：https://hihocoder.com/contest/hiho121/problem/1
# 题意
最长可重叠重复K次子串问题

# 思路
二分答案，转化成判定问题。

看看能不能找出不重叠的重复子串。对于每一组，我们检查这些后缀对应的sa值(也就是后缀起点在原串中的位置i）。如果max{sa} - min{sa} >= k，那么就说明我们能找出一组不重叠的重复子串


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

int sa[200005 ];
int h[200005 ];
int  str[200005 ];
int n;

bool check(int K)
{
	int minsa,maxsa;
    for(int i=0;i<n;i++)
    if(h[i]< K)
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


int main()
{
	
	scanf("%d",&n);
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
		if(check(mid))
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