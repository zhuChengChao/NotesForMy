# LeetCode：怒刷 “剑指 Offer”

> 刷题小菜鸡，花了几天时间做了一遍 LeetCode 上给出的 “剑指 Offer” 在此做一下记录
>
> LeetCode主页：[贤余超](https://leetcode-cn.com/u/xian-yu-chao/)

#### [剑指 Offer 03. 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

```java
// 方法1：
// hash表来做:空间换时间的思想
// 时间复杂度 O(n)
// 空间复杂度 O(n)
public int findRepeatNumber(int[] nums) {
    Set<Integer> hashset = new HashSet<>();

    for(int i=0; i<nums.length; i++){
        if(hashset.contains(Integer.valueOf(nums[i]))){
            return nums[i];
        }
        hashset.add(Integer.valueOf(nums[i]));
    }

    return -1;
}


// 方法2：
// 先排序再去比较
// 时间复杂度 O(nlogn + n)
// 空间复杂度 O(1)
public int findRepeatNumber(int[] nums) {

    Arrays.sort(nums);

    for(int i=0; i<nums.length-1; i++){
        if(nums[i] == nums[i+1]){
            return nums[i];
        }
    }
    return -1;
}


// 方法3：
// 原地交换算法：一个萝卜对应一个坑
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int findRepeatNumber(int[] nums) {
    int tmp;

    for(int i=0; i<nums.length; i++){
        while(nums[i] != i){
            if(nums[i] == nums[nums[i]]){
                return nums[i];
            }
            tmp = nums[i];
            nums[i] = nums[tmp];
            nums[tmp] = tmp;
        }
    }
    return -1;
}
```

#### [剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

```java
// 方法1：
// 先来个暴力法
// 时间复杂度：O(m*n)
// 空间复杂度：O(1)
public boolean findNumberIn2DArray(int[][] matrix, int target) {
 
    for(int m=0; m<matrix.length; m++){
        for(int n=0; n<matrix[m].length; n++){
            if(target == matrix[m][n]){
                return true;
            }
        }
    }

    return false;
}

// 方法2：
// 从右上角开始往左下角走，类似二叉搜索树，右上角为根节点
// 时间复杂度：O(m+n)
// 空间复杂度：O(1)
public boolean findNumberIn2DArray(int[][] matrix, int target) {

    if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
        return false;
    }

    int rows = matrix.length, cols = matrix[0].length;
    int row = 0, col = cols-1;  // 从右上角开始
    while(row < rows && col >=0){
        if(target == matrix[row][col]){
            return true;
        }else if(target < matrix[row][col]){
            // 当前值大于target则往左边：左子树方向
            col --;
        }else{
            // 当前值小于target则往右边：右子树方向
            row ++;
        }
    }

    return false;
}
```

#### [剑指 Offer 05. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

```java
// 方法1：正常解法
// 时间复杂度：O(n)
// 空间复杂度：O(n)
public String replaceSpace(String s) {

    StringBuffer sb = new StringBuffer();

    for(int i=0; i<s.length(); i++){
        char c = s.charAt(i);
        if(c == ' '){
            sb.append("%20");
        }else{
            sb.append(c);
        }
    }

    return sb.toString();
}

// 方法2：库函数
public String replaceSpace(String s) {
    return s.replace(" ", "%20");
}
```

#### [剑指 Offer 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

```java
// 方法1：两次遍历
// 时间复杂度：O(2n)
// 空间复制度：O(n)
public int[] reversePrint(ListNode head) {

    if(head == null){
        return new int[0];
    }
    
 	// 第一次遍历
    List<Integer> tmp = new ArrayList<>();
    while(head != null){
        tmp.add(head.val);
        head = head.next;
    }
	
    // 第二次遍历
    int[] res = new int[tmp.size()];
    for(int i=tmp.size()-1, j=0; i>=0; i--){
        res[j++] = tmp.get(i);
    }

    return res;
}

// 方法2：使用栈
// 时间复杂度：O(n)
// 空间复制度：O(n)
public int[] reversePrint(ListNode head) {

    Stack<Integer> stack = new Stack<>();

    while(head != null){
        stack.push(head.val);
        head = head.next;
    }

    int[] res = new int[stack.size()];
    int size = stack.size();
    for(int i=0; i<size; i++){
        res[i] = stack.pop();
    }

    return res;
}

// 方法3：递归，其实也是栈
// 时间复杂度：O(n)
// 空间复制度：O(n)
List<Integer> tmp = new ArrayList<Integer>();

public int[] reversePrint(ListNode head) {
    recur(head);
    int[] res = new int[tmp.size()];

    for(int i = 0; i < res.length; i++){
        res[i] = tmp.get(i);
    }
    return res;
}

private void recur(ListNode head) {
    if(head == null) return;
    // 递归调用
    recur(head.next);
    // 递归回来的时候加入，最后的一个节点最先到这里
    tmp.add(head.val);
}
```

#### [剑指 Offer 07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

> [树专题](https://www.cnblogs.com/zhuchengchao/p/14348664.html)中对重建二叉树的题目有涉及

```java
// 复杂度分析：
// 空间复杂度：建了一个hash表，递归过程中也需要空间 O(n)
// 时间复杂度：还是相当于遍历了一棵树 O(n)
// 加速inorder中对于的根节点的下标寻找
private Map<Integer, Integer> indexMap;

public TreeNode buildTree(int[] preorder, int[] inorder) {
    // 前序遍历：[root [root.left] [root.right]]
    // 中序遍历：[[root.left] root [root.right]]
    if (preorder.length == 0 || inorder.length ==0 || preorder.length != inorder.length){
        return null;
    }

    indexMap = new HashMap<>();
	
    // 加速根据前序遍历中根节点在中序遍历的结果中找该节点的过程
    for(int i=0; i<preorder.length; i++){
        indexMap.put(inorder[i], i);
    }

    return helper(preorder, 0, preorder.length-1, inorder, 0, inorder.length-1);

}

private TreeNode helper(int[] preorder, int pre_s, int pre_e, int[] inorder, int in_s, int in_e){

    if(pre_s > pre_e){
        // 递归结束条件
        return null;
    }
	
    // 很显然，前序遍历的第一个元素就是根节点
    TreeNode root = new TreeNode(preorder[pre_s]);
    // 找到根节点在中序遍历的位置，这样就能分左右子树了
    int index = indexMap.get(preorder[pre_s]);
	
    // 左子树的长度
    int len = index  - in_s;
    root.left = helper(preorder, pre_s+1, pre_s+len, inorder, in_s, index-1);
    root.right = helper(preorder, pre_s+1+len, pre_e, inorder, index+1, in_e);

    return root;
}
```

#### [剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

```java
// 复杂度分析：
// 时间复杂度 入队：O(1)  出队：O(n)
// 空间复杂度：O(n)
LinkedList<Integer> stack1; // 入队都在这里
LinkedList<Integer> stack2; // 出队的时候都从这里出

public CQueue() {
    // // stack1
    // stack1 = new Stack<>();
    // // stack2
    // stack2 = new Stack<>();

    // Stack能做的事LinkedList都能做
    // stack1
    stack1 = new LinkedList<>();
    // stack2
    stack2 = new LinkedList<>();
}

public void appendTail(int value) {
    stack1.push(value);
}

public int deleteHead() {
    // 弹出做特殊处理，都是从2弹出
    if(stack2.isEmpty()){
        // 当2为空，则把1中的全部给2
        while(!stack1.isEmpty()){
            stack2.push(stack1.pop());
        }
    }

    if(stack2.isEmpty()){
        return -1;
    }else{
        return stack2.pop();
    }
}
```

#### [剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

```java
// 求余运算规则: (a+b)%c  --->  (a%c + b%c)%c
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int fib(int n) {

    if(n==0){
        return 0;
    }
    if(n == 1){
        return 1;
    }

    int a = 0, b = 1;
    for(int i=2; i<=n; i++){
        int tmp = b;
        // 这里就相当于用了上面的公式
        b = (a+b) % 1000000007;
        a = tmp;
    }

    return b;
}
```

#### [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

```java
// 和上一题基本一样，不做分析
public int numWays(int n) {

    if(n == 0 || n == 1){
        return 1;
    }

    int a=1, b=1;
    for(int i=2; i<=n; i++){
        int tmp = b;
        b = (a+b) % 1000000007;
        a = tmp;
    }

    return b;
}
```

#### [剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

```java
// 方法0：啊？这？
// 时间复杂度：O(nlogn)
// 空间复杂度：O(1)
public int minArray(int[] numbers) {

    if(numbers.length == 0){
        return 0;
    }
    Arrays.sort(numbers);

    return numbers[0];
}


// 正确解法：二分查找
// 时间复杂度：O(logn)，但是由于后续使用了线性查找，最坏的情况下会到O(n)
// 空间复杂度：O(1)
public int minArray(int[] numbers) {

    if(numbers.length <= 0){
        return 0;
    }

    // 开始二分
    int left = 0, right = numbers.length-1;
    while(left <= right){
        int mid = left + (right - left) / 2;

        if(numbers[mid] > numbers[right]){
            left = mid+1;
        }else if(numbers[mid] < numbers[right]){
            right = mid;
        }else if(numbers[mid] == numbers[right]){
            // 直接线性查找好了
            return findMin(numbers, left, right);
        }
    }
    return left >= numbers.length ? numbers[numbers.length-1] : numbers[left];
}

private int findMin(int[] numbers,int start,int end){
    int result = numbers[start];
    for(int i = start;i <= end;i++){
        if (numbers[i] < result) result = numbers[i];
    }
    return result;
}
```

#### [剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

```java
// dfs+回溯的思想
// 时间复杂度: 不太好分析
// 空间复杂度：即递归的深度，也不太好分析
public boolean exist(char[][] board, String word) {

    char[] words = word.toCharArray(); 

    for(int i=0; i<board.length; i++){
        for(int j=0; j<board[0].length; j++){
            if(dfs(board, words, i, j, 0)) return true;
        }
    }

    return false;
}

private boolean dfs(char[][] board, char[] words, int i, int j, int k){
    
    if(i >= board.length || i < 0 || j >=board[0].length || j < 0 || board[i][j] != words[k]){
        return false;
    }

    if(k == words.length - 1){
        return true;
    }

    // 回溯：
    // 到这里说明的是第k个已经相等了，接下来要判断第k+1个字符了
    board[i][j] = '\0';  // 用于剪枝操作
    boolean res = dfs(board, words, i+1, j, k+1) || dfs(board, words, i-1, j, k+1) || 
        dfs(board, words, i, j+1, k+1) || dfs(board, words, i, j-1, k+1);
    board[i][j] = words[k];
    return res;
}
```

#### [剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

```java
// 方法1：dfs+回溯思想
// 时间复杂度 O(mn)
// 空间复杂度:O(mn)
public int movingCount(int m, int n, int k) {
    boolean[][] visited = new boolean[m][n];
    return dfs(visited, m, n, k, 0, 0);
}

private int dfs(boolean[][] visited, int m, int n, int k, int i, int j){
    // 判断是否能到这个坐标
    if(i >= m || j >= n || visited[i][j] || (bitSum(i) + bitSum(j) > k)) {
        return 0;
    }
	
    // 能到这里，则继续向左边/下边走
    visited[i][j] = true;
    return dfs(visited, m, n, k, i+1, j) + dfs(visited, m, n, k, i, j+1) + 1;
}

private int bitSum(int n){

    int sum = 0;
    while(n != 0){
        sum += n%10;
        n /= 10;
    }

    return sum;
}

// 方法2：bfs
// 时间复杂度 O(mn)
// 空间复杂度:O(mn)
public int movingCount(int m, int n, int k) {

    boolean[][] visited = new boolean[m][n];
    int res = 0;
    // bfs求解
    Queue<int[]> queue = new LinkedList<>();
    // 坐标 i j  数位和: i的数位和  j的数位和
    queue.add(new int[]{0, 0, 0, 0});

    while(!queue.isEmpty()){
        int[] point = queue.poll();
        int i=point[0], j=point[1], s_i=point[2], s_j=point[3];
        if( i>=m || j>=n || k<s_i+s_j || visited[i][j]){
            continue;
        }

        visited[i][j] = true;
        res ++;
        queue.add(new int[]{i+1, j, bitSum(i+1), bitSum(j)});
        queue.add(new int[]{i, j+1, bitSum(i), bitSum(j+1)});
    }

    return res;
}

private int bitSum(int n){

    int sum = 0;
    while(n != 0){
        sum += n%10;
        n /= 10;
    }

    return sum;
}
```

#### [剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

```java
// 动态规划：
// 时间复杂度 O(n^2)
// 空间复杂度:O(n)
public int cuttingRope(int n) {

    // 明确状态： d[i] 绳子长为i时的最大值
    int[] dp = new int[n+1];

    // base case
    dp[0] = 0;  // 长度为0当然为0
    dp[1] = 0;  // 长度为1时，不能切分
	
    // 状态转移
    for(int i=2; i<=n; i++){       // 状态1的转移：绳子的长度增大
        for(int j=1; j<i; j++){    // 状态2的转移：把绳切成 i*(i-j) 
            int cut_more = dp[i-j] * j;  // i-j这段能切的最大值*j这段的最大值
            int cut_two = (i-j) * j;     // 不继续切了，切成两段
            dp[i] = Math.max(dp[i], Math.max(cut_more, cut_two));
        }
    }

    return dp[n];
}
```

#### [剑指 Offer 14- II. 剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)

```java
// 和上一题不同，次数已经不能再使用动态规划了
// 参考：Java贪心 思路讲解 —— fanhua
// 最优：3  次优：2 等价于 4  最差：1
// 结合公式：(x*y)%p = ((x%p) * (y%p))%p
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int cuttingRope(int n) {

    // base case
    if(n == 2){
        return 1;
    }
    if(n == 3){
        return 2;
    }

    long res = 1;
    while(n > 4){  // 每次都拆一个3，3是最优的情况
        res = res * 3;  
        res %= 1000000007;
        n -= 3;
    }
    // 最后还剩下长度4/2这种长度，直接乘上就行
    return (int) (res * n % 1000000007);
}
```

#### [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

```java
// 方法1：与运算
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int hammingWeight(int n) {
    
    int ans = 1;  // 标志位
    int res = 0;
    for(int i=1; i<=32; i++){
        if((n & ans) != 0){
            res++;
        }
        ans = ans << 1;
    }
    return res;
}

// 方法2：另一种方式进行与运算
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int hammingWeight(int n) {
    int res = 0;
    while(n != 0) {
        res += n & 1;
        // java中的无符号右移
        n >>>= 1;
    }
    return res;
}
```

#### [剑指 Offer 16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

```java
// 方法1：暴力法
// 直接超出时间限制.... 1.00000   2147483647
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public double myPow(double x, int n) {

    double res = 1;
    boolean flag = false;
    if(n < 0){
        flag = true;
        n = -n;
    }

    for(int i=n; i>=1; i--){
        res *= x;
    }
    if(flag){
        res = 1/res;
    }
    return res;
}

// 快速幂算法：递归形式
// 依靠了公式: 偶数：x^n = (x^2)^(n/2)  奇数：x^n = x(x^2)^(n/2)
// 时间复杂度：O(logn)
// 空间复杂度：O(1),这里忽略了递归占用的空间
public double myPow(double x, int n) {
    // 如果n等于0，直接返回1
    if (n == 0)
        return 1;
    // 如果n小于0，把它改为正数，并且把1/x提取出来一个
    // 这是因为最小值溢出的问题
    if (n < 0)
        return 1/x * myPow(1 / x, -n - 1);
    // 根据n是奇数还是偶数来做不同的处理
    return (n % 2 == 0) ? myPow(x * x, n / 2) : x * myPow(x * x, n / 2);
}

// 快速幂算法：迭代形式
// 时间复杂度：O(logn)
// 空间复杂度：O(1)
public double myPow(double x, int n) {
    double res = 1.0;
    long t = n;
    if(t < 0){
        t = -t;
    }
    while(t > 0){
        if(t % 2 == 1){
            // 奇数的情况多*一个x
            res *= x;
        }
        // 变为 x^2 , 则t就可以减半了
        x *= x;
        t /= 2;
    }
    return n < 0 ? 1.0 / res : res;
}
```

#### [剑指 Offer 17. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

```java
// 不考虑大数问题时解题：
// 时间复杂度：0(10^n)
// 空间复杂度：0(1) 返回结果不计入
public int[] printNumbers(int n) {
    int max = 9;// 最大的n位数，最终肯定是n个9
    for(int i=2; i<=n; i++){
        max = max * 10;
        max += 9;
    }
	
    // 打印出结果
    int[] res = new int[max];
    for(int i=1; i<=max; i++){
        res[i-1] = i;
    }

    return res;
}


// 考虑大数问题解题：分析如下
// 即当n较大时，max会超出int32的取值范围
// 无论是什么类型，都会越界，因此使用String类型
// 因此通过全排列的方式来组成数字，即回溯算法的思想
// 时间复杂度：0(10^n)
// 空间复杂度：0(10^n)

// 代码：
// 用于存储满足条件的字符串
List<Integer> list;
StringBuilder sb;

public int[] printNumbers(int n) {

    // 存储符合条件的'数'  '数'的类型是字符串
    list = new ArrayList<>();  // 用LinkedList的最后一个案例会超时
    sb = new StringBuilder();

    // 这里进行回溯
    dfs(n, 0);

    int[] res = new int[list.size()];
    // 存入数组
    for (int i = 0; i < res.length; i++) {
        res[i] = list.get(i);
    }
    return res;
}

private void dfs(int n, int index) {
    // 递归结束的条件
    if(index == n){
        while (sb.length() != 0 && sb.charAt(0) == '0') {
            // 将左边多余的0删除
            sb.deleteCharAt(0);
        }
        // 将字符串形式的'数'，转化为整数
        if(sb.length() != 0){
            list.add(Integer.valueOf(sb.toString()));
        }
        return;
    }

    for(int j=0; j<10; j++){
        // 深度搜索下一位
        sb.append(j);
        dfs(n, index + 1);
        if(sb.length() != 0){
            sb.deleteCharAt(sb.length() - 1);
        }
    }
}
```

#### [剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

```java
// 正常遍历：
// 时间复杂度：0(n)
// 空间复杂度：0(1)
public ListNode deleteNode(ListNode head, int val) {

    if(head == null){
        return head;
    }

    // 特殊处理：头结点就是要删除的节点
    if(head.val == val){
        return head.next;
    }
	
    ListNode ans = new ListNode(-1);
    ans.next = head;

    while(head.next != null){
        if(head.next.val == val){
            head.next = head.next.next;
            return ans.next;
        }
        head = head.next;
    }

    return ans.next;
}
```

#### [剑指 Offer 19. 正则表达式匹配](https://leetcode-cn.com/problems/zheng-ze-biao-da-shi-pi-pei-lcof/)

```java
// 方法1：递归
public boolean isMatch(String s, String p) {
 
    if(p.length() == 0){
        return s.length() == 0;
    }

    // 匹配首位字符
    boolean fist = false;
    if(s.length() != 0){
        char s1 = s.charAt(0);
        char p1 = p.charAt(0);
        fist = (s1 == p1);
        if(p1 == '.'){
            // . 在这里判断了
            fist = true;
        }
    }

    if(p.length() >= 2 && p.charAt(1) == '*'){
        // 判断是否有*
        return isMatch(s, p.substring(2, p.length())) || 
                (fist && isMatch(s.substring(1, s.length()), p));
    }else{
        return fist && isMatch(s.substring(1, s.length()), p.substring(1, p.length()));
    }
}

// 方法2：上述方法的改进带个备忘录的dp
int[][] cache;  // 0 没算过  1 为true -1 为false
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
		ans = dp(s, i, p, j+2) ||          // 直接掉过*
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

#### [剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

> 这种题目真的很烦人，直接看答案+CV的

```java
public boolean isNumber(String s) {
    if(s == null || s.length() == 0){
        return false;
    }
    // 标记是否遇到相应情况
    boolean numSeen = false;
    boolean dotSeen = false;
    boolean eSeen = false;
    // 去除空格转换为字符数组
    char[] str = s.trim().toCharArray();
    for(int i = 0;i < str.length; i++){
        if(str[i] >= '0' && str[i] <= '9'){
            // 出现数组了标记一下
            numSeen = true;
        }else if(str[i] == '.'){
            // .之前不能出现.或者e
            if(dotSeen || eSeen){
                return false;
            }
            dotSeen = true;
        }else if(str[i] == 'e' || str[i] == 'E'){
            // e之前不能出现e，必须出现数
            if(eSeen || !numSeen){
                return false;
            }
            eSeen = true;
            numSeen = false; // 重置numSeen，排除123e或者123e+的情况,确保e之后也出现数
        }else if(str[i] == '-' || str[i] == '+'){
            //+-出现在0位置或者e/E的后面第一个位置才是合法的
            if(i != 0 && str[i-1] != 'e' && str[i-1] != 'E'){
                return false;
            }
        }else{//其他不合法字符
            return false;
        }
    }
    return numSeen;
}
```

#### [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

```java
// 方法1：暴力法
// 时间空间复杂度：O(n)
public int[] exchange(int[] nums) {

    int[] res = new int[nums.length];
    int left = 0, right = nums.length-1;
    for(int i=0; i<nums.length; i++){
        if(nums[i] % 2 == 0){
            res[right] = nums[i];
            right--;
        }else{
            res[left] = nums[i];
            left++;
        }
    }
    return res;
}

// 方法2：双指针
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int[] exchange(int[] nums) {

    int left = 0, right = nums.length-1;
    while(left < right){
        // 左指针向右遍历找到第一个偶数
        while(nums[left]%2==1 && left < right){
            left ++;
        }
		
        // 右指针向左遍历找到第一个奇数
        while(nums[right]%2==0 && left < right){
            right --;
        }
		
        // 交换左右指针指向的值
        if(left < right){
            int tmp = nums[left];
            nums[left] = nums[right];
            nums[right] = tmp;
        }
    }

    return nums;
}
```

#### [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

```java
// 快慢指针/双指针
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public ListNode getKthFromEnd(ListNode head, int k) {
    if(head == null){
        return null;
    }

    ListNode node1 = head;
    ListNode node2 = head;
    // 先让快指针走个k步
    while(k-- != 0){
        node1 = node1.next;
    }
	
    // 然后快慢指针一起走，当快指针走到头的时候，返回慢指针即可
    while(node1 != null){
        node2 = node2.next;
        node1 = node1.next;
    }

    return node2;
}
```

#### [剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

```java
// 正常迭代反转
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public ListNode reverseList(ListNode head) {
    ListNode curr = head;
    ListNode pre = null;
    ListNode temp = null;

    while(curr != null){
        temp = curr.next;
        curr.next = pre;
        pre = curr;
        curr = temp;
    }
    return pre;
}
```

#### [剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

```java
// 正常迭代反转
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {

    ListNode res = new ListNode(-1);
    ListNode head = new ListNode(-1);
    res = head;

    while(l1 != null && l2 != null){

        if(l1.val <= l2.val){
            head.next = l1;
            l1 = l1.next;
        }else{
            head.next = l2;
            l2 = l2.next;
        }

        head = head.next;
    }
	
    // 口少口巴 
    head.next = l1==null ? l2 : l1;

    return res.next;
}
```

#### [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

```java
// 树专题中双递归的技巧
// 时间复杂度：O(mn),其中m为A的节点数量，n为B的节点数量，每次都是以A作为根节点去匹配B数的n个节点
// 空间复杂度：O(m) 最大递归深度
public boolean isSubStructure(TreeNode A, TreeNode B) {

    if(A==null || B==null){
        return false;
    }

    // 递归遍历A树与B树进行判断
    return dfs(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
}

private boolean dfs(TreeNode A, TreeNode B){
    
    if(A == null && B != null){
        // A已经空了，B还有节点，很显然B不是A的子结构
        return false;
    }else if(B == null){
        // B已经空了，A还有节点，能递归到这里，说明B是A的子结构
        return true;
    }else if(A.val != B.val){
        // 值不同，那就不用看了，B必不是A的子结构
        return false;
    }
	// 继续迭代
    return dfs(A.left, B.left) && dfs(A.right, B.right);
}
```

#### [剑指 Offer 27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

```java
// bfs
// 时间复杂度：O(n)
// 空间复杂度：O(n)
public TreeNode mirrorTree(TreeNode root) {

    if(root == null){
        return null;
    }

    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
 

    while(!queue.isEmpty()){
     
        int size = queue.size();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();

            if(node.right != null){
                queue.offer(node.right);
            }

            if(node.left != null){
                queue.offer(node.left);
            }

            // 这里反转一下就好了,自顶向下，
            TreeNode tmp = node.left;
            node.left = node.right;
            node.right = tmp;
        }
    }

    return root;
}

// dfs
// 时间复杂度：O(n)，每个节点都还是要被遍历到
// 空间复杂度：O(n)，递归所需的空间
public TreeNode mirrorTree(TreeNode root) {
    if(root == null){
        return root;
    }
	
    // 借助个临时节点来避免覆盖后找不到左（右）节点
    TreeNode tmp = root.left;
    // 递归反转
    root.left = mirrorTree(root.right);
    root.right = mirrorTree(tmp);

    return root;
}
```

#### [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

```java
// bfs：建立两个队列
// 时间复杂度：O(n)
// 空间复杂度：O(2n)，这里建立了两个队列
public boolean isSymmetric(TreeNode root) {

    if(root == null){
        return true;
    }
 
    // 从左到右层序遍历
    Queue<TreeNode> queue1 = new LinkedList<>();
    // 从右到左层序遍历
    Queue<TreeNode> queue2 = new LinkedList<>();
    queue1.offer(root);
    queue2.offer(root);

    while(!queue1.isEmpty()){

        int size1 = queue1.size();
        int size2 = queue2.size();
        if(size1 != size2){
            return false;
        }

        for(int i=0; i<size1; i++){
            TreeNode node1 = queue1.poll();
            TreeNode node2 = queue2.poll();

            if(node1 == null && node2 == null){
                continue;
            }
            
            // 对每个节点进行比较
            if((node1 == null && node2 != null)||
                (node2 == null && node1 != null)||
                (node1.val != node2.val)){
                return false;
            }

            queue1.offer(node1.left);
            queue1.offer(node1.right);

            queue2.offer(node2.right);
            queue2.offer(node2.left);
        }
    }

    return true;
}

// dfs：
// 时间复杂度：O(n)
// 空间复杂度：O(n)，递归消耗
public boolean isSymmetric(TreeNode root) {
    if (root == null)
        return true;
    // 1.明确：递归的函数要干什么？
    // 函数的作用是判断传入的两个树是否镜像
    // 输入：TreeNode left, TreeNode right  
    // 输出：是：true，不是：false
    return helper(root.left, root.right);
}

public boolean helper(TreeNode root1, TreeNode root2) {
    // 2.明确：递归停止的条件是什么
    if (root1 == null && root2 == null){
        // 左节点和右节点都为空 -> 倒底了都长得一样 ->true
        return true;
    }
    if (root1 == null || root2 == null){
        // 左节点为空的时候右节点不为空，或反之 -> 长得不一样-> false
        return false;
    }
    if (root1.val != root2.val){
        // 左右节点值不相等 -> 长得不一样 -> false
        return false;
    }
    // 3.明确：从某层到下一层的关系是什么
    // 要想两棵树镜像，那么一棵树左边的左边要和二棵树右边的右边镜像，一棵树左边的右边要和二棵树右边的左边镜像
    // 调用递归函数传入左左和右右 调用递归函数传入左右和右左
    // 只有左左和右右镜像且左右和右左镜像的时候，我们才能说这两棵树是镜像的
    return helper(root1.left, root2.right) && helper(root1.right, root2.left);
}
```

#### [剑指 Offer 29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

```java
// 直接遍历,最好画图理解一下这个过程
// 时间复杂度：O(m*n)
// 空间复杂度：O(1) 
public int[] spiralOrder(int[][] matrix) {

    if(matrix == null || matrix.length == 0 || matrix[0].length == 0){
        return new int[0];
    }

    // 存储当前最外层的边界,从最外层开始遍历，每遍历完最外层就对下面的参数修改
    int rows = matrix.length-1, cols = matrix[0].length-1, row = 0, col = 0;
    int size = (rows+1)*(cols+1);
    int[] res = new int[size];

    int index = 0;
    while(row <= rows && cols <= cols){

        // 上
        for(int i=col; i <= cols; i++){
            res[index++] = matrix[row][i];
            // 元素个数相同，则无需再遍历了，直接返回
            if(index == size) return res;
        }

        // 右
        for(int i=row+1; i<=rows; i++){
            res[index++] = matrix[i][cols];
            if(index == size) return res;
        }

        // 下
        for(int i=cols-1; i>=col; i--){
            res[index++] = matrix[rows][i];
            if(index == size) return res;
        }

        // 左
        for(int i=rows-1; i>row; i--){
            res[index++] = matrix[i][col];
            if(index == size) return res;
        }

        // 遍历完最外层就修改下面的参数，开始遍历内层
        row++; col++; rows--; cols--;

    }

    return res;
}
```

#### [剑指 Offer 30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

```java
// 题目要求的是调用 min、push 及 pop 的时间复杂度都是 O(1)
class MinStack {

    /** initialize your data structure here. */
    Stack<Integer> stack1, stack2;
    public MinStack() {
        // 这里建立两个栈来实现 min 为O(1)的复杂度
        stack1 = new Stack<>();
        stack2 = new Stack<>();
    }
    
    public void push(int x) {
		// 栈1存储的是正常的元素
        stack1.push(x);
        // 栈2专门存储最小元素
        if(stack2.isEmpty() || x <= stack2.peek()){
            // 设法维护好 栈B的元素，使其保持非严格降序
            stack2.push(x);
        }
    }
    
    public void pop() {
        // 栈1弹出元素的时候，若栈2的元素相同也需要弹出
        if(stack1.pop().equals(stack2.peek())){
            stack2.pop();
        }
    }
    
    public int top() {
        // 栈1存储的是正常的元素
        return stack1.peek();
    }
    
    public int min() {
        // 栈2专门存储最小元素
        return stack2.peek();
    }
}
```

#### [剑指 Offer 31. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

```java
// 复杂度分析：
// 时间：O(n)
// 空间：O(n)
public boolean validateStackSequences(int[] pushed, int[] popped) {

    // Stack<Integer> stack = new Stack<>();
    // Java里面所有和栈相关的操作都应该用Deque，避免使用Stack
    Deque<Integer> stack = new LinkedList<>();
	
    // 判断合不合法，用个栈试一试
    int pop_index = 0;
    for(int i=0; i<pushed.length; i++){
        // 压栈是顺序是固定的
        stack.push(pushed[i]);
        // 当栈顶的元素和此时出栈序列的元素相同时，则出栈
        while(!stack.isEmpty() && stack.peek() == popped[pop_index]){
            stack.pop();
            pop_index ++;
        }
    }
	
    // 最后更具栈是否为空来判断出栈序列是否合法
    if(stack.isEmpty()){
        return true;
    }else{
        return false;
    }
}
```

#### [剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

```java
// 标准的层序遍历
// 时间复杂度：O(n)
// 空间复杂度：O(n) 
public int[] levelOrder(TreeNode root) {

    if(root == null){
        return new int[0];
    }

    List<Integer> res = new ArrayList<>();
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while(!queue.isEmpty()){

        int size = queue.size();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();

            res.add(node.val);

            if(node.left != null){
                queue.offer(node.left);
            }
            if(node.right != null){
                queue.offer(node.right);
            }
        }
    }

    int[] ans = new int[res.size()];
    for(int i=0; i<res.size(); i++){
        ans[i] = res.get(i);
    }
    return ans;
}
```

#### [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

```java
// 和上一题一样的层序遍历
public List<List<Integer>> levelOrder(TreeNode root) {

    List<List<Integer>> res = new LinkedList<>();
    if(root == null){
        return res;
    }
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while(!queue.isEmpty()){

        int size = queue.size();
        List<Integer> oneLayer = new LinkedList<>();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();

            oneLayer.add(node.val);

            if(node.left != null){
                queue.offer(node.left);
            }
            if(node.right != null){
                queue.offer(node.right);
            }
        }
        res.add(oneLayer);
    }
    return res;
}
```

#### [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

```java
// 方法1：bfs
// 时间复杂度：O(n)
// 空间复杂度：O(n) 
public List<List<Integer>> levelOrder(TreeNode root) {

    List<List<Integer>> res = new LinkedList<>();
    if(root == null){
        return res;
    }
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    boolean flag = true;

    while(!queue.isEmpty()){
        int size = queue.size();
        List<Integer> oneLayer = new LinkedList<>();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();

            oneLayer.add(node.val);
            if(node.left != null){
                queue.offer(node.left);
            }
            if(node.right != null){
                queue.offer(node.right);
            }
        }
        // 区分一下层,做到之字形的效果
        if(flag){
            res.add(oneLayer);
        }else{
            List<Integer> tmp = new LinkedList<>();
            for(int i=oneLayer.size()-1; i>=0; i--){
                tmp.add(oneLayer.get(i));
            }
            res.add(tmp);
        }
        flag = !flag;
    }
    return res;
}


