1
`random_shuffle`
 重排序给定范围 [first, last) 中的元素，使得这些元素的每个排列拥有相等的出现概率。
```
random_shuffle(RandomIt first, RandomIt last)
```

2
`nth_element`
将第n_th 元素放到它该放的位置上，左边元素都小于它，右边元素都大于它.
```
void nth_element (RandomAccessIterator first, RandomAccessIterator nth,
                    RandomAccessIterator last);

void nth_element (RandomAccessIterator first, RandomAccessIterator nth,
                    RandomAccessIterator last, Compare comp);
```

3`bitset`
```
template< std::size_t N > class bitset;
# 表示一个 N 位的固定大小序列，可以用标准逻辑运算符操作位集，并将它与字符串和整数相互转换。

# 常见方法
operator[] 
test() 
all()
any()
none ()
count()
size()
set()
reset()
to_string()
to_ulong()

```

4.`fill` `fill_n`
```
# 制定范围内填充元素
fill(ForwardIterator first, ForwardIterator last, const T& val);

# 填充从first开始的n个元素
fill_n(OutputIterator first, Size n, const T& val);


```

5.`sort`
排序序列，时间复杂度为O(N*lgN)
```
# 默认用 operator< 比较元素，也可以制定comp方法
# RandomIt 也可以使用 指针
void sort( RandomIt first, RandomIt last )
sort( RandomIt first, RandomIt last, Compare comp )

```

6.`unique` `unique_copy`
移除重复元素
```
unique( ForwardIt first, ForwardIt last )
ForwardIt unique( ForwardIt first, ForwardIt last, BinaryPredicate p )

# demo code
    std::vector<int> v{1,2,3,1,2,3,3,4,5,4,5,6,7};
    std::sort(v.begin(), v.end()); // 1 1 2 2 3 3 3 4 4 5 5 6 7 
    auto last = std::unique(v.begin(), v.end());
    // v 现在保有 {1 2 3 4 5 6 7 x x x x x x} ，其中 x 不确定
    v.erase(last, v.end()); 
    for (int i : v)
      std::cout << i << " ";
    std::cout << "\n";
```

7.`lower_bound()`
标记了第一个大于等于value 的值
`upper_bound()`
这个位置标记了第一个大于value 的值。
如下图所示：
![](https://img2018.cnblogs.com/blog/1484685/201810/1484685-20181011200416406-89585791.png)
```
ForwardIt lower_bound( ForwardIt first, ForwardIt last, const T& value );
ForwardIt  lower_bound( ForwardIt first, ForwardIt last, const T& value, Compare comp );
```

8.`priority_queue`
优先级队列
```
# 常见方法
push()
top()
pop()
# 默认为大根堆
# 需要更改为小根堆，这样写
std::priority_queue<int, std::vector<int>, std::greater<int> > q2;

# 需要设置元素为pair<int,int>
std::priority_queue<pair<int,int>> pq;

# 需要设置元素为pair<int,int>，且为小根堆
class CompareDist
{
public:
    bool operator()(pair<int,int> n1,pair<int,int> n2) {
        return n1.first>n2.first;
    }
};

priority_queue<pair<int,int>,vector<pair<int,int>>,CompareDist> pq;

# 需要设置元素为其他类
struct cs{
    int x;
    bool operator < (const cs &rhs) const {
        return x > rhs.x;
    }
}a;
priority_queue<cs>Q;
```

9.`greater<T>`,`less<T>`
进行比较的函数对象。调用类型 T 上的 operator> 和 <

10.`perv(it)`，`advance(it,n)`，`next(it)`
```
前一个迭代器，迭代器步近n，后一个迭代器
```

11. `list<T> `类
线性表，链表实现，与vector<T> 相对应。
```
front()
back()
push_back(num)
pop_back()
push_front()
pop_front()
insert(iterator)
erase(iterator)
```