# 一、基础算法

## 快排

```cpp
void quick_sort(std::vector<int> &q , int l , int r ){
    if(l >= r)   return;
    int i = l - 1 , j = r + 1 , x = q[(l+r)>>1] ;
    while(i < j){
        do i++ ; while(q[i] < x) ;
        do j-- ; while(q[j] > x) ;
        if(i < j)   std::swap(q[i] , q[j]) ;
    }
    quick_sort(q , l , j) ;
    quick_sort(q , j+1 , r) ;

}
```

## 快速选择(第k个数)

```cpp
int quick_sort(int *q , int l , int r , int k){
    if(l >= r)  return q[l];
    int i = l - 1 , j = r + 1 , x = q[(l+r)>>1] ;
    while(i < j){
        do i++ ; while(q[i] < x) ;
        do j-- ; while(q[j] > x) ;
        if(i < j) std::swap(q[i] , q[j]) ;
    }
    if(k <= (j - l + 1))    return quick_sort(q , l , j , k) ;
    else return quick_sort(q , j+1 , r , k - (j - l + 1)) ;
}
```

[215. 数组中的第K个最大元素 - 力扣（LeetCode）](https://leetcode.cn/problems/kth-largest-element-in-an-array/description/)

## 归并排序

**分治思想**

<img src = "https://pic.leetcode-cn.com/1614274007-nBQbZZ-Picture1.png"></img>

```cpp
#include<iostream>
#include<algorithm>
const int N = 1e5 + 10 ;
int q[N] ;
void merge_sort(int *q , int l , int r){
    if(l >= r)  return;
    int mid = l + r >> 1 , i = l , j = mid + 1 ;
    merge_sort(q , l , mid) ;
    merge_sort(q , mid+1 , r) ;
    int temp[r-l+1] , k = 0 ;
    while(i <= mid && j <= r){
        if(q[i] < q[j]) temp[k++] = q[i++] ;
        else temp[k++] = q[j++] ;
    }
    while(i <= mid) temp[k++] = q[i++] ;
    while(j <= r)   temp[k++] = q[j++] ;
    for(int i = l , j = 0 ; i <= r ; i++ , j++) 
        q[i] = temp[j] ;
}
int main()
{
    int n ;
    std::cin >> n ;
    for(int i = 0 ; i < n ; i++){
        std::scanf("%d" , &q[i]) ;
    }
    // std::sort(q , q+n) ;
    merge_sort(q , 0 , n-1);
    for(int i ; i < n ; i++)  std::printf("%d ", q[i]) ;
}
```

## 逆序对数量

```cpp
#include<iostream>
typedef long long LL ;
const int N = 1e5 + 10 ;
int q[N] ;
LL merge_sort(int *q , int l , int r){
    if(l >= r)  return 0 ;
    int mid = (l+r) >> 1 ;
    LL res = merge_sort(q , l , mid) + merge_sort(q , mid+1 , r) ;
    int i = l , j = mid+1 , k = 0 ;
    int temp[r-l+1] ;
    while(i <= mid && j <= r){
        if(q[i] <= q[j])    temp[k++] = q[i++] ;
        else{
            res += mid - i + 1 ;
            temp[k++] = q[j++] ;
        }

    }
    while(i <= mid) temp[k++] = q[i++] ;
    while(j <= r)   temp[k++] = q[j++] ;
    for(int i = l , k = 0 ; i <= r ; i++ , k++)
        q[i] = temp[k] ;
    return res ;
}

int main()
{
    int n ;
    std::cin >> n ;
    for(int i = 0 ; i < n ; i++)
        scanf("%d" , &q[i]) ;
    std::cout << merge_sort(q , 0 , n-1) ;
    return 0 ;
}
```

[LCR 170. 交易逆序对的总数 - 力扣（LeetCode）](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

## 二分查找

```cpp
#include<iostream>
const int N = 1e5 + 10 ;
int q[N] ;
int SL(int x , int l , int r){
    //查找第一个≥x的数 lower_bound
    int mid ;
    while(l < r){
        mid = (l+r) >> 1 ;
        if(q[mid] >= x) r = mid ;
        else l = mid + 1 ;
    }
    if(q[l] == x)   return l ;
    else return -1 ;
}
int SR(int x , int l , int r){
    //查找最后一个≤x的数
    int mid ;
    while(l < r){
            mid = (l+r+1) >> 1 ;
            if(q[mid] <= x) l = mid ;
            else r = mid - 1 ;
    }
    if(q[l] == x) return l ;
    else return -1 ;
}
int main()
{
    int n , m ; 
    std::cin >> n >> m ;
    for(int i = 0 ; i < n ; i++)
        std::scanf("%d" , &q[i]) ;
    while(m--){
        int x ;
        std::cin >> x ;
        std::cout << SL(x , 0 , n-1) << " " << SR(x , 0 , n-1 ) << std::endl ;

    }

    return 0 ;
}
```

[69. x 的平方根 - 力扣（LeetCode）](https://leetcode.cn/problems/sqrtx/)

## 高精度计算

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
using namespace std ;
string add(const string &a , const string &b){
    int t = 0 , i = a.size() - 1 , j = b.size() - 1 ;
    string c ;
    while(i >= 0 || j >= 0){
        if(i >= 0)  t += a[i--] - '0' ;
        if(j >= 0)  t += b[j--] - '0' ;
        c.push_back(t % 10 + '0') ;
        t /= 10 ;
    }
    if(t)   c.push_back(t + '0') ;
    reverse(c.begin() , c.end()) ;
    return c ;
}
bool cmp(const string& a , const string& b){
    if(a.size() != b.size())    return a.size() > b.size() ;
    else {
        int i ;
        for(i = 0 ; a[i] == b[i] ; i++) ;
        return a[i] > b[i] ;
    }
    return 0 ;
}
string sub(const string& a , const string& b){
    if(a == b)  return "0" ;
    string c ;
    if(cmp(a , b)){
           int i = a.size() - 1 , j = b.size() - 1 , t = 0 , k = 0 ;
           while(i >= 0 || j >= 0){
            t = k ;
            if(i >= 0)  t += a[i] - '0' ;
            if(j >= 0)  t = t - (b[j] - '0') ;
            c.push_back((t+10)%10 + '0') ;
            k = (t >= 0) ? 0 : -1 ;
            i-- , j-- ;
           }
    }
    else {
        c.push_back('-') ;
        c += sub(b,a) ; 
    }
    int i = c.size() - 1 ;
    while(c.size() > 1 && c[i--] == '0')  c.pop_back() ;
    if(c[0] != '-') reverse(c.begin() , c.end()) ;
    return c ;
}
string mul(const string &a , const string &b){
    if(a == "0" || b == "0")    return "0" ;
    int len_a = a.size() , len_b = b.size()  , t = 0 ;
    string c = string(len_a + len_b , '0') ;
    for(int i = len_a - 1 ; i >= 0 ; i--){
        for(int j = len_b - 1 ; j >= 0 ; j--){
            t = (a[i] - '0') * (b[j] - '0') + (c[i+j+1] - '0');
            c[i+j+1] = t % 10 + '0' ;
            c[i+j] += t / 10 ;
        }
    }
    int i ;
    for(i = 0 ; c[i] == '0' && i < len_a + len_b - 1 ; i++) ;
    c = c.substr(i) ;
    return c ;
}
string div(const string &a , int b , int& r){
    int r0 = 0 ;//ro is reminder .
    string c ;
    for(auto it = a.begin() ; it != a.end() ; it++){
        r0 = r0 * 10 + (*it - '0') ;
        c.push_back(r0 / b + '0') ;
        r0 %= b ;
    }
    int idx = 0 ;
    while(c[idx] == '0' && idx < c.size() - 1)    idx++ ;
    c = c.substr(idx) ;
    r = r0 ;
    return c ;
}

int main()
{

}
```

## 前缀和

```cpp
#include<iostream>
using namespace std ;

int main()
{
    int n , m , l , r ;
    const int N = 1e5 + 10 ;
    cin >> n >> m ;
    int q[N] , s[N] ;
    s[0] = 0 ;
    for(int i = 1 ; i <= n ; i++){
        scanf("%d" , &q[i]) ;
        s[i] = s[i-1] + q[i] ;
    }

    while(m--){
        cin >> l >> r ;
        cout << s[r] - s[l-1] << endl ;
    }
}
```

## 子矩阵的和

```cpp
#include<iostream>
using namespace std ;
const int N = 1010 ;
int main()
{
    int a[N][N] = {0} , s[N][N] = {0} ;
    int n , m , q , x1 , y1 , x2 , y2 ;
    cin >> n >> m >> q ;
    for(int i = 1 ; i <= n ; i++)
        for(int j = 1 ; j <= m ; j++){
            scanf("%d" , &a[i][j]) ;
            s[i][j] = s[i-1][j] + s[i][j-1] + a[i][j] - s[i-1][j-1] ;
        }
    while(q--){
        // cin >> x1 >> y1 >> x2 >> y2 ;
        scanf("%d %d %d %d" , &x1 , &y1 , &x2 , &y2) ;
        printf("%d\n" , s[x2][y2] - s[x2][y1-1] - s[x1-1][y2] + s[x1-1][y1-1]) ; 
    } 

}
```

## 差分

## 双指针

**最长连续不重复子序列**

```cpp
#include<iostream>
using namespace std ;
int main()
{
    const int N = 1e5 + 10 ;
    int a[N] , s[N] = {0} , n , res = 0 ;
    cin >> n ;
    for(int i = 0 ; i < n ; i++)
        scanf("%d" , &a[i]) ;
    for(int i = 0 , j = 0 ; i < n ; ++i){
        s[a[i]]++ ;
        while(j < i && s[a[i]] > 1) s[a[j++]] -- ;
        res = res > (i-j+1) ? res : (i-j+1) ;
    }
    cout << res ;
    return 0 ;
}
```

[3. 无重复字符的最长子串 - 力扣（LeetCode）](https://leetcode.cn/problems/longest-substring-without-repeating-characters/submissions/492530011/)

**数组元素的目标和**

```cpp
#include<iostream>
using namespace std ;
int main()
{
    int n , m , x ;
    const int N = 1e5 + 10 ;
    int a[N] , b[N] ;
    cin >> n >> m >> x ;
    for(int i = 0 ; i < n ; i++){
        scanf("%d" , &a[i]) ;
    }
    for(int i = 0 ; i < m ; i++)
        scanf("%d" , &b[i]) ;
    for(int i = 0 , j = m-1 ; i < n ; i++){
        while(j>=0 && a[i]+b[j] > x)    j-- ;
        if(j >= 0 && a[i]+b[j] == x){
            cout << i << ' ' << j ;
            break ;
        }
    }

}
```

**位运算**

```cpp
#include<iostream>
using namespace std ;
int main()
{
    int x ;
    cin >> x ;
    for(int k = 7 ; k >= 0 ; --k)//8位二进制表示输出
        cout << (x>>k & 1) ;
}
```

**lowbit**

求出最后一个二进制1所表示的数的大小(描述得不清楚)，看示例：

`24`的8位二进制表示为`0001_1000` , 则`lowbit(24) = 0000_1000` ;

`lowbit(26) = 0000_0010` ……

原理是`x&(-x)` , 本质是`x & (~x + 1)` , `x`与其补码相与。

`lowbit`常用于统计二进制数中`1`的个数

```cpp
#include<iostream>
using namespace std ;
int lowbit(int x){
    return x&(-x) ;
}
int main()
{
    int n , x , res = 0 ;
    cin >> n ;
    while(n--){
        cin >> x ;
        res =  0 ;
        while(x)    x -= lowbit(x) , res++ ;
        printf("%d " , res) ;
    }
}
```

[LCR 133. 位 1 的个数 - 力扣（LeetCode）](https://leetcode.cn/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/submissions/492746547/)

[461. 汉明距离 - 力扣（LeetCode）](https://leetcode.cn/problems/hamming-distance/)

利用`x&(x-1)`也可求二进制表示中1的个数

例如汉明距离：

```cpp
class Solution {
public:
    int hammingDistance(int x, int y) {
        int t = x ^ y , res = 0 ;
        while(t)    t = t & (t-1) , res++ ;
        return res ;
    }
};
```

## 离散化

使用场景：**值域大，但稀疏**

先看离散化：[数组序号转换](https://leetcode.cn/problems/rank-transform-of-an-array/description/)

这道题是将原数组中的元素映射到从1开始的编号，也就是将范围未定的元素**离散化**到自然数序列：

```cpp
class Solution {
public:
    vector<int> arrayRankTransform(vector<int>& arr) {
        vector<int> temp = arr;
        sort(arr.begin() , arr.end());
        arr.erase(unique(arr.begin() , arr.end()) , arr.end());

        vector<int> res;
        for(int i = 0 ; i < temp.size() ; i++){
            res.push_back(upper_bound(arr.begin() , arr.end() , temp[i]) - arr.begin() );
        }

        return res;
    }
};
```

```python
class Solution:
    def arrayRankTransform(self, arr: List[int]) -> List[int]:
        dict = {val:i+1 for i,val in enumerate(sorted(set(arr)))}
        return [dict[elem] for elem in arr]
```



[802. 区间和](https://www.acwing.com/problem/content/description/804/)

acwing这道题的难点就在，题目让你在$a[x]$ ($x$表示数轴坐标)上进行多次加法操作，但$x$范围在$-10^9$~$10^9$ , 且不说数组索引无法为负数，就算转换到$0到2×10^9$,也开不了这么大的数组。这就需要将所有输入的$x$离散化。$a[x] += c$ 转化为$a'[x'] += c$  ,  在$[l,r]$求前缀和转换为在$[l',r']$上求前缀和。($x'、l'、r'$表示$x,l,r$离散化后对应的值)注意:

* $l,r$也要加入到$x$中求离散化，这样才能保证$l',r'$在$x'$中能找到值。
* $a[N]$中$N$开多大合适？$N$的大小要保证能装下$x ,l,r$离散化后的值，$n$,$m$的个数最大均为$10^5$,因此$N$要大于$3×10^5$
* `std::unique`只对相邻的重复元素有效，所以在应用 unique 之前，需要对容器进行排序
* `upper_bound_`函数的作用就是找出$x$离散化后的坐标$x'$,这里是用二分查找做的。

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
const int N = 3e5 + 10;
int a[N] , s[N];
int upper_bound_(const vector<int>& nums , int x){
    int l = 0 , r = nums.size() - 1;
    
    while(l<r){
        int mid = (l+r)>>1;
        if(nums[mid]>=x)    r = mid;
        else l = mid + 1;
    }
    return r+1;
}
int main()
{
    int n , m;
    cin >> n >> m;
    vector<pair<int,int>> add , query;
    vector<int> idx;
    for(int i = 0 ; i < n ; i++){
        int x , c;
        scanf("%d %d" , &x , &c);
        add.push_back({x,c});
        idx.push_back(x);
    }


    for(int i = 0 ; i < m ; i++){
        int l , r;
        scanf("%d %d" , &l , &r);
        query.push_back({l,r});
        idx.push_back(l);
        idx.push_back(r);
    }

    sort(idx.begin() , idx.end());
    idx.erase(unique(idx.begin() , idx.end()) , idx.end());

    for(auto item : add){
        int x = upper_bound_(idx , item.first);
        a[x] += item.second;
    }

    for(int i = 1 ; i <= idx.size() ; ++i){
        s[i] = s[i-1] + a[i];
    }

    for(auto item : query){
        int l = upper_bound_(idx , item.first);
        int r = upper_bound_(idx , item.second);
        printf("%d\n" , s[r] - s[l-1]);
    }
}
```

## 区间合并

[803. 区间合并](https://www.acwing.com/problem/content/805/)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;


int main()
{
    int n;
    cin >> n;
    vector<pair<int,int>> interval ;
    for(int i = 0 ; i < n ; i++){
        int l , r;
        scanf("%d %d" , &l , &r);
        interval.push_back({l,r});
    }

    sort(interval.begin() , interval.end());
    int start = interval[0].first;
    int end   = interval[0].second;
    int cnt = 1;

    for(auto item : interval){
        if(item.first <= end && item.second > end)  end = item.second;
        else if(item.first > end) {
            cnt++;
            start = item.first;
            end   = item.second;
        }
    }
    cout << cnt;
}
```

[56. 合并区间 - 力扣（LeetCode）](https://leetcode.cn/problems/merge-intervals/description/)





## 单调栈

所谓的单调栈就是在将一列数依次入栈时，始终保持栈的单调性(栈底到栈顶，递增or递减)：

```cpp
template<typename T>
stack<T> monotonic_stack(const vector<T>& a){
    //monotonically incremental
    stack<T> st;
    for(auto item : a){
        while(!st.empty() && item < st.top()){
            st.pop();
        }
        st.push(item);
    }
    return st;
}
```

单调栈可以用来解决`NGE`(Next Greater Element)问题，这一问题也可以扩展成`左边第一个大的元素`  `右边第一个小的元素`  `左边第一个小的元素`  `右边第一个大的元素`等问题。解决这类问题时，先要想清楚是要构造单调增的栈or单调减的栈。

[830. 单调栈 - AcWing题库](https://www.acwing.com/problem/content/description/832/)

这道题是求`左边第一个比它小的数` , 先想清楚是要构造一个递增的栈。

```cpp
#include<iostream>
#include<vector>
#include<stack>
#include<algorithm>
using namespace std;
const int N = 1e5 + 10;
int a[N];
int main()
{
    int N , t;
    cin >> N;
    stack<int> st;
    for(int i = 0 ; i < N ; i++){
        scanf("%d" , &t);

        while(!st.empty() && t<=st.top()){
            st.pop();
        }
        if(st.empty()){
            printf("-1 ");
            st.push(t);
        }
        else{
            printf("%d ",st.top());
            st.push(t);
        }
    }
}
```



[739. 每日温度 - 力扣（LeetCode）](https://leetcode.cn/problems/daily-temperatures/description/) 这道题是求右边第一个大的数，需要维护一个单调减的栈，而且要注意：

* 求右边第一个大(或小)的数，可以从右到左遍历数组然后把结果翻转。也可以：对比栈顶元素与即将入栈元素，给出栈顶元素对应的答案。
* 这道题是求右边第一个大的数与该数间隔的天数，所以栈里面存元素的索引，通过求差值得到间隔的天数。
* 一些判断逻辑与`左边第一个大的数`不同

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        st = []
        N = len(temperatures)
        res = [0] * N
        for i , temp in enumerate(temperatures):
            while st and temp > temperatures[st[-1]]:
                top = st.pop()
                res[top] = i - top

            st.append(i)
        
        return res
```



## 单调队列

* 单调队列常用于解决**滑动窗口**问题

[239. 滑动窗口最大值 - 力扣（LeetCode）](https://leetcode.cn/problems/sliding-window-maximum/description/)

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int len = nums.size();
        deque<int> q;
        vector<int> res;

        for(int i = 0 ; i < len ; i++){
            while(q.size() && nums[i] > q.back()){
                q.pop_back();
            }

            q.push_back(nums[i]);
            if(i >= k && q.front() == nums[i-k])  q.pop_front();
            if(i >= k-1)    res.push_back(q.front());  
        }

        return res;
    }
```

* 注意在弹出窗口最前端的数时，需要判断`q.front() == nums[i-k]`,因为`q.front()`不一定是窗口最前端需要弹出的数。



## KMP

`KMP`常用于寻找一个字符串中是否包含子串。

[28. 找出字符串中第一个匹配项的下标 - 力扣（LeetCode）](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/description/)

暴力做法肯定是遍历母串中每一个元素开始的字符串，寻找是否有与子串相同的部分：

```cpp
int strStr(string haystack, string needle) {
        int len_hay = haystack.size() , len_needle = needle.size();

        if(len_hay < len_needle)    return -1;
        for(int i = 0  , j = 0; i < len_hay ; i++){
            int k = i; 
            while(haystack[k] == needle[j] && i < len_hay && j < len_needle){
                k++ , j++;
            }

            if(j == len_needle) return i;
            else {
                j = 0;
            }
        }
        return -1;
    }
```

* 暴力解法的缺点在于`i`指针需要不断回溯，时间复杂度为$O(n^2)$ 但实际上，`i`指针无需回溯，见下图：

![KMP](images/KMP.png)

**KMP的做法**

* 字符串S的指针`i`不会回溯：当比较进行到`S[3] != P[3]`时，`i`指针不回溯，`j`指针回溯到`next[j]`处，即`j = next[j]`，再继续对比下去。时间复杂度为$O(n)$
* `next`数组是什么？为什么`j`回溯到`next[j]`?   `next`数组是当前元素前的字符串的最长公共前后缀。例如上图左下角的字符串`ababab`，第一个元素`str[0] = a`的`next[0] = -1` , 因为`str[0]`之前没有字符串，也没有前后缀；`str[1] = b`之前的字符串为`a`，前后缀是重合的，`next[1] = 0` ;可见`next`数组前两项为`-1  0` ; `str[2] = a`前的字符是`ab`， 没有公共前后缀，`next[2] = 0` ; `str[3] = b`前的字符串是`aba`，最长公共前后缀分别为`a`和`a` , `next[2] = 1`
* 求`next`的算法：

**证明next数组的求法：**
$$
求next[k] , 令cn = next[k-1]\\
判断str[k-1] == str[cn]\\
①true: str[k-1] = str[cn]\\
由next[k-1] = cn \\
\rarr str[0:cn-1] = str[k-cn-1:k-2](即str[k-1]前长度为cn的前后缀相同)\\
又str[k-1] = str[cn]\\
\rarr str[0:cn] = str[k-cn-1:k-1]\\
\therefore next[k] = cn + 1\\
②false: str[k-1] \neq str[cn]\\
令cn = next[cn]\\
判断str[k-1] == str[cn]\\
true:\   \because str[0:cn-1] = str[k-cn-1:k-2]\\
又str[cn] = str[k-1]\\
\rarr str[0:cn] = str[k-cn-1:k-1]\\
\rarr next[k] = cn+1
$$
注：

* `next[k]`是否可以大于`cn+1` ？ 不可能。可以反证：若`next[k] = cn+2`,则`str[0:cn+1] = str[k-cn-2:k-1]` , 也有`str[0:cn] = str[k-cn-2:k-2]` $\rarr$ `next[k-1] = cn + 1` 与`next[k-1] = cn`矛盾。类似的，`next[k] = cn+3 , cn+4……`不成立，`next[k]`只能满足$next[k] \leq cn+1$ 

* 由上面这一点$next[k] \leq cn+1$ , 因此是去判断`str[k-1]是否与str[cn]`相等，而不是：$str[k-1] == str[x] , x > cn$
* 判断失败后$$str[k-1] \neq str[cn]$$ , 继续迭代$str[k-1] == str[next[cn]]$.  求next数组本质是在迭代：$$str[k-1] == str[next[k-1]] , str[k-1] == str[next[next[k-1]]]$$……

```cpp
 vector<int> get_next(const string& needle){
        int m = needle.size();
        if(m == 1)  return {-1};
        vector<int> next(m);
        next[0] = -1;
        next[1] = 0;
        int i = 2 , cn = 0;

        while(i < m){
            if(needle[i-1] == needle[cn]){
                next[i++] = ++cn;
            }
            else if(cn > 0){
                cn = next[cn];
            }
            else    next[i++] = 0;
        }

        return next;
 	}
```





**证明为什么`i`指针无需回溯，`j`指针只要回溯到`next[j]`处：**
$$
若s为母串，p为子串，i为s的指针，j为p的指针，此时发生不匹配，有\\
s[i] \neq p[j] \ 且\ s[i-j : i-1] = p[0:j-1]① \\
设next[j] = cn,\ 则有p[0:cn-1] = p[j-cn:j-1]② \\
由最长公共前后缀的性质：最长公共前后缀长度不大于整个字符串可知cn < j③\\
由①③两式可推出s[i-cn:i-1] = p[j-cn:j-1]④\\
由②④两式得s[i-cn:i-1] = p[0:cn-1],也就是说p的前cn个字符已经与s匹配\\
只需继续匹配s[i] == p[cn],因此j指针需要回溯到cn,next[j]\\
证毕
$$


* 完整`kmp`:

```cpp
class Solution{
public:
    vector<int> get_next(const string& needle){
        int m = needle.size();
        if(m == 1)  return {-1};
        vector<int> next(m);
        next[0] = -1;
        next[1] = 0;
        int i = 2 , cn = 0;

        while(i < m){
            if(needle[i-1] == needle[cn]){
                next[i++] = ++cn;
            }
            else if(cn > 0){
                cn = next[cn];
            }
            else    next[i++] = 0;
        }

        return next;
    }



    int strStr(string haystack , string needle){
        vector<int> next = get_next(needle);
        int len_hay = haystack.size() , len_needle = needle.size();
        if(len_hay < len_needle) return -1;


        int i = 0 , j = 0;
        while(i < len_hay && j < len_needle){
            if(haystack[i] == needle[j]){
                i++ , j++;
            }
            else if(j == 0){
                i++;
            }
            else {
                j = next[j];
            }
        }
        if(i == len_hay && j != len_needle) return -1;
        else return i - j;
    }
};
```





[831. KMP字符串 - AcWing题库](https://www.acwing.com/problem/content/description/833/)

`AcWing`的这道题需要考虑子串在母串中多次出现的情况，这时完成一次匹配后仍需继续匹配:

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
using namespace std;

vector<int> get_next(const string& P){
    int len = P.size();
    if(len == 1)    return {-1};
    vector<int> next(len+1);
    next[0] = -1 , next[1] = 0;

    int i = 2 , cn = 0;
    while(i <= len){
        if(P[i-1] == P[cn]){
            next[i++] = ++cn;
        }
        else if(cn > 0){
            cn = next[cn];
        }
        else next[i++] = 0;
    }

    return next;
}

void kmp_(const string& S , int M , const string& P , int N){
    vector<int> next = get_next(P);
    // vector<int> res;
    

    int i = 0 , j = 0 ;
    while(i < M){
        if(S[i] == P[j]){
            i++ , j++;
        }
        else if(j == 0){
            i++;
        }
        else{
            j = next[j];
        }

        if(j == N){
            printf("%d " , i-j);
            j = next[j];
        }
    }

}

int main()
{
    int N , M ;
    string P , S;
    std::cin >> N >> P >> M >> S;
    kmp_(S , M , P , N);
}
```