// 方法2：双端队列
// 时间复杂度：O(n)
// 空间复杂度：O(n)
public List<List<Integer>> levelOrder(TreeNode root) {

    List<List<Integer>> res = new LinkedList<>();
    if(root == null){
        return res;
    }
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    boolean flag = true;

    while(!queue.isEmpty()){
        int size = queue.size();
        // 使用双端队列
        LinkedList<Integer> oneLayer = new LinkedList<>();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();
			// 在这里分辨层
            if(flag){
                // add == addLast 添加到队尾
                oneLayer.add(node.val);
            }else{
                oneLayer.addFirst(node.val);
            }
            if(node.left != null){
                queue.offer(node.left);
            }
            if(node.right != null){
                queue.offer(node.right);
            }
        }
        res.add(oneLayer);
        flag = !flag;
    }
    return res;
}
```

#### [剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

```java
// dfs递归解题：后续遍历有 [[root.left][root.right]root]
// 时间复杂度：O(n^2) 递归深度n，递归函数中还要进行左右子树的判断
// 空间复杂度：O(n) n为数的节点个数，递归的深度
public boolean verifyPostorder(int[] postorder) {

    return dfs(postorder, 0, postorder.length-1);
}

private boolean dfs(int[] postorder, int s, int e){

    if(s>=e){
        // 单节点
        return true;
    }
	// 找到二叉搜索树的根节点
    int root = postorder[e];

    // 左子树树的跨度
    int left_index = s;
    while(postorder[left_index] < root) left_index++;
    // 右子树的跨度
    int right_index = left_index;
    while(postorder[right_index] > root) right_index++;

    // 判断：最终左子树+右子树的长度是否满足二叉搜索树
    if(right_index != e){
        return false;
    }
    // 递归判断左子树是否有效
    boolean left = dfs(postorder, s, left_index-1);
    // 递归判断右子树是否有效
    boolean right = dfs(postorder, left_index, e-1);

    return left && right;
}
```

#### [剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

```java
// 正常的dfs/回溯思想
// 时间复杂度：O(n)
// 空间复杂度：O(n)
List<List<Integer>> res;
public List<List<Integer>> pathSum(TreeNode root, int sum) {
    res = new LinkedList<>();
    if(root == null){
        return res;
    }

    List<Integer> path = new LinkedList<>();
    dfs(root, sum, path, 0);

    return res;

}

