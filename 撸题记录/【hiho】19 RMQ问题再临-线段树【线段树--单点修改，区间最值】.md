传送门：[RMQ问题再临-线段树](https://hihocoder.com/contest/hiho19/problem/1)

线段树牛逼啊，这都是谁发明的。想起大三下学期在南理工打比赛的时候，当时都不知道线段树，
一道线段树裸题，只知道用暴力来写（肯定超时啊）
虽然现在知道线段树了也不一定写出来，因为菜啊。
线段树比较裸的内容有：
1. 单点增减，区间求和
2. 单点更新，区间求最值
3. 区间替换，区间求和
4. 区间更新，区间求和
5. 区间替换、区间求最值（这个有吗？？？）
6. 区间更新、区间求最值（这个有吗？？？）

**还有一些和线段树相关的题：**
1. [区间合并](http://chuanwang66.iteye.com/blog/1418459)
2. [扫描线](http://chuanwang66.iteye.com/blog/1418459)

**与之有相似功能的数据结构有：**
树状数组


当然比赛的时候，肯定不会这么裸的，线段树的变形（那就更打不出来了o(╥﹏╥)o）

# 题意
线段树教程题，考察**单点修改，区间最值**

# 思路
下面这张图用来讲线段树原理我觉得特别好。
![](http://media.hihocoder.com/problem_images/20141108/14154249916279.png)

# Online AC Code
```
/*
算法：线段树--单点修改，区间最值
*/
#include <cstdio>
#include <cstring>
#include <climits>
#include <algorithm>
using namespace std;

int const N = 1e4 + 10;

struct segmentTree {
	int mn[N<<2];
	void init(int r, int L, int R) {
		if (L > R) return;
		if (L == R) {
			scanf("%d", mn+r);
		} else {
			int ls = r << 1, rs = ls | 1, M = (L + R) >> 1;
			init(ls, L, M);
			init(rs, M+1, R);
			mn[r] = min(mn[ls], mn[rs]);
		}
	}
	void modify(int r, int L, int R, int p, int v) {
		if (R < p || L > p) return;
		if (p <= L && R <= p) {
			mn[r] = v;
		} else {
			int ls = r << 1, rs = ls | 1, M = (L + R) >> 1;
			modify(ls, L, M, p, v);
			modify(rs, M+1, R, p, v);
			mn[r] = min(mn[ls], mn[rs]);
		}
	}
	/*
	r:root
	L:left
	R:right
	巧妙，线段树牛逼
	*/
	int query(int r, int L, int R, int a, int b) {
		if (R < a || L > b) return INT_MAX;
		if (a <= L && R <= b) return mn[r];
		int ls = r << 1, rs = ls | 1, M = (L + R) >> 1;
		return min(query(ls,L,M,a,b), query(rs,M+1,R,a,b));
	}
} T;

int main() {
	int n, q;
	scanf("%d", &n);
	T.init(1, 0, n-1);
	scanf("%d", &q);
	for (int op,a,b; q--; ) {
		scanf("%d%d%d", &op, &a, &b);
		if (op == 0) printf("%d\n", T.query(1, 0, n-1, a-1, b-1));
		else T.modify(1, 0, n-1, a-1, b);
	}
	return 0;
}

```