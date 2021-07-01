# 常见经典DP题目

判断字符串是否为回文

```python
f = [[True] * n for _ in range(n)]
		#从末尾开始dp
        for i in range(n - 1, -1, -1):
            for j in range(i + 1, n):
                f[i][j] = (s[i] == s[j]) and f[i + 1][j - 1]
```



最长公共子序列

```python
class Solution:
    def longestCommonSubsequence(self, A: str, B: str) -> int:
        m, n = len(A), len(B)
        ans = 0
        dp = [[0 for _ in range(n + 1)] for _ in range(m + 1)]
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if A[i - 1] == B[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1] + 1
                    ans = max(ans, dp[i][j])
                else:
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
        return ans

    #leetcode1035. 不相交的线
```

