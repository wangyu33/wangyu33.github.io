# python常用算法库

chr asc码转字符

ord 字符转asc码

## heapq 最小堆函数

**建堆**:

 heapq.heapify()

**加入堆**:

heapq.heappush()

**弹出堆**:

heapq.heappop()

**前k大**:

heapq.nlargest(3, lyst)

**前k小**:

heapq.nsmallest(3, lyst)

更新：

heapq.heapreplace()

等价于先pop再push

默认按**第一个元素**进行堆排序

```python
from typing import List
from collections import  defaultdict
import heapq
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        dict = defaultdict(int)
        for i in nums:
            dict[i] = dict[i] + 1
        ans = []
        for key in dict:
            ans.append((dict[key],key))
        # print(ans)
        heapq.heapify(ans)
        # 取出前k小的元素
        while len(ans) > k:
            heapq.heappop(ans)
        temp = []
        for i in range(k):
            temp.append(heapq.heappop(ans)[1])
        return temp
```

## PriorityQueue优先队列

**声明：**

queue.PriorityQueue()

**往队列中加入元素：**

queue.put()

**往队列中删除元素**:

queue.get()

默认以加入队列元素中的第一个值进行从小到大排序，例如put进来的元素为tuple （a,b），则按a的大小从小到大排序

```python
class Solution:
    def minCostConnectPoints(self, points: List[List[int]]) -> int:
        def dst(a, b):
            return abs(a[0]-b[0]) + abs(a[1]-b[1])

        from queue import PriorityQueue
        
        pq = PriorityQueue()  # 来，复习一下优先队列的API | PS:要是卡常，就换成"配对堆"
        visit = set([i for i in range(len(points))])  # 待访问的节点集
        res = 0

        pq.put((0, 0))  # (distance, point_id)  # Prim算法从任何一个节点出发都是一样的，这里从0点开始
        while visit:  # 当没有访问完所有节点 | 其它语言的coder请把这行理解成 => while(visit.length() > 0){...}
            dis, now = pq.get()  # 获取优先队列中最小的项 => (到扩展集中某最近点的距离，某最近点的序号)
            if now not in visit:  # 已访问过的直接跳过
                continue
            visit.remove(now)  # 随手剪枝，移除出待访问的节点集
            res += dis
            for i in visit:  # 构建扩展集，就是把当前点对所有未访问点的距离都求一遍
                # 以距离为cost丢进优先队列排序就好，想不清明白其它题解费劲构建边结构干啥...
                pq.put((dst(points[now], points[i]), i))

        return res
```

## dict与defaultdict

dict判断字典里是否存在key

dict.get(key, 默认键值)	若无键值赋值默认键值

**合并字典**

dict.update()



## set

集和

求集和交集与并集

a = set()

b = set()

a&b

a|b

差集

a-b

补集

a^b

添加元素

a.add

## frozenset

**set(可变集合)与frozenset(不可变集合)的区别**：
set无序排序且不重复，是可变的，有add（），remove（）等方法。既然是可变的，所以它不存在哈希值。基本功能包括关系测试和消除重复元素. 集合对象还支持union(联合), intersection(交集), difference(差集)和sysmmetric difference(对称差集)等数学运算.
sets 支持 x in set, len(set),和 for x in set。作为一个无序的集合，sets不记录元素位置或者插入点。因此，sets不支持 indexing, 或其它类序列的操作。
frozenset是冻结的集合，它是不可变的，存在哈希值，**好处是它可以作为字典的key**，也可以作为其它集合的元素。缺点是一旦创建便不能更改，没有add，remove方法。



## bisect二分查找

import bisect

```python
import bisect
a = [0, 1, 5, 7, 19, 25]
a1 = bisect.bisect(a, 6)

# 这里返回的位置是3是因为：
# 为了保证插入这个数，还能保持列表升序，这个位置显而易见就在值5后面
print(a1) 
```

## @lru_cache(None)

functools模块中的lru_cache(maxsize,typed)

加速递归函数