private void dfs(TreeNode root, int sum, List<Integer> path, int target){

    if(root==null){
        return;
    }

    // 叶子节点
    if(root.left == null && root.right == null){
        if(target+root.val == sum){
            // 满足一个路径
            path.add(root.val);
            List<Integer> tmp = new LinkedList(path);
            res.add(tmp);
            path.remove(path.size()-1);
        }
        return;
    }

    // 非叶子节点
    target += root.val;
    path.add(root.val);
    dfs(root.left, sum, path, target);
    dfs(root.right, sum, path, target);
    path.remove(path.size() - 1);

}
```

#### [剑指 Offer 35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

```java
// 方法1：建立一个hash表，两次遍历完成复制
// 时间复杂度：O(2n)
// 空间复杂度：O(2n)
public Node copyRandomList(Node head) {
	
    // 虚拟头
    Node ans = new Node(-1);
    ans.next = head;

    Node tmp;
    // node1: 原节点  node2：新节点
    HashMap<Node, Node> hashMap = new HashMap<>();
	// 第一次遍历链表，记录节点
    while(head != null){
        tmp = new Node(head.val);
        hashMap.put(head, tmp);
        head = head.next;
    }

    // 在第二次遍历，完成新链表的指向操作
    head = ans.next;
    while(head != null){
        // 拿出新的节点
        tmp = hashMap.get(head);
        // next指针的指向
        tmp.next = hashMap.get(head.next);
        // 随机指针的指向
        tmp.random = hashMap.get(head.random);
        head = head.next;
    }

    return hashMap.get(ans.next);
}

