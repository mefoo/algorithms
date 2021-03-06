# 康托展开

康托展开是一个全排列到一个自然数的双射，常用于构建hash表时的空间压缩。

## 定义

公式为:

```
X = a[n - 1] * (n - 1)! + a[n - 2] * (n - 2)! + ... + a[0] * 0!,
其中, a[i]为整数，并且0 <= a[i] <= i, 0 <= i < n, 表示当前未出现的的元素
中排第几个。
```
比如31254, 

* 在3后面比3小的有2个，分别为1,2
* 在1后面比1小的有0个
* 在2后面比2小的有0个
* 在5后面比5小的有1个，为4
* 在4后面比4小的有0个

因此a数组序列为2 0 0 1 0
```
X = 2 * 4! + 0 * 3! + 0 * 2! + 1 * 1! + 0 * 0!
  = 2 * 24 + 1
  = 49
```
既然是双射关系那一定可以反推出来31254这个序列。
首先我们需要推出a序列。

* 49 / 4! = 2,因此a[4] = 2, 此时49 % 4! = 1, 
* 1 / 3! = 1 / 2! = 0,因此a[3] = a[2] = 0,
* 而 1 / 1 = 1, 因此a[1] = 1,1 % 1! = 0,
* 0 / 0! = 0,因此a[0] = 0
* 所以得到a数组为2 0 0 1 0

再由a数组推出序列，根据a数组的意义反推。

* a[4] = 2, 表示它在1 2 3 4 5 序列中比它小的有2个，即它自己排第3，它等于3
* a[3] = 0, 表示它在1 2 4 5序列中比它小的有0个，即最小数，等于1
* a[2] = 0, 表示它在2 4 5 中最小，等于2
* a[1] = 1, 表示它在4 5中比它小的有一个，即排第2,等于5
* a[0] = 0, 表示它在4中最小，只能是4

因此序列为3 1 2 5 4

## 应用

一个直观的应用就是给定一个自然数集合和一个序列，求这个序列中按从小到大的顺序
排第几个?

比如1 2 3 4 5, 3 1 2 5 4比它小的序列有49个，即它排第50

另一个应用就是逆推第K个排列是多少，比如第50个全排列是多少？则首先减1得到49,
再反推即可得到3 1 2 5 4

另外在保存一个序列，我们可能需要开一个数组，如果能够把它映射成一个自然数，
则只需要保存一个整数，大大压缩空间.


## 计算

计算包括编码（根据序列求对应的自然数）和解码（康托展开）。

### 编码
```c
static const int FAC[] = {1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880};
int cantor(int *a, int n)
{
	assert(n < 10);
	int x = 0;
	for (int i = 0; i < n; ++i) {
		int smaller = 0;
		for (int j = i + 1; j < n; ++j) {
			if (a[j] < a[i])
				smaller++;
		}
		x += FAC[n - i - 1] * smaller;
	}
	return x;
}
```

### 解码
```c
int decantor(int *a, int n, int k)
{
	int *num = malloc(sizeof(int) * n );
	int len = n;
	for (int i = 0; i < n; ++i)
		num[i] = i + 1;
	int cur = 0;
	for (int i = n - 1; i > 0; --i) {
		int index = k / FAC[i];
		k %= FAC[i];
		a[cur++] = num[index];
		listRemove(num, &len, index);
	}
	a[cur] = num[0];
	free(num);
	return 0;
}
```
