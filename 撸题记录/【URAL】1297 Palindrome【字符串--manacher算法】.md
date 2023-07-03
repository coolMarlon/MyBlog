传送门：[Palindrome](http://acm.timus.ru/problem.aspx?space=1&num=1297)

# 题意
求最长回文字符串，在学manacher算法，所以用了manacher，看到网上好多题解使用后缀数组来做的。

# 思路
manacher算法，参考《ACM国际大学生程序设计竞赛 算法与实现》的板子，一开始我以为板子的manacher算法是错误的，然后上网看题解。
直到我看到 https://blog.csdn.net/u012717411/article/details/53363444 文章，我才知道其实人家是对的，只不过我没理解。

manacher算法在O（N）的时间复杂度内求得了字符串所有子串的最长回文子串的长度。

例如输入
`abccba` 最终的字符串会变成这样 ` a#b#c#c#b#a `
下标从0开始。
len数组表示以当前位置开始的 最长回文班级，其中# 不考虑。
所以上面例子的len数组如下图所示。

![](https://img2018.cnblogs.com/blog/1484685/201810/1484685-20181016184642086-702944704.png)

至于manacher算法的原理，我大概理解，但还是不够透彻。可以参考这篇文章。
https://blog.csdn.net/dyx404514/article/details/42061017



#AC Code
```
/*
参考：https://blog.csdn.net/u012717411/article/details/53363444

算法过程: abccba -> a#b#c#c#b#a 
下标从0开始。
#位置即(pos&1)==1 的位置 

*/
#include<bits/stdc++.h>
#define rep(i,a,b) for(int i=a;i<=b;i++)
using namespace std;
const int maxn=1e6+5;

void manacher(char str[],int len[],int n) //接口
{
	len[0] = 1;
	for(int i = 1,j = 0; i < (n<<1) - 1; ++ i)
	{
		int p = i >> 1,q = i - p, r = ((j+1) >> 1) + len[j] - 1;
		len[i] = r < q?0:min(r-q+1,len[(j<<1) - i]);
		while(p > len[i] - 1 && q + len[i] < n && str[p - len[i]] == str[q+len[i]]) ++len[i];
		if(q + len[i] - 1 > r) j = i;
	}
}



string longestPalindrome(string s)
{
	int n = s.size();
	/*
	len数组：
	*/
	
	int len[2000];
	char *str = &s[0];
	manacher(str,len,n);//调用接口，得到len[]
	for(int i=0;i<n*2;i++)
	{
		cout<<len[i]<<" ";
	}
	cout<<endl;
	
	
	string tmp = "";
	int pos = 0,max_len = 0;
	for(int i = 0; i < (n<<1) - 1; ++ i)
	{
		//以‘#’or字符为中心，串长不一样
		int tmp_len = (i&1)?len[i]<<1:(len[i]<<1)-1; 
		//pos记录目标串的中心点，max_len表示目标串的串长（不含#）
		if(tmp_len > max_len) pos = i,max_len = tmp_len; 
		//作一个tmp[0..2n-1]的字符串，便于输出
		if(i&1) tmp+="#";
		else tmp+=s[i>>1];    
	}
	//# 为中心
	if(pos&1)    //找到要打印的串tmp的起始位置pos和打印长度max_len（便于打印输出）
	{
		max_len = (len[pos] << 2) - 1;
		pos = pos - (len[pos] << 1) + 1;
	}
	else
	{
		max_len = (len[pos] << 2) - 3;
		pos = pos - ((len[pos]-1)<<1);
	}
	string ans = "";
	for(int i = pos,j = 0; j < max_len; ++ j,++ i)
	{
		if(i&1) continue;
		ans+=tmp[i];     //tmp中找到所要打印的字符，链接起来
	}
	return ans;
}

int main()
{
	string s;
	cin>>s;
	cout<<longestPalindrome(s)<<endl;
}
```