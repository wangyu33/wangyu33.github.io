# Python高级操作

```python
class Solution:
    def reorderLogFiles(self, logs: List[str]) -> List[str]:
        #sorted当返回值为一个元祖时，这个元祖中的多个元素即为多个排序条件，从前到后重要程度依次降低
        def f(log):
            idx,content=log.split(' ',1)
            return (0,content,idx) if content[0].isalpha() else (1,)#这个0太精髓，前后呼应，让所有字母的在前

        return sorted(logs,key=f)

作者：bo-xia-zhu
链接：https://leetcode-cn.com/problems/reorder-data-in-log-files/solution/python3guan-fang-jie-jian-dan-zhu-shi-by-bo-xia-zh/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```



nonlocal

```python
def make_averager():
    count = 0
    total = 0
    def averager(new_value):
        nonlocal count, total
        count += 1
        total += new_value
        return total / count
    return averager
```

