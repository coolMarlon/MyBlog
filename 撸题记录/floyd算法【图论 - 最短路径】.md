floyd大神发明的算法？
挺难理解的。

源代码
```
#include <cstdio>
#include <cstring>
using namespace std;

#define INF 0x3f3f3f3f

int dist[105][105];
int n;
void Floyd()
{
	for(int k=1; k<=n; k++)
		for(int i=1; i<=n; i++)
			for(int j=1; j<=n; j++)
				if(dist[i][k] + dist[k][j] < dist[i][j])
					dist[i][j] = dist[i][k]+dist[k][j];
}

int main()
{
	int  m, x, y, z;
	while(~scanf("%d%d",&n, &m))
	{
		memset(dist, INF, sizeof dist);
		for(int i=0; i<=n; i++)
			dist[i][i] = 0;
		for(int i=0; i<m; i++)
		{
			scanf("%d%d%d", &x, &y, &z);
			if(z < dist[x][y])
				dist[x][y] = dist[y][x] = z;
		}
		Floyd();
		for(int i=1; i<=n; i++)
		{
			for(int j=1; j<n; j++)
				printf("%d ", dist[i][j]);
			printf("%d\n",dist[i][n]);
		}
	}
	return 0;
}

```