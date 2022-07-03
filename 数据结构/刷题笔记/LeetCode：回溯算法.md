# LeetCode：回溯算法

> 这部分主要是学习了 **labuladong** 公众号中对于回溯算法的讲解
>
> 刷了些 leetcode 题，在此做一些记录，不然没几天就忘完了

## 总结

### 概述

回溯是 DFS 中的一种技巧。回溯法采用 **试错** 的思想，尝试分步的去解决一个问题，在分步解决问题的过程中，当它通过尝试发现现有的分步答案不能得到有效的正确的解答的时候，它将**取消上一步甚至是上几步的计算，再通过其它的可能的分步解答再次尝试寻找问题的答案**。

通俗上讲，回溯是一种走不通就回头的算法。

> 摘录自：https://leetcode-solution-leetcode-pp.gitbook.io/leetcode-solution/thinkings/backtrack

**在进行回溯时，需要考虑3个问题**

1. **路径**：也就是已经做出的选择。
2. **选择列表**：也就是你当前可以做的选择。
3. **结束条件**：也就是到达决策树底层，无法再做选择的条件。

算法框架如下：

```java
List<List<...>> res = new LinkedList<>();
// List<...> res = new LinkedList<>();

void backtrack(路径, 选择列表){
	if(满足结束条件){
        res.add(路径);
        return;
    }

    for(选择 :选择列表){
        // 做选择
        backtrack(路径, 选择列表)
        // 撤销选择
    }
}
```

> **特殊说明**：
>
> 在撤销选择时，若是如下定义：`List<List<Integer>> res = new LinkedList<>();`，`List`类若要移除最后一个元素：
>
> * 方式1：`track.remove(track.size() - 1);`，推荐，因为选择列表出现重复的元素时，方式2就会有问题！
> * 方式2：`track.remove(Integer.valueOf(nums[i]));`，直接`remove(i)`是不对的，因为 i 为 index 而不是添加的元素

### 经典问题

#### 排列问题

