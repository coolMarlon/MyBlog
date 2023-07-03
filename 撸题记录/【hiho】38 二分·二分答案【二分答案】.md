传送门：[二分·二分答案](https://hihocoder.com/contest/hiho38/problem/1)
# 分析
因为 两个相邻战略点之间可能不止一条航线，所以不能够用邻接矩阵来存储图。
用邻接表来存储，用数组模拟。

二分答案就是二分枚举答案，验证答案的正确性，每次验证可将范围缩小一半。
因为如果要用二分答案法的话，验证单个答案的正确性必须是一个简单的步骤。
因为二分查找需要满足一个性质：即答案的临界性，临界2遍分别满足和不满足。


# Online AC Code
```
/*
因为 两个相邻战略点之间可能不止一条航线，所以不能够用邻接矩阵来存储图

*/
#include <cstdio>
#include <cstring>
#include <queue>
using namespace std;
int n,m,k,t,cnt=0,head[10005];
int len[10005];

struct edge
{
    int n,t,v;
    edge(int a=0,int b=0,int c=0):n(a),t(b),v(c){}
}e[200005];

inline void add(int a,int b,int v)
{
    e[cnt]=edge(head[a],b,v);
    head[a]=cnt++;
}
queue<int>que;
bool check(int tt)
{
    int i,j,u,v;
    memset(len,0,sizeof(len));
    while(!que.empty())que.pop();
    que.push(1);
    while(!que.empty())
    {
        u=que.front();que.pop();
        if(u==t)return len[u]<=k;
        for(i=head[u];i!=-1;i=e[i].n)
        {
            v=e[i].t;
            if(e[i].v>tt)continue;
			/*
			len数组有点像visited数组或者是path数组
			*/
            if(!len[v])
            {
                len[v]=len[u]+1;
                que.push(v);
            }
        }
    }
    return 0;
}
/*
是正确的，虽然r=mid-1可能会把正确的r减1，但是l=mid+1可以把正确答案找回来。
6啊
*/
int b_search(int l,int r)
{
    int mid;
    while(l<=r)
    {
        mid=l+r>>1;
        if(check(mid))
            r=mid-1;
        else l=mid+1;
    }
    return l;
}
int main()
{
    int i,j,a,b,c;
    scanf("%d%d%d%d",&n,&m,&k,&t);
    memset(head,-1,sizeof(head));
    for(i=1;i<=m;++i)
    {
        scanf("%d%d%d",&a,&b,&c);
        add(a,b,c);add(b,a,c);
    }
    printf("%d\n",b_search(1,1000000));
    return 0;
}

```