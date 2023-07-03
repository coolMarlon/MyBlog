dijkstra大神发明的算法

朴素的dijkstra算法时间复杂度为O(n*n)，只能处理包含正权边的图。
使用优先级队列或堆优化过的dijkstra算法时间复杂度为O(N*log(N))

下面是优先级队列优化的dijkstra代码源码
```
/*
input:点数 N，边数 M，起点S，终点T，以及M组路线(起点 终点 终点)
*/
#include <iostream>
#include <queue>
#include <cstdio>
#include <cstring>
using namespace std;
#define maxN 1024
#define maxM 10240
int n, m, s, t;
struct node{
	int to, cost, next;
}edge[maxM<<1];
int first[maxN], edgeCnt;
int addEdge(int from, int to, int cost){
	edge[edgeCnt].to = to;
	edge[edgeCnt].cost = cost;
	edge[edgeCnt].next = first[from];
	return first[from] = edgeCnt++;
}
void buildGraph(){
	int a, b, c;
	edgeCnt = 0;
	memset(first, -1, sizeof(first));
	for(int i=0; i<m; i++) {
		scanf("%d %d %d", &a, &b, &c);
		addEdge(a, b, c);
		addEdge(b, a, c);
	}
}

int dist[maxN];
struct qnode{
	int pos, cost;
	friend bool operator < (qnode a, qnode b){
		return a.cost > b.cost;
	}
};
priority_queue<struct qnode> myQueue;
int solve(){
	struct qnode tmp;
	memset(dist, -1, sizeof(dist));
	tmp.pos = s; tmp.cost = 0;
	myQueue.push(tmp);
	while(myQueue.size() > 0 && dist[t] == -1){
		tmp = myQueue.top(); myQueue.pop();
		if(dist[tmp.pos] != -1) continue;
		int from = tmp.pos;
		dist[from] = tmp.cost;
		for(int e = first[from]; e != -1; e = edge[e].next){
			if(dist[edge[e].to] != -1) continue;
			tmp.pos = edge[e].to, tmp.cost = dist[from] + edge[e].cost;
			myQueue.push(tmp);
		}
	}
	return dist[t];
}
int main(){
//	freopen("1.txt", "r", stdin);
	scanf("%d %d %d %d", &n, &m, &s, &t);
	buildGraph();
	cout << solve();
	return 0;
}

```

下面是朴素的dijkstra算法
```
#include<stdio.h>
#include<string.h>
#define maxa 1005
#define INF 10000000

int n,m;
int dis[maxa];
bool mark[maxa];
int map[maxa][maxa];
int mon[maxa];

void dijkstra(int s)
{
    int i,j,k,Min,visa,p;
    memset(mark,0,sizeof(mark));
    for(i=1; i<=n; i++)
    {
        dis[i]=map[s][i];        
    }
    dis[s]=0;
    mon[s]=0;
    mark[s]=1;
    for(i=1; i<n; i++)
    {
        Min=INF;
        for(j=1; j<=n; j++)
        {
            if(mark[j]) continue;
            if(dis[j]<Min)
            {
                Min=dis[j];
                k=j;
            }
        }
        mark[k]=1;
        for(p=1; p<=n; p++)
        {
            if(!mark[p] && map[k][p]!=INF)
            {
                visa=dis[k]+map[k][p];
                if(visa<dis[p])
                {
                    dis[p]=visa;                    
                }
            }
        }
    }
}

int main()
{
    int i,j,a,b,c,d,from,to;
    scanf("%d%d",&n,&m);
	scanf("%d%d",&from,&to);    
    for(i=0; i<=n; i++)
    {
        for(j=0; j<=n; j++)
        {
            map[i][j]=INF;            
        }
    }
    for(i=0; i<m; i++)
    {
        scanf("%d%d%d",&a,&b,&c);
        if(c<map[a][b])
        {
            map[a][b]=map[b][a]=c;            
        }
    }
    
    dijkstra(from);
    printf("%d",dis[to]);    
    return 0;
}
```