问题描述：[46. 全排列](https://leetcode-cn.com/problems/permutations/)

```java
List<List<Integer>> res = new LinkedList<>();

/* 主函数，输入一组不重复的数字，返回它们的全排列 */
List<List<Integer>> permute(int[] nums) {
    // 记录「路径」
    LinkedList<Integer> track = new LinkedList<>();
    // 输入：选择列表, 路径
    backtrack(nums, track);
    return res;
}

// 路径：记录在 track 中
// 选择列表：nums 中不存在于 track 的那些元素
// 结束条件：nums 中的元素全都在 track 中出现
void backtrack(int[] nums, LinkedList<Integer> track) {
    // 触发结束条件
    if (track.size() == nums.length) {
        res.add(new LinkedList(track));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        // 排除不合法的选择
        if (track.contains(nums[i]))
            continue;
        // 前序：做选择
        track.add(nums[i]);
        // 进入下一层决策树
        backtrack(nums, track);
        // 后序：取消选择
        track.removeLast();
        // 注意：若要用reomve的话，track.remove(i)是错误的，应该是track.remove(Integer.valueOf(i));
    }
}
```

> ```java
> public static void main(String[] args) {
>     int[] nums = {1,2,3};
>     Demo demo = new Demo();
>     demo.permute(nums);
> 
>     System.out.println(demo.res.toString());
>     // [[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]
> }
> ```

#### N皇后

题目描述：[51. N 皇后](https://leetcode-cn.com/problems/n-queens/)

> 输入：4
>
> 输出：`[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]`

```java
List<List<String>> res = new LinkedList<>();

/* 输入棋盘边长 n，返回所有合法的放置 */
public List<List<String>> solveNQueens(int n) {
    // '.' 表示空，'Q' 表示皇后，初始化空棋盘。
    List<char[]> board = new LinkedList<>();
	
    // 初始化路径
    for(int i=0; i<n; i++){
        char[] arr = new char[n];
        Arrays.fill(arr, '.');
        board.add(arr);
    }

    backtrack(board, 0);

    return res;
}

// 路径：board 中小于 row 的那些行都已经成功放置了皇后
// 选择列表：第 row 行的所有列都是放置皇后的选择
// 结束条件：row 超过 board 的最后一行
private void backtrack(List<char[]> board, int row){
    // 触发结束条件
    if(board.size() == row){
        res.add(transform(board));
        return;
    }
	
    // 第 row 行的所有列都是放置皇后的选择
    int n = board.get(row).length;
    for(int col=0; col<n; col++){
        // 排除不合法选择
        if(!isValid(board, row, col)){
            continue;
        }

        // 做选择
        board.get(row)[col] = 'Q';
        // 进入下一行决策,即row行已经放好皇后了，要在row+1行放
        backtrack(board, row+1);
        // 撤销选择
        board.get(row)[col] = '.';

    }
}

private Boolean isValid(List<char[]> board, int row, int col) {
    int n = board.size();

    // 遍历所有行，固定列：检查列是否有皇后互相冲突
    for(int i = 0; i < n; i++) {
        if(board.get(i)[col] == 'Q'){
            return false;
        }
    }

    // 检查右上方是否有皇后互相冲突，因为是一行一行放皇后因此只要检查上方
    for(int i = row - 1, j = col + 1; i >= 0 && j < n; i--,j++){
        if(board.get(i)[j] == 'Q'){
            return false;
        }
    }

    // 检查左上方是否有皇后互相冲突，因为是一行一行放皇后因此只要检查上方
    for(int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--){
        if(board.get(i)[j] == 'Q'){
            return false;
        }
    }
    return true;
}


private List<String> transform(List<char[]> board){
    List<String> newBoard = new LinkedList<>();
    for(char[] row : board){
        newBoard.add(new String(row));
    }
    return newBoard;
}
```

#### 子集问题

题目描述：[78. 子集](https://leetcode-cn.com/problems/subsets/)

```java
// 数学归纳思想
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> res = new LinkedList<>();
    // 先加入[]的情况
    res.add(new LinkedList<Integer>());

    for(int i=0; i<nums.length; i++){
		// 每多一个数字，都在原先的基础上添加这个数字再加到res中
        int size = res.size();
        for(int j=0; j<size; j++){
            List<Integer> sub = new LinkedList<>(res.get(j));
            sub.add(nums[i]);
            res.add(sub);
        }
    }
    return res;
}

// 回溯解法
List<List<Integer>> res = new LinkedList<>();

public List<List<Integer>> subsets(int[] nums) {

    List<Integer> track = new LinkedList<>();;
    backtrack(nums, 0, track);

    return res;
}

private void backtrack(int[] nums, int start, List<Integer> track){
    res.add(new LinkedList(track));
    // 这里的i=start做了可选集和的限定
    for(int i=start; i<nums.length; i++){
        // 做选择
        track.add(nums[i]);
        // 回溯
        backtrack(nums, i+1, track);
        // 撤销选择
        track.remove(Integer.valueOf(nums[i]));
    }
}
```

#### 组合问题

题目描述：[77. 组合](https://leetcode-cn.com/problems/combinations/)

```java
List<List<Integer>> res = new LinkedList<>();
public List<List<Integer>> combine(int n, int k) {

    List<Integer> track = new LinkedList<>();
	// k限定了尺寸，start限定了选择列表
    backtrack(n, k, 1, track);

    return res;
}

private void backtrack(int n, int k, int start, List<Integer> track){
    
    // 剪枝：可有可无，加上后效率提升很多
    if(track.size() + n-start + 1 < k){
        return;
    }

    if(k == track.size()){
        res.add(new LinkedList(track));
        return;
    }

    for(int i=start; i<=n; i++){
        track.add(i);
        backtrack(n, k, i+1, track);
        track.remove(Integer.valueOf(i));
    }
}
```

#### 排列问题

问题描述：[46. 全排列](https://leetcode-cn.com/problems/permutations/)，和之前的全排列一样，再贴一个代码吧

```java
List<List<Integer>> res = new LinkedList<>();

public List<List<Integer>> permute(int[] nums) {

    List<Integer> track = new LinkedList<>();
    backtrack(nums, track);
    return res;
}

private void backtrack(int[] nums, List<Integer> track){
    if(nums.length == track.size()){
        res.add(new LinkedList(track));
        return;
    }

    for(int i=0; i<nums.length; i++){
        // 和组合问题不同就是在这里多了一个判断，并且开始的索引不需要修改
        if(track.contains(nums[i])){
            continue;
        }
        track.add(nums[i]);
        backtrack(nums, track);
        track.remove(Integer.valueOf(nums[i]));
    }
}
```

#### 数独问题

问题描述：[37. 解数独](https://leetcode-cn.com/problems/sudoku-solver/)

```java
public void solveSudoku(char[][] board) {

    backtrack(board, 0, 0);
}

private boolean backtrack(char[][] board, int r, int c){

    int m=9, n=9;

    if(c == n){
        // 穷举到最后一列的话就换到下一行重新开始
        return backtrack(board, r+1, 0);
    }

    if(r == m){
        // 找到一个可行解，触发 base case
        return true;
    }

    // 对棋盘的每个位置进行穷举
    for(int i=r; i< m; i++){
        for(int j=c; j< n; j++){
            if(board[i][j] != '.'){
                // 如果该位置是预设的数字，不用我们操心
                return backtrack(board, i, j + 1);
            }
            // 对每个数都进行穷举
            for(char ch='1'; ch <= '9'; ch++){
                // 如果遇到不合法的数字，就跳过
                if( !isValid(board, i, j, ch)){
                    continue;
                }

                // 做选择:
                board[i][j] = ch;
                // 继续穷举下一个位置
                // 如果找到一个可行解，立即结束
                if(backtrack(board, i, j+1)){
                    return true;
                }
                // 撤销选择
                board[i][j] = '.';
            }
            // 穷举完 1~9，依然没有找到可行解，此路不通, 回溯
            return false;
        }
    }
    return false;
}

// 判断 board[i][j] 是否可以填入 n
private boolean isValid(char[][] board, int r, int c, char n) {
    for (int i = 0; i < 9; i++) {
        // 判断行是否存在重复
        if (board[r][i] == n) return false;
        // 判断列是否存在重复
        if (board[i][c] == n) return false;
        // 判断 3 x 3 方框是否存在重复
        if (board[(r/3)*3 + i/3][(c/3)*3 + i%3] == n) 
            return false;
    }
    return true;
}
```

#### 括号生成

题目描述：[22. 括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

```java
// 解法一：效率更高
List<String> res = new LinkedList<>();
public List<String> generateParenthesis(int n) {

    if(n == 0){
        return res;
    }

    // 回溯过程中的路径
    char[] track = new char[n*2];
    Arrays.fill(track, '.');
    backtrack(n, n, 0, track);

    return res;

}

// 可用的左括号数量为 left 个，可用的右括号数量为 rgiht 个,已经放的个数index
private void backtrack(int left, int right, int index, char[] track){

    // 若左括号剩下的多，说明不合法
    if(left > right){
        return;
    }
    // 数量小于 0 肯定是不合法的
    if(left < 0 || right < 0){
        return;
    }

    // 当所有括号都恰好用完时，得到一个合法的括号组合
    if(left == 0 && right == 0){
        res.add(new String(track));
        return;
    } 

    // 尝试放一个左括号
    track[index] = '(';
    backtrack(left-1, right, index+1, track);
    track[index] = '.';

    // 尝试放一个右括号
    track[index] = ')';
    backtrack(left, right-1, index+1, track);
    track[index] = '.';
}

// 解法二：效率没有解法一高
List<String> res = new ArrayList<>();

public List<String> generateParenthesis(int n) {

    StringBuilder track = new StringBuilder();

    backTrack(n, 0, 0, track);    

    return res;
}

private void backTrack(int n, int left, int right, StringBuilder track){

    // 剪枝: 当前用掉的右边括号大于用掉的左边的括号，不可能更加合理了，直接return
    if(left < right){
        return;
    }
	
    if(2*n == track.length()){
        String str = track.toString();
        if(isValid(str)){
            res.add(str);
        }
        return;
    }

    // 填充左括号
    // for(int i=left; i<n; i++){
        track.append('(');
        backTrack(n, i+1, right, track);
        track.deleteCharAt(track.length()-1);
    // }
	
    // 填充右括号
    // for(int i=right; i<n; i++){
        track.append(')');
        backTrack(n, left, i+1, track);
        track.deleteCharAt(track.length()-1);
    // }
}

// 验证此时的s是否合理
private boolean isValid(String s){

    int count = 0;

    for(int i=0; i<s.length(); i++){
        if(s.charAt(i) == '('){
            count++;
        }else if(s.charAt(i) == ')'){
            count--;
            if(count < 0){
                return false;
            }
        }else{
            return false;
        }
    }

    return count == 0;
}
```

### 回溯算法VS动态规划

题目描述：[494. 目标和](https://leetcode-cn.com/problems/target-sum/)

```java
// 回溯算法：击败30%
int res = 0;
public int findTargetSumWays(int[] nums, int S) {

    backbarck(nums, S, 0, 0);
    return res;
}

private void backbarck(int[] nums, int target, int start, int sum){

    if(start == nums.length){
        if(sum == target){
            res+=1;
        }
        return;
    }

    // 选择+
    backtrack(nums, target, sum+nums[start], start+1);
    // 选择-
    backtrack(nums, target, sum-nums[start], start+1);
}


// 存在子问题：加备忘录的优化
Map<String, Integer> cache = new HashMap<>();
public int findTargetSumWays(int[] nums, int S) {
    
    return dp(nums, S, 0, 0);
}

private int dp(int[] nums, int target, int start, int sum){

    if(start == nums.length){
        if(sum == target){
            return 1;
        }
        return 0;
    }

    // 把它俩转成字符串才能作为哈希表的键
    // 即：前面的计算相同，后面就不用去计算了，直接用之前的结果即可
    String key = start + "," + sum;

    // 避免重复计算
    if (cache.containsKey(key)) {
        return cache.get(key);
    }

    // 还是穷举
    int res = dp(nums, target, start+1, sum+nums[start]) + dp(nums, target, start+1, sum-nums[start]);
    cache.put(key, res);

    return res;
}

// 动态规划：把 nums 划分成两个子集A和B，分别代表分配+的数和分配-的数
// sum(A) - sum(B) = target
// sum(A) = target + sum(B)
// sum(A) + sum(A) = target + sum(B) + sum(A)
// 2 * sum(A) = target + sum(nums)
// sum(A) = (target + sum(nums)) / 2
// 把原问题转化成：nums中存在几个子集A，使得A中元素的和为 (target + sum(nums)) / 2
// 这不就是背包问题了嘛
public int findTargetSumWays(int[] nums, int S) {

    int n = nums.length;
    int sum = 0;

    for(int i=0; i<n; i++){
        sum += nums[i];
    }

    // 不可能的情况
    if(sum < S || (sum + S) % 2 == 1){
        return 0;
    }

    sum = (S + sum) / 2;

    // 1.明确状态：dp[i][j]  当有i个物体时，背包容量为j时，有几种方式能装满背包
    int[][] dp = new int[n+1][sum+1];

    // 2.base case
    for(int i=0; i<=n; i++){
        // 有东西可以装  但是背包已经满了  啥也不做是一种情况
        dp[i][0] = 1;
    }
    for(int j=1; j<=sum; j++){
        // 没东西了 但背包还没满 没有操作空间
        dp[0][j] = 0;
    }

    // 3.状态转移：
    for(int i=1; i<=n; i++){       // 可选物品的状态转移
        for(int j=0; j<=sum; j++){ // 背包容量的状态转移，这里注意的是从0开始转移，因为存在重量为0的物体
            if(nums[i-1] > j){
                // 东西放不进背包,继承之前的状态
                dp[i][j] = dp[i-1][j];
            }else{
                // 能放进去的情况
                dp[i][j] = dp[i-1][j-nums[i-1]] + dp[i-1][j];
            }
        }  
    }

    return dp[n][sum];
}
```

## 题目

### 索引

[39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

[40. 组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii/)

[216. 组合总和 III](https://leetcode-cn.com/problems/combination-sum-iii/)

[47. 全排列 II](https://leetcode-cn.com/problems/permutations-ii/)

[90. 子集 II](https://leetcode-cn.com/problems/subsets-ii/)

[131. 分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)

[93. 复原 IP 地址](https://leetcode-cn.com/problems/restore-ip-addresses/)

### 解题

#### [39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

```java
List<List<Integer>> res = new LinkedList<>();
public List<List<Integer>> combinationSum(int[] candidates, int target) {

    List<Integer> track = new LinkedList<>();
    Arrays.sort(candidates);
    backtrack(candidates, target, 0, 0, track);

    return res;
}

// param1：可选的数组
// param2: 目标值
// param3: 当前的总和
// param4: 开始的索引，为了避免重复的组合出现多次
// param5：路径
private void backtrack(int[] candidates, int target, int sum, int start, List<Integer> track){

    if(sum >= target){
        if(sum == target){
            res.add(new LinkedList<>(track));
        }
        return;
    }

    // 剪枝：可有可无，有了效率高
    if(sum+candidates[start] > target){
        return;
    }

    for(int i=start; i<candidates.length; i++){
        sum += candidates[i];
        track.add(candidates[i]);
        backtrack(candidates, target, sum, i, track);
        sum -= candidates[i];
        track.remove(Integer.valueOf(candidates[i]));
    }
}
```

#### [40. 组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii/)

```java
// 耗时 917ms
List<List<Integer>> res = new LinkedList<>();
// 用于去重，虽然效率很低，但能实现去重的效果
Set<String> visited = new HashSet<>();
public List<List<Integer>> combinationSum2(int[] candidates, int target) {

    List<Integer> track = new LinkedList<>();

    Arrays.sort(candidates);

    backtrack(candidates, target, 0, 0, track);

    return res;
}

private void backtrack(int[] candidates, int target, int sum, int start, List<Integer> track){

    if(sum >= target){
        if(sum == target && !visited.contains(track.toString())){
            res.add(new LinkedList<>(track));
            visited.add(track.toString());
        }
        return;
    }

    for(int i=start; i<candidates.length; i++){

        sum += candidates[i];
        track.add(candidates[i]);
        backtrack(candidates, target, sum, i+1, track);
        sum -= candidates[i];
        track.remove(Integer.valueOf(candidates[i]));
    }
}

// 优化版本，加入剪枝提高效率，耗时：4ms
List<List<Integer>> res = new LinkedList<>();
boolean[] visited;
public List<List<Integer>> combinationSum2(int[] candidates, int target) {

    List<Integer> track = new LinkedList<>();
    visited = new boolean[candidates.length];

    Arrays.sort(candidates);
    backtrack(candidates, target, 0, 0, track);

    return res;
}

private void backtrack(int[] candidates, int target, int sum, int start, List<Integer> track){

    if(sum >= target){
        if(sum == target){
            res.add(new LinkedList<>(track));
        }
        return;
    }

    // 剪枝1：
    if(start<candidates.length && sum + candidates[start] > target){
        return;
    }

    for(int i=start; i<candidates.length; i++){

        // 剪枝2：☆☆☆如果数组相连元素相等，没有先访问后面的元素，就不会存在重复
        if (visited[i] || (i > 0 && candidates[i] == candidates[i - 1] && !visited[i - 1])) {
            continue;
        }

        sum += candidates[i];
        track.add(candidates[i]);
        visited[i] = true;
        backtrack(candidates, target, sum, i+1, track);
        sum -= candidates[i];
        track.remove(Integer.valueOf(candidates[i]));
        visited[i] = false;
    }
}
```

#### [216. 组合总和 III](https://leetcode-cn.com/problems/combination-sum-iii/)

```java
List<List<Integer>> res = new LinkedList<>();

public List<List<Integer>> combinationSum3(int k, int n) {

    List<Integer> track = new LinkedList<>();

    backtrack(k, n, 1, 0, track);

    return res;
}

private void backtrack(int k, int n, int start, int sum, List<Integer> track){

    if(track.size() == k && sum == n){
        res.add(new LinkedList<>(track));
        return;
    }
    
    // 剪枝: 可有可无，有了提高效率
    if(start>9 || sum > n || sum+start > n){
        return;
    }

    for(int i=start; i<=9; i++){
        sum += i;
        track.add(i);
        backtrack(k, n, i+1, sum, track);
        sum -= i;
        track.remove(Integer.valueOf(i));
    }
}
```

#### [47. 全排列 II](https://leetcode-cn.com/problems/permutations-ii/)

```java
// 方法1：通过set来去除重复的组合情况
boolean[] visited;
List<List<Integer>> res = new ArrayList<>();
Set<String> set = new HashSet<>();

public List<List<Integer>> permuteUnique(int[] nums) {

    visited = new boolean[nums.length];

    List<Integer> track = new ArrayList<>();

    backtrack(nums, track);

    return res;
}

private void backtrack(int[] nums, List<Integer> track){

    if(track.size() == nums.length){
        String temp = track.toString();
        if(!set.contains(temp)){
            set.add(temp);
            res.add(new ArrayList(track));
        }
        return;
    }

    for(int i=0; i<nums.length; i++){

        if(visited[i]){
            continue;
        }

        track.add(nums[i]);
        visited[i] = true;
        backtrack(nums, track);
        visited[i] = false;
        track.remove(track.size()-1);
    }
}

// 方法2：优化了去重的方式，效率由7% ---> 99%
boolean[] visited;
List<List<Integer>> res = new ArrayList<List<Integer>>();

public List<List<Integer>> permuteUnique(int[] nums) {

    List<Integer> track = new ArrayList<Integer>();
    visited = new boolean[nums.length];

    // 这里是需要排序的，不然无法优化
    Arrays.sort(nums);
    // 开始回溯
    backtrack(nums, track);

    return res;
}

private void backtrack(int[] nums, List<Integer> track) {
    if (track.size() == nums.length) {
        res.add(new ArrayList<Integer>(track));
        return;
    }
    for (int i = 0; i < nums.length; ++i) {
        // ☆☆☆如果数组相连元素相等，没有先访问后面的元素，就不会存在重复
        if (visited[i] || (i > 0 && nums[i] == nums[i - 1] && !visited[i - 1])) {
            continue;
        }
        track.add(nums[i]);
        visited[i] = true;
        backtrack(nums, track);
        visited[i] = false;
        track.remove(track.size()-1);
    }
}
```

#### [90. 子集 II](https://leetcode-cn.com/problems/subsets-ii/)

```java
// 回溯解法：正常的模版解题
List<List<Integer>> res = new LinkedList<>();
Set<String> visited = new HashSet<>();

public List<List<Integer>> subsetsWithDup(int[] nums) {

    List<Integer> track = new LinkedList<>();
    // 因为集和具有无序性，因此需要排序来排除顺序造成的重复子集
    // 即：两个子集如果只是元素的排列顺序不一样，则认为重复
    Arrays.sort(nums);
    backtrack(nums, 0, track);

    return res;
}

private void backtrack(int[] nums, int start, List<Integer> track){

    if(!visited.contains(track.toString())){
        visited.add(track.toString());
        res.add(new LinkedList(track));
    }else{
        return;
    }

    for(int i=start; i<nums.length; i++){
        // 做选择
        track.add(nums[i]);
        // 回溯
        backtrack(nums, i+1, track);
        // 撤销选择
        track.remove(Integer.valueOf(nums[i]));
    }
}

// 回溯解法：+ 剪枝
List<List<Integer>> res = new LinkedList<>();
boolean[] visited;

public List<List<Integer>> subsetsWithDup(int[] nums) {

    visited = new boolean[nums.length];
    List<Integer> track = new LinkedList<>();
    // 因为集和具有无序性，因此需要排序来排除顺序造成的重复子集
    // 即：两个子集如果只是元素的排列顺序不一样，则认为重复
    Arrays.sort(nums);
    backtrack(nums, 0, track);

    return res;
}

private void backtrack(int[] nums, int start, List<Integer> track){

    res.add(new LinkedList(track));

    for(int i=start; i<nums.length; i++){

        // ☆☆☆如果数组相连元素相等，没有先访问后面的元素，就不会存在重复
        if (visited[i] || (i > 0 && nums[i] == nums[i - 1] && !visited[i - 1])) {
            // 这个判断条件：
            // 	1. i位置元素还没访问，若不满足这个判断，则要访问i位置元素了
            //  2. i存在前一个元素和i位置元素值相同，但是前一个元素还未被访问！
            continue;
        }

        // 做选择
        track.add(nums[i]);
        visited[i] = true;
        // 回溯
        backtrack(nums, i+1, track);
        // 撤销选择
        track.remove(track.size() - 1);
        visited[i] = false;
    }
}
```

#### [131. 分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)

```java
List<List<String>> res = new LinkedList<>();

public List<List<String>> partition(String s) {
    List<String> track = new LinkedList<>();

    backtrack(s, 0, track);

    return res;
}

private void backtrack(String s, int start, List<String> track){
    if(start == s.length()){
        res.add(new LinkedList<>(track));
        return;
    }

    for(int i=start; i<s.length(); i++){
        String sub = s.substring(start, i+1);
        if(!isPalindrome(sub)){
            continue;
        }
        track.add(sub);
        backtrack(s, i+1, track);
        track.remove(track.size()-1);
    }
}

private boolean isPalindrome(String s){
    if(s == null || s.length() <= 1){
        return true;
    }

    int left = 0;
    int right = s.length() - 1;

    while(left < right){
        if(s.charAt(left) == s.charAt(right)){
            left++;
            right--;
        }else{
            return false;
        }
    }

    return true;
}
```

#### [93. 复原 IP 地址](https://leetcode-cn.com/problems/restore-ip-addresses/)

```java
List<String> res = new ArrayList<>();

public List<String> restoreIpAddresses(String s) {

    StringBuilder track = new StringBuilder();

    if(s.length() > 16){
        return res;
    }

    backtrack(s, 0, 0, track);

    return res;
}

private void backtrack(String s, int start, int idx, StringBuilder track){

    // 回溯结束条件: 4个id已经满足 + s用完了
    if(idx == 4 && start == s.length()){
        res.add(track.toString().substring(0, track.length()-1));
        return;
    }

    for(int i=start; i<s.length(); i++){

        String str = s.substring(start, i+1);

        if(!isValid(str)){
            continue;
        }

        track.append(str + ".");
        backtrack(s, i+1, idx+1, track);
        track.delete(track.length()-str.length()-1, track.length());
    }
}

private boolean isValid(String str){

    // 两位数但是用0开头的 || 长度超过3了
    if((str.length() > 1 && str.charAt(0) == '0') || str.length() > 3){
        return false;
    }

    int num = Integer.valueOf(str);
    if(num > 255 || num < 0){
        return false;
    }

    return true;
}
```

