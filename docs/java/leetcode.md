- [数组](#数组)
  - [二分法(待完善)](#二分法待完善)
    - [两种不同的写法](#两种不同的写法)
    - [35.搜索插入位置](#35搜索插入位置)
- [二叉树](#二叉树)
  - [层序遍历](#层序遍历)
  - [平衡二叉搜索树](#平衡二叉搜索树)
- [数组](#数组-1)
  - [滑动窗口](#滑动窗口)
- [回溯算法](#回溯算法)
  - [N皇后](#n皇后)
- [动态规划](#动态规划)
  - [题目类型划分](#题目类型划分)
    - [组合数，排列数](#组合数排列数)
  - [494.目标和](#494目标和)
  - [279.完全平方数](#279完全平方数)
  - [322.零钱兑换](#322零钱兑换)

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

# 二叉树

## 层序遍历

层序遍历也称作广度优先遍历，可以利用队列的`FIFO`特性，进行一层一层的遍历，代表题目有：

- [102.二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)
- [107.二叉树的层序遍历II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

下面是102题的代码实现：

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if (root != null) queue.add(root);
        while (!queue.isEmpty()) {
            // 每一层的节点数
            int size = queue.size();
            // 记录每一层的节点值
            List<Integer> tempList = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                TreeNode tempNode = queue.poll();
                tempList.add(tempNode.val);
                // 添加的顺序是从左往右，因此层序遍历的结果也是从左往右
                if (tempNode.left != null) queue.add(tempNode.left);
                if (tempNode.right != null) queue.add(tempNode.right);
            }
            res.add(tempList);
        }
        return res;
    }
}
```

102题是从树的根节点开始层序遍历，107题是从树的叶子节点开始层序遍历，与102相反。但遍历的思路是相同的，都是利用队列的`FIFO`特性，进行一层一层的遍历。

我们通过查看同一个树的遍历结果，102题是：

```java
[[3],[9,20],[15,7]]
```

107题是：

```java
[[15,7],[9,20],[3]]
```

我们102题向`res`中添加`tempList`是从尾部开始添加的，因此107题可以从头部开始添加，这样就可以得到从叶子节点开始的层序遍历结果。即将`res`的声明类型从`List<List<Integer>>`改为`LinkedList<List<Integer>>`，然后将`res.add(tempList)`改为`res.addFirst(tempList)`即可。

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

## 滑动窗口

代表题目：[209.长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)

这道题中要求长度最小的子数组，滑动窗口在这道题的思路就是：

- 只用一个for循环，当窗口内的元素大于等于目标数时，把窗口从左向右缩减

```java
for (int fastIndex = 0; fastIndex < length; fastIndex++) {
    sum += nums[fastIndex];
    while (sum >= target) {
        // 向左缩减的同时比较此时窗口长度是否最小
        res = Math.min(res, fastIndex - slowIndex + 1);
        sum -= nums[slowIndex++];
    }
}
```

- 本题要注意的细节是：

```java
// 窗口长度res一开始是整数最大值
int res = Integer.MAX_VALUE;
...
// 如果数组内所有元素加起来都小于目标数，要返回0，不能直接返回res
return res == Integer.MAX_VALUE ? 0 : res;
```

# 回溯算法

## N皇后

题目链接：[51.N皇后](https://leetcode-cn.com/problems/n-queens/)

回溯的经典题目，先想到利用回溯算法是第一步，这点不难，难在接下来的细节，如下：

- 如何判断当前棋盘位置能否下皇后？ -> `isValid`方法的实现
- 对于Java语言，棋盘采用`char[][]`的数据类型时，如何转换为需要返回的结果数据类型：`List<List<String>>`

首先第一步是回溯的方法实现：

```java
private void backtracking(int n, char[][] chessboard, int row) {
    // 当row == n时，说明已经成功摆放了n个皇后，可以返回
    if (row == n) {
        // res是一个全局变量，即我们最重要返回的结果
        res.add(Array2List(chessboard));
        return;
    }
    for (int col = 0; col < n; col++) {
        // 判断当前位置是否可以放皇后
        if (isValid(row, col, chessboard, n)) {
            chessboard[row][col] = 'Q';
            backtracking(n, chessboard, row + 1);
            // 回溯，将之前位置的皇后移除
            chessboard[row][col] = '.';
        }
    }
}
```

然后先看如何判断当前棋盘位置能否下皇后，即`isValid`方法的实现：

```java
private boolean isValid(int row, int col, char[][] chessboard, int n) {
    // 检查列
    for (int i = 0; i < row; i++) {
        if (chessboard[i][col] == 'Q') {
            return false;
        }
    }
    // 检查左上
    for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
        if (chessboard[i][j] == 'Q') {
            return false;
        }
    }
    // 检查右上
    for (int i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
        if (chessboard[i][j] == 'Q') {
            return false;
        }
    }
    return true;
}
```

当时写到这产生了几个问题：

- 为什么不检查右下和左下是否有皇后？
  - A：我们是通过行(`row`)来进行层序遍历的，因此在我们判断当前行`isValid`时，下方的行还没有放置皇后，因此不需要进行遍历判断
- 为什么不遍历行？
  - A：因为在`backtracking`方法中，我们回溯的时候将原本位置上的皇后移除了，保证每一行只会存在一个皇后，因此不需要遍历行

最后是如何将`char[][]`转换为`List<List<String>>`，即`Array2List`方法的实现：

```java
private List<String> Array2List(char[][] chessboard) {
    List<String> res = new ArrayList<>();
    for (char[] chars : chessboard) {
        // 将char[]转换为Strin
        res.add(String.copyValueOf(chars));
    }
    return res;
}
```

还有一个需要注意的细节是我们在初始化棋盘`char[][] chessboard`时，也要将棋盘内的值都初始化为`.`，因此主函数中的初始化为：

```java
public List<List<String>> solveNQueens(int n) {
    char[][] chessboard = new char[n][n];
    // 初始化棋盘
    for (char[] chars : chessboard) {
        Arrays.fill(chars, '.');
    }
    backtracking(n, chessboard, 0);
    return res;
}
```

完整代码见[这里](https://programmercarl.com/0051.N%E7%9A%87%E5%90%8E.html#java)

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
