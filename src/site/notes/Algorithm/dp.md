---
{"dg-publish":true,"permalink":"/algorithm/dp/"}
---



## 股票问题

### 买卖股票最佳时机

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。

```Java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices==null||prices.length==0){
            return 0;
        }
        int n =prices.length;
        int min = Integer.MAX_VALUE;
        int max = Integer.MIN_VALUE;
        for(int i=0;i<n;i++){
            min = Math.min(min,prices[i]);
            max = Math.max(max,prices[i]-min);
        }
        return max;
    }
}
```

### 买卖股票最佳时机2

给你一个整数数组 `prices` ，其中 `prices[i]` 表示某支股票第 `i` 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。

返回 _你能获得的 **最大** 利润_ 。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices==null||prices.length==0){
            return 0;
        }
        int n = prices.length;
        int[][] dp = new int[n][2];
        dp[0][0] = -prices[0];
        for(int i=1;i<n;i++){
            //持有股票
            dp[i][0] = Math.max(dp[i-1][0],dp[i-1][1]-prices[i]);
            //未持有股票
            dp[i][1] = Math.max(dp[i-1][1],dp[i-1][0]+prices[i]);
        }
        return dp[n-1][1];
    }
}
```


### 买卖股票的最佳时机3

给定一个数组，它的第 `i` 个元素是一支给定的股票在第 `i` 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 **两笔** 交易。

**注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length<=1){
            return 0;
        }
        int maxK = 2;
        int n = prices.length;
        int [][][] dp=new int[n][maxK+1][2];

        for(int i=0;i<n;i++){
            for(int k=maxK;k>=1;k--){
                //处理最基本的情况
                if(i==0){
                    //未持有股票，收益为0
                    dp[i][k][0] = 0;
                    //最开始购入股票，收益为负的第一天股票价钱
                    dp[i][k][1] = -prices[0];
                    continue;
                }
                //未持有股票,与前一天的情况一致，同样未购入股票；或者当天卖掉前一天的股票
                dp[i][k][0] = Math.max(dp[i-1][k][0],dp[i-1][k][1]+prices[i]);
                //持有股票，依旧持有股票未出售；或者当天买入一支股票。
                dp[i][k][1] = Math.max(dp[i-1][k][1],dp[i-1][k-1][0]-prices[i]);
            }
        }
        return dp[n-1][maxK][0];
    }
}
```


### 买卖股票的最佳时机4

给你一个整数数组 `prices` 和一个整数 `k` ，其中 `prices[i]` 是某支给定的股票在第 `i` 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 `k` 笔交易。也就是说，你最多可以买 `k` 次，卖 `k` 次。

**注意：**你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。


```java
class Solution {
    public int maxProfit(int k, int[] prices) {
        if(prices.length<=1){
            return 0;
        }
        int maxK = k;
        int n = prices.length;
        int [][][] dp=new int[n][maxK+1][2];

        for(int i=0;i<n;i++){
            for(int j=maxK;j>=1;j--){
                //处理最基本的情况
                if(i==0){
                    //未持有股票，收益为0
                    dp[i][j][0] = 0;
                    //最开始购入股票，收益为负的第一天股票价钱
                    dp[i][j][1] = -prices[0];
                    continue;
                }
                //未持有股票,与前一天的情况一致，同样未购入股票；或者当天卖掉前一天的股票
                dp[i][j][0] = Math.max(dp[i-1][j][0],dp[i-1][j][1]+prices[i]);
                //持有股票，依旧持有股票未出售；或者当天买入一支股票。
                dp[i][j][1] = Math.max(dp[i-1][j][1],dp[i-1][j-1][0]-prices[i]);
            }
        }
        return dp[n-1][maxK][0];
    }
}
```


## 打家劫舍

### 打家劫舍1

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。

给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。

```java
class Solution {
    public int rob(int[] nums) {
        int len = nums.length;
        if(len==0){
            return 0;
        }
        int[] dp = new int[len+1];
        dp[0] = 0;
        dp[1] = nums[0];
        for(int i=2;i<len+1;i++){
            dp[i] = Math.max(nums[i-1]+dp[i-2],dp[i-1]);
        }
        return dp[len];

    }
}
```

### 打家劫舍2

