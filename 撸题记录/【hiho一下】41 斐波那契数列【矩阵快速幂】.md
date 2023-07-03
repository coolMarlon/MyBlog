计算 斐波那契数列 第n项 MOD 19999997的结果
```
#include <iostream>
using namespace std;

/*2x2矩阵相乘与2x1矩阵相乘*/
void mul(long long F[2][2], long long M[2][2]) {
	long long lt, rt,
		bl, br;
	lt = F[0][0] * M[0][0] + F[0][1] * M[1][0], rt = F[0][0] * M[0][1] + F[0][1] * M[1][1];
	bl = F[1][0] * M[0][0] + F[1][1] * M[1][0], br = F[1][0] * M[0][1] + F[1][1] * M[1][1];
	lt = lt % 19999997;
	rt = rt % 19999997;
	bl = bl % 19999997;
	br = br % 19999997;
	F[0][0] = lt, F[0][1] = rt;
	F[1][0] = bl, F[1][1] = br;
}
/*2x2矩阵求乘方*/
void power2(long long F[2][2], long long n) {
	if (n == 0 || n == 1) {
		return;
	}
	long long M[2][2] = { { 1, 1 },{ 1, 0 } };
	power2(F, n / 2);
	mul(F, F);
	if (n % 2 != 0) {
		mul(F, M);
	}
}
/*求斐波那契数列高效算法(O(log2n))*/
/*当n较小时比int fibonacci(int n)慢,当n较大时明显快于其他算法*/
long long fibnacci_3(long long n) {
	long long F[2][2] = { { 1, 1 },{ 1, 0 } };
	if (n == 0) {
		return 0;
	}
	power2(F, n - 1);
	return F[0][0];
}

int main() {
	long long n;
	cin >> n;
	cout << fibnacci_3(n + 1) << endl;
	return 0;
}
```