// 方法2：原地操作，出去必要的输出占用，只占用常量级别的空间
// 时间复杂度：O(n)
// 空间复杂度：O(1) 除去必要的输出
public Node copyRandomList(Node head) {

    if(head == null){
        return head;
    }

    // 虚拟头
    Node ans = new Node(-1);
    ans.next = head;

    // 将拷贝节点放到原节点后面
    // 例如1->2->3这样的链表就变成了这样1->1'->2->2'->3->3'
    while(head != null){
        Node tmp = new Node(head.val);
        tmp.next = head.next;
        head.next = tmp;
        head = tmp.next;
    }

    // 把拷贝节点的random指针安排上
    head = ans.next;
    while(head != null){
        if(head.random != null){
            head.next.random = head.random.next;
        }
        head = head.next.next;
    }

    // 分离拷贝节点和原节点，变成1->2->3和1'->2'->3'两个链表，后者就是答案
    head = ans.next;
    Node res = new Node(-1);  // 需要返回的虚拟头
    res.next = head.next;
    while(head != null){
        Node tmp = head.next.next;
        if (tmp == null){
            // 到末尾了，但还是要回复原链表最后一个节点指向的null
            head.next = tmp;
            break;
        }
        // 修改新链表的指向
        head.next.next = tmp.next;
        // 恢复原链表的指向
        head.next = tmp;
        // 向后遍历原连表，继续断开接上
        head = tmp;
    }

    return res.next;
}
```

#### [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

```java
// 复杂度分析：
// 时间：O(n)
// 空间：O(n)
Node head, pre;
public Node treeToDoublyList(Node root) {
    if(root == null) return root;
	
    // dfs递归修改指向
    dfs(root);

    // 进行头节点和尾节点的相互指向
    head.left = pre;
    pre.right = head;

    return head;
}

