传送门:[二分·二分查找](https://hihocoder.com/problemset/problem/1128)

# 分析
AC Code 1 简洁
AC Code 2首先排序，然后再用原始的二分查找法，也行吧。

# Online AC Code 1
```
#include <stdio.h>

int main(void) {
    int n, k, ans = 1, appeared = 0, num;
    scanf("%d%d", &n, &k);
    while (n--) {
        scanf("%d", &num);
        if (num < k)
            ++ans;
        else if (num == k)
            appeared = 1;
    }
    if (!appeared)
        puts("-1");
    else
        printf("%d\n", ans);
    return 0;
}

```
# Online AC Code 2
```
//hiho36
#include<algorithm>
#include<cassert>
#include<cstdio>
#include<cmath>
#include<cstring>
using namespace std;

int a[1000100];
//l是其应当插入的位置
int bsearch(int posl,int posr,int key){
	//[l,r]=[l,m][m+1,r]
	int m;
	int l=posl;
	int r=posr;
	while(l<r){
		m=(l+r)>>1;
		if(key>a[m]){//->
			l=m+1;
		}
		else{//<-
			r=m;
		}
	}
	//assert(l==r);
	//assert(key<=a[l]||posr==l);
	return l;
}
int main(){
	int u,k,v;
	int i;
	int j;
	int n;
	scanf("%d%d",&n,&k);
	for(i=1;i<=n;i++){
		scanf("%d",a+i);
	}
	sort(a+1,a+1+n);
	j=bsearch(1,n,k);
	if(k==a[j]){
		printf("%u\n",j);
	}
	else{
		puts("-1");
	}
	return 0;
}

```