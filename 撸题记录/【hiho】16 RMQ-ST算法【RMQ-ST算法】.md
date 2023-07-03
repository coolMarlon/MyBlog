传送门：[RMQ-ST算法](https://hihocoder.com/contest/hiho16/problem/1)

RMQ(Range  Minimum/Maximum Query)区间范围最值查询问题

# 题意
求指定区间值最小的元素

# 思路
其实就是二分法的思路，统计所有长度为2的非负整数次幂的区间。
然后将所求转化到在包含的几个区间之中寻找最小值。

# Online AC Code
```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <algorithm>
#include <cmath>
#include <map>
#include <vector>
#include <queue>
#include <set>
using namespace std;

#define FOR(i, s, t ) for( int i = s; i < t; ++i )
#define REP( i, s, t ) for( int i = s; i <= t; ++i ) 
#define eps 1e-8
#define inf 0x3f3f3f3f
#define LL long long 
#define ULL unsigned long long
#define pii pair<int, int >
#define MP make_pair
#define lson id << 1, l, m
#define rson id << 1 | 1, m + 1, r
#define maxn ( 1000000 + 10 )
#define maxe ( 200000 + 100 )

int d[maxn][30],num[maxn];

void rmq_init ( int n ){
	for( int i = 0; i < n; i++ ) d[i][0] = num[i];
	for( int j = 1; (1<<j) <= n; j++ )
		for( int i = 0; i + (1<<j ) - 1 < n; i ++ )
			d[i][j] = min( d[i][j-1], d[i+(1<<(j-1))][j-1] );
}

int rmq ( int l, int r ){
	int k = 0;
	while( ( 1 << (k+1) ) <= ( r - l + 1 ) ) k++ ;
	return min( d[l][k], d[r - (1<<k) + 1][k] );
}

int main () {
	int n;
	scanf("%d", &n );
	FOR( i,0 ,n ) scanf("%d", num+i );
	rmq_init( n );

	int m ;
	scanf("%d", &m );
	int l, r;
	while( m-- ) {
		scanf("%d%d", &l, &r );
		--l, --r;
		printf("%d\n", rmq( l, r ) ) ;
	}
}

```