- [动态规划](#动态规划)
    - [494.目标和](#494目标和)


# 动态规划

### 494.目标和

这道题感觉看代码随想录上写的不是很清晰，结合了leetcode官网该题评论的第一条后豁然开朗，在这里记录下以便之后回顾。

题目要求给一个数组`nums`和一个目标数`S`，求出数组中的元素通过`+`或`-`组合成`S`的方案数。

举例：`nums=[1, 1, 1, 1, 1]`, `S: 3`

**思路介绍**：

我们可以讲nums分为正子集P和负子集N，那么目标数和正负子集之间的关系是：

$$
sum(P)-sum(N)=S
$$

我们可以对上式做进一步的推导如下：

$$
sum(P)-sum(N)+sum(P)+sum(N)=S+sum(P)+sum(N) \\
2\times sum(P)=S+sum(nums)
$$

因此可以得到$sum(P)=(S+sum(s))\div2$。

> 当时到这里卡住，拿到这个公式有什么用呢？

答案是我们可以将问题简化为：`nums`有多少种组合方案可以得到$sum(P)$（省去了减法，只考虑加法，就转换成背包问题了，Amazing！）

转换为背包问题，那么`sum(P)`就是背包的容量，那么状态转移方程就是：

$$
dp[j]=dp[j-num[i]]
$$
> 这个状态转移方程常用来求**装满背包有几种方法**

其中`j=sum(P); j>num[i]`，到这里程式的思路应该就很清晰了，实现的程式如下所示：

```java
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        // 官方有个案例是nums=[100], S=-200
        // sum<target || (target + sum) % 2 !=0 对上例无效
        // 并且会造成capacity过大发生OOM，因此需要加上下方判断
        if (target < 0 && sum < -target) {
            return 0;
        }
        if ((target + sum) % 2 != 0) {
            return 0;
        }
        // 背包容量sum(P)=(target+sum(nums))/2
        int capacity = (target + sum) >>> 1;
        int[] dp = new int[capacity + 1];
        dp[0] = 1;
        for (int num : nums) {
            for (int i = capacity; i >= num; i--) {
                dp[i] += dp[i - num];
            }
        }
        return dp[capacity];
    }
}
```