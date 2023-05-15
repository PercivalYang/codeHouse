- [二叉树](#二叉树)
  - [平衡二叉搜索树](#平衡二叉搜索树)
- [数组](#数组)
  - [二分法(待完善)](#二分法待完善)
    - [两种不同的写法](#两种不同的写法)
    - [35.搜索插入位置](#35搜索插入位置)
- [动态规划](#动态规划)
  - [题目类型划分](#题目类型划分)
    - [组合数，排列数](#组合数排列数)
  - [494.目标和](#494目标和)
  - [279.完全平方数](#279完全平方数)
  - [322.零钱兑换](#322零钱兑换)

# 二叉树

## 平衡二叉搜索树

题目链接：[1382.将二叉搜索树变平衡](https://leetcode-cn.com/problems/balance-a-binary-search-tree/)

这道题其实是将 [108.将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/) 和 [98.验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/) 结合起来的题目。

题目给的是二叉搜索树，因此通过中序遍历后一定会得到一个递增的数组(98题的思路：利用中序遍历验证二叉搜索树)，然后将该有序数组转换成平衡二叉搜索树(108题的思路：利用二分法将有序数组转换成平衡二叉搜索树)。

```java
class Solution {
    public TreeNode balanceBST(TreeNode root) {
        List<Integer> list = new ArrayList<>();
        traversal(root, list);
        return balanceBST(list, 0, list.size() - 1);
    }

    private void traversal(TreeNode root, List<Integer> list) {
        // 中序遍历二叉搜索树，得到一个递增的数组list
        if (root == null) return;
        traversal(root.left, list);
        list.add(root.val);
        traversal(root.right, list);
    }

    private TreeNode balanceBST(List<Integer> list, int left, int right) {
        // 二分法将有序数组转换成平衡二叉搜索树
        // 注意迭代的left和right下标范围
        if (left > right) return null;
        int middle = left + (right - left) / 2;
        return new TreeNode(list.get(middle),
                balanceBST(list, left, middle - 1),
                balanceBST(list, middle + 1, right));
    }
}
```

# 数组

## 二分法(待完善)

### 两种不同的写法

**左闭右闭**

```java
while(left <= right){ // 此时是[left,right]
    int middle = left + ((right - left) >> 1);
    if (nums[middle] > target)
        right = middle - 1; // 因为middle位置的元素在本轮已经比较过了，因此下一个区间内不应该包含middle处的元素
    else if (nums[middle] < target)
        left = middle + 1; // 同理如上
    else
        return middle;
}
```

**左闭右开**

```java
while(left < right){ // 此时是[left,right)
    int middle = left + ((right - left) >> 1);
    if (nums[middle] > target)
        right = middle; // 同理如上，但是此刻right是开区间，所以不需要-1
    else if (nums[middle] < target)
        left = middle + 1; 
    else
        return middle;
}
```

### 35.搜索插入位置

# 动态规划

## 题目类型划分

### 组合数，排列数

**区别**

组合不强调元素之间的顺序，排列强调元素之间的顺序。例如：`[1, 2, 3]`和`[2, 1, 3]`是两种不同的组合，但是是同一种排列。

若想得到组合数，通常先遍历背包，再遍历物品

```java
for (coin : coins){
    for (int i = coin; i <= target; i ++){
        dp[i] += dp[i - coin];
    }
}
```

若想得到排列数，通常先遍历物品，再遍历背包

```java
for (int i = 1; i <= target; i ++){
    for (coin : coins){
        if( i >= coin){
            dp[i] += dp[i - coin];
        }
    }
}
```

**组合数**

- [377.组合求和IV](https://leetcode.cn/problems/combination-sum-iv/)

**排列数**

- [518.零钱兑换II](https://leetcode.cn/problems/coin-change-ii/)

## 494.目标和

[link](https://leetcode.cn/problems/target-sum/)

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

## 279.完全平方数

[link](https://leetcode-cn.com/problems/perfect-squares/)

这道题不限制物品的数量，因此是一道完全背包问题。

同时排列数和组合数的思路都可以用在这道题上，即先遍历物品或背包不影响最终结果，因为是求组合物品个数的最小值，不管组合数还是排列数的遍历方法都会包含这个最小物品个数。

先遍历背包的程序如下：

```java
public int numSquares(int n) {
    int[] dp = new int[n + 1];
    int max = Integer.MAX_VALUE;
    for (int i = 1; i < dp.length; i++) {
        dp[i] = max;
    }
    dp[0] = 0;
    // 先遍历背包
    for (int i = 0; i <= n; i++) {
        // j*j指平方数，即本题的“物品价值”
        for (int j = 1; j*j <= i; j++) {
            dp[i] = Math.min(dp[i - j * j] + 1, dp[i]);
        }
    }
    return dp[n];
}
```

先遍历物品的程序如下：

```java
public int numSquares(int n) {
    int[] dp = new int[n + 1];
    int max = Integer.MAX_VALUE;
    for (int i = 1; i < dp.length; i++) {
        dp[i] = max;
    }
    dp[0] = 0;
    // 先遍历物品
    for (int i = 1; i * i <= n; i++) {
        // 再遍历背包，注意背包起点不是从0开始
        // 因为要满足 j-i*i >= 0，不然就IndexOutOfBound了
        for (int j = i * i; j <= n; j++) {
            dp[j] = Math.min(dp[j - i * i] + 1, dp[j]);
        }
    }
    return dp[n];
}
```

与这道题类似的是[322.零钱兑换](https://leetcode-cn.com/problems/coin-change/)

## 322.零钱兑换

[link](https://leetcode-cn.com/problems/coin-change/)

与[279.完全平方数](https://leetcode-cn.com/problems/perfect-squares/)类似，只是这道题可能不存在满足目标数的组合，需要返回`-1`。

```java
...
// 前面的和完全平方数一致
    // 如果等于max说明dp[i-coin]没有满足的组合
    if (dp[i - coin] != max) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
    }
...
// 根据背包内容判断是否返回-1
if (dp[amount] != max) {
    return dp[amount];
}
return -1;
```
