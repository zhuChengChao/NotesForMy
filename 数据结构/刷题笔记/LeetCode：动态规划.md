# LeetCode：动态规划

> 动态规划永远的神
>
> 这部分主要是学习了 **labuladong** 公众号中对于动态规划的讲解
>
> 刷了些 leetcode 题，在此做一些记录，不然没几天就忘光光了

## 题目

> 这部分内容直接上题目了，解题的路子都体现在题里了

[70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

[509. 斐波那契数](https://leetcode-cn.com/problems/fibonacci-number/)

***

[5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

[516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

***

[300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

[53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

[354. 俄罗斯套娃信封问题](https://leetcode-cn.com/problems/russian-doll-envelopes/)

***

[1143. 最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

[583. 两个字符串的删除操作](https://leetcode-cn.com/problems/delete-operation-for-two-strings/)

[712. 两个字符串的最小ASCII删除和](https://leetcode-cn.com/problems/minimum-ascii-delete-sum-for-two-strings/)

[72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

***

[416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)

[279. 完全平方数](https://leetcode-cn.com/problems/perfect-squares/)

[322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

[518. 零钱兑换 II](https://leetcode-cn.com/problems/coin-change-2/)

***

[10. 正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

[887. 鸡蛋掉落](https://leetcode-cn.com/problems/super-egg-drop/)

[198. 打家劫舍](https://leetcode-cn.com/problems/house-robber/)

[213. 打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

[337. 打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii/)

[121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

[122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

[123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

[188. 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

[309. 最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

[714. 买卖股票的最佳时机含手续费](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

***

[139. 单词拆分](https://leetcode-cn.com/problems/word-break/)

[91. 解码方法](https://leetcode-cn.com/problems/decode-ways/)

[392. 判断子序列](https://leetcode-cn.com/problems/is-subsequence/)

[62. 不同路径](https://leetcode-cn.com/problems/unique-paths/)

[64. 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

[120. 三角形最小路径和](https://leetcode-cn.com/problems/triangle/)

[面试题 17.16. 按摩师](https://leetcode-cn.com/problems/the-masseuse-lcci/)

***

#### [70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

```java
// 方法1-1：直接递归，超出时间限制
public int climbStairs(int n) {

    if(n <= 1){
        return 1;
    }
    if(n == 2){
        return 2;
    }
    return climbStairs(n-1) + climbStairs(n-2);
}

// 方法1-2：用一个hashtable缓存结果，避免重复计算
Map<Integer ,Integer> cash = new HashMap<>();
public int climbStairs(int n) {

    if(n <= 1){
        return 1;
    }

    if(n == 2){
        return 2;
    }

    if(cash.containsKey(n)){
        return cash.get(n);
    }

    int result = climbStairs(n-1) + climbStairs(n-2);
    cash.put(n, result);

    return result;
}

// 方法2-1：动态规划，使用数组
// 1. 明确状态：dp[i], 爬到第i层的可能方案
// 2. 状态转移方程: dp[i] = dp[i-2] + dp[i-1]
// 3. 选择：dp[n] 爬n阶之后达到楼顶
// 4. base case： dp[1] = 1  dp[2] = 2
public int climbStairs(int n) {

    if(n <= 1){
        return 1;
    }

    int[] dp = new int[n+1];

    dp[1] = 1;
    dp[2] = 2;

    for(int i=3; i<=n; i++){
        dp[i] = dp[i-1] + dp[i-2];
    }

    return dp[n];
}

// 方法2-2：动态规划，爬楼梯问题可以直接使用两个变量，优化求解
public int climbStairs(int n) {

    if(n <= 1){
        return 1;
    }else if(n == 2){
        return 2;
    }else{
        int a=1, b=2;
        for(int i=3; i<=n; i++){
            int tmp = a + b;
            a = b;
            b = tmp;
        }
        return b;
    }
}
```

#### [509. 斐波那契数](https://leetcode-cn.com/problems/fibonacci-number/)

```java
// 方法1：暴力法
public int fib(int n) {
    if(n == 0 || n == 1){
        return n;
    }

    return fib(n-1) + fib(n-2);
}

// 方法2：带备忘录的递归解法
Map<Integer, Integer> map = new HashMap<>();
public int fib(int n) {
    if(n == 0 || n == 1){
        return n;
    }

    if(map.containsKey(n)){
        return map.get(n);
    }else{
        int result = fib(n-1) + fib(n-2);
        map.put(n, result);
        return result;
    }
}

// 方法3：动态规划
// 1. 明确状态：dp[i] 第i个数
// 2. 状态转移方程：dp[i] = dp[i-1] + dp[i-2]
// 3. 明确选择：dp[n] 第n个斐波那契数
// 4. basecase: dp[1] = 1  dp[2] = 1;
public int fib(int n) {
    if(n < 1) return 0;
    if(n == 1 || n == 2) return 1;

    int[] dp = new int[n+1];
    dp[1] = 1;
    dp[2] = 1;
    for(int i=3; i<=n; i++){
        dp[i] = dp[i-1] + dp[i-2];
    }

    return dp[n];
}

// 方法4：动态规划，只使用两个变量实现
public int fib(int n) {

    if(n < 1){
        return 0;
    }
	// 明确两个状态：状态1 状态2 + base case
    int a = 0, b = 1;
	// 开始状态转移
    for(int i=2; i<=n; i++){
        int tmp = a+b;
        a = b;
        b = tmp;
    }

    return b;
}
```

#### [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

```java
// 方法1：暴力法
public String longestPalindrome(String s) {

    int len = s.length();
    if (len < 2) {
        return s;
    }

    // 记录最长回文子串
    int maxLen = 1;  
    // 记录回文子串开始索引
    int begin = 0;
    // s.charAt(i) 每次都会检查数组下标越界，因此先转换成字符数组
    char[] charArray = s.toCharArray();

    // 枚举所有长度大于 1 的子串 charArray[i..j]
    for (int i = 0; i < len - 1; i++) {
        for (int j = i + 1; j < len; j++) {
            // 当子串长度j - i + 1 小于 maxLen，则没必要去判断了
            if (j - i + 1 > maxLen && validPalindromic(charArray, i, j)) {
                maxLen = j - i + 1;
                begin = i;
            }
        }
    }
    return s.substring(begin, begin + maxLen);
}

/**
  * 验证子串 s[left..right] 是否为回文串
  */
private boolean validPalindromic(char[] charArray, int left, int right) {
    while (left < right) {
        if (charArray[left] != charArray[right]) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}

// 方法2：动态规划
// 1.明确状态：dp[i][j] 表示字符串s[i...j]是否为回文子串
// 2.状态转移方程： dp[i][j] = dp[i+1][j-1] and (s[i] == s[j])，
//        这个dp数组是可以画图的，在状态转移时，前面的状态一定要已经好了，体现在代码中就是遍历dp数组的方向
// 3.选择：当dp[i][j]为true时，记录下此时的字符串起始与长度
// 4.base case: i==j 时， dp[i][j] 必为回文串
// 5.优化...emmmm
public String longestPalindrome(String s) {

    int length = s.length();

    if(length < 2){
        return s;
    }

    int start = 0;
    int maxLen = 1;
    // 定义dp数组
    boolean[][] dp = new boolean[length][length];

    // base case
    for(int i=0; i<length; i++){
        dp[i][i] = true;
    }

    // 开始遍历
    for(int i=length-1; i>=0; i--){  // 状态1的转移
        for(int j=i+1; j<length; j++){  // 状态2的转移
            if(s.charAt(i) == s.charAt(j)){
                if(j - i <= 2){
                    // 只有两个或者三个字符时，是不需要依赖前面的状态的
                    dp[i][j] = true;
                }else{
                    // 而超过三个字符时，就依赖前面的状态了
                    dp[i][j] = dp[i+1][j-1];
                }
            }else{
                dp[i][j] = false;
            }

            // 只要 dp[i][j] == true 成立，就表示子串 s[i..j] 是回文，此时记录回文长度和起始位置
            if(dp[i][j] == true && j-i+1 > maxLen){
                start = i;
                maxLen = j-i+1;
            } 
        }
    }

    return s.substring(start, start+maxLen);
}

// 方法3：双指针，中心扩散法
public String longestPalindrome(String s) {
    String res = "";
    for(int i=0; i<s.length(); i++){
        // 以s[i]为中心的最长回文子串
        String s1 = palinedrome(s, i, i);
        // 以s[i] 和 s[i+1] 为中心的最长回文子串
        String s2 = palinedrome(s, i, i+1);

        res = res.length() > s1.length() ? res: s1;
        res = res.length() > s2.length() ? res: s2;
    }

    return res;
}

private String palinedrome(String s, int l, int r){
    // 防止索引越界
    while(l>=0 && r<s.length() && s.charAt(l) == s.charAt(r)){
        l--;
        r++;
    }
    return s.substring(l+1, r);  // 因为r是取不到的，因此不需要-1了
}
```

#### [516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

```java
// 1. 明确状态：dp[i][j]，表示字符串s[i...j]中的最长回文子序列长度
// 2. 状态转移方程：
//       当s[i] == s[j]  dp[i][j] = dp[i + 1][j - 1] + 2;
//       当s[i] != s[j]  dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
//       最终状态：dp[0][n - 1]
// 3. 选择：回文子序列的长度
// 4. base case：i==j 时， dp[i][j] 必为回文串
public int longestPalindromeSubseq(String s) {

    int n = s.length();
    // 定义dp数组
    int[][] dp = new int[n][n];
	// base case
    for(int i=0; i<n; i++){
        dp[i][i] = 1;
    }
	
    // 开始状态转移
    for(int i=n-1; i>=0; i--){
        for(int j=i+1; j<n; j++){
            if(s.charAt(i) == s.charAt(j)){
                // 它俩一定在最长回文子序列中
                dp[i][j] = dp[i+1][j-1] + 2;
            }else{
                // 把i/j 加入后，s[i+1..j] 和 s[i..j-1] 谁的回文子序列更长？
                dp[i][j] = Math.max(dp[i+1][j], dp[i][j-1]);
            }
        }
    }

    return dp[0][n-1];
}
```

#### [300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

```java
// 动态规划求解
// 1. 明确状态：dp[i],表示数组nums[0...i]的最长子序列且i必须取
// 2. 确定状态转移方程：dp[i] = Math.max(dp[i], dp[j] + 1); 
// 3. 选择：dp中的最大值
// 4. base case dp[i]=1;
public int lengthOfLIS(int[] nums) {
	
    // 定义了dp数组
    int[] dp = new int[nums.length];
    // base case
    for(int i=0; i<nums.length; i++){
        dp[i] = 1;
    }

    for(int i=0; i<nums.length; i++){  // 状态转移
        for(int j=0; j<i; j++){  // s[0...i]数组的最长递增子序列的情况，但是必须包含i元素的情况下
            if(nums[i] > nums[j]){
                dp[i] = Math.max(dp[i], dp[j] + 1); 
            }
        }
    }
	
    // 明确输出
    int max = 1;
    for(int i=0; i<dp.length; i++){
        if(max < dp[i]){
            max = dp[i];
        }
    }

    return max;
}
```

#### [53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

```java
// 动态规划求解
// 1. 明确状态：以nums[i]为结尾的「最大子数组和」为dp[i]
// 2. 确定状态转移方程：dp[i] = Math.max(nums[i], dp[i-1] + nums[i])
// 3. 最终选择：dp中的最大值
// 4. basecase dp[0]=nums[0];
public int maxSubArray(int[] nums) {
    
    int n = nums.length;
    int[] dp = new int[n];
    // base case
    // 第一个元素前面没有子数组
    dp[0] = nums[0];

    // 状态转移方程
    for(int i=1; i<n; i++){
        // 要么自成一派，要么和前面的子数组合并
        dp[i] = Math.max(nums[i], dp[i-1] + nums[i]);    
    }

    // 得到 nums 的最大子数组
    int res = Integer.MIN_VALUE;
    for (int i = 0; i < n; i++) {
        res = Math.max(res, dp[i]);
    }
    return res;
}
```

#### [354. 俄罗斯套娃信封问题](https://leetcode-cn.com/problems/russian-doll-envelopes/)

```java
// 这题关键点在与理解为什么要这么排序：
// 1. 按照信封的w进行升序排序，很好理解，因为只有大的w可以包含小的w
// 2. 那为什么要按照h的降序排列呢？这是考虑到w相同时，只有一个h能被选择，而在计算 LIS 时，如升序排列，则存在相同的w同时被选中的情况
public int maxEnvelopes(int[][] envelopes) {

    if(envelopes == null || envelopes.length == 0){
        return 0;
    }

    int n = envelopes.length;
    // 按照w对envelopes进行升序排序，若h相同，则按照h降序排序
    Arrays.sort(envelopes, new Comparator<int[]>() {
        @Override
        public int compare(int[] o1, int[] o2) {
            return o1[0] == o2[0] ? o2[1] - o1[1] : o1[0] - o2[0];
        }
    });

    // 对h数组寻找LIS （Longes Increasing Subsequence）300题的内容
    int[] height = new int[n];
    for(int i=0; i<n; i++){
        height[i] = envelopes[i][1];
    }

    return lengthOfLIS(height);
}

// 求最长递增子序列，300题的内容
private int lengthOfLIS(int[] nums){

    int n = nums.length;
    int[] dp = new int[n];
    for(int i=0; i<n; i++){
        dp[i] = 1;    
    }

    for(int i=0; i<n; i++){
        for(int j=0; j<i; j++){
            if(nums[i] > nums[j]){
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
    }

    int max = 1;
    for(int i=0; i<n; i++){
        if(max < dp[i]){
            max = dp[i];
        }
    }

    return max;
}
```

#### [1143. 最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

```java
// 方法1：自顶向下，带备忘录的方式
// 备忘录，消除重叠子问题
int[][] memo = null;

public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length();
    int n = text2.length();

    memo = new int[m][n];
    for(int i=0; i<m; i++){
        for(int j=0; j< n; j++){
            memo[i][j] = -1;
        }
    }

    // 计算 s1[0..] 和 s2[0..] 的 lcs 长度
    return dp(text1, 0, text2, 0);
}

// 定义：计算 s1[i..] 和 s2[j..] 的最长公共子序列长度
private int dp(String s1, int i, String s2, int j){
    // base case
    if(i==s1.length() || j==s2.length()){
        return 0;
    }

    if(memo[i][j] != -1){
        return memo[i][j];
    }

    if(s1.charAt(i) == s2.charAt(j)){
        // s1[i] 和 s2[j] 必然在 lcs 中，两个字符串都往后移动+1
        // 加上 s1[i+1..] 和 s2[j+1..] 中的 lcs 长度，就是答案
        memo[i][j] = 1 + dp(s1, i+1, s2, j+1);
    }else{
        // s1[i] 和 s2[j] 中至少有一个字符不在 lcs 中，
        // 穷举三种情况的结果，取其中的最大结果
        int condition1 = dp(s1, i+1, s2, j);   // 删除s1中的字符
        int condition2 = dp(s1, i, s2, j+1);   // 删除s2中的字符
        int condition3 = dp(s1, i+1, s2, j+1);  // 其实这个可以不用的，因为已经包括在了上两种情况中
        memo[i][j] = Math.max(condition1, Math.max(condition2, condition3));
    }

    return memo[i][j];
}


// 方法2：动态规划
// 1. 明确状态：dp[i][j] 为 s1[0..i-1] 和 s2[0..j-1] 的 lcs 长度
// 2. 确定状态转移：用两个指针i和j从后往前遍历s1和s2，
//    如果s1[i]==s2[j]那么这个字符一定在lcs中   如果s1[i]!=s2[j]这两个字符至少有一个不在lcs中 
// 3. 明确选择：dp[m][n]为最长的lcs
// 4. basecase：dp[0][..] = dp[..][0] = 0
public int longestCommonSubsequence(String text1, String text2) {

    // 自底向上的递归
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m+1][n+1];  // 这里都+1，是为了给base case留位置

    // 定义：s1[0..i-1] 和 s2[0..j-1] 的 lcs 长度为 dp[i][j]
    // 目标：s1[0..m-1] 和 s2[0..n-1] 的 lcs 长度，即 dp[m][n]
    // base case: dp[0][..] = dp[..][0] = 0
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            // 现在 i 和 j 从 1 开始，所以要减一
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                // s1[i-1] 和 s2[j-1] 必然在 lcs 中
                dp[i][j] = 1 + dp[i - 1][j - 1];
            }else{
                // s1[i-1] 和 s2[j-1] 至少有一个不在 lcs 中
                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
            }
        }
    }

    return dp[m][n];
}
```

#### [583. 两个字符串的删除操作](https://leetcode-cn.com/problems/delete-operation-for-two-strings/)

```java
// 这题目就是寻找最长公共子序列，同1143
public int minDistance(String word1, String word2) {

    int lcs = findlcs(word1, word2);

    return word1.length() - lcs + word2.length() - lcs;
}

private int findlcs(String s1, String s2){

    int m = s1.length(), n = s2.length();

    int[][] dp = new int[m+1][n+1];

    for(int i=1; i<=m; i++){
        for(int j=1; j<=n; j++){
            if(s1.charAt(i-1) == s2.charAt(j-1)){
                dp[i][j] = 1 + dp[i-1][j-1];
            }else{
                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
            }
        }
    }

    return dp[m][n];
}
```

#### [712. 两个字符串的最小ASCII删除和](https://leetcode-cn.com/problems/minimum-ascii-delete-sum-for-two-strings/)

```java
// 自顶向下，带备忘录的方式
int[][] memo = null;  // 备忘录，消除重叠子问题

public int minimumDeleteSum(String s1, String s2) {

	int m = s1.length();
    int n = s2.length();

    memo = new int[m][n];
    for(int i=0; i<m; i++){
        for(int j=0; j< n; j++){
            memo[i][j] = -1;
        }
    }

    return dp(s1, 0, s2, 0);
}

// 定义：将 s1[i..] 和 s2[j..] 删除成相同字符串，
// 最小的 ASCII 码之和为 dp(s1, i, s2, j)。
private int dp(String s1, int i, String s2, int j){
    int res = 0;
    // base case
    if (i == s1.length()) {
        // 如果 s1 到头了，那么 s2 剩下的都得删除
        for (; j < s2.length(); j++)
            res += s2.charAt(j);
        return res;
    }
    if (j == s2.length()) {
        // 如果 s2 到头了，那么 s1 剩下的都得删除
        for (; i < s1.length(); i++)
            res += s1.charAt(i);
        return res;
    }

    if (memo[i][j] != -1) {
        return memo[i][j];
    }

    if (s1.charAt(i) == s2.charAt(j)) {
        // s1[i] 和 s2[j] 都是在 lcs 中的，不用删除
        memo[i][j] = dp(s1, i + 1, s2, j + 1);
    } else {
        // s1[i] 和 s2[j] 至少有一个不在 lcs 中，删一个,选择更小的结果
        memo[i][j] = Math.min(
            s1.charAt(i) + dp(s1, i + 1, s2, j),
            s2.charAt(j) + dp(s1, i, s2, j + 1)
        );
    }
    return memo[i][j];
}

// 动态规划
// 同1143，在其基础上修改些许内容即可
// 1. base case 需要修改
// 2. 状态转移方程需要修改一下
//     当s[i] == s[j] 时，不需要删除，就是前面的状态 dp[i][j] = dp[i - 1][j - 1];
//     当s[i] != s[j] 时, 取小的那个删除
public int minimumDeleteSum(String s1, String s2) {
    // 自底向上的递归
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m+1][n+1];
	
    // base case，当一个字符串为""时，剩下的字符串都要删除
    for (int i = 1; i <= m; i++) {
        dp[i][0] = dp[i-1][0] + s1.charAt(i-1);
    }

    for (int j = 1; j <= n; j++) {
        dp[0][j] = dp[0][j-1] + s2.charAt(j-1);
    }

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            // 现在 i 和 j 从 1 开始，所以要减一
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                // s1[i-1] 和 s2[j-1] 必然在 lcs 中
                dp[i][j] = dp[i - 1][j - 1];
            }else{
                // s1[i-1] 和 s2[j-1] 至少有一个不在 lcs 中,删一个
                dp[i][j] = Math.min(dp[i-1][j] + (int)s1.charAt(i-1), dp[i][j-1] + (int)s2.charAt(j-1));
            }
        }
    }

    return dp[m][n];
}
```

#### [72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

```java
// 自顶向下的动态规划，从后往前的方式
int[][] cache;  // 备忘录
public int minDistance(String word1, String word2) {

    int m = word1.length();
    int n = word2.length();
    cache = new int[m][n];
    for(int i=0; i<m; i++){
        for(int j=0; j<n; j++){
            cache[i][j] = -1;
        }
    }
    return dp(word1, m-1, word2, n-1);
}

private int dp(String s1, int i, String s2, int j){
    // base case
    if(i == -1) return j+1;
    if(j == -1) return i+1;

    if(cache[i][j] != -1){
        return cache[i][j];
    }

    if(s1.charAt(i) == s2.charAt(j)){
        cache[i][j] = dp(s1, i-1, s2, j-1);
    }else{
        int insert = dp(s1, i, s2, j-1) + 1;
        int del = dp(s1, i-1, s2, j) + 1;
        int change = dp(s1, i-1, s2, j-1) + 1;

        cache[i][j] = Math.min(insert, Math.min(del, change));
    }

    return cache[i][j];
}

// 自顶向下的动态规划，从前往后的方式
int[][] memo = null;

public int minDistance(String word1, String word2) {

    int m = word1.length();
    int n = word2.length();

    memo = new int[m][n];

    for(int i=0; i<m; i++){
        for(int j=0; j<n; j++){
            memo[i][j] = -1;
        }
    }

    return dp(word1, 0, word2, 0);
}

private int dp(String s1, int i, String s2, int j){

    if(i==s1.length()){
        return s2.length() - j;
    }

    if(j == s2.length()){
        return s1.length() - i;
    }

    if(memo[i][j] != -1){
        return memo[i][j];
    }

    if(s1.charAt(i) == s2.charAt(j)){
        memo[i][j] = dp(s1, i+1, s2, j+1);
    }else{

        int insert = 1 + dp(s1, i, s2, j+1);
        int remove = 1 + dp(s1, i+1, s2, j);
        int replace = 1 + dp(s1, i+1, s2, j+1);

        memo[i][j] = Math.min(insert, Math.min(remove, replace));
    }

    return memo[i][j];

}

// 自底向上的动态规划
public int minDistance(String word1, String word2) {

    int m = word1.length();
    int n = word2.length();

    int[][] dp = new int[m+1][n+1];

    // base case
    for(int i=1; i<=m; i++){
        dp[i][0] = i;
    }

    for(int j=1; j<=n; j++){
        dp[0][j] = j;
    }

    // 自底向上的动态规划
    for(int i=1; i<=m; i++){
        for(int j=1; j<=n; j++){
            if(word1.charAt(i-1) == word2.charAt(j-1)){
                dp[i][j] = dp[i-1][j-1];
            }else{
                dp[i][j] = min(
                    dp[i][j-1] + 1,    // 删除
                    dp[i-1][j-1] + 1,  // 替换
                    dp[i-1][j] + 1     // 插入
                );
            }
        }
    }

    return dp[m][n];

}

private int min(int a, int b, int c){
    return Math.min(a, Math.min(b, c));
}
```

#### [416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)

```java
// 0-1背包问题，动态规划
// 1. 明确状态：背包的容量/可选的物品--->子集的和/集和中可选的数
// 2. 状态转移：dp[i][j] 对于集和中的i个数，是否能填满j容量的背包
// 3. 选择：装/不装 ---> 选/不选，最终：dp[N][sum/2]，是否装满
// 4. base case：dp[..][0] = true   dp[0][..] = false
public boolean canPartition(int[] nums) {

    int n = nums.length;

    int sum = 0;
    for(int i=0; i<n; i++){
        sum += nums[i];
    }

    // 排除不可能情况
    if(sum%2 != 0){
        return false;
    }
    sum /= 2;  // 等价背包容量
	
    // dp数组的定义
    boolean[][] dp = new boolean[n+1][sum+1];

    // base case
    for(int i=0; i<=n; i++){
        // 当我还有元素，但是已经满足sum/2了
        dp[i][0] = true;
    }
    for(int i=0; i<=sum; i++){
        // 当我的元素没了，但是还没加到sum/2;
        dp[0][i] = false;
    }

    // 开始dp
    for(int i=1; i<=n; i++){        // 状态1，元素个数增加
        for(int j=1; j<=sum; j++){  // 状态2, 背包容量
            // 判断能否装入
            if(nums[i-1] > j){
                // 放不下这个元素，这时的状态和i-1个物体时相同
                dp[i][j] = dp[i-1][j];
            }else{
                // 能装下，则选择装/不装
                // 不装 dp[i-1][j]，就和之前的状态相同
                // 装：dp[i-1][j - nums[i-1]]，在装了i这个物体后背包的容量在有i-1个物体时的状态
                dp[i][j] = dp[i-1][j] | dp[i-1][j - nums[i-1]];
            } 
        }
    }

    return dp[n][sum];
}
```

#### [279. 完全平方数](https://leetcode-cn.com/problems/perfect-squares/)

```java
// 完全背包问题：即每个物品的数量是无限的
// 1. 明确状态：dp[i]当背包容量为i时，最少需要装多少个完全平方数
// 2. base case: dp[i] 全部用1来装，最快情况的个数，即能装满的情况
// 3. 状态转移：可选的完全平方数增多/背包容量增加
public int numSquares(int n) {

    int len = (int)Math.sqrt(n);
    int[] nums = new int[len+1];

    for(int i=1; i<=len; i++){
        nums[i] = i*i;
    }

    int[] dp = new int[n+1];

    // base case：最坏的情况
    for(int j=0; j<=n; j++){
        dp[j] = j;
    }

    for(int i=1; i<=len; i++){    // 状态1,可选的数字增加
        for(int j=1; j<=n; j++){  // 状态2,容量
            if(nums[i] > j){
                // 装不进背包,子问题无解，跳过
                continue;
            }else{
                // 能装下，则选择装/不装
                dp[j] = Math.min(dp[j], dp[j-nums[i]] + 1);
            }
        }
    }
    return dp[n];
}
```

#### [322. 零钱兑换](https://leetcode-cn.com/problems/coin-change/)

```java
// 1. 暴力递归--->超出时间限制
public int coinChange(int[] coins, int amount) {
    return dp(amount, coins);
}

// 要凑出金额 n，至少要 dp(n) 个硬币
private int dp(int n, int[] coins){
    if(n == 0) return 0;
    if(n < 0) return -1;

    int res = 10000;
    for(int i=0; i<coins.length; i++){
        int sub = dp(n-coins[i], coins);
        if (sub == -1){
            continue;
        }
        res = Math.min(sub+1, res);
    }
    return res==10000 ? -1: res;
}

// 2. 带备忘录的递归
HashMap<Integer, Integer> cache = new HashMap<>();
public int coinChange(int[] coins, int amount) {
    return dp(amount, coins);
}

// 要凑出金额 n，至少要 dp(n) 个硬币
private int dp(int n, int[] coins){

    if(cache.containsKey(n)) return cache.get(n);
    if(n == 0) return 0;
    
    int res = 10000;
    for(int i=0; i<coins.length; i++){
        int sub = dp(n-coins[i], coins);
        if (sub == -1){
            continue;
        }
        res = Math.min(sub+1, res);
    }
    int result = res==10000 ? -1: res;
    cache.put(n, result);
    return result;
}


// 3. 动态规划：一个完全背包问题
// 		1. 定义dp数组，明确状态，dp[i]表示当前背包容量为i时，放满背包的最少物品个数
// 		2. base case: dp[0] 当背包空间为0时，无需放入东西，当然是0
//		3. 状态转移：
//			3.1 状态1：背包的容量逐渐增大
//			3.2 状态2：可选的物品不断增多
//		4. 最终输出：dp[amount]
public int coinChange(int[] coins, int amount) {

    // 建立dp数组
    int[] dp = new int[amount+1];
    // base case
    dp[0] = 0;
    // 外层 for 循环在遍历所有状态的所有取值
    for(int i=1; i<=amount; i++){
        dp[i] = amount+1;  // 最差的情况
        for(int j=0; j<coins.length; j++){
            // 子问题无解，跳过
            if(i - coins[j] < 0) continue;
            dp[i] = Math.min(dp[i], dp[i-coins[j]]+1);
        }
    }

    return dp[amount] == amount+1 ? -1: dp[amount];
}
```

#### [518. 零钱兑换 II](https://leetcode-cn.com/problems/coin-change-2/)

```java
// 完全背完问题，体现出来的就是物品可以重复使用
public int change(int amount, int[] coins) {

    int n = coins.length;
    // dp数组定义:dp[i][j] 表示当有i个硬币可用，总额为j时的可能组合数
    int[][] dp = new int[n+1][amount+1];

    // base case
    for(int j=0; j<=amount; j++){
        dp[0][j] = 0;
    }
    for(int i=0; i<=n; i++){
        // 总金额为0，则不适用任何硬币为一种凑法
        dp[i][0] = 1;
    }

    for(int i=1; i<=n; i++){           // 状态1：可选择的物品
        for(int j=1; j<=amount; j++){  // 状态2：背包的容量
            // 选择：装/不装
            if(coins[i-1] > j){
                // 硬币的额度大于余额，为原先的状态
                dp[i][j] = dp[i-1][j];
            }else{
                // 状态转移
                // dp[i-1][j] 用了 i-1 个硬币，在金额为j时的可能
                // dp[i][j - coins[i-1]] 用了i个硬币的情况, 
                // i是关键：i-1就是0-1背包，i不减1就是完全背包
                // 最终的状态转移方程就是：使用了硬币i的可能+不使用i硬币的可能
                dp[i][j] = dp[i-1][j] + dp[i][j - coins[i-1]];  
            }
        }
    }
	// 最终状态，即有n个硬币时，总金额为amount时的可能的组合数
    return dp[n][amount];
}
```

#### [10. 正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

```java
// 方法1：直接暴力法
public boolean isMatch(String s, String p) {

    if(p.length() == 0){
        return s.length() == 0;
    }

    // 匹配首位字符，s的首位字符和p的首位字符是否匹配
    boolean fist = false;
    if(s.length() != 0){
        char s1 = s.charAt(0);
        char p1 = p.charAt(0);
        fist = (s1 == p1);  // 首位字符是否匹配
        if(p1 == '.'){      // p的首位字符是.的情况
            fist = true;
        }
    }
    // 判断*的情况
    if(p.length() >= 2 && p.charAt(1) == '*'){
        // 在pattern字符的第二位是*
        return isMatch(s, p.substring(2, p.length())) || 
                (fist && isMatch(s.substring(1, s.length()), p));
    }else{
        return fist && isMatch(s.substring(1, s.length()), p.substring(1, p.length()));
    }
}

// 暴力法 --> 带个备忘录的dp
int[][] cache;  // 0：没算过  1：处理了为true   -1：处理为false
public boolean isMatch(String s, String p) {

    cache = new int[s.length()+1][p.length()+1];

    for(int i=0; i<=s.length(); i++){
        for(int j=0; j<=p.length(); j++){
            cache[i][j] = 0;
        }
    }

    return dp(s, 0, p, 0);
}

private boolean dp(String s, int i, String p, int j){

    if(cache[i][j] != 0){
        return cache[i][j] == 1 ? true : false;
    }

    if(j == p.length()){
        return i == s.length();
    }

    // 匹配首位字符
    boolean fist = false;
    if(i < s.length()){
        fist = (s.charAt(i) == p.charAt(j));
        if(p.charAt(j) == '.'){
            fist = true;
        }
    }

    boolean ans = false;
    if(p.length() - j >= 2 && p.charAt(j+1) == '*'){
        ans = dp(s, i, p, j+2) ||        // 直接掉过*，即匹配0个
            (fist && dp(s, i+1, p, j));  // 第一个字符是匹配的，则继续匹配任意多个
    }else{
        ans = fist && dp(s, i+1, p, j+1);
    }

    if(ans){
        cache[i][j] = 1;
    }else{
        cache[i][j] = -1;
    }

    return ans;
}
```

#### [887. 鸡蛋掉落](https://leetcode-cn.com/problems/super-egg-drop/)

```java
// 自顶向下递归：
int[][] cache;
public int superEggDrop(int K, int N) {

    cache = new int[K+1][N+1];

    return dp(K, N);
}

private int dp(int k, int n){

    if(k == 1){
        // 只有一个鸡蛋了，那只能一层一层尝试了
        return n;
    }
    if(n==0){
        // 第0层，不需要扔了，直接返回0
        return 0;
    }

    if(cache[k][n] != 0){
        return cache[k][n];
    }

    // 最坏的情况就是线性扔，从1-N
    int res = n+1;  
    
    // 下面的遍历会超市WTF
    // 穷举所有可能的选择，这些选择都是从i层出发的最坏的情况下找到F的，然后找到最小移动的情况
    // for(int i=1; i<=n; i++){
    //     // 最坏情况下的最少扔鸡蛋次数
    //     // min:首先从i层开始扔最终的扔的次数最少
    //     res = Math.min(
    //         res,
    //         Math.max(          // max最坏，取后续两者的最大值
    //             dp(k-1, i-1),  // 在i层碎了 
    //             dp(k, n-i)     // i层没碎
    //         ) + 1              // 在i层扔了一次鸡蛋，因此要+1
    //     );
    // }
   	
    // 修改为：这里用二分法进行优化操作
    // 能用二分搜索是因为状态转移方程的函数图像具有单调性，可以快速找到最值
    int lo=1, hi=n;
    while( lo <= hi){
        int mid = (lo + hi) / 2;
        // 在mid层是否破了
        int broken = dp(k-1, mid-1);
        int not_broken = dp(k, n-mid);

        if(broken > not_broken){
            hi = mid - 1;
            res = Math.min(res, broken + 1);
        }else{
            lo = mid + 1;
            res = Math.min(res, not_broken + 1);
        }
    }

    cache[k][n] = res;

    return cache[k][n];
}


// 使用自底向上的写法
public int superEggDrop(int K, int N) {

    // 明确状态：dp[i][j] 有i个楼层k个鸡蛋找到F的最优情况
    int[][] dp = new int[N+1][K+1];

    // base case
    for(int i=0; i<=N; i++){
        // 只有一个鸡蛋，线性去扔鸡蛋
        dp[i][1] = i;
    }

    for(int i=1; i<=N; i++){
        for(int k=2; k<=K; k++){
            int res = Integer.MAX_VALUE;
            /*
            // dp[i][j] 有i个楼层k个鸡蛋找到F的最优情况
            // 随机开始从i层扔鸡蛋，找最优情况
            for(int j=1; j<=i; j++){
                res = Math.min(
                    res,
                    Math.max(          // 对于F的最坏情况
                        dp[j-1][k-1],  // 碎了
                        dp[i-j][k]     // 没碎
                    ) + 1
                );
            }
            */
            // 二分不超时
            int lo = 1, hi = i;
            while( lo <= hi){
                int mid = (lo + hi) / 2;
                // 在mid层是否破了
                int broken = dp[mid-1][k-1];
                int not_broken = dp[i-mid][k];

                if(broken > not_broken){
                    hi = mid - 1;
                    res = Math.min(res, broken + 1);
                }else{
                    lo = mid + 1;
                    res = Math.min(res, not_broken + 1);
                }
            }
            dp[i][k] = res;
        }
    }

    return dp[N][K];
}
```

#### [198. 打家劫舍](https://leetcode-cn.com/problems/house-robber/)

```java
public int rob(int[] nums) {

    int n = nums.length;

    if(n == 0){
        return 0;
    }

    // 明确状态：dp[i] 表示偷[0..i]间房子的最大值
    int[] dp = new int[n+1];

    // base case 
    dp[0] = 0;
    dp[1] = nums[0];

    for(int i=2; i<=n; i++){
        // 状态转移方程：
        // 第i间屋子偷/不偷哪个情况能取最大
        dp[i] = Math.max(dp[i-2]+nums[i-1], dp[i-1]);
    }

    return dp[n];
}
```

#### [213. 打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

```java
public int rob(int[] nums) {

    int n = nums.length;

    if(n == 0){
        return 0;
    }
	
    // 明确状态：dp[i] 表示偷[0..i]间房子的最大值
    int[] dp = new int[n+1];

    // case 1: 偷第一家，则最后一家不能偷了
    // base case
    dp[0] = 0;
    dp[1] = nums[0];
    for(int i=2; i<n; i++){
        dp[i] = Math.max(dp[i-2] + nums[i-1], dp[i-1]);
    }
    // int res1 = dp[n-1];
    // 解决只有一家的情况
    int res1 = Math.max(dp[n-1], dp[n]);

    // case 2: 第一家不偷，我能偷最后一家
    dp[0] = 0;
    dp[1] = 0;
    for(int i=2; i<=n; i++){
        dp[i] = Math.max(dp[i-2] + nums[i-1], dp[i-1]);
    }
    int res2 = dp[n];

    return Math.max(res1, res2);
}
```

#### [337. 打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii/)

```java
// 自顶向下的备忘录
Map<TreeNode, Integer> cache = new HashMap<>();
public int rob(TreeNode root) {

    if(root == null){
        return 0;
    }

    if(cache.containsKey(root)){
        return cache.get(root);
    }
	
    // 进行选择
    // 抢，然后只能偷该节点的子节点的子节点
    int steal = root.val + 
        (root.left == null ? 0 : rob(root.left.left) + rob(root.left.right)) + 
        (root.right == null ? 0 : rob(root.right.left) + rob(root.right.right));
    // 不抢，那能偷该节点的子节点
    int not_steal = rob(root.left) + rob(root.right);

    // 上述两者取最大作为当前节点的最大情况
    int res = Math.max(steal, not_steal);
    cache.put(root, res);

    return res;
}
```

#### [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

```java
// 暴力法1:超出时间限制 201/210
public int maxProfit(int[] prices) {
 
    int res=0;

    // 遍历所有可能
    for(int i=0; i<prices.length; i++){
        for(int j=i+1; j<prices.length; j++){
            if(prices[j] - prices[i] > res){
                res = prices[j] - prices[i];
            }
        }
    }

    return res;
}

// 暴力法2:这里其实已经有动态规划的意思在了
public int maxProfit(int[] prices) {

    int res = 0;
    // 固定买出时间
    int hold = prices[0];  
    for(int j=1; j<prices.length; j++){
        // 寻找最低买入价格的时间，遍历一遍肯定能找到最小值
        hold = Math.min(hold, prices[j]);
        // 计算最佳时机
        res = Math.max(res, prices[j] - hold);
    }
    return res;
}

// 动态规划
// 1.明确状态：定义dp数组表示第i天的最大收入,
//			 i：天数  j:持有股票的状态 0-没持有  1-持有
// 2.base case：
//		第1天没持有，当然是0: dp[0][0] = 0; 
//		第1天持有，收益当然是负的：dp[0][1] = -prices[0];
// 3.状态转移：见下面
// 4.明确输出：第n天，没持有股票 dp[n-1][0]
public int maxProfit(int[] prices) {

    int n = prices.length;

    // 1.定义dp数组表示第i天的最大收入,有2个状态，
    // i：天数  j:持有股票的状态 0-没持有  1-持有
    int[][] dp = new int[n][2];

    // 2.base case
    dp[0][0] = 0;           // 第1天没持有，当然是0
    dp[0][1] = -prices[0];  // 第1天持有，收益当然是负的

    // 3.开始状态转移
    for(int i=1; i<n; i++){
        // 第i天没持有，可以由：上一天没持有 / 上一天持有，这一天卖了 转移而来
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1]+prices[i]);
        // 第i天持有，可以由：上一天没持有 这一天买入/上一天持有 转移而来
        // 这里不是 dp[i-1][0]-prices[i] 而是 -prices[i] 是因为只能卖一次
        dp[i][1] = Math.max(-prices[i], dp[i-1][1]);
    }

    // 最终输出：第n天，没持有股票
    return dp[n-1][0];
}
```

#### [122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

```java
// 贪心：说明：买出后其实还可以在当天买入的！！！
public int maxProfit(int[] prices) {

    int max = 0;
    for(int i=1; i<prices.length; i++){
        if(prices[i] > prices[i-1]){
            max += prices[i] - prices[i-1];
        }
    }
    return max;
}


// 暴力法：
public int maxProfit(int[] prices) {
    return dp(prices, 0);
}

private int dp(int[] prices, int index){

    int res = 0;
 
    // 暴力计算从第i天买入股票的情况
    for(int i=index; i<prices.length; i++){
        // 暴力计算j天买出股票的情况
        for(int j=i+1; j<prices.length; j++){
            res = Math.max(
                res,
                dp(prices, j+1) + 		 // 剩下j到最后一天的最大情况
                + prices[j] - prices[i]  // i天买入j天买出赚的
            );
        }
    }

    return res;
}


// 暴力法：优化--->加了备忘录
int[] cache;
public int maxProfit(int[] prices) {

    cache = new int[prices.length];
    Arrays.fill(cache, -1);

    return dp(prices, 0);
}

private int dp(int[] prices, int index){

    int res = 0;
 
    if(index >= prices.length){
        return 0;
    }

    if(cache[index] != -1){
        return cache[index];
    }
	// 消除一层循环
    int cur_min = prices[index];
    for(int i=index+1; i<prices.length; i++){
        cur_min = Math.min(cur_min, prices[i]);
        res = Math.max(res, dp(prices, i+1) + prices[i] - cur_min);
    }

    cache[index] = res;

    return res;
}

// 动态规划
public int maxProfit(int[] prices) {

    int n = prices.length;

    // 1. 明确状态：定义dp数组表示第i天的最大收入,
    //    有2个状态，i：天数  j:持有股票的状态 0-没持有  1-持有
    int[][] dp = new int[n][2];

    // base case
    dp[0][0] = 0;           // 第1天没持有，当然是0
    dp[0][1] = -prices[0];  // 第1天持有，收益当然是负的

    // 开始状态转移
    for(int i=1; i<n; i++){
        // 第i天没持有，可以由 上一天没持有/上一天持有，这一天卖了 转移而来
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1]+prices[i]);
        // 第i天持有，可以由 上一天没持有 这一天买入/上一天持有 转移而来
        dp[i][1] = Math.max(dp[i-1][0]-prices[i], dp[i-1][1]);
    }

    // 最终输出：第n天，没持有股票
    return dp[n-1][0];
}
```

#### [123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

```java
// 自顶向下的递归：超时 205 / 214
int[][] cache;
public int maxProfit(int[] prices) {

    cache = new int[prices.length][2];

    return dp(prices, 0, 2);
}

private int dp(int[] prices, int index, int k){

    int res = 0;
 
    if(index >= prices.length){
        return 0;
    }
	// 只能进行k笔交易，大于k笔交易后不能再交易了，返回0
    if(k==0){
        return 0;
    }

    if(cache[index][k-1] != 0){
        return cache[index][k-1];
    }

    int cur_min = prices[index];
    for(int i=index+1; i<prices.length; i++){
        // 动态选择最小的价格作为买入价格
        cur_min = Math.min(cur_min, prices[i]);
        // 表示第i天是否应该卖出，若卖出则为：i+1天的最大收益+第i天卖出的收益，什么时候买入不需要考虑，因为在i天之前是实时选择最小价格作为买入的
        res = Math.max(res, dp(prices, i+1, k-1) + prices[i] - cur_min);
    }

    cache[index][k-1] = res;

    return res;
}

// 动态规划：状态转移，即穷举所有的状态
public int maxProfit(int[] prices) {

    int n = prices.length;
    int k = 2;  // 只能交易两次

    if(n <= 1){
        return 0;
    }
	
    // 1.明确状态
    // 定义dp数组,有三个状态，i：天数  j：可交易数  k:持有股票的状态 ，如
    // dp[3][2][1]  第3天 还能交易2次 手上有股票
    // dp[4][1][0]  第4天 还能交易1次 手上无股票
    // 最终的答案为：dp[n-1][K][0]  最后一天，最多K此交易，此时也没股票了
    int[][][] dp = new int[n][k+1][2];

    // 2.base case
    // 当交易次数为0次时，不可能持有股票
    for(int i=0; i<n; i++){
        dp[i][0][1] = Integer.MIN_VALUE;
    }
    // 在第1天时，若这一天买入股票，则收益为负的
    for(int j=1; j<=k; j++){
        dp[0][j][1] = -prices[0];
    }

    // 3.开始状态转移
    for(int i=1; i<n; i++){        // 天数转移
        for(int j=k; j > 0; j--){  // 可交易次数的转移
            // 0:无股票状态
            dp[i][j][0] = Math.max(dp[i-1][j][0], dp[i-1][j][1] + prices[i]); 
            // 1:有股票状态
            dp[i][j][1] = Math.max(dp[i-1][j][1], dp[i-1][j-1][0] - prices[i]);  
        }
    }
	
    // 4.明确输出：dp[n-1][K][0]  最后一天，最多K此交易，此时也没股票了
    return dp[n-1][k][0];
}
```

#### [188. 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

```java
// 暴力法：优化--->加了备忘录
int[][] cache;
public int maxProfit(int k, int[] prices) {

    cache = new int[prices.length][k];

    return dp(prices, 0, k);
}

private int dp(int[] prices, int index, int k){

    int res = 0;

    if(index >= prices.length){
        return 0;
    }
    // 只能进行k笔交易，大于k笔交易后不能再交易了，返回0
    if(k==0){
        return 0;
    }

    if(cache[index][k-1] != 0){
        return cache[index][k-1];
    }

    int cur_min = prices[index];
    for(int i=index+1; i<prices.length; i++){
        cur_min = Math.min(cur_min, prices[i]);
        res = Math.max(res, dp(prices, i+1, k-1) + prices[i] - cur_min);
    }

    cache[index][k-1] = res;

    return res;
}

// 动态规划：状态转移，即穷举所有的状态
// 对于k的优化点：若k大于price的一半，则这个k就不是限制条件了
public int maxProfit(int k, int[] prices) {

    int n = prices.length;

    if(n <= 1){
        return 0;
    }
	
    // 1.明确状态：dp表示当前的最大收益
    // 定义dp数组,有三个状态，i：天数  j：可交易数  k:持有股票的状态 ，如
    // dp[3][2][1]  第3天 还能交易2次 手上有股票
    // dp[4][1][0]  第4天 还能交易1次 手上无股票
    // 最终的答案为：dp[n-1][K][0]  最后一年，最多K此交易，此时也没股票了
    int[][][] dp = new int[n][k+1][2];

    // 2.base case
    // 当交易次数为0次时，不可能持有股票
    for(int i=0; i<n; i++){
        dp[i][0][1] = Integer.MIN_VALUE;
    }
    // 在第1天时，若这一天买入股票，则收益为负的
    for(int j=1; j<=k; j++){
        dp[0][j][1] = -prices[0];
    }

    // 3.开始状态转移
    for(int i=1; i<n; i++){      // 天数转移
        for(int j=k; j > 0; j--){  // 可交易次数的转移
            dp[i][j][0] = Math.max(dp[i-1][j][0], dp[i-1][j][1] + prices[i]);  // 0:无股票状态
            dp[i][j][1] = Math.max(dp[i-1][j][1], dp[i-1][j-1][0] - prices[i]);  // 1:有股票状态
        }
    }
	
    // 4.最终状态；dp[n-1][K][0]  最后一年，最多K此交易，此时也没股票了，这时候的值即为最大值
    return dp[n-1][k][0];
}
```

#### [309. 最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

```java
// 暴力法：优化--->加了备忘录
int[] cache;
public int maxProfit(int[] prices) {

    cache = new int[prices.length];
    Arrays.fill(cache, -1);

    return dp(prices, 0);
}

private int dp(int[] prices, int index){

    int res = 0;
 
    if(index >= prices.length){
        return 0;
    }

    if(cache[index] != -1){
        return cache[index];
    }

    int cur_min = prices[index];
    for(int i=index+1; i<prices.length; i++){
        cur_min = Math.min(cur_min, prices[i]);
        // 冷冻期体现在： i+2而不是之前的i+1了
        res = Math.max(res, dp(prices, i+2) + prices[i] - cur_min);
    }

    cache[index] = res;

    return res;
}

// 动态规划
public int maxProfit(int[] prices) {

    int n = prices.length;

    if(n <= 1){
        return 0;
    }

    // 定义dp数组表示第i天的最大收入,有2个状态，i：天数  j:持有股票的状态 0-没持有  1-持有
    int[][] dp = new int[n][2];

    // base case
    dp[0][0] = 0;
    dp[0][1] = -prices[0];  // 第一天持有，结果当然是负的
    dp[1][0] = Math.max(dp[0][0], dp[0][1] + prices[1]);
    // 第二天持有：第一天没持有，第二天买入  / 第一天持有
    dp[1][1] = Math.max(-prices[1], dp[0][1]);  

    // 开始状态转移
    for(int i=2; i<n; i++){
        // 第i天没持有，可以由 上一天没持有/上一天持有，这一天卖了 转移而来
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1]+prices[i]);
        // 第i天持有，可以由 上两天没持有(要求隔一天) 这一天买/上一天持有 转移而来
        dp[i][1] = Math.max(dp[i-2][0]-prices[i], dp[i-1][1]);
    }

    // 最终输出：第n天，没持有股票
    return dp[n-1][0];
}
```

#### [714. 买卖股票的最佳时机含手续费](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

```java
// 暴力法：优化--->加了备忘录
// 超时：34 / 44
int[] cache;
public int maxProfit(int[] prices, int fee) {

    cache = new int[prices.length];
    Arrays.fill(cache, -1);

    return dp(prices, 0, fee);
}

private int dp(int[] prices, int index, int fee){

    int res = 0;

    if(index >= prices.length){
        return 0;
    }

    if(cache[index] != -1){
        return cache[index];
    }

    int cur_min = prices[index];
    for(int i=index+1; i<prices.length; i++){
        cur_min = Math.min(cur_min, prices[i]);
        // 手续费体现了 -fee 这一项
        res = Math.max(res, dp(prices, i+1, fee) + prices[i] - cur_min - fee);
    }

    cache[index] = res;

    return res;
}

// 动态规划
public int maxProfit(int[] prices, int fee) {

    int n = prices.length;

    // 定义dp数组表示第i天的最大收入,有2个状态，i：天数  j:持有股票的状态 0-没持有  1-持有
    int[][] dp = new int[n][2];

    // base case
    dp[0][0] = 0;
    dp[0][1] = -prices[0];  // 第一天持有，结果当然是负的

    // 开始状态转移
    for(int i=1; i<n; i++){
        // 第i天没持有，可以由 上一天没持有/上一天持有，这一天卖了 转移而来
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1]+prices[i]-fee);  // 加了个手续费
        // 第i天持有，可以由 上一天没持有 这一天买入/上一天持有 转移而来
        dp[i][1] = Math.max(dp[i-1][0]-prices[i], dp[i-1][1]);
    }

    // 最终输出：第n天，没持有股票
    return dp[n-1][0];
}
```

#### [139. 单词拆分](https://leetcode-cn.com/problems/word-break/)

```java
// 经典动态规划
public boolean wordBreak(String s, List<String> wordDict) {

    int n = s.length();
    // 1. 明确状态：dp[i]表示前i个字符能否拆分
    boolean[] dp = new boolean[n+1];

    // 2. base case：考虑了从 [0...i] 个字符能否拆分
    dp[0] = true;

    // 3. 状态转移
    for(int i=1; i<=n; i++){      // 状态：字符个数增加
        for(int j=0; j<i; j++){   // j用作拆分字符的指针
            // 状态转移方程：j把字符拆分成立  [0,,,j) [j...i)
            dp[i] = dp[j] && wordDict.contains(s.substring(j, i));
            if(dp[i]) break;
        }
    }
	// 4. 明确输出：dp[n]，n个字符能否拆分
    return dp[n];
}

// 这个效率高很多
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> set = new HashSet<>();
    for(String str:wordDict){
        set.add(str);
    }

    // dp[i]表示s前i个字符能否拆分
    // dp[8] = dp[5] + check("pen")
    // 翻译一下：前八位能否拆分取决于前五位能否拆分，加上五到八位是否属于字典
    boolean[] dp = new boolean[s.length()+1];
    dp[0] = true;
    for(int i=1; i<=s.length(); i++){
        for(int j=i-1; j>=0; j--){
            dp[i] = dp[j] && set.contains(s.substring(j,i));
            if(dp[i]) break; 
        }
    }
    return dp[s.length()];
}
```

#### [91. 解码方法](https://leetcode-cn.com/problems/decode-ways/)

```java
public int numDecodings(String s) {

    int n = s.length();

    // 1. 明确状态:dp[i] s[0...i]个字符能解码的总数
    int[] dp = new int[n+1];

    // 2. base case: 第一个字符能否正常解码
    if(s.charAt(0) == '0'){
        return 0;
    }else{
        dp[0] = 1;  // 这个不能省
        dp[1] = 1;
    }

    for(int i=2; i<=n; i++){

        // 当前第i个字符
        int s1 = (int)s.charAt(i-1) - '0'; 
        // 当前第i个字符和前面i-1个字符组成的二位数
        int s2 = ((int)s.charAt(i-2) - '0') * 10 + s1;
        // 3.状态转移方程：
        if(s1 != 0 && (s2 <= 26 && s2 >= 10)){
            // 当s1和s2都能解码
            dp[i] = dp[i-1] + dp[i-2];
        }else if(s1 != 0){
            // 只有当s1能解码
            dp[i] = dp[i-1];
        }else if(s2 <= 26 && s2 >= 10){
            // 只有当s2能解码
            dp[i] = dp[i-2];
        }
        else{
            // s1和s2都无法解码，直接进就是不合法的情况
            return 0;
        }
    }
    // 4.明确输出：dp[n]全部字符串长度能解码的情况
    return dp[n];
}
```

#### [392. 判断子序列](https://leetcode-cn.com/problems/is-subsequence/)

```java
// 双指针
public boolean isSubsequence(String s, String t) {

    int i=0;
    int j=0;
    while(i<s.length() && j<t.length()){
        if(s.charAt(i) == t.charAt(j)){
            i++;j++;
        }else{
            j++;
        }
    }

    if(i==s.length()){
        return true;
    }else{
        return false;
    }
}

// 动态规划，方式1，选择最长子序列，同1143
public boolean isSubsequence(String s, String t) {

    int m = s.length();
    int n = t.length();

    // 1.明确状态：dp[i][j]  表示s[0...i] 是否为 t[0...j] 的子序列，即记录的是子序列的长度
    int[][] dp = new int[m+1][n+1];

    // 2.base case
    for(int i=0; i<=m; i++){
        // t为空时，肯定就不是了
        dp[i][0] = 0;
    }
    for(int j=0; j<=n; j++){
        // s为空，可以把t全部删除，是子序列，但长度就是0
        dp[0][j] = 0;
    }

    // 3. 状态转移
    for(int i=1; i<=m; i++){        // 状态1：随着s长度的增大
        for(int j=i; j<=n; j++){	// 状态2：t的长度必须是大于等于s的长度，不然就没有子序列了
            if(s.charAt(i-1) == t.charAt(j-1)){
                // 只有字符相等，才让匹配的长度+1
                dp[i][j] = dp[i-1][j-1] + 1;
            }else{
                // 否则就去之前匹配的最大长度
                dp[i][j] = Math.max(dp[i][j-1], dp[i-1][j]);
            }
        }
    }
	// 4.明确输出:考虑到本题是求s是否是t的子序列，因此最终判定最长公共子序列的长度是否等于s的长度
    return dp[m][n] == m;
}


// 动态规划：比上的合适，更加适合这题
public boolean isSubsequence(String s, String t) {

    int m = s.length();
    int n = t.length();

    // 1.状态定义：dp[i][j]  表示s[0...i] 是否为 s[0...j] 的子序列
    boolean[][] dp = new boolean[m+1][n+1];

    // 2.base case
    for(int i=0; i<=m; i++){
        // t为空时，肯定就不是了
        dp[i][0] = false;
    }
    for(int j=0; j<=n; j++){
        // s为空，可以把t全部删除，是子序列
        dp[0][j] = true;
    }

    // 3.状态转移
    for(int i=1; i<=m; i++){
        for(int j=i; j<=n; j++){
            // 下面就是状态转移方程：
            if(s.charAt(i-1) == t.charAt(j-1)){
                dp[i][j] = dp[i-1][j-1];
            }else{
                dp[i][j] = dp[i][j-1];
            }
        }
    }
	// 4.明确输出
    return dp[m][n];
}
```

#### [62. 不同路径](https://leetcode-cn.com/problems/unique-paths/)

```java
// 自顶向下：带备忘录的递归
int[][] cache;
public int uniquePaths(int m, int n) {
 
    cache = new int[m+1][n+1];

    return dp(m, n);
}

private int dp(int m, int n){
    // base case1:
    if((m==1 && n==2) || (m==2 && n==1)){
        return 1;
    }
	// base case2:
    if(m==1 || n==1){
        return 1;
    }

    if(m < 1 || n < 1){
        return 0;
    }

    if(cache[m][n] != 0){
        return cache[m][n];
    }

    cache[m][n] = dp(m-1, n) + dp(m, n-1);
    return cache[m][n]; 
}

// 自顶向下：带备忘录的递归---优化
int[][] cache;
public int uniquePaths(int m, int n) {
 
    cache = new int[m+1][n+1];

    return dp(m, n);
}

private int dp(int m, int n){
    // base case:
    if(m==1 || n==1){
        return 1;
    }
    if(cache[m][n] != 0){
        return cache[m][n];
    }

    cache[m][n] = dp(m-1, n) + dp(m, n-1);
     return cache[m][n]; 
}

// 自底向上，dp
public int uniquePaths(int m, int n) {

    // 1.明确状态  dp[i][j]  在i和j位置时的可能
    int[][] dp = new int[m+1][n+1];

    // base case
    dp[1][1] = 1;  // 在 1 1 这个位置当然只有一种可能
    for(int i=1; i<=m; i++){
        // 这些位置也只有一种可能
        dp[i][1] = 1;
    }
    for(int j=1; j<=n; j++){
        // 这些位置也只有一种可能
        dp[1][j] = 1;
    }

    // 3.状态转移
    for(int i=2; i<=m; i++){
        for(int j=2; j<=n; j++){
            // 状态转移方程式，i,j位置的可能只能有 i-1,j / i,j-1 这时候过来
            dp[i][j] = dp[i-1][j] + dp[i][j-1];
        }
    }
	
    // 4.明确输出
    return dp[m][n];
}
```

#### [63. 不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii/)

```java
public int uniquePathsWithObstacles(int[][] obstacleGrid) {

    int m = obstacleGrid.length;
    int n = obstacleGrid[0].length;

    if(obstacleGrid[0][0] == 1 || obstacleGrid[m-1][n-1] == 1){
        return 0;
    }

    // 1.明确状态，dp[i][j]到达i,j的可能路径和
    int[][] dp = new int[m][n];

    // 2.base case
    for(int i=0; i<m; i++){
        // 当出现障碍物时，直接break
        if(obstacleGrid[i][0] == 1){
            break;
        }  
        dp[i][0] = 1;
    }
    for(int j=0; j<n; j++){
        // 当出现障碍物时，直接break
        if(obstacleGrid[0][j] == 1){
            break;
        }  
        dp[0][j] = 1;
    }

    // 3. 状态转移
    for(int i=1; i<m; i++){
        for(int j=1; j<n; j++){
            // 当没出现障碍物
            if(obstacleGrid[i][j] != 1){
                dp[i][j] = dp[i-1][j] + dp[i][j-1]; 
            }
        }
    }

    // 4.明确输出
    return dp[m-1][n-1];
}
```

#### [64. 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

```java
public int minPathSum(int[][] grid) {

    int m = grid.length;
    int n = grid[0].length;

    // 1.明确状态： dp[i][j] 从起始点到 i,j坐标的最小值
    int[][] dp = new int[m][n];

    // 2.base case
    dp[0][0] = grid[0][0];  // 在位置0,0的情况
    for(int i=1; i<m; i++){
        dp[i][0] += grid[i][0] + dp[i-1][0];
    }
    for(int j=1; j<n; j++){
        dp[0][j] += grid[0][j] + dp[0][j-1];
    }

    // 3.状态转移
    for(int i=1; i<m; i++){
        for(int j=1; j<n; j++){
            // 状态方程转移式
            dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
        }
    }
	
    // 4.明确输出
    return dp[m-1][n-1];
}
```

#### [120. 三角形最小路径和](https://leetcode-cn.com/problems/triangle/)

```java
// 这题目就是和找最下的路径相同的，只是这里的走法略有不同，可以向下/右下走
public int minimumTotal(List<List<Integer>> triangle) {

    int m = triangle.size();
    int n = triangle.get(m-1).size();

    // 1.明确状态 dp[i][j] 表示在i行j位置时的最小值
    int[][] dp = new int[m][n];

    // 2.base case
    dp[0][0] = triangle.get(0).get(0);
    for(int i=1; i<m; i++){
        dp[i][0] = dp[i-1][0] + triangle.get(i).get(0);
        dp[i-1][i] = 10000;  // 这是题目条件给的最大值
    }

    // 3.进行状态转移
    for(int i=1; i<m; i++){
        for(int j=1; j<triangle.get(i).size(); j++){
            dp[i][j] = Math.min(dp[i-1][j-1], dp[i-1][j]) + triangle.get(i).get(j);
        }
    }

    // 4.明确输出：走完了，在最下一行找最小值
    int min = Integer.MAX_VALUE;
    for(int j=0; j<n; j++){
        if(dp[m-1][j] < min){
            min = dp[m-1][j];
        }
    }
    return min;
}
```

#### [面试题 17.16. 按摩师](https://leetcode-cn.com/problems/the-masseuse-lcci/)

```java
// 动态规划：同小偷问题
public int massage(int[] nums) {

    int n = nums.length;

    if(n <= 0){
        return 0;
    }

    // 1.明确状态：dp[i] 来了i个预约的最长时间 
    int[] dp = new int[n+1];

    // 2.base case
    dp[0] = 0;
    dp[1] = nums[0];

    // 3.开始状态转移
    for(int i=2; i<=n; i++){
        // 接：上次不接+这次接   不接：上次接的情况
        dp[i] = Math.max(dp[i-1], dp[i-2] + nums[i-1]);
    }
	
    // 4.明确输出
    return dp[n];
}
```

#### [746. 使用最小花费爬楼梯](https://leetcode-cn.com/problems/min-cost-climbing-stairs/)

```java
// 方式1：多加一层
public int minCostClimbingStairs(int[] cost) {

    int n = cost.length;

    if(n <= 0){
        return 0;
    }

    // 给cost加一层，作为无消耗的一层
    int[] new_cost = new int[n+1];
    System.arraycopy(cost, 0, new_cost, 0, n);
    new_cost[n] = 0;
    n += 1;

    // 1.明确状态：dp[i] 在第i个阶梯所消耗的体力
    int[] dp = new int[n+1];

    // 2.base case
    dp[0] = 0;
    dp[1] = new_cost[0];
	
    // 3.状态转移
    for(int i=2; i<=n; i++){  // 从第二层开始推断
        // 状态转移方程式，在i层从哪里跨过来比较省体力
        dp[i] = Math.min(dp[i-2], dp[i-1])  + new_cost[i-1];
    }
    
    // 4.明确输出
    return dp[n];
}

// 方式2：无需再多加，最后判断一下就好了
public int minCostClimbingStairs(int[] cost) {

    int n = cost.length;
	
    // 1.明确状态，在第i层时所消耗的体力
    int[] dp = new int[n];
	
    // 2.base case：从第一层还是第二层开始
    dp[0] = cost[0];
    dp[1] = cost[1];
	
    // 3.状态转移，明确状态方程式
    for(int i=2; i<n; i++){
        dp[i] = Math.min(dp[i-1], dp[i-2]) + cost[i];
    }
	
    // 4.明确输出：这里最后一层为消耗为0，是从倒数第一层还是到倒数第二层跨过来，选择消耗小的那一个
    return Math.min(dp[n-1], dp[n-2]);
}
```

#### [剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

```java
// 方法1：找规律
public int cuttingRope(int n) {
	
    // 1和2直接返回1即可
    if(n == 1 || n == 2){
        return 1;
    }
	
    // 当是3的时候，只能剪成1+2
    if(n==3){
        return 2;
    }
	
    // 当大于4时，则从里面找有几个3
    int res = 1;
    while(n > 4){
        n -= 3;
        res *= 3;
    }

    return res * n;
}

// 方法2：动态规划
public int cuttingRope(int n) {

    // 1.明确状态： d[i] 绳子长为i时的最大值
    int[] dp = new int[n+1];

    // 2.base case
    dp[0] = 0;
    dp[1] = 0;
	
    // 3.状态转移
    for(int i=2; i<=n; i++){       // 状态1的转移：绳子的长度增大
        for(int j=1; j<i; j++){    // 状态2的转移：把绳切成 j*(i-j) 
            int cut_more = dp[i-j] * j;  // 继续切 i-j 这段
            int cut_two = (i-j) * j;     // 不继续切了，就切成两段
            dp[i] = Math.max(dp[i], Math.max(cut_more, cut_two));
        }
    }
	
    // 4.明确输出
    return dp[n];
}
```

#### [剑指 Offer 47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

```java
public int maxValue(int[][] grid) {

    int m = grid.length;
    int n = grid[0].length;

    // 1.明确状态:  dp[i][j] 就是在ij位置时的最大礼物价值
    int[][] dp = new int[m][n];

    // 2.base case: 只有一行/只有一列
    dp[0][0] = grid[0][0];
    for(int i=1; i<m; i++){
        dp[i][0] = dp[i-1][0] + grid[i][0];
    }
    for(int j=1; j<n; j++){
        dp[0][j] = dp[0][j-1] + grid[0][j];
    }

    // 3.开始状态转移
    for(int i=1; i<m; i++){
        for(int j=1; j<n; j++){
            // 状态转移方程
            dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]) + grid[i][j];
        }
    }
	
    // 4.明确输出
    return dp[m-1][n-1];
}
```

#### [221. 最大正方形](https://leetcode-cn.com/problems/maximal-square/)

```java
public int maximalSquare(char[][] matrix) {
	
    int m = matrix.length;
    int n = matrix[0].length;
	
    // 1.明确状态：dp[i][j]表示matrix[i][j]能组成的正方形边长
    int[][] dp = new int[m][n];
	// 存储结果
    int res = 0;
    // 2.base case: 出现'1'说明已经出现了一个小正方形
    for(int i=0; i<m; i++){
        for(int j=0; j<n; j++){
            if(matrix[i][j] == '1'){
                dp[i][j] = 1;
                res = 1;
            }
        }
    }
	
    // 3. 状态转移
    for(int i=1; i<m; i++){
        for(int j=1; j<n; j++){
            if(matrix[i][j] == '1'){
                // 状态转移方程式
                dp[i][j] = Math.min(dp[i-1][j-1], Math.min(dp[i-1][j], dp[i][j-1])) + 1;  // 这里+1正方形扩大了
                // 4. 明确输出，就是最大的边长
                res = Math.max(dp[i][j], res);
            }
        }
    }
	// 4. 明确输出，就是最大的边长
    return res * res;

}
```

