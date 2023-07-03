传送门：[Buried memory](http://acm.hdu.edu.cn/showproblem.php?pid=3007)

苍天饶过谁，第三次在hdoj上 交计算几何的题了，没一次是AC的。
┭┮﹏┭┮都是模板题啊，我都是抄板子的啊，为什么会这样，我怎么这么菜。


# 题意：
求最小圆覆盖 的 圆心，半径，保留2位小数

# 分析
我的代码参考的是 俞勇老师的 《ACM国际大学生程序设计竞赛 算法与实现》 中的最小圆覆盖代码。

算法过程如下
参考：https://blog.csdn.net/commonc/article/details/52291822
![](https://img2018.cnblogs.com/blog/1484685/201810/1484685-20181015154451578-1664939155.png)

# 知识点

三角形外接圆的圆心（外心）：任意两边的垂直平分线的交点
三角形的内切圆的圆心（内心）：三角形三条角平分线的交点。
[重心](https://baike.baidu.com/item/%E4%B8%89%E8%A7%92%E5%BD%A2%E4%BA%94%E5%BF%83%E5%AE%9A%E5%BE%8B/2755449?fr=aladdin)：中线的交点
垂心：垂心的交点
旁心：三角形的旁切圆（与三角形的一边和其他两边的延长线相切的圆）的圆心



# Online AC Code And My Wrong Code
```
/*************************************************************************
    > File Name:            hdu_3007.cpp
    > Author:               Howe_Young
    > Mail:                 1013410795@qq.com
    > Created Time:         2015年05月04日 星期一 18时42分33秒
 ************************************************************************/
/*最小圆覆盖*/
/*给定n个点, 让求半径最小的圆将n个点全部包围,可以在圆上*/
#include <cstdio>
#include <iostream>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <algorithm>
#define EPS 1e-8
using namespace std;
const int maxn = 550;
struct point{
    double x, y;
};
int sgn(double x)
{
    if (fabs(x) < EPS)
        return 0;
    return x < 0 ? -1 : 1;
}
double get_distance(const point a, const point b)//两点之间的距离
{ 
    return sqrt((a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y)); 
}
point get_circle_center(const point a, const point b, const point c)//得到三角形外接圆的圆心
{
    point center;
    double a1 = b.x - a.x;
    double b1 = b.y - a.y;
    double c1 = (a1 * a1 + b1 * b1) / 2.0;
    double a2 = c.x - a.x;
    double b2 = c.y - a.y;
    double c2 = (a2 * a2 + b2 * b2) / 2.0;
    double d = a1 * b2 - a2 * b1;
    center.x = a.x + (c1 * b2 - c2 * b1) / d;
    center.y = a.y + (a1 * c2 - a2 * c1) / d;
    return center;
}
//p表示定点, n表示顶点的个数, c代表最小覆盖圆圆心, r是半径
void min_cover_circle(point *p, int n, point &c, double &r)//找最小覆盖圆(这里没有用全局变量p[], 因为是为了封装一个函数便于调用)
{
    random_shuffle(p, p + n);//随机函数,使用了之后使程序更快点,也可以不用
    c = p[0];
    r = 0;
    for (int i = 1; i < n; i++)
    {
        if (sgn(get_distance(p[i], c) - r) > 0)//如果p[i]在当前圆的外面, 那么以当前点为圆心开始找
        {
            c = p[i];//圆心为当前点
            r = 0;//这时候这个圆只包括他自己.所以半径为0
            for (int j = 0; j < i; j++)//找它之前的所有点
            {
                if (sgn(get_distance(p[j], c) - r) > 0)//如果之前的点有不满足的, 那么就是以这两点为直径的圆
                {
                    c.x = (p[i].x + p[j].x) / 2.0;
                    c.y = (p[i].y + p[j].y) / 2.0;
                    r = get_distance(p[j], c);
                    for (int k = 0; k < j; k++)
                    {
                        if (sgn(get_distance(p[k], c) - r) > 0)//找新作出来的圆之前的点是否还有不满足的, 如果不满足一定就是三个点都在圆上了
                        {
                            c = get_circle_center(p[i], p[j], p[k]);
                            r = get_distance(p[i], c);
                        }
                    }
                }
            }
        }
    }
}
int main()
{
    int n;
    point p[maxn];
    point c; double r;
    while (~scanf("%d", &n) && n)
    {
        for (int i = 0; i < n; i++)
            scanf("%lf %lf", &p[i].x, &p[i].y);
        min_cover_circle(p, n, c, r);
        printf("%.2lf %.2lf %.2lf\n", c.x, c.y, r);
    }
    return 0;
}


//My Wrong Code
/*
Wrong Answer!!!
Why ???

绝望了，和网上的AC代码思路一模一样啊。
*/
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=a;i<=b;i++)
using namespace std;
const int maxn=505;

const int pi=acos(-1.0);

const double eps=1e-8;
int cmp(double x)
{
	if(fabs(x)<eps) return 0;
	if(x>0) return 1;
	return -1;
}

inline double sqr(double x)
{
	return x*x;
}

struct point
{
	double x,y;
	point() {}
	point(double a,double b):x(a),y(b) {}
	void input()
	{
		scanf("%lf%lf",&x,&y);
	}
	friend point operator + (const point &a,const point &b)
	{
		return point(a.x+b.x,a.y+b.y);
	}
	friend point operator - (const point &a,const point &b)
	{
		return point(a.x-b.x,a.y-b.y);
	}
	double norm()
	{
		return sqrt(sqr(x)+sqr(y));
	}
	
};

double dist(const point& a,const point &b)
{
	return (a-b).norm();
}
void circle_center(point p0,point p1,point p2,point &cp)
{
	double a1=p1.x-p0.x,b1=p1.y-p0.y,c1=(a1*a1+b1*b1)/2;
	double a2=p2.x-p0.x,b2=p2.y-p0.y,c2=(a2*a2+b2*b2)/2;
	double d=a1*b2 - a2*b1;
	cp.x=p0.x+(c1*b2 - c2*b1)/d;
	cp.y=p0.y+(a1*c2 - a2*c1)/d;
}

void circle_center(point p0,point p1,point &cp)
{
	cp.x=(p0.x+p1.x)/2;
	cp.y=(p0.y+p1.y)/2;
}

point center;
double radius;

bool point_in(const point &p)
{
	return cmp((p-center).norm()-radius)<=0;
}

void min_circle_cover(point a[],int n)
{
	// 打乱
	random_shuffle(a, a + n);
	radius =0;
	center=a[0];
	for(int i=1; i<n; i++)
	{
		if(!point_in(a[i]))
		{
			center=a[i];
			radius=0;
			for(int j=0;j<i;j++)
			{
				if(!point_in(a[j]))
				{
					circle_center(a[i],a[j],center);
					radius = (a[j]-center).norm();
					for(int k=0;k<j;k++)
					{
						if(!point_in(a[k]))
						{
							circle_center(a[i],a[j],center);
							radius=(a[k]-center).norm();
						}
					}
				}
			}
		}
	}
}

int main()
{
	int n;
	point A[505];
	while(~scanf("%d",&n) &&n!=0)
	{		
		for(int i=0;i<n;i++)
		{
			scanf("%lf%lf",&A[i].x,&A[i].y);
		}
		min_circle_cover(A,n);
		printf("%.2lf %.2lf %.2lf\n",center.x,center.y,radius);
	}
	return 0;
}

```