private void dfs(Node root){

    if(root==null) return;
    // 递归遍历树的左节点
    dfs(root.left);
    // 二叉搜索树中序遍历是有序的，处理中间节点，左边的比你小，右边的比你大
    if(pre == null){
        // pre用于记录双向链表中位于root左侧的节点，即上一次迭代中的root,当pre==null时，root左侧没有节点,即此时root为双向链表中的头节点
        // 第一次遍历到最左节点，记录此时的root为head
        head = root;
    }else{
        // 反之，pre!=null时，root左侧存在节点pre，需要进行pre.right=root的操作。
        pre.right = root;
    }

    // 这里只需要指明left即可
    root.left = pre;
    // 更新pre
    pre = root;
    // 递归遍历树的右节点
    dfs(root.right);
}
```

#### [剑指 Offer 37. 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/)

```java
// 复杂度分析：
// 序列化： 层序遍历时间，空间复杂度都为O(n)
// 反序列化： 还是遍历的所有的节点且用到了队列，空间复杂度都为O(n)
public class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        
        StringBuilder sb = new StringBuilder();
        if(root == null){
            return null;
        }

        sb.append("[");

        // 层序遍历
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while(!queue.isEmpty()){
            int size = queue.size();

            for(int i=0; i<size; i++){
                TreeNode node = queue.poll();

                if(node == null){
                    sb.append("null,");
                    continue;
                }else{
                    sb.append(node.val + ",");
                }

                queue.offer(node.left);
                queue.offer(node.right);
            }
        }

        String substring = sb.substring(0, sb.length() - 1);
        substring = substring + "]";

        return substring;

    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {

        if(data == null){
            return null;
        }

        String s_data = data.substring(1, data.length() - 1);
        String[] datas = s_data.split(",");

        Queue<TreeNode> queue = new ArrayDeque<>();
        TreeNode root = new TreeNode(Integer.parseInt(datas[0]));
        queue.offer(root);
        
		// 第一次做的时候用了两个指针，其实一个指针就行了，更加简洁明了
        int index = 1;  
        while(!queue.isEmpty()){
            TreeNode node = queue.poll();

            if(!datas[index].equals("null")){
                node.left = new TreeNode(Integer.parseInt(datas[index]));
                queue.offer(node.left);
            }
            index++;

            if(!datas[index].equals("null")){
                node.right = new TreeNode(Integer.parseInt(datas[index]));
                queue.offer(node.right);
            }
            index++;
        }
        return root;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec codec = new Codec();
// codec.deserialize(codec.serialize(root));
```

#### [剑指 Offer 38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

```java
// 一个经典的回溯问题
// 时间复杂度：O(n!) n为字符串长度，n*(n-1)*(n-2)*...*1
// 空间复杂度：O(n^2) 
List<String> res = new LinkedList<>();
boolean[] visited;
public String[] permutation(String s) {

    char[] chars = s.toCharArray();
    // 排序一下后面好剪枝
    Arrays.sort(chars);
    StringBuilder track = new StringBuilder();
    visited = new boolean[s.length()];

    backtrack(chars, track);

    // 输出
    return res.toArray(new String[res.size()]);
}

private void backtrack(char[] chars, StringBuilder track){

    // 触发结束条件
    if(track.length() == chars.length){
        res.add(track.toString());
        return;
    }

    for(int i=0; i<chars.length; i++){
        // 排除不合法的选择，当有重复的字符串出现时，不允许先访问后面再访问前面
        if(visited[i] || (i>0 && chars[i] == chars[i-1] && !visited[i-1])){
            continue;
        }

        // 前序：做选择
        visited[i] = true;
        track.append(chars[i]);
        // 进入下一层决策树
        backtrack(chars, track);
        // 后序：取消选择
        track.deleteCharAt(track.length() - 1);
        visited[i] = false;
    }
}
```

#### [剑指 Offer 39. 数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

```java
// 方法1：直接hash表
// 时间复杂度：O(n)
// 空间复杂度：O(n/2)
public int majorityElement(int[] nums) {

    Map<Integer, Integer> map = new HashMap<>();

    for(int i=0; i<= nums.length; i++){

        map.put(Integer.valueOf(nums[i]), map.getOrDefault(Integer.valueOf(nums[i]), 0) + 1);

        if(map.get(Integer.valueOf(nums[i])) > nums.length/2){
            return nums[i];
        }
    }    

    return -1;
}

// 解法2：排序取中位数
// 排序算法时间复杂度O(nlogn)，
// 空间复杂度O(1)
public int majorityElement(int[] nums) {

    Arrays.sort(nums);

    return nums[nums.length/2];
} 

// 解法3：摩尔投票法
// 也可以理解成混战极限一换一，不同的两者一旦遇见就同归于尽，最后活下来的值都是相同的，即要求的结果
// 时间复杂度O(n)，空间复杂度O(1)
public int majorityElement(int[] nums) {
    int res = 0, count = 0;
    for(int i=0; i<nums.length; i++){
        if(count == 0){
            // 假设当前的数为众数，给你投票后续让你去火拼
            res = nums[i];
            count++;
        }else{
            if(res == nums[i]) count++;  // 票数增加
            else count--;  // 抵消
        }
    }
    // 最终一定是票数大于一半的获胜
    return res;
}
```

#### [剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

```java
// 方法1：固定堆：大顶堆，固定一个大小为k的大顶堆可以快速求出第k小的数
// 时间复杂度: O(nlog k)
// 空间复杂度：O(k)
public int[] getLeastNumbers(int[] arr, int k) {

    if(arr.length <= 0 || k == 0){
        return new int[0];
    }

    // 建立大顶堆：固定大小为k，
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(3, (a, b) -> b-a);

    for(int i=0; i<arr.length; i++){
        if(maxHeap.size() < k){
            maxHeap.offer(arr[i]);
            continue;
        }

        if(maxHeap.peek() > arr[i]){
            maxHeap.poll();
            maxHeap.offer(arr[i]);
        }
    }

    int size = maxHeap.size();
    int[] res = new int[size];
    for(int i=0; i<size; i++){
        res[i] = maxHeap.poll();
    }

    return res;
}


// 方法2：快排，用了https://www.cnblogs.com/zhuchengchao/p/14403781.html中快排的代码
// 时间复杂度：根据已经分好的数组与k比较，其实能到O(k)的
// 空间复杂度：原地排序O(1)
public int[] getLeastNumbers(int[] arr, int k) {

    if(arr.length == 0 || k == 0){
        return new int[0];
    }

    // 最后一个参数表示我们要找的是下标为k-1的数
    return quickSort(arr, 0, arr.length-1, k-1);
}

private int[] quickSort(int[] nums, int start, int end, int k){
    int j = partition(nums, start, end);
    if(k == j){
        // 已经到达个数了 返回即可
        return Arrays.copyOf(nums, j + 1);
    }
    // 优化在此，根据下标j与k的大小关系来决定继续切分左段还是右段
    if (j > k){
        // 对分区左侧进行快排
        return quickSort(nums, start, j-1, k);
    }else{
        // 对分区右侧进行快排
        return quickSort(nums, j+1, end, k);
    }

}

// 快排切分，返回下标pivotIndex
// 使得比nums[pivotIndex]小的数都在pivotIndex的左边，比nums[pivotIndex]大的数都在pivotIndex的右边。
private static int partition(int[] nums, int begin, int end){
    // 默认数组中待分区区间的最后一个是 pivot 元素
    int	pivot = nums[end];
    // 定义分区后 pivot 元素的下标
    int pivotIndex = begin;
    for(int i=begin; i<end; i++){
        // 判断如果该区间内如果有元素小于 pivot 则将该元素从区间头开始一直向后填充 有点类似选择排序
        if(nums[i] < pivot){
            if(i > pivotIndex){
                // 交换元素
                swap(nums, i, pivotIndex);
            }
            pivotIndex++;
        }
    }
    swap(nums, pivotIndex, end);
    return pivotIndex;
}

// 交换数组内下标为 i j 的两个元素
private static void swap(int[] nums,int i,int j){
    int temp = nums[j];
    nums[j] = nums[i];
    nums[i] = temp;
}

// 方法3：数据范围有限时可以用计数排序：O(N)
public int[] getLeastNumbers(int[] arr, int k) {

    if(k == 0 || arr.length == 0){
        return new int[0];
    }

    // 统计每个数字出现的次数
    // arr[i]的范围是0~10000
    int[] counter = new int[10001];
    for(int num: arr){
        counter[num] ++;
    }

    // 根据counter数组从头找出k个数作为返回结果
    int[] res = new int[k];
    int index = 0;
    for(int i=0; i<counter.length; i++){
        while(counter[i]-- > 0 && index < k){
            res[index++] = i;
        }
        if(index == k){
            break;
        }
    }

    return res;
}
```

#### [剑指 Offer 41. 数据流中的中位数](https://leetcode-cn.com/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/)

```java
class MedianFinder {

    // 建立两个堆，一个小顶堆 一个大顶堆
    private Queue<Integer> minHeap, maxHeap;
    /** initialize your data structure here. */
    public MedianFinder() {
        // 建立两个堆，一个大顶堆，一个小顶堆，只需要平衡两者的大小即可
        // 大顶堆中放入 (n+1)/2 个小元素，栈顶就是第(n+1)/2小的元素
        // 小顶堆中放入 (n+1)/2 个大元素，栈顶就是第(n+1)/2大的元素
        minHeap = new PriorityQueue<>();
        maxHeap = new PriorityQueue<>((a, b) -> b-a);
    }
    
    public void addNum(int num) {

        // 两个堆之间的元素是需要平衡的
        // 保证：大顶堆中的元素个数 >= 小顶堆中的元素个数，且最多相差一个元素
        maxHeap.offer(num);
        minHeap.offer(maxHeap.poll());
        if(maxHeap.size() < minHeap.size()){
            maxHeap.offer(minHeap.poll());
        }
    }
    
    public double findMedian() {
		
        if(maxHeap.size() == minHeap.size()){
            // 偶数情况下的中位数
            return (maxHeap.peek() + minHeap.peek()) / 2.0;
        }else{
            // 计数情况下的中位数
            return maxHeap.peek();
        }
    }
}
```

#### [剑指 Offer 42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)

```java
// 经典动态规划
// 时间复杂度：O(n)
// 空间复杂度：O(n)，是可以降为O(1)的，将dp数组修改为两个变量即可
public int maxSubArray(int[] nums) {
	// 明确状态，dp[i]在包含第nums[i]时的最大情况 
    int[] dp = new int[nums.length + 1];

    // base case
    dp[0] = 0;
    int res = nums[0];
	// 状态转移
    for(int i=1; i<=nums.length; i++){
        // 继承 / 另辟蹊径
        dp[i] = Math.max(nums[i-1], dp[i-1] + nums[i-1]);
        res = Math.max(res, dp[i]);
    }

    return res;
}
```

#### [剑指 Offer 43. 1～n 整数中 1 出现的次数](https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/)

> 这种题目真的很烦人，直接看答案+CV的；
>
> 这题其实也是动态规划类型的题目吧，解答中是有明确的状态转移方程的

```java
// 参考了题解中：xujunyi的答案
public int countDigitOne(int n) {
    return helper(n);
}

private int helper(int n){
    if(n <= 0){
        return 0;
    }

    String s = String.valueOf(n);
    int high = s.charAt(0) - '0';
    int pow = (int) Math.pow(10, s.length() - 1);
    int last = n - high*pow;
    if(high == 1){
        return helper(pow-1) + (last+1) + helper(last);
    }else{
        return pow + high*helper(pow-1) + helper(last);
    }
}
```

#### [剑指 Offer 44. 数字序列中某一位的数字](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/)

> 先空着吧

#### [剑指 Offer 45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

```java
// 这个排序很巧妙，万万没想到还能这么玩
// 时间复杂度：O(NlogN)
// 空间复杂度：O(n), strs需要额外的空间
public String minNumber(int[] nums) {

    String[] strs = new String[nums.length];
    for(int i=0; i<nums.length; i++){
        strs[i] = String.valueOf(nums[i]);
    }
	// 对string组成的字符串进行排序，是从小到大的排序顺序
    // 例如："30" + "3" < "3" + "30"
    // 则，把 “30” 放到 “3" 前面
    Arrays.sort(strs, (o1, o2) -> (o1+o2).compareTo(o2+o1));

    StringBuilder res = new StringBuilder();

    for(String s: strs){
        res.append(s);
    }

    return res.toString();
}
```

#### [剑指 Offer 46. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)

```java
// 动态规划解题
// 时间空间复杂度都为O(n)
public int translateNum(int num) {

    if(num == 0){
        // 特殊处理一下
        return 1;
    }

    // 把num修改为nums数组，便于后续操作
    List<Integer> lists = new ArrayList<>();
    while(num != 0){
        lists.add(num%10);
        num /= 10;
    }
    int[] nums = new int[lists.size()];
    for(int i=0; i<lists.size(); i++){
        nums[i] = lists.get(lists.size()-i-1);
    }

    int n = nums.length;
    // 明确状态: dp[i] 当有i个字符时可能的组合数
    int[] dp = new int[n+1];

    // base case
    dp[0] = 1;
    if(nums[0] >=0 && nums[0] <= 25){
        dp[1] = 1;
    }else{
        return 0;
    }
	
    // 开始状态转移
    for(int i=2; i<=n; i++){
		
        // 当前的数字能否转换成字符
        int one = nums[i-1];
        if(one >=0 && one <= 25){
            dp[i] += dp[i-1];
        }
		
        // 当前的字符和上一个字符能否转换成字符
        int two = nums[i-2] * 10 + nums[i-1]; 
        if(nums[i-2] != 0 && two >=0 && two <= 25){
            dp[i] +=  dp[i-2];
        }
    }

    return dp[n];
}
```

#### [剑指 Offer 47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

```java
// 基础的动态规划题
// 时间/空间复杂度：O(m*n)
public int maxValue(int[][] grid) {

    int m = grid.length;
    int n = grid[0].length;

    // 明确状态:  dp[i][j] 就是在ij位置时的最大礼物价值
    int[][] dp = new int[m][n];

    // base case: 只有一行/只有一列的情况下
    dp[0][0] = grid[0][0];
    for(int i=1; i<m; i++){
        dp[i][0] = dp[i-1][0] + grid[i][0];
    }
    for(int j=1; j<n; j++){
        dp[0][j] = dp[0][j-1] + grid[0][j];
    }

    // 开始状态转移
    for(int i=1; i<m; i++){
        for(int j=1; j<n; j++){
            // 状态转移方程
            dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]) + grid[i][j];
        }
    }

    return dp[m-1][n-1];
}
```

#### [剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

```java
// 滑动窗口解题
// 时间/空间复杂度：O(n)
public int lengthOfLongestSubstring(String s) {
	
    // 定义窗口，窗口中包含了出现过的字符
    Map<Character, Integer> window = new HashMap<>();
    
    int left = 0, right = 0, len = 0;
    while(right < s.length()){
        // c 是将移入窗口的字符
        char c = s.charAt(right);

        // 进行窗口内数据的一系列更新
        if(window.getOrDefault(c, 0) < 1){
            // 没有重复的时候的情况：右移窗口
            right++;
            window.put(c, window.getOrDefault(c, 0)+1);
            // 动态获取最长长度
            if(right - left > len){
                len = right - left;
            }
        }else{
            // 一旦出现重复的时候：开始移左窗口
            char d = s.charAt(left);
            left++;
            window.put(d, window.get(d) - 1);
        }
    }

    return len;
}
```

#### [剑指 Offer 49. 丑数](https://leetcode-cn.com/problems/chou-shu-lcof/)

```java
// 方法1：直接上个小顶堆完事
// 空间复杂度：O(3n)
// 时间复杂度：O(nlogn)
public int nthUglyNumber(int n) {
	
    // 建立一个最小堆，每次都从堆中弹出最小的那个元素 * 2 3 5后入堆
    PriorityQueue<Long> minHeap = new PriorityQueue<>();
    // int有案例过不了
    long res = 1;
    minHeap.offer(res);
    for(int i=0; i<n; i++){

        res = minHeap.poll();

        while(!minHeap.isEmpty() && res == minHeap.peek()){
            // 为了删除重复的元素，如2*3  3*2 就重复了
            minHeap.poll();
        }
        minHeap.offer(res*2);
        minHeap.offer(res*3);
        minHeap.offer(res*5);
    }
    return (int)res;
}

