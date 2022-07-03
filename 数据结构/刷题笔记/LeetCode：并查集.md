# LeetCode：并查集

> 这部分主要是学习了 **labuladong** 公众号中对于并查集的讲解，文章链接如下：
>
> [Union-Find 并查集算法详解](https://mp.weixin.qq.com/s?__biz=MzAxODQxMDM0Mw==&mid=2247484751&idx=1&sn=a873c1f51d601bac17f5078c408cc3f6&chksm=9bd7fb47aca07251dd9146e745b4cc5cfdbc527abe93767691732dfba166dfc02fbb7237ddbf&scene=21#wechat_redirect)
>
> [Union-Find 算法怎么应用？](https://mp.weixin.qq.com/s?__biz=MzAxODQxMDM0Mw==&mid=2247484759&idx=1&sn=a88337164c741b9740e50523b41b7659&chksm=9bd7fb5faca07249c15e925e596e8ab071731f0c996b1ba3e58a1b45052900a23278114f2720&scene=21#wechat_redirect)

## 概述

并查集用于解决图论中「动态连通性」问题；

主要实现以下几个API

```java
class UF {
    /* 将 p 和 q 连接 */
    public void union(int p, int q);
    /* 判断 p 和 q 是否连通 */
    public boolean connected(int p, int q);
    /* 返回图中有多少个连通分量 */
    public int count();
}
```

关于上述提到的连通性，其具有以下三个特性：

1. **自反性**：节点`p`和`p`是连通的；

2. **对称性**：如果节点`p`和`q`连通，那么`q`和`p`也连通。

3. **传递性**：如果节点`p`和`q`连通，`q`和`r`连通，那么`p`和`r`也连通。

## 案例代码

```java
class UF {
    // 连通分量个数
    private int count;
    // 存储一棵树，用于记录节点x的父节点
    private int[] parent;
    // 记录树的“重量”，出于平衡的目的
    private int[] size;
	
    // 构造函数，n为图的节点总数
    public UF(int n) {
        // 初始状态：节点之间互不连通
        this.count = n;
        // 父节点指向自身
        parent = new int[n];
        // 重量都为1
        size = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }
	
    // 连通p节点与q节点
    public void union(int p, int q) {
        int rootP = find(p);  // 找到p节点的父节点
        int rootQ = find(q);  // 找到q节点的父节点
        if (rootP == rootQ)
            return;

        // 小树接到大树下面，较平衡
        if (size[rootP] > size[rootQ]) {
            // 让小树rootQ指向大树rootP
            parent[rootQ] = rootP;
            // 大树的尺寸要加上小树的尺寸
            size[rootP] += size[rootQ];
        } else {
            parent[rootP] = rootQ;
            size[rootQ] += size[rootP];
        }
        // 连通量-1
        count--;
    }
	
    // 判断p节点与q节点是否相连
    public boolean connected(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        return rootP == rootQ;
    }
    
    // 获取此时连通量的个数
    public int getCount(){
        return this.count;
    }
	
    // 找x节点的父节点：带路径压缩的
    private int find(int x) {
        while (parent[x] != x) {
            // 进行路径压缩，使得整体的复杂度都降至O(1)
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }
    
    // // 找x节点的父节点：不带路径压缩的
    // private int find(int x) {
    //     // 根节点的 parent[x] == x
    //     while (parent[x] != x){
    //     	x = parent[x];
    //     }
    //     return x;
    // }
}
```

## 题目

### [130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)

> 更好的解法是通过dfs进行求解
>
> 但这里也可以用并查集的解法去求解，虽然代码很长，效率不高，但是不失为一种解法

```java
class Solution {

    // 并查集解题
    public void solve(char[][] board) {

        int m = board.length;
        int n = board[0].length;

        // 给 dummy 留一个额外位置
        UF uf = new UF(m*n + 1);
        int dummy = m*n;

        // 将首列和末列的 O 与 dummy 连通
        for(int i=0; i<m; i++){
            if(board[i][0] == 'O'){
                uf.union(i*n, dummy);
            }
            if(board[i][n-1] == 'O'){
                uf.union(i*n + n-1, dummy);
            }
        }

        // 将首行和末行的 O 与 dummy 连通
        for(int j=0; j<n; j++){
            if(board[0][j] == 'O'){
                uf.union(j, dummy);
            }
            if(board[m-1][j] == 'O'){
                uf.union((m-1)*n + j, dummy);
            }
        }

        // 方向数组d是上下左右搜索的常用手法
        int[][] d = new int[][]{{1,0}, {0,1}, {0,-1}, {-1,0}};
        for(int i=1; i<m-1; i++){
            for(int j=1; j<n-1; j++){
                if(board[i][j] == 'O'){
                    // 上
                    if(board[i-1][j] == 'O') uf.union((i-1)*n + j , i*n + j);
                    // 下
                    if(board[i+1][j] == 'O') uf.union((i+1)*n + j , i*n + j);
                    // 左
                    if(board[i][j-1] == 'O') uf.union(i*n + (j-1) , i*n + j);
                    // 右
                    if(board[i][j+1] == 'O') uf.union(i*n + (j+1) , i*n + j);
                }
            }
        }

        // 所有不和 dummy 连通的 O，都要被替换
        for(int i=1; i<m-1; i++){
            for(int j=1; j<n-1; j++){
                if(!uf.connected(i*n + j, dummy)){
                    board[i][j] = 'X';
                }
            }
        }
    }


    class UF{
		// ... 见上方案例代码
    }
}
```

### [990. 等式方程的可满足性](https://leetcode-cn.com/problems/satisfiability-of-equality-equations/)

```java
boolean equationsPossible(String[] equations) {
    // 26 个英文字母
    UF uf = new UF(26);
    // 先让相等的字母形成连通分量
    for (String eq : equations) {
        if (eq.charAt(1) == '=') {
            char x = eq.charAt(0);
            char y = eq.charAt(3);
            uf.union(x - 'a', y - 'a');
        }
    }
    // 检查不等关系是否打破相等关系的连通性
    for (String eq : equations) {
        if (eq.charAt(1) == '!') {
            char x = eq.charAt(0);
            char y = eq.charAt(3);
            // 如果相等关系成立，就是逻辑冲突
            if (uf.connected(x - 'a', y - 'a'))
                return false;
        }
    }
    return true;
}
```

### [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

```java
public int numIslands(char[][] grid) {

    int m = grid.length;
    int n = grid[0].length;

    UF uf = new UF(m*n + 1);
    int water = m*n;

    for(int i=0; i<m; i++){
        for(int j=0; j<n; j++){
            if(grid[i][j] == '0'){
                uf.union(i*n + j, water);
            }else{
                // 上下左右
                if(i-1 >=0 && grid[i-1][j] == '1'){
                    uf.union(i*n+j, (i-1)*n+j);
                }
                if(i+1 < m && grid[i+1][j] == '1'){
                    uf.union(i*n+j, (i+1)*n+j);
                }
                if(j-1 >=0 && grid[i][j-1] == '1'){
                    uf.union(i*n+j, i*n+j-1);
                }
                if(j+1 < n && grid[i][j+1] == '1'){
                    uf.union(i*n+j, i*n+j+1);
                }
            }
        }
    }

    return uf.getCount() - 1;

}

class UF {
    // ... 见上方案例代码
}
```

### [547. 省份数量](https://leetcode-cn.com/problems/number-of-provinces/)

```java
// 并查集开整
public int findCircleNum(int[][] isConnected) {

    int m = isConnected.length;

    UF uf = new UF(m);

    for(int i=0; i<m; i++){
        for(int j=0; j<i; j++){
            if(isConnected[i][j] == 1)
                uf.union(i, j);
        }
    }
    return uf.getCount();
}

class UF {
    // ... 见上方案例代码
}
```

### [684. 冗余连接](https://leetcode-cn.com/problems/redundant-connection/)

```java
public int[] findRedundantConnection(int[][] edges) {

    UF uf = new UF(edges.length+1);

    int[] res = new int[2];
    for(int i=0; i<edges.length; i++){
		
        if(!uf.connected(edges[i][0], edges[i][1])){
            // 当两个节点还没有相连时，则让其相连
            uf.union(edges[i][0], edges[i][1]);
        }else{
            // 当存在新的连接已经有共同的父节点了，说明这个连接会使树成环
            res[0] = edges[i][0];
            res[1] = edges[i][1];
        }
    }

    return res;
}

class UF {
    // ... 见上方案例代码
}
```

### [1319. 连通网络的操作次数](https://leetcode-cn.com/problems/number-of-operations-to-make-network-connected/)

```java
public int makeConnected(int n, int[][] connections) {

    UF uf = new UF(n);

    int times = 0;
    for(int i=0; i<connections.length; i++){
        if(!uf.connected(connections[i][0], connections[i][1])){
            // 没连接，则连接
            uf.union(connections[i][0], connections[i][1]);
        }else{
            // 已经连接了，则不需要再连接，这次连接的线后续可以用
            times ++;
        }
    }
	// 还需要连接的计算机群
    int need_link = uf.getCount() - 1;
	// 判断省下的线是否能满足连接需求来返回相应的结果
    return need_link <= times ? need_link : -1;
}

class UF {
    // ... 见上方案例代码
}
```

### [778. 水位上升的泳池中游泳](https://leetcode-cn.com/problems/swim-in-rising-water/)

```java
// 自己做的：一顿操作猛如虎，真TMD击败5%
public int swimInWater(int[][] grid) {

    int n = grid.length;
    UF uf = new UF(n*n);

    // 由于每个格子的值都不重复，则记录一下起点与终点
    int s = grid[0][0];
    int e = grid[n-1][n-1];
    // 计算初始水位
    int t = Math.max(s, e);

    // 遍历水位，合并可以联通的区域
    for(int i=t; i<n*n; i++){
        for(int r=0; r<n; r++){
            for(int c=0; c<n; c++){
                if(grid[r][c] <= i){
                    // 上下左右
                    if(r-1 >=0 && grid[r-1][c] <= i)
                        uf.union(grid[r][c], grid[r-1][c]);
                    if(r+1 < n && grid[r+1][c] <= i)
                        uf.union(grid[r][c], grid[r+1][c]);
                    if(c-1 >=0 && grid[r][c-1] <= i)
                        uf.union(grid[r][c], grid[r][c-1]);
                    if(c+1 < n && grid[r][c+1] <= i)
                        uf.union(grid[r][c], grid[r][c+1]);
                    // 其他四个方向... 这里为什么不需要不是说可以从游到相邻任意的嘛
                    // if(r-1 >=0 && c-1 >=0 && grid[r-1][c-1] <= i)
                    //     uf.union(grid[r][c], grid[r-1][c-1]);
                    // if(r-1 >=0 && c+1 < n && grid[r-1][c+1] <= i)
                    //     uf.union(grid[r][c], grid[r-1][c+1]);
                    // if(r+1 <n && c-1 >=0 && grid[r+1][c-1] <= i)
                    //     uf.union(grid[r][c], grid[r+1][c-1]);
                    // if(r+1 <n && c+1 < n && grid[r+1][c+1] <= i)
                    //     uf.union(grid[r][c], grid[r+1][c+1]);
                }

                if(uf.connected(s, e)){
                    return i;
                }
            }
        }
    }

    return -1;
}

// 看了题解之后的优化思路：从0开始找到对应的位置填充
public int swimInWater(int[][] grid) {

    int n = grid.length;
    UF uf = new UF(n*n);

    // 由于每个格子的值都不重复，则记录一下起点与终点
    int s = grid[0][0];
    int e = grid[n-1][n-1];

    // 记录一下初始值，下标：对应grad[i][j]，值：对应grad[i][j]的索引
    int[] idxs = new int[n*n];
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            idxs[grid[i][j]] = i*n + j;
        }
    }

    // 开始合并
    for(int i=0; i<n*n; i++){
        int r = idxs[i] / n;
        int c = idxs[i] % n;

        // 上下左右
        if(r-1 >=0 && grid[r-1][c] <= i)
            uf.union(grid[r][c], grid[r-1][c]);
        if(r+1 < n && grid[r+1][c] <= i)
            uf.union(grid[r][c], grid[r+1][c]);
        if(c-1 >=0 && grid[r][c-1] <= i)
            uf.union(grid[r][c], grid[r][c-1]);
        if(c+1 < n && grid[r][c+1] <= i)
            uf.union(grid[r][c], grid[r][c+1]);


        if(uf.connected(s, e)){
            return i;
        }
    }

    return -1;
}
```

### [721. 账户合并](https://leetcode-cn.com/problems/accounts-merge/)

```java
// 这题完全被建立缓存给搞崩了
public List<List<String>> accountsMerge(List<List<String>> accounts) {

    // 简历n个用户的并查集
    int n = accounts.size();
    UF uf = new UF(n);

    // 搞个缓存，来缓存所有的邮箱，即 email->id
    Map<String, Integer> emailToId = new HashMap<>();
    for(int i=0; i<n; i++){
        int size = accounts.get(i).size();
        for(int j=1; j<size; j++){
            emailToId.put(accounts.get(i).get(j), i);
        }
    }

    // 开始合并
    for(int i=0; i<n; i++){
        // 账户中邮箱的数量
        int size = accounts.get(i).size();
        // 取出每一个邮箱
        for(int j=1; j<size; j++){
            String account = accounts.get(i).get(j);
            if(emailToId.containsKey(account)){
                uf.union(emailToId.get(account), i);
            }
        }
    }

    // 再搞个缓存 id 对应 email
    Map<Integer, List<String>> idToEamil = new HashMap<>();
    for(Map.Entry<String, Integer> entry: emailToId.entrySet()){
        // 通过并查集使得相同的账户都对应同一个id
        int id = uf.find(entry.getValue());
        List<String> emails = idToEamil.getOrDefault(id, new ArrayList<>());
        // 添加账户
        emails.add(entry.getKey());
        idToEamil.put(id, emails);
    }

    // 通过上面的步骤，现在的id和email都已经对应了，且id是唯一的
    List<List<String>> res = new ArrayList<>();
    for(Map.Entry<Integer, List<String>> entry: idToEamil.entrySet()){
        // 排序
        List<String> emails = entry.getValue();
        Collections.sort(emails);
        List<String> temp = new ArrayList<>();
        // 添加用户名
        temp.add(accounts.get(entry.getKey()).get(0));
        temp.addAll(emails);
        res.add(temp);
    }

    return res;
}
```

### [399. 除法求值](https://leetcode-cn.com/problems/evaluate-division/)

> u1s1 这题真有点难...

```java
public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {

    int equationsSize = equations.size();
    // 缓存的建立，key：'除数/被除数'  value：id
    Map<String, Integer> cache = new HashMap<>();

    // 将除数/被除数 添加到缓存中与 id 一一对应
    int idx = 0;
    for(List<String> equation: equations){
        String str1 = equation.get(0);
        String str2 = equation.get(1);
        if(!cache.containsKey(str1)){
            cache.put(str1, idx++);
        }
        if(!cache.containsKey(str2)){
            cache.put(str2, idx++);
        }
    }

    // 建立并查集，其中的节点个数，就是不同的除数被除数的个数
    UF uf = new UF(idx--);
    idx = 0;
    // 再次遍历等式关系，进行union操作
    for(List<String> equation: equations){
        String str1 = equation.get(0);
        String str2 = equation.get(1);
        double value = values[idx++];
        int p = cache.get(str1);
        int q = cache.get(str2);
        uf.union(p, q, value);
    }

    // 根据建立的并查集来进行值的判断操作
    double[] res = new double[queries.size()];
    idx = 0;
    for(List<String> query: queries){
        String str1 = query.get(0);
        String str2 = query.get(1);
        if(!cache.containsKey(str1) || !cache.containsKey(str2)){
            res[idx++] = -1.0;
        }else{
            int p = cache.get(str1);
            int q = cache.get(str2);
            res[idx++] = uf.connected(p, q);
        }
    }

    return res;
}


public static void main(String[] args) {

    List<List<String>> equations = new ArrayList<>();
    double[] values = new double[]{3.4, 1.4, 2.3};
    List<List<String>> queries = new ArrayList<>();

    equations.add(Arrays.asList("a", "b"));
    equations.add(Arrays.asList("e", "f"));
    equations.add(Arrays.asList("b", "e"));
    queries.add(Arrays.asList("b", "a"));
    queries.add(Arrays.asList("a", "f"));
    queries.add(Arrays.asList("f", "f"));
    queries.add(Arrays.asList("f", "e"));

    double[] doubles = new Solution().calcEquation(equations, values, queries);
    System.out.println(Arrays.toString(doubles));
}

class UF {
    // 连通分量个数
    private int count;
    // 存储一棵树，用于记录节点x的父节点
    private int[] parent;
    // 原先的size，这里需要修改为权重信息
    private double[] weights;

    // 构造函数，n为图的节点总数
    public UF(int n) {
        // 初始状态：节点之间互不连通
        this.count = n;
        // 父节点指向自身
        parent = new int[n];
        // 重量都为1
        weights = new double[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            // 初始的权重都是1.0
            weights[i] = 1.0;
        }
    }

    // 连通p节点与q节点
    public void union(int p, int q, double weight) {
        int rootP = find(p);  // 找到p节点的父节点
        int rootQ = find(q);  // 找到q节点的父节点
        if (rootP == rootQ)
            return;

        parent[rootQ] = rootP;  // 这里必须是q指向p，即除数指向了被除数
        // 权重的更新
        weights[rootQ] = weight * weights[p] / weights[q];
        // 连通量-1
        count--;
    }

    // 判断p节点与q节点是否相连
    public double connected(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if(rootP == rootQ){
            return weights[q] / weights[p];
        }else{
            return -1.0;
        }
    }

    // 获取此时连通量的个数
    public int getCount(){
        return this.count;
    }

    // 找x节点的父节点：同样具有路径压缩的！！！
    private int find(int x) {
        // 根节点的 parent[x] == x
        if (parent[x] != x){
            int origin = parent[x];  // x节点当前的父节点，用于权重的更新
            parent[x] = find(parent[x]);  // x节点直接指向根节点
            weights[x] = weights[x] * weights[origin];  // 权重更新
        }
        // 返回根节点
        return parent[x];
    }
}
```