你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 **围成一圈** ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警** 。

给定一个代表每个房屋存放金额的非负整数数组，计算你 **在不触动警报装置的情况下** ，今晚能够偷窃到的最高金额。

```java
class Solution {
    public int rob(int[] nums) {
        if(null==nums||nums.length==0){
            return 0;
        }
        if(nums.length==1){
            return nums[0];
        }
        return Math.max(myRob(Arrays.copyOfRange(nums,0,nums.length-1)),myRob(Arrays.copyOfRange(nums,1,nums.length)));
    }

    private int myRob(int[] nums){
        int length = nums.length;
        int[] dp = new int[length+1];
        dp[0] = 0;
        dp[1] = nums[0];
        for(int i=2;i<length+1;i++){
            dp[i] = Math.max(dp[i-1],dp[i-2]+nums[i-1]);
        }
        return dp[length];
    }
}
```


### 打家劫舍3

小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为 `root` 。

除了 `root` 之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果 **两个直接相连的房子在同一天晚上被打劫** ，房屋将自动报警。

给定二叉树的 `root` 。返回 _**在不触动警报的情况下** ，小偷能够盗取的最高金额_ 。


```Java
class Solution {
    public int rob(TreeNode root) {
        int[] rootStatus = dfs(root);
        return Math.max(rootStatus[0], rootStatus[1]);
    }

    public int[] dfs(TreeNode node) {
        if (node == null) {
            return new int[]{0, 0};
        }
        int[] l = dfs(node.left);
        int[] r = dfs(node.right);
        int selected = node.val + l[1] + r[1];
        int notSelected = Math.max(l[0], l[1]) + Math.max(r[0], r[1]);
        return new int[]{selected, notSelected};
    }
}
```


## 课程表

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程  `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

```Java
import java.util.ArrayList;  
import java.util.List;  
  
class Solution {  
  
    List<List<Integer>> list;  
    int[] rudu;  
    int[] visited;  
    boolean valid = true;  
  
    public boolean canFinish(int numCourses, int[][] prerequisites) {  
        list = new ArrayList<>();  
        for(int i=0;i<numCourses;i++){  
            list.add(new ArrayList<Integer>());  
        }  
        rudu = new int[numCourses];  
        visited = new int[numCourses];  
        for(int[] pre : prerequisites){  
            list.get(pre[1]).add(pre[0]);  
            rudu[pre[0]]++;  
        }  
        for(int i =0;i<numCourses&&valid;i++){  
            if(visited[i]==0){  
                dfs(i);  
            }  
        }  
  
        return valid;  
  
    }  
  
    private void dfs(int v){  
        visited[v] = 1;  
        for(int tmp : list.get(v)){  
            if(visited[tmp]==0){  
                dfs(tmp);  
                if(!valid){  
                    return;  
                }  
            }else if(visited[tmp]==1){  
                valid = false;  
                return;  
            }  
        }  
        visited[v] = 2;  
    }  
}
```


## 课程表2
现在你总共有 `numCourses` 门课需要选，记为 `0` 到 `numCourses - 1`。给你一个数组 `prerequisites` ，其中 `prerequisites[i] = [ai, bi]` ，表示在选修课程 `ai` 前 **必须** 先选修 `bi` 。

- 例如，想要学习课程 `0` ，你需要先完成课程 `1` ，我们用一个匹配来表示：`[0,1]` 。

返回你为了学完所有课程所安排的学习顺序。可能会有多个正确的顺序，你只要返回 **任意一种** 就可以了。如果不可能完成所有课程，返回 **一个空数组** 。

```Java
import java.util.*;  
  
