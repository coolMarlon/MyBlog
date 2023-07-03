**题目来源**

[hiho一下 第147周](https://hihocoder.com/contest/hiho147/problem/1)

**“五维数点”问题**

现在有n个在五维空间中的点(X_i,Y_i,Z_i,Q_i,W_i)。现在对于每个点，我们需要知道所有坐标均比它小的点的数量。
转化为集合操作，用bitset 的and 来模拟2个集合之间的交集，优化后的时间复杂度为成O(N^2/32)



**AC code**
```
#include<iostream>
#include<bitset>
#include<cstdio>
using namespace std;
#define maxn 30010

bitset<maxn> nums[5][maxn];

int nodes[maxn][5];

int order[maxn];

/*
非常的巧妙，充分地利用了学生的排名
*/
int main()
{
    int N,i,j;
    scanf("%d",&N);
    for(i=0;i<N;i++)
        for(j=0;j<5;j++)
            scanf("%d",&nodes[i][j]);
	// 时间复杂度O(N)
    for(i=0;i<5;i++)
    {
        for(j=0;j<N;j++)
        {
			/*order数组表示排名
			order[a]=b
			排名为a的人，所在的位置是b
			*/
			
            order[nodes[j][i]]=j+1;
        }
        for(j=2;j<=N;j++)
        {
            nums[i][order[j]]=nums[i][order[j-1]];
            nums[i][order[j]][order[j-1]]=1;
        }
    }
    for(i=1;i<=N;i++)
    {
        bitset<maxn> tmp;
        tmp = nums[0][i] & nums[1][i] & nums[2][i] & nums[3][i] & nums[4][i];
        printf("%lu\n",tmp.count());
    }
    return 0;
}

```