// 方法2：三指针法
// 空间复杂度：O(n)
// 时间复杂度：O(1)
public int nthUglyNumber(int n) {

    int[] reuslt = new int[n];

    reuslt[0] = 1;

    // 定义三个指针
    int p1=0, p2=0, p3=0;
    for(int i=1; i<n; i++){

        reuslt[i] = Math.min(reuslt[p1]*2, Math.min(reuslt[p2]*3, reuslt[p3]*5));

        if(reuslt[i] == reuslt[p1]*2) p1++;
        if(reuslt[i] == reuslt[p2]*3) p2++;
        if(reuslt[i] == reuslt[p3]*5) p3++;
    }

    return reuslt[n-1];
}
```

#### [剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

```java
// 方法1：好家伙我直接就是hash表
// 时间复杂度：O(2n)
// 空间复杂度：O(n)
public char firstUniqChar(String s) {

    Map<Character, Integer> map = new HashMap<>();

    for(char c: s.toCharArray()){
        map.put(c, map.getOrDefault(c, 0) + 1);
    }

    for(char c: s.toCharArray()){
        if(map.get(c) == 1){
            return c;
        }
    }
    return ' ';
}

// 方法2：优化方式1：
// 时间复杂度：O(2n)
// 空间复杂度：O(26)
public char firstUniqChar(String s) {
    
    if (s.equals("")) return ' ';
    
    // 创建‘a'-'z'的字典
    int[] target = new int[26];
    
    // 第一次遍历，将字符统计到字典数组
    for (int i = 0; i < s.length(); i++) {
        target[s.charAt(i) - 'a']++;
    }
    
    // 第二次遍历，从字典数组获取次数
    for (int i = 0; i < s.length(); i++) {
        if (target[s.charAt(i) - 'a'] == 1) return s.charAt(i);
    }

    return ' ';
}

// 优化方式2：
// 时间复杂度：O(2n)
// 空间复杂度：O(26)
public char firstUniqChar(String s) {
    // key:字符  value:标记其是否出现过
    Map<Character, Boolean> dic = new LinkedHashMap<>();
    char[] sc = s.toCharArray();
    for(char c : sc){
        // 这里的逻辑很巧妙，只要出现过2次及以上就是false
        dic.put(c, !dic.containsKey(c)); 
    } 

    for(Map.Entry<Character, Boolean> d : dic.entrySet()){
        if(d.getValue()) return d.getKey();
    }
    return ' ';
}
```

#### [剑指 Offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

> 就很烦

```java
// 回溯：组合问题  虽然超出时间限制，但还是贴一下吧
// 时间复杂度：O(n^2)
// 空间复杂度：O(n^2), 应...该递归的深度和时间复杂度差不多的吧...
int count = 0;
public int reversePairs(int[] nums) {

    if(nums.length <= 1){
        return 0;
    }

    List<Integer> track = new LinkedList<>();
    backtrack(nums, track, 0);
    return count;
}

private void backtrack(int[] nums, List<Integer> track, int index){

    if(track.size() == 2){
        if(track.get(0) > track.get(1)){
            // 满足逆序的条件
            count++;
        }
        return;
    }

    for(int i=index; i<nums.length; i++){
        // 剪枝
        if(index>0 && nums[index-1] < nums[i]){
            continue;
        }
        track.add(nums[i]);
        backtrack(nums, track, i+1);
        track.remove(track.size() - 1);
    }
}


// 正确解法：
// 分治算法:归并排序
// 时间复杂度：O(nlogn), 归并排序的时间复杂度
// 空间复杂度：O(n)
int count = 0;
public int reversePairs(int[] nums) {
    mergeSort(nums);
    return count;
}

private int[] mergeSort(int[] arr){
    if(arr.length < 2){
        return arr;
    }
    // 将数组从中间拆分成左右两部分
    int mid = arr.length/2;
    int[] left = Arrays.copyOfRange(arr, 0, mid);
    int[] right = Arrays.copyOfRange(arr, mid, arr.length);
    return merge(mergeSort(left), mergeSort(right));
}

// 合并两个有序数组并返回新的数组
private int[] merge(int[] left, int[] right){
    // 创建一个新数组,长度为两个有序数组的长度之和
    int[] newArray = new int[left.length+right.length];
    // 定义两个指针,分别代表两个数组的下标
    int lindex = 0;
    int rindex = 0;
    for(int i=0; i<newArray.length;i++){
        // 归并的过程
        if(lindex >= left.length){
            newArray[i] = right[rindex++];
        }else if(rindex >= right.length){
            newArray[i] = left[lindex++];
        }else if(left[lindex] > right[rindex]){
            newArray[i] = right[rindex++];
            // ☆☆☆只有这里有区别☆☆☆
            // 当左边的数组大于时右边的值时，表示左区间lindex及之后的数都将大于rindex指向的数，所以出现了left.length - lindex个逆序对
            count += left.length - lindex;
        }else{
            newArray[i] = left[lindex++];
        }
    }
    return newArray;
}
```

#### [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

```java
// 方法1：暴力解法：hash表 走一波
// 空间时间复杂度都是 O(n)
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {

    Set<ListNode> set = new HashSet<>();

    while(headA != null){
        set.add(headA);
        headA = headA.next;
    }

    while(headB != null){
        if(set.contains(headB)){
            return headB;
        }
        headB = headB.next;
    }

    return null;
}


// 方法2：双指针法，浪漫相遇，看呆了
// 贴一个骚评论：两个结点不断的去对方的轨迹中寻找对方的身影，只要二人有交集，就终会相遇❤
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode h1 = headA, h2 = headB;

    while(h1 != h2){
        h1 = h1 == null ? headB : h1.next;
        h2 = h2 == null ? headA : h2.next;
    }

    return h1;
}
```

#### [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/)

```java
// 方法1：暴力法
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int search(int[] nums, int target) {

    int res = 0;
    for(int i=0; i<nums.length; i++){
        if(nums[i] == target){
            res ++;
        }
    }

    return res;
}

// 方法2：二分法所搜左右边界，主要给了这个数组是排序的这个条件
// 时间复杂度：O(logn)
// 空间复杂度：O(1)
public int search(int[] nums, int target) {

    int left_index = 0, right_index = 0;
    
    // 找左边界
    int left = 0, right = nums.length-1;
    while(left <= right){
        int mid = left + (right-left)/2;
        if(nums[mid] < target){
            left = mid+1;
        }else if(nums[mid] > target){
            right = mid-1;
        }else if(nums[mid] == target){
            // 别返回，锁定左侧边界
            right = mid - 1;
        }
    }
    // 最后要检查 left 越界的情况
    if (left >= nums.length || nums[left] != target)
        return 0;
    left_index = left;

    // 找右侧边界
    left = 0;
    right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定右侧边界
            left = mid + 1;
        }
    }
    // 最后要检查 right 越界的情况
    if (right < 0 || nums[right] != target)
        return 0;
    right_index = right;
	
    // 最后返回长度即可
    return right_index - left_index + 1;
}
```

#### [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/)

```java
// 方法1：暴力法，顺序遍历
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int missingNumber(int[] nums) {

    for(int i=0; i<nums.length; i++){
        if(nums[i] != i){
            return i;
        }
    }

    return nums[nums.length-1] + 1;
}


// 方法2：二分法
// 时间复杂度：O(logn)
// 空间复杂度：O(1)
public int missingNumber(int[] nums) {

    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;

        if(nums[mid] == mid){
            left = mid+1;
        }else{
            right = mid-1;
        }
    }
    return left;
}
```

#### [剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

```java
// 方法1：第k大的节点：动态极值，用固定k大小的最小堆
// 时间复杂度：O(n), 遍历每一个节点
// 空间复杂度：O(k) 没有算递归的消耗
PriorityQueue<Integer> minHeap;
public int kthLargest(TreeNode root, int k) {
 
    if(root == null){
        return 0;
    }
	// 小顶堆k尺寸，找第K大的值
    minHeap = new PriorityQueue<>();
    dfs(root, k);

    return minHeap.peek();
}

private void dfs(TreeNode root, int k){

    if(root == null){
        return;
    }

    if(minHeap.size() < k){
        minHeap.offer(root.val);
    }else if(minHeap.peek()<root.val){
        minHeap.poll();
        minHeap.offer(root.val);
    }

    dfs(root.left, k);
    dfs(root.right, k);
}


