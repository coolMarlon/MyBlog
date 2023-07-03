参考：[扩展KMP算法](https://segmentfault.com/a/1190000008663857)

问题定义：给定两个字符串S和T（长度分别为n和m），下标从0开始，定义extend[i]等于S[i]...S[n-1]与T的最长相同前缀的长度，求出所有的extend[i]。
如下表所示：

|i			|0	|1	|2	|3	|4	|5	|6	|7|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|S			|a	|a	|a	|a	|a	|b	|b	|b
|T			|a	|a	|a	|a	|a	|c	|	|
|extend[i]	|5	|4	|3	|2	|1	|0	|0	|0


```
#include<iostream>
#include<string>
using namespace std;

/* 求解T中next[]，注释参考GetExtend() */
void GetNext(string T, int next[])
{
    int t_len = T.size();
    next[0] = t_len;
    int a;
    int p;

    for (int i = 1, j = -1; i < t_len; i++, j--)
    {
        if (j < 0 || i + next[i - a] >= p)
        {
            if (j < 0)
                p = i, j = 0;

            while (p < t_len&&T[p] == T[j])
                p++, j++;

            next[i] = j;
            a = i;
        }
        else
            next[i] = next[i - a];
    }
}

/* 求解extend[] */
void GetExtend(string S, string T, int extend[], int next[])
{
    GetNext(T, next);  //得到next
    int a;             
    int p;             //记录匹配成功的字符的最远位置p，及起始位置a
    int s_len = S.size();
    int t_len = T.size();

    for (int i = 0, j = -1; i < s_len; i++, j--)  //j即等于p与i的距离，其作用是判断i是否大于p（如果j<0，则i大于p）
    {
        if (j < 0 || i + next[i - a] >= p)  //i大于p（其实j最小只可以到-1，j<0的写法方便读者理解程序），
        {                                   //或者可以继续比较（之所以使用大于等于而不用等于也是为了方便读者理解程序）
            if (j < 0)
                p = i, j = 0;  //如果i大于p

            while (p < s_len&&j < t_len&&S[p] == T[j])
                p++, j++;

            extend[i] = j;
            a = i;
        }
        else
            extend[i] = next[i - a];
    }
}

int main()
{
    int next[100] = { 0 };
    int extend[100] = { 0 };
    string S = "aaaaabbb";
    string T = "aaaaac";

    GetExtend(S, T, extend, next);

    //打印next和extend
    cout << "next:    " << endl;
    for (int i = 0; i < T.size(); i++)
        cout << next[i] << " ";

    cout << "\nextend:  " << endl;
    for (int i = 0; i < S.size(); i++)
        cout << extend[i] << " ";

    cout << endl;
    return 0;
}	
```