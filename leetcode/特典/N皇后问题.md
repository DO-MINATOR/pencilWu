### [N皇后问题 leetcode](https://leetcode-cn.com/problems/n-queens/)

*n* 皇后问题研究的是如何将 *n* 个皇后放置在 *n*×*n* 的棋盘上，并且使皇后彼此之间不能相互攻击。

![image-20200825163235496](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200825163235496.png)

上图为 8 皇后问题的一种解法。

给定一个整数 n，返回所有不同的 n 皇后问题的解决方案。

每一种解法包含一个明确的 n 皇后问题的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。

![image-20200825163305080](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200825163305080.png)

```c
class Solution {
public:
    int *col, *dg, *udg;//列、对角线占用情况
    vector<string> a;//存储中间结果
    vector<vector<string>> res;//返回结果

    vector<vector<string>> solveNQueens(int n) {
        col = new int[n]{0}, dg = new int[2 * n - 1]{0}, udg = new int[2 * n - 1]{0};
        a.resize(n, string(n, '.'));
        dfs(0, n);//从第0行开始DFS
        return res;
    }

    void dfs(int u, int n) {
        if (u == n) {
            res.push_back(a);//已经保存的结果==n行数时，得到结果之一，并保存下来
        }
        for (int i = 0; i < n; i++) {
            if (!col[i] && !dg[u + i] && !udg[u - i + n-1]) {//求解当前行可行的情况，如果找到，则继续递归，否则回溯至上层
                a[u][i] = 'Q';
                col[i] = dg[u + i] = udg[u - i + n-1] = 1;
                dfs(u + 1, n);
                a[u][i] = '.';
                col[i] = dg[u + i] = udg[u - i + n-1] = 0;
            }
        }
    }
};
```

时间复杂度：通过剪枝，第1层n个解，第2层不超过n-2个解，第三层不超过n-4个解，因此总体时间复杂度为O(n!)

空间复杂度：字符串数组保存中间结果O(n^2^)，递归调用栈O(n)，列、对角线数组O(n)