// 方法2：利用二叉搜索树的中序遍历是有序的这个性质
// 时间复杂度：O(n), 遍历每一个节点
// 空间复杂度：O(n) 
public int kthLargest(TreeNode root, int k) {

    List<Integer> res = new ArrayList<>();

    dfs(root, res);
 
    return res.get(res.size() - k);
}

private void dfs(TreeNode root, List<Integer> res){
    if(root == null){
        return;
    }

    dfs(root.left, res);
    res.add(root.val);
    dfs(root.right, res);
}

// 方法3：二叉搜索树的中序遍历优化: 逆序遍历 “右中左” 的顺序
// 时间复杂度：O(k)  k <= n(树节点)
// 空间复杂度：O(k) 
int ans = 0, count=0;
public int kthLargest(TreeNode root, int k) {

    dfs(root, k);

    return ans;
}

private void dfs(TreeNode root, int k){
    if(root == null){
        return;
    }

    dfs(root.right, k);
    if(++count == k){
        ans = root.val;
        return;
    }
    dfs(root.left, k);
}
```

#### [剑指 Offer 55 - I. 二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)

```java
// 正常的dfs即可
// 时间复杂度：O(n)  
// 空间复杂度：O(1)  没有算递归的消耗
public int maxDepth(TreeNode root) {

    if(root == null){
        return 0;
    }

    return dfs(root, 1);
}

private int dfs(TreeNode root, int layer){

    if(root == null){
        return layer-1;
    }

    int left_layer = dfs(root.left, layer+1);
    int right_layer = dfs(root.right, layer+1);

    return Math.max(left_layer, right_layer);
}

// 优化写法，其实可以一行代码解决的
public int maxDepth(TreeNode root) {
    return root == null ? 
        0 : Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
}
```

#### [剑指 Offer 55 - II. 平衡二叉树](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)

```java
// 和上一题一样，dfs计算左右子树的深度
// 时间复杂度：O(n)  
// 空间复杂度：O(1)  没有算递归的消耗
boolean res = true;
public boolean isBalanced(TreeNode root) {

    if(root == null){
        return true;
    }

    dfs(root, 0);
    
    return res;
}

private int dfs(TreeNode root, int depth){

    if(!res){
    	// 剪枝
        return -1;
    }

    if(root == null){
        return depth - 1;
    }
	
    int left_depth = dfs(root.left, depth+1);
    int right_depth = dfs(root.right, depth+1);

    if(Math.abs(left_depth - right_depth) > 1){
        res = false;
        return -1;
    }

    return Math.max(left_depth, right_depth);
}
```

#### [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

```java
// 方法1：排序,但是时间复杂度不满足要求
// 时间复杂度：O(nlogn)
// 空间复杂度：O(1)
public int[] singleNumbers(int[] nums) {
 
    Arrays.sort(nums);

    int[] res = new int[2];
    int index = 0;
    if(nums[0] != nums[1]){
        res[index++] = nums[0];
    }
    if(nums[nums.length-1] != nums[nums.length-2]){
        res[index++] = nums[nums.length-1];
    }
    for(int i=1; i<nums.length-1; i++){

        if(index == 2){
            break;
        }
        if(nums[i] != nums[i+1] && nums[i] != nums[i-1]){
            res[index++] = nums[i];
        }
    }

    return res;
}


// 方法2：位运算: 参考了题解中eddieVim的解答
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int[] singleNumbers(int[] nums) {

    // 用于将所有的数异或起来
    int k = 0;

    for(int num: nums) {
        // 相同的数都抵消掉了，只有两个不同的数，最终的为k为1的位置就是不同处
        k ^= num;
    }

    // 获得k中最低位的1，以此为依据将nums分为两组
    int mask = 1;
    while((k & mask) == 0) {
        mask <<= 1;
    }

    int a = 0;
    int b = 0;
    for(int num: nums) {
        // 更具mask分成了两组数据，两个不同数会被分到不同的组中
        // 而相同的数会被分到同一组中，并且在异或后会被抵消的即变为0
        // 又有0异或任何数就是该数本身
        if((num & mask) == 0) {
            // 找第一个数
            a ^= num;
        } else {
            // 找第二个数
            b ^= num;
        }
    }

    return new int[]{a, b};
}
```

#### [剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

```java
// 论位运算的巧妙
// 时间复杂度：O(32n)
// 空间复杂度：O(1)
public int singleNumber(int[] nums) {

    int res = 0;
    for(int i=0; i<32; i++){
        int mask = 1 << i;
        int cnt = 0;
        for(int j=0; j<nums.length; j++){
            if((nums[j] & mask) != 0){
                // 这个位置上的值是1
                cnt++;
            }
        }

        if(cnt % 3 != 0){
            // 一个数字出现3遍，如果这个位置是1，则%3就没了
            // 只有出现1次的数字若是1则能保存下来，若是0那就是0没有区别
            res |= mask;
        }
    }

    return res;
}
```

#### [剑指 Offer 57. 和为s的两个数字](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/)

```java
// 递增数列：双指针完事
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int[] twoSum(int[] nums, int target) {

    if(nums.length < 2){
        return new int[0];
    }

    int left = 0;
    int right = nums.length - 1;

    while(left < right){
        if(nums[left] + nums[right] == target){
            return new int[]{nums[left], nums[right]};
        }else if(nums[left] + nums[right] > target){
            right --;
        }else{
            left ++;
        }
    }
    
    return new int[0];
}
```

#### [剑指 Offer 57 - II. 和为s的连续正数序列](https://leetcode-cn.com/problems/he-wei-sde-lian-xu-zheng-shu-xu-lie-lcof/)

```java
// 暴力前缀和数组：超出时间限制 24 / 32 
// 时间复杂度：O(n^2)
// 空间复杂度：O(n)
public int[][] findContinuousSequence(int target) {
 
    List<List<Integer>> res = new LinkedList<>();

    int len = target/2 + 1;
    // 数组
    int[] nums = new int[len];
    // 数组的前缀和
    int[] preSum = new int[len+1];
    preSum[0] = 0;
    for(int i=0; i<len; i++){
        nums[i] = i+1;
        preSum[i+1] = preSum[i] + nums[i];
    }

    for(int i=0; i<len; i++){
        for(int j=i+1; j<len; j++){
            if(preSum[j+1] - preSum[i] == target){
                List tmp = new ArrayList<>();
                for(int k=i; k<=j; k++){
                    tmp.add(nums[k]);
                }
                res.add(tmp);
            }
        }
    }

    int[][] ans = new int[res.size()][];
    for(int i=0; i<res.size(); i++){
        ans[i] = new int[res.get(i).size()];
        for(int j=0; j<res.get(i).size(); j++){
            ans[i][j] = res.get(i).get(j);
        }
    }
    return ans;
}


// 正确解法：滑动窗口
// 时间复杂度：O(target)
// 空间复杂度：O(1)
public int[][] findContinuousSequence(int target) {

    List<int[]> res = new LinkedList<>();

    int left = 1, right = 1;
    int sum = 0;
    while(right <= target/2+1){
        // 右移窗口
        sum += right;
        right++;

        while(sum > target){
            // 左移窗口
            sum -= left;
            left++;
        }

        if(sum == target){
            // 条件满足，存储结果
            int[] tmp = new int[right - left];
            for(int i=left; i<right; i++){
                tmp[i-left] = i;
            }
            res.add(tmp);
        }
    }

    int[][] ans = new int[res.size()][];
    for(int i=0; i<res.size(); i++){
        if(res.get(i).length <= 1){
            continue;
        }
        ans[i] = res.get(i);
    }

    return ans;
}
```

#### [剑指 Offer 58 - I. 翻转单词顺序](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)

```java
// 直接库函数
public String reverseWords(String s) {
    // 删除首尾空格
    s = s.trim();
    // 直接按照空格分隔字符串
    String[] words = s.split(" ");

    StringBuilder res = new StringBuilder();
    for(int i=words.length-1; i>=0; i--){
        if(words[i].equals("")){
            continue;
        }
        res.append(words[i]+" ");
    }

    return res.toString().trim();
}


// 正常解法：双指针
// 空间复杂度：O(n)
// 时间复杂度：O(n)
public String reverseWords(String s) {
    // 删除首尾空格
    s = s.trim();
    int n = s.length();
    StringBuilder res = new StringBuilder();

    int left = n-1, right = n-1, index = 0;
    while(left >= 0){
        while(left >= 0 && s.charAt(left) != ' ') left--;  // 搜索首个空格
        res.append(s.substring(left+1, right+1)+" ");       // 添加单词
        while(left >= 0 && s.charAt(left) == ' ') left --;  // 跳过单词间空格
        // 更新右指针
        right = left;

    }

    return res.toString().trim();
}
```

#### [剑指 Offer 58 - II. 左旋转字符串](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

```java
// 暴力做法
// 时间复杂度 O(n)
// 空间复杂度 O(N)
public String reverseLeftWords(String s, int n) {

    int len = s.length();
    char[] res = new char[len];

    for(int i=0; i<len; i++){
        if(i<n){
            res[len - n + i] = s.charAt(i); 
        }else{
            res[i-n] = s.charAt(i);
        }
    }

    return new String(res);
}
```

#### [剑指 Offer 59 - I. 滑动窗口的最大值](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

```java
// 方法1：暴力法
// 时间复杂度 O(n*k)
// 空间复杂度 O(1)  除去必要的输出
public int[] maxSlidingWindow(int[] nums, int k) {

    if(nums.length == 0){
        return new int[0];
    }

    int left = 0, right = k-1;
    int res[] = new int[nums.length - k + 1];

    while(right < nums.length){
     
        res[left] = nums[left];
        for(int i=left; i<=right; i++){
            res[left] = Math.max(res[left], nums[i]);
        }

        left++; right++;
    }

    return res;
}

