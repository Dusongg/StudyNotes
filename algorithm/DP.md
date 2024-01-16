# 1 数位dp

> 模板：[2719. 统计整数数目](https://leetcode.cn/problems/count-of-integers/)
>
> ```cpp
> class Solution {
> public:
>     const int MOD = 1e9 + 7;
>     int count(string num1, string num2, int min_sum, int max_sum) {
>         int n2 = num2.size(), n1 = num1.size();
>         num1 = string(n2 - n1, '0') + num1;
>         vector<vector<int>> memo(n2, vector<int>(9 * n2 + 1, -1));
>         function<int(int, int, bool, bool)> f = [&](int i, int sum, bool up_limit, bool low_limit) ->int {
>             if (i == n2) return (sum >= min_sum && sum <= max_sum) ? 1 : 0;
> 
>             if (!up_limit && !low_limit && memo[i][sum] != -1) return memo[i][sum];
>             int res = 0;
> 
>             int up = up_limit ? num2[i] - '0' : 9;
>             int low = low_limit ? num1[i] -'0' : 0;
>             
>             for (int d = low; d <= up; d++) {
>                 res = (f(i + 1, sum + d, up_limit && d == up, low_limit && d == low) + res) % MOD;
>             }
>             if (!up_limit && !low_limit) memo[i][sum] = res;
>             return res;
>         };
>         return f(0, 0, true, true);
>     }
> };
> ```
>
> 

```cpp
class Solution {
public:
    long long numberOfPowerfulInt(long long start, long long finish, int limit, string s) {
        string low = to_string(start);
        string high = to_string(finish);
        int n = high.size();
        low = string(n - low.size(), '0') + low; // 补前导零，和 high 对齐
        int diff = n - s.size();

        vector<long long> memo(n, -1);
        function<long long(int, bool, bool)> dfs = [&](int i, bool limit_low, bool limit_high) -> long long {
            if (i == low.size()) {
                return 1;
            }

            if (!limit_low && !limit_high && memo[i] != -1) {
                return memo[i]; // 之前计算过
            }

            // 第 i 个数位可以从 lo 枚举到 hi
            // 如果对数位还有其它约束，应当只在下面的 for 循环做限制，不应修改 lo 或 hi
            int lo = limit_low ? low[i] - '0' : 0;
            int hi = limit_high ? high[i] - '0' : 9;

            long long res = 0;
            if (i < diff) { // 枚举这个数位填什么
                for (int d = lo; d <= min(hi, limit); d++) {
                    res += dfs(i + 1, limit_low && d == lo, limit_high && d == hi);
                }
            } else { // 这个数位只能填 s[i-diff]
                int x = s[i - diff] - '0';
                if (lo <= x && x <= min(hi, limit)) {
                    res = dfs(i + 1, limit_low && x == lo, limit_high && x == hi);
                }
            }

            if (!limit_low && !limit_high) {
                memo[i] = res; // 记忆化 (i,false,false)
            }
            return res;
        };
        return dfs(0, true, true);
    }
};

作者：灵茶山艾府
链接：https://leetcode.cn/problems/count-the-number-of-powerful-integers/solutions/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```







# 2 状压dp

## 2.1 位运算

- lowbit(最低位的1)：

  ```cpp
  lowbit = x & -x;
  ```

- 枚举x关于位为1的子集

  ```cpp
  for (int s = x; s; s = (s - 1) & x)
  ```

## 2.2 例题

> ```cpp
> // 会超时的递归代码
> class Solution {
> public:
>     int maxStudents(vector<vector<char>> &seats) {
>         int m = seats.size(), n = seats[0].size();
>         vector<int> a(m); // a[i] 是第 i 排可用椅子的下标集合
>         for (int i = 0; i < m; i++) {
>             for (int j = 0; j < n; j++) {
>                 if (seats[i][j] == '.') {
>                     a[i] |= 1 << j;
>                 }
>             }
>         }
> 
>         function<int(int, int)> dfs = [&](int i, int j) -> int {
>             if (i == 0) {
>                 if (j == 0) {
>                     return 0;
>                 }
>                 int lb = j & -j;
>                 return dfs(i, j & ~(lb * 3)) + 1;
>             }
>             int res = dfs(i - 1, a[i - 1]); // 第 i 排空着
>             for (int s = j; s; s = (s - 1) & j) { // 枚举 j 的子集 s
>                 if ((s & (s >> 1)) == 0) { // s 没有连续的 1
>                     int t = a[i - 1] & ~(s << 1 | s >> 1); // 去掉不能坐人的位置
>                     res = max(res, dfs(i - 1, t) + __builtin_popcount(s));
>                 }
>             }
>             return res;
>         };
>         return dfs(m - 1, a[m - 1]);
>     }
> };
> 
> 作者：灵茶山艾府
> 链接：https://leetcode.cn/problems/maximum-students-taking-exam/solutions/2580043/jiao-ni-yi-bu-bu-si-kao-dong-tai-gui-hua-9y5k/
> 来源：力扣（LeetCode）
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
> ```
>
> 
