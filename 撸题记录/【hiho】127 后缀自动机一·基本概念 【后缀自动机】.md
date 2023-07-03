传送门：[后缀自动机一·基本概念][3]

后缀自动机（Suffix Automaton，简称SAM）是处理字符串的很强大的工具，对于一个字符串S，它对应的后缀自动机是一个最小的确定有限状态自动机（DFA），接受且只接受S的后缀。
对于字符串S="aabbabd"，它的后缀自动机是：
![](https://img2018.cnblogs.com/blog/1484685/201810/1484685-20181015203140679-659856353.png)

介绍一个概念子串的结束位置集合 endpos。对于S的一个子串s，endpos(s) = s在S中所有出现的结束位置集合。还是以S="aabbabd"为例，endpos("ab") = {3, 6}，因为"ab"一共出现了2次，结束位置分别是3和6。同理endpos("a") = {1, 2, 5}, endpos("abba") = {5}。

对于S="aabbabd"：

|状态	|子串			|endpos|
|:-:|:-:|:-:|
|S		|空串								|{0,1,2,3,4,5,6}
|1		|a									|{1,2,5}
|2		|aa									|{2}
|3		|aab									|{3}
|4		|aabb,abb,bb							|{4}
|5		|b									|{3,4,6}
|6		|aabba,abba,bba,ba					|{5}
|7		|aabbab,abbab,bbab,bab				|{6}
|8		|ab									|{3,6}
|9		|aabbabd,abbabd,bbabd,babd,abd,bd,d	|{7}

# hiho 大神
[094791][1]
van048
[FakerInHeart][2]    


# 094791's beautiful Code
```
//后缀自动机一·基本概念
//hiho127
#include <bits/stdc++.h>
using namespace std;

void print(set<int>&s){
	set<int>::iterator it;
	for(it=s.begin();it!=s.end();it++){
		printf("%u ",*it);
	}
	putchar('\n');
}
int main(){
	string s;
	map<string,set<int> >m;
	cin>>s;
	int i;
	int l;
	// 好优美！
	for(l=1;l<=s.length();l++){
		for(i=0;i+l<=s.length();i++){
			string sub=s.substr(i,l);
			set<int>tmp=m[sub];
			tmp.insert(i+l);
			m[sub]=tmp;
		}
	}
	int t;
	cin>>t;
	while(t--){
		cin>>s;
		string lstr=s;
		string sstr=s;
		set<int>tmp=m[s];
		map<string,set<int> >::iterator it;
		for(it=m.begin();it!=m.end();it++){
			if(it->second==tmp){
				if(lstr.length()<it->first.length()){
					lstr=it->first;
				}
				if(sstr.length()>it->first.length()){
					sstr=it->first;
				}
			}
		}
		printf("%s %s ",sstr.c_str(),lstr.c_str());
		print(tmp);
	}
    return 0;
}

```

[1]:https://hihocoder.com/user/17318
[2]:https://hihocoder.com/user/29735
[3]:https://hihocoder.com/contest/hiho127/problem/1