class Solution {  
    public int[] findOrder(int numCourses, int[][] prerequisites) {  
        //使用拓扑排序  
        //定义入度  
        int[] inDegree = new int[numCourses];  
        //有来保存课程和其对应的后续课程  
        Map<Integer, List<Integer>> map = new HashMap<>();  
        Queue<Integer> queue = new LinkedList<>();  
        //创建入度表和哈希表  
        for(int i=0;i<prerequisites.length;i++){  
            inDegree[prerequisites[i][0]]++;  
            if(!map.containsKey(prerequisites[i][1])){  
                map.put(prerequisites[i][1],new ArrayList<Integer>());  
            }  
            map.get(prerequisites[i][1]).add(prerequisites[i][0]);  
        }  
        //遍历将入度为0的index入队  
        List<Integer> res = new ArrayList<>();  
        for(int i=0;i<numCourses;i++){  
            if(inDegree[i]==0){  
                queue.offer(i);  
            }  
        }  
  
        //BFS  
        while(!queue.isEmpty()){  
            int cur = queue.poll();  
            res.add(cur);  
            if(map.containsKey(cur)){  
                for(Integer n : map.get(cur)){  
                    inDegree[n]--;  
                    if(inDegree[n]==0){  
                        queue.offer(n);  
                    }  
                }  
            }  
        }  
        return res.size() == numCourses ? res.stream().mapToInt(Integer::valueOf).toArray() : new int[0];  
    }  
}

```


## 97、交错字符串
给定三个字符串 `s1`、`s2`、`s3`，请你帮忙验证 `s3` 是否是由 `s1` 和 `s2` **交错** 组成的。

两个字符串 `s` 和 `t` **交错** 的定义与过程如下，其中每个字符串都会被分割成若干 **非空** 子字符串：

- `s = s1 + s2 + ... + sn`
- `t = t1 + t2 + ... + tm`
- `|n - m| <= 1`
- **交错** 是 `s1 + t1 + s2 + t2 + s3 + t3 + ...` 或者 `t1 + s1 + t2 + s2 + t3 + s3 + ...`

**注意：**`a + b` 意味着字符串 `a` 和 `b` 连接。

```Java
class Solution {  
    public boolean isInterleave(String s1, String s2, String s3) {  
        int l1 = s1.length();  
        int l2 = s2.length();  
        int l3 = s3.length();  
        if(l1+l2 != l3){  
            return false;  
        }  
        boolean[][] dp = new boolean[l1+1][l2+1];  
        dp[0][0] = true;  
        for(int i=1;i<l1+1;i++){  
            dp[i][0] = dp[i-1][0] && s1.charAt(i-1) == s3.charAt(i-1);  
        }  
        for(int i=1;i<l2+1;i++){  
            dp[0][i] = dp[0][i-1] && s2.charAt(i-1) == s3.charAt(i-1);  
        }  
        for(int i=1;i<l1+1;i++){  
            for(int j=1;j<l2+1;j++){  
                dp[i][j] = (dp[i][j-1] && s2.charAt(j-1)==s3.charAt(i+j-1)) || (dp[i-1][j] && s1.charAt(i-1)==s3.charAt(i+j-1));  
            }  
        }  
        return dp[l1][l2];  
    }  
}
```


## [环形子数组的最大和](https://leetcode.cn/problems/maximum-sum-circular-subarray/)
给定一个长度为 `n` 的**环形整数数组** `nums` ，返回 _`nums` 的非空 **子数组** 的最大可能和_ 。

**环形数组** 意味着数组的末端将会与开头相连呈环状。形式上， `nums[i]` 的下一个元素是 `nums[(i + 1) % n]` ， `nums[i]` 的前一个元素是 `nums[(i - 1 + n) % n]` 。

**子数组** 最多只能包含固定缓冲区 `nums` 中的每个元素一次。形式上，对于子数组 `nums[i], nums[i + 1], ..., nums[j]` ，不存在 `i <= k1, k2 <= j` 其中 `k1 % n == k2 % n` 。

```Java
class Solution {  
    public int maxSubarraySumCircular(int[] nums) {  
        int n = nums.length;  
        int[] leftMax = new int[n];  
        leftMax[0] = nums[0];  
        int leftNum = nums[0];  
        int pre = nums[0];  
        int res = nums[0];  
        for(int i=1;i<n;i++){  
            pre = Math.max(pre+nums[i],nums[i]);  
            res = Math.max(pre,res);  
            leftNum += nums[i];  
            leftMax[i] = Math.max(leftMax[i-1],leftNum);  
        }  
        //从右到左枚举后缀，固定后缀，选择最大前缀  
        int rightNum = 0;  
        for(int i=n-1;i>0;i--){  
            rightNum += nums[i];  
            res = Math.max(res,rightNum+leftMax[i-1]);  
        }  
        return res;  
    }  
}
```