// 方法2：单调队列，还是需要画图理解，看的K神的讲解
// 时间复杂度 O(n)
// 空间复杂度 O(k)  
public int[] maxSlidingWindow(int[] nums, int k) {

    if(nums.length <= 0){
        return new int[0];
    }

    int[] res = new int[nums.length - k + 1];
    // res数组的下标
    int index = 0;  
    // 单调队列
    Deque<Integer> deque = new ArrayDeque<>();
    // 未形成窗口区间
    for(int i=0; i<k; i++){
        // 队列不为空时，当前值与队列尾部值比较，如果大于，删除队列尾部值
        // 一直循环删除到队列中的值都大于当前值，或者删到队列为空
        while(!deque.isEmpty() && nums[i] > deque.peekLast()) deque.pollLast();
        // 执行完上面的循环后，队列中要么为空，要么值都比当前值大，然后就把当前值添加到队列中
        deque.addLast(nums[i]);
    }
    
    // 窗口区间刚形成后，把队列首位值添加到队列中
    // 因为窗口形成后，就需要把队列首位添加到数组中，而下面的循环是直接跳过这一步的，所以需要我们直接添加
    res[index++] = deque.peek();

    // 窗口区间形成
    for(int i=k; i<nums.length; i++){
        // i-k是已经在区间外了，如果首位等于nums[i-k]，那么说明此时首位值已经不再区间内了，需要删除
        if(deque.peek() == nums[i-k]) deque.poll();
        // 删除队列中比当前值小的值
        while(!deque.isEmpty() && nums[i] > deque.peekLast()) deque.pollLast();
        // 把当前值添加到队列中
        deque.addLast(nums[i]);
        // 把队列的首位值添加到arr数组中
        res[index++] = deque.peek();
    }

    return res;
}
```

#### [剑指 Offer 59 - II. 队列的最大值](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/)

```java
// 参考的是题解中 “腐烂的橘子” 的题解
// 函数max_value、push_back 和 pop_front 的均摊时间复杂度都是O(1)
class MaxQueue {

    Queue<Integer> queue;  // 正常队列
    Deque<Integer> deque;  // 存了最大值的队列，是个双端队列
    
    public MaxQueue() {
        queue = new LinkedList<>();
        deque = new LinkedList<>();
    }
    
    public int max_value() {
        if(deque.isEmpty()){
            return -1;
        }else{
            // 双端队列存的是最大值
            return deque.peek();
        }
    }
    
    public void push_back(int value) {
        queue.offer(value);
        
        // 队列是空的时候就直接添加
        if(deque.isEmpty()){
            deque.offer(value);
            return;
        }
		
        // 若列队不为空，则从后比较，是的队列头中存的是最大值
        // 即这个deque中按照递减的顺序存了queue中的部分数据
        while(!deque.isEmpty() && value > deque.peekLast()){
            deque.pollLast();
        }
        deque.offer(value);
    }
    
    public int pop_front() {
        
        if(queue.isEmpty()){
            return -1;
        }

        Integer res = queue.poll();
	
        // 当队列中的值和deque中的相同时，deque也要出队
        if(res.equals(deque.peek())){
            deque.poll();
        }

        return res;
    }
}
```

#### [剑指 Offer 60. n个骰子的点数](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/)

```java
// 动态规划解题
public double[] dicesProbability(int n) {

    // 明确状态：dp[i][j] 当前有i个骰子时，出现j点数的次数
    int[][] dp = new int[n+1][6*n+1];

    // base case：只有一个骰子的情况，每种可能都是1次
    for(int j=1; j<=6; j++){
        dp[1][j] = 1;
    }

    // 开始状态转移
    for(int i=2; i<=n; i++){          // 骰子个数的状态转移
        for(int j=i; j<=6*i; j++){    // 出现点数j的状态
            for(int k=1; k<=6; k++){  // 然后遍历这个骰子出点的点数可能
                if(j-k<=0){
                    // 不存在的情况，提前结束
                    break;
                }
                dp[i][j] += dp[i-1][j-k];
            }
        }
    }
	
    // 由于结果需要的是输出的概率值，进行转换
    double[] res = new double[6*n -n + 1];
    double all = Math.pow(6, n);
    for(int i=n; i<=6*n; i++){
        res[i-n] = dp[n][i] / all;
    }

    return res;
}
```

#### [剑指 Offer 61. 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

```java
// 复杂度分析
// 时间：O(nlogn + n)
// 空间：O(1)
public boolean isStraight(int[] nums) {

    // 排序一下
    Arrays.sort(nums);

    // 记录一下0的个数，0可能是有好几个的 TMD
    int count = 0;
    for(int i=0; i<nums.length; i++){
        if(nums[i] == 0){
            count++;
        }else{
            break;
        }
    }

    // 记录一下不连续的情况
    int fix = 0;
    for(int i=count+1; i<nums.length; i++){
        if(nums[i] - nums[i-1] != 1){
            // 计算不连续度
            fix += nums[i] - nums[i-1] - 1;
        }
        if(nums[i] == nums[i-1] || fix > count){
            // 重复牌的出 || 已经修复不了了
            return false;
        }
    }

    return true;
}
```

#### [剑指 Offer 62. 圆圈中最后剩下的数字](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)

```java
// 暴力法:模拟这个过程
// 时间复杂度：O(n)
// 空间复杂度：O(n)
public int lastRemaining(int n, int m) {

    List<Integer> nums = new ArrayList<>();
    for(int i=0; i<n; i++){
        nums.add(i);
    }

    int index = 0;
    int res = 0;
    while(n > 0){
        // 关键逻辑
        index = (index + m - 1) % n;
        res = nums.remove(index);
        n--;
    }

    return res;
}
```

#### [剑指 Offer 63. 股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)

```java
// 方法1：暴力法 超出时间限制 201/210
// 时间复杂度：O(n^2)
// 空间复杂度：O(1)
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

// 方法2：优化后减去一个循环
// 时间复杂度：O(n)
// 空间复杂度：O(1)
public int maxProfit(int[] prices) {

    int res = 0;
    // 固定买出时间
    int cur_min = prices[0];  
    for(int j=1; j<prices.length; j++){
        // 寻找最低买入价格的时间，遍历一遍肯定能找到最小值
        cur_min = Math.min(cur_min, prices[j]);
        // 计算最佳时机
        res = Math.max(res, prices[j] - cur_min);
    }
    return res;
}

// 方法3：动态规划
// 时间复杂度：O(n)
// 空间复杂度：O(2n)
public int maxProfit(int[] prices) {

    int n = prices.length;:
    
    // 定义dp数组表示第i天的最大收入
    // 有2个状态，i：天数  j:持有股票的状态 0-没持有  1-持有
    int[][] dp = new int[n][2];

    // base case
    dp[0][0] = 0;
    dp[0][1] = -prices[0];  // 第一天持有，结果当然是负的

    // 开始状态转移
    for(int i=1; i<n; i++){
        // 第i天没持有，可以由 上一天没持有/上一天持有，这一天卖了 转移而来
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1]+prices[i]);
        // 第i天持有，可以由 上一天没持有 这一天买入/上一天持有 转移而来
        dp[i][1] = Math.max(-prices[i], dp[i-1][1]);
    }

    // 最终输出：第n天，没持有股票
    return dp[n-1][0];
}
```

#### [剑指 Offer 64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

```java
// 不能用for：递归解决
// 不能用if判断递归结束条件，用 && 的短路特性实现
public int sumNums(int n) {

    boolean bool = n > 1 && (n += sumNums(n-1)) > 0;

    return n; 
}
```

#### [剑指 Offer 65. 不用加减乘除做加法](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

> 还没做

#### [剑指 Offer 66. 构建乘积数组](https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/)

```java
// 妙蛙种子吃着妙脆角走进了米奇妙妙屋
// 时间复杂度：O(2n)  
// 空间复杂度：O(1)  除去了必要的输出
public int[] constructArr(int[] a) {
    int n = a.length;
    int[] res = new int[n];
    for (int i = 0, cur = 1; i < a.length; i++) {
        res[i] = cur;   // 先乘左边的数(不包括自己)
        cur *= a[i];
    }
    for (int i = a.length - 1, cur = 1; i >= 0; i--) {
        res[i] *= cur;  // 再乘右边的数(不包括自己)
        cur *= a[i];
    }
    return res;
}
```

#### [剑指 Offer 67. 把字符串转换成整数](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

```java
public int strToInt(String str) {

    // 删除空格，转换成字符数组
    char[] c = str.trim().toCharArray();
    if(c.length == 0) return 0; 
    // 大数边界，这个边界用的很巧妙
    int res = 0, bndry = Integer.MAX_VALUE / 10;
    // 符号位的提取
    int i = 1, sign = 1;
    if(c[0] == '-') sign = -1;
    else if(c[0] != '+') i = 0;
    // 开始遍历
    for(int j = i; j < c.length; j++) {
        // 不是数字了 break
        if(c[j] < '0' || c[j] > '9') break;

        // 关键步骤：防止溢出
        // int的最大值为 -2147483648 ~ 2147483647
        // 大于边界了，再*10必然溢出 || 刚刚等于边界，但是下一个数大于7加上后也溢出
        if(res > bndry || res == bndry && c[j] > '7'){
            return  sign == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;
        }
        res = res * 10 + (c[j] - '0');
    }

    return sign*res;
}
```

#### [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

```java
// 方法1:一般dfs，没有利用二叉搜索树的性质
// 时间复杂度：O(n)
// 空间复杂度：O(n)  加上了递归的消耗
TreeNode res = null;
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    dfs(root, p, q);
    return res;
}

private boolean dfs(TreeNode root, TreeNode p, TreeNode q){

    if(res != null){
        return false;
    }

    boolean left = false, right = false;
    if(root.left != null){
        left = dfs(root.left, p, q);
    }

    if(root.right != null){
        right = dfs(root.right, p, q);
    }

    if(left && right){
        // 左边右边都满足val相等，则root则是这个公共祖先
        res = root;
    }else if((left || right) && (root.val == p.val || root.val == q.val)){
        // 左边或右边满足val相等，且现在的root满足等于p/q，则现在的root就是祖先
        res = root;
    }

    return root.val == p.val || root.val == q.val || left || right;
}


// 方法2：利用二叉搜索树的性质的解法：迭代
// 时间复杂度：O(logn)
// 空间复杂度：O(1) 
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {

    while(root != null){
        if(root.val < p.val && root.val < q.val){
            // p,q 都在 root 的右子树中
            root = root.right;
        }else if(root.val > p.val && root.val > q.val){ 
            // p,q 都在 root 的左子树中
            root = root.left;
        }else{
            // 此时有root的值刚好介于 p q 之间
            break;
        }
    }

    return root;
}
```

#### [剑指 Offer 68 - II. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

```java
// 一般解法：做了68-I再做的这题...直接用上述的一般解法解的
TreeNode res = null;
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {

    dfs(root, p, q);
    return res;
}

private boolean dfs(TreeNode root, TreeNode p, TreeNode q){

    if(res != null){
        return false;
    }

    boolean left = false, right = false;
    if(root.left != null){
        left = dfs(root.left, p, q);
    }

    if(root.right != null){
        right = dfs(root.right, p, q);
    }

    if(left && right){
        res = root;
    }else if((left || right) && (root.val == p.val || root.val == q.val)){
        res = root;
    }

    return root.val == p.val || root.val == q.val || left || right;
}
```

