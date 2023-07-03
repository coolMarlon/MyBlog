SPFA(Super Programming Festival Algorithm)
其实是 Shortest Path Faster Algorithm啦^^-o-^^

简单介绍：复杂度只和边的数量相关，适用边的数量很少的最短路问题，BELLMAN-FORD算法的一种优化版本。
算法实现是BFS+剪枝构成的。


算法步骤：

 基于BFS遍历，尝试将(p, q)加入到队尾的时候，发现队列中已经存在一个(p, q')（p表示当前结点，q表示目标点到当前结点的距离）了，那么你就可以比较q和q'：如果q>=q'，那么(p, q)这个节点实际上是没有继续搜索下去的必要的——算是一种最优化剪枝吧。而如果q<q'，那么(p, q')也是没有必要继续搜索下去的——但是它已经存在于队列里了怎么办呢？很简单，将队列中的(p, q')改成(p, q)就可以了！

通过维护一个position[1..N]的数组就可以判断队列中是不是存在着一个(p, q')，如果不在队列里就是-1，否则就是所在的位置！

源代码：
```
/*
input:点数 N，边数 M，起点S，终点T，以及M组路线(起点 终点 终点)
*/
#include <stdio.h>
#include <string.h>
#include <queue>

std::queue<int>q;
//用head数组 v数组 next数组 w数组 来模拟临接图
int e,head[100005],v[2000005],next[2000005],w[2000005];
/*	
dis表示		距离
inqi标识	顶点i是不是在队列中
*/
int dis[100005],inq[100005];
int n,m,s,t,a,b,c;

void add(int a,int b,int c){
	/*
	头插法
	v表示 指向
	w表示 权重
	next  表示后继
	head  头结点数组
	*/
    v[e]=b;w[e]=c;next[e]=head[a];
    head[a]=e++;
}

int spfa(int s,int t){
    memset(inq,0,sizeof inq);
    memset(dis,-1,sizeof dis);
	//设s结点已经访问过，初始距离为0
    inq[s]=1;dis[s]=0;
    q.push(s);
    while(q.size()){
        int x=q.front();q.pop();inq[x]=0;
        for(int i=head[x];i!=-1;i=next[i]){
            int ww=dis[x]+w[i];
            if(dis[v[i]]==-1 || dis[v[i]]&gt;ww){
                dis[v[i]]=ww;
                //inq数组相当于visited数组
                if(!inq[v[i]]){
                    inq[v[i]]=1;
                    q.push(v[i]);
                }
            }
        }
    }
    return dis[t];
}

int main(){
    e=0;memset(head,-1,sizeof head);
    scanf("%d%d%d%d",&n,&m,&s,&t);
    for(int i=0;i<m;i++){
        scanf("%d%d%d",&a,&b,&c);
        add(a,b,c);add(b,a,c);
    }
    printf("%d\n",spfa(s,t));
    return 0;
}

```



Q:
1、权值为负？
可以，而dijkstra无法使用了，但是不能存在负权回路，负权回路存在可以用拓扑排序进行判断。
2、双点之间多条路径？
可以
3、环路？
可以。
4、自环？


期望的时间复杂度O(ke)， 其中k为所有顶点进队的平均次数，可以证明k一般小于等于2。