算法题目中可以使用

相当于记忆化搜索，保存已递归的数值

加了cache_clear()貌似会变快？

```python
from typing import *
from functools import lru_cache
import bisect
from queue import Queue
import math

class Solution:
    def maximumScore(self, nums: List[int], multipliers: List[int]) -> int:
        n, m = len(nums), len(multipliers)
        
        
        @lru_cache(None)
        def helper(i, j, idx):
            if idx == m:
                return 0
            r1 = multipliers[idx] * nums[i] + helper(i+1, j, idx+1)
            r2 = multipliers[idx] * nums[j] + helper(i, j-1, idx + 1)
            return max(r1, r2)
        
        res = helper(0, n-1, 0)
        
        helper.cache_clear()
        return res
```

## import itertools

### itertools.combinations(p, i)

itertools模块combinations(iterable, r)方法可以创建一个迭代器，返回iterable中所有长度为r的**子序列**，返回的子序列中的项按输入iterable中的顺序排序。

itertools.permutations

返回可迭代对象的所有数学**全排列方式**



## 排序

envelopes.sort(key=lambda x:(x[0],-x[1]))



## functools.reduce

**reduce()** 函数会对参数序列中元素进行累积

```python
from functools import reduce

def add(x, y) :            # 两数相加
    return x + y
sum1 = reduce(add, [1,2,3,4,5])   # 计算列表和：1+2+3+4+5
sum2 = reduce(lambda x, y: x+y, [1,2,3,4,5])  # 使用 lambda 匿名函数
print(sum1)
print(sum2)
```



## collections.Counter

声明为迭代器

```python
from collections import Counter
class Solution:
    def beautySum(self, s: str) -> int:
        ans = 0
        for i in range(len(s)):
            count = Counter()
            for j in range(i, len(s)):
                count[s[j]] += 1
                max_value = max(count.values())
                min_value = min(count.values())
                ans += (max_value - min_value)
        return ans

```



### 素数筛

```python
class Solution:
    def countDifferentSubsequenceGCDs(self, nums: List[int]) -> int:
        f = [0 for i in range(200010)]
        ans = 0
        for num in nums:
            if f[num] == 0:
                f[num] += 1
                ans += 1
        maxn = max(nums)
        for i in range(1, maxn + 1):
            if f[i]: continue
            r = 0
            for j in range(i, maxn + 1, i):
                if f[j]:
                    r = math.gcd(r, j)
                    if r == i:
                        ans += 1
                        break
        return ans
    #leetcode1819 
```



十进制数字转为二进制字符串

```python
x = str(bin(x))[2:].rjust(32,'0')
```



# Python SortedContainers对字典排序

```python
from sortedcontainers import SortedList
lt = SortedList()
```





```python
class MovieRentingSystem:#leetcode1912
    def __init__(self, n: int, entries: List[List[int]]):
        self.t_price = dict()
        self.t_valid = defaultdict(sortedcontainers.SortedList)
        self.t_rent = sortedcontainers.SortedList()

        for shop, movie, price in entries:
            self.t_price[(shop, movie)] = price
            self.t_valid[movie].add((price, shop))

    def search(self, movie: int) -> List[int]:
        t_valid_ = self.t_valid

        if movie not in t_valid_:
            return []

        return [shop for (price, shop) in t_valid_[movie][:5]]

    def rent(self, shop: int, movie: int) -> None:
        price = self.t_price[(shop, movie)]
        self.t_valid[movie].discard((price, shop))
        self.t_rent.add((price, shop, movie))

    def drop(self, shop: int, movie: int) -> None:
        price = self.t_price[(shop, movie)]
        self.t_valid[movie].add((price, shop))
        self.t_rent.discard((price, shop, movie))

    def report(self) -> List[List[int]]:
        return [(shop, movie) for price, shop, movie in self.t_rent[:5]]
```



sortedcontainers.SortedList()

添加元素

add

如果元素存在，删除指定元素

discard

删除指定元素

remove

删除指定index元素

pop

