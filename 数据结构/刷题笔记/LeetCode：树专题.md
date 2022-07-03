# LeetCode：树专题

> 参考了**力扣加加**对与树专题的讲解，刷了些 leetcode 题，在此做一些记录，不然没几天就没印象了
>
> [力扣加加-树专题](https://leetcode-solution-leetcode-pp.gitbook.io/leetcode-solution/thinkings/tree)

## 总结

### 树的定义

```java
// Definition for a binary tree node.
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

// Definition for a binary tree node.
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
```

### 一个中心：树的遍历

**按照主逻辑处理的顺序：先序遍历，后续遍历**

**树的遍历主要分为两个类型**：

1. 深度优先遍历 dfs
2. 广度优先遍历 bfs

**树的迭代遍历写法：从递归写法 转换为 迭代写法**

双色标记法，核心思想如下：

* 使用颜色标记节点的状态，新节点为白色，已访问的节点为灰色。
* 如果遇到的节点为白色，则将其标记为灰色，然后将其右子节点、自身、左子节点依次入栈。
* 如果遇到的节点为灰色，则将节点的值输出。

```java
public List<Integer> xxxorderTraversal(TreeNode root){

    List<Integer> result = new LinkedList<>();
    Stack<TreeNode> stack = new Stack<>();
    // 标记作用：参数Integer：1-已经被遍历了  0-还没有被遍历  ---> 双色
    Map<TreeNode, Integer> map = new HashMap<>();  
	// 根节点入栈
    stack.push(root);
    map.put(root, 0);  

    while(!stack.isEmpty()){
        TreeNode node = stack.pop();
        if (node == null){
            continue;
        }
		
        if (map.get(node) == 0){
    	    // 这个节点还没有被遍历过
            // 将其标记为灰色，然后将其右子节点、自身、左子节点依次入栈
            
            // ----前序遍历 144 题---- //
            stack.push(node.right);
            stack.push(node.left);
            stack.push(node);
            
            // ---- 中序遍历 94 题---- //
            // stack.push(node.right);
            // stack.push(node);
            // stack.push(node.left);
            
            // ----后续遍历 145 题---- //
            // stack.push(node);
            // stack.push(node.right);
            // stack.push(node.left);
            
            map.put(node, 1);
            map.put(node.right, 0);
            map.put(node.left, 0);
        }else{
            // 这个已经被遍历过了，则加入result结果中
            // 即：如果遇到的节点为灰色，则将节点的值输出
            result.add(node.val);
        }
    }
    return result;
}
```

> 关于二叉树的遍历：前后中
>
> * [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)
> * [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)
> * [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)
>
> 层序遍历：
>
> * [102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)
> * [117. 填充每个节点的下一个右侧节点指针 II](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii/)

### 两个基本点

#### 深度优先遍历 Depth-First-Search，DFS：

> 摘录算法流程：
>
> 1. 首先将根节点放入**stack** 中。
> 2. 从 **stack** 中取出第一个节点，并检验它是否为目标。如果找到所有的节点，则结束搜寻并回传结果。否则将它某一个尚未检验过的直接子节点加入**stack**中。
> 3. 重复步骤 2。
> 4. 如果不存在未检测过的直接子节点。将上一级节点加入**stack**中，重复步骤 2。
> 5. 重复步骤 4。
> 6. 若**stack**为空，表示整张图都检查过了——亦即图中没有欲搜寻的目标。结束搜寻并回传“找不到目标”。
>
> 这里的 stack 可以理解为自己实现的栈，也可以理解为调用栈。**如果是调用栈的时候就是递归，如果是自己实现的栈的话就是迭代。**

一个典型的通用的 DFS 模板可能是这样的：

```java
boolean[] visited;
void dfs(int i) {
    if (满足特定条件）{
        // 返回结果 or 退出搜索空间
    }

    visited[i] = true // 将当前状态标为已搜索
    for (根据i能到达的下个状态j) {
        if (!visited[j]) { // 如果状态j没有被搜索过
            dfs(j)
        }
    }
}
```

但是树不存在环，因此不需要visited，因此一个树的 DFS 更多是

```java
void dfs(TreeNode root) {
    if (满足特定条件）{
        // 返回结果 or 退出搜索空间
    }
    for (TreeNode child: root.children) {
        dfs(child)
    }
}
```

**算法模版：而几乎所有的题目几乎都是二叉树，因此下面这个模板更常见。**

```java
void dfs(TreeNode root) {
    if (满足特定条件）{
        // 返回结果 or 退出搜索空间
    }
    // 主要处理逻辑在此：前序遍历
    dfs(root.left);
    dfs(root.right);
   // 主要处理逻辑在此：后续遍历
}
```

#### 广度优先遍历 Breadth-First-Search，BFS：

> 文中将了：BFS不同与层序遍历，**BFS 的核心在于求最短问题时候可以提前终止，这才是它的核心价值，层次遍历是一种不需要提前终止的 BFS 的副产物**。
>
> **算法流程**
>
> 1. 首先将根节点放入队列中。
> 2. 从队列中取出第一个节点，并检验它是否为目标。
>    - 如果找到目标，则结束搜索并回传结果。
>    - 否则将它所有尚未检验过的直接子节点加入队列中。
> 3. 若队列为空，表示整张图都检查过了——亦即图中没有欲搜索的目标。结束搜索并回传“找不到目标”。
> 4. 重复步骤 2。

**算法模版：**

```java
void bfs(TreeNode root){
    // 使用双端队列，而不是数组
    Queue<TreeNode> queue = new ArrayDeque<>();
    // 注意：ArrayDeque不允许null值，LinkedList允许null值
    // Queue<TreeNode> queue = new LinkedList<>();
    // 记录层数
    int steps = 0;
    
    queue.offer(root);
    while(!queue.isEmpty()){
        // 当前层的节点数
        int size = queue.size();
        // 遍历当前层的所有节点数
        for (int i=0; i<size; i++){
            TreeNode node = queue.poll();
            result.add(node);
            // 判断节点是否满足，而决定是否返回等操作
            if(node.left != null){
                queue.offer(node.left);
            }
            if(node.right != null){
                queue.offer(node.right);
            }
        }
        steps += 1; // 遍历完一层，层数+1
    }
    return;
}
```

> 典型题目：[513. 找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value/)

### 三种题型

#### 搜索类

通过 BFS / DFS 进行求解

**使用DFS求解的模版**：[113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

```java
void dfs(TreeNode root, List<TreeNode> path){
    if (root == null){
        return;
    }
    
    if(root.left == null && root.right == null){
        // 叶子节点，做相应处理
        return;
    }
    
    path.add(root);
    // 主逻辑在此，则为前序遍历
    dfs(root.left, path);
    dfs(root.right, path);
    // 主逻辑在此，则为后序遍历
    path.remove(root);
}
```

**使用BFS求解的模版**：见上方

#### 构建类

**普通二叉树的构建：**

1. 根据两种 DFS 的遍历结果，构建出原始的树结构
   * [105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
   * [106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)
   * [889. 根据前序和后序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)
2. 根据一个 BFS 的遍历结果，构建出原始树结构

   * [297. 二叉树的序列化与反序列化](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/)

   * [654. 最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)

   * [894. 所有可能的满二叉树](https://leetcode-cn.com/problems/all-possible-full-binary-trees/)

**二叉搜索树的构建：**

根据二叉搜索树的性质，有可能根据**一种遍历序列**构造出来，如：[1008. 前序遍历构造二叉搜索树](https://leetcode-cn.com/problems/construct-binary-search-tree-from-preorder-traversal/)

#### 修改类

**题目要求的修改**：

* 修改指针：[116. 填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

* 添加/删除节点：[450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)，[669. 修剪二叉搜索树](https://leetcode-cn.com/problems/trim-a-binary-search-tree/)

**算法需要，自己修改**

* **这题很经典**：[863. 二叉树中所有距离为 K 的结点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)

  > 自己添加一个节点：
  >
  > ```java
  > class Solution {
  >        Map<TreeNode, TreeNode> parent;
  >        public void dfs(TreeNode node, TreeNode parent) {
  >            if (node != null) {
  >                parent.put(node, parent);
  >                dfs(node.left, node);
  >                dfs(node.right, node);
  >            }
  >        }
  > }
  > ```
  >

### 四个重要概念

#### 二叉搜索树

* 天生适合查找

* 中序遍历是有序的：[98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)，[99. 恢复二叉搜索树](https://leetcode-cn.com/problems/recover-binary-search-tree/)


#### 完全二叉树

* [222. 完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes/)
* [297. 二叉树的序列化与反序列化](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/)
* [662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/)

> 二叉堆就是完全二叉树的一个应用

#### 路径

* [124. 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)
* [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

#### 距离

* [834. 树中距离之和](https://leetcode-cn.com/problems/sum-of-distances-in-tree/)

  > 待做

* [863. 二叉树中所有距离为 K 的结点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)

### 七个技巧

关于dfs的技巧，bfs直接用模版即可

#### dfs(root)

```java
void dfs(root){
    // your code    
}
```

#### 单/双递归

如果题目有类似，**任意节点开始 xxx 或者所有 xxx**这样的说法，就可以考虑使用双递归；

双递归的基本套路就是一个主递归函数和一个内部递归函数。

主递归函数负责计算以某一个节点开始的 xxx，内部递归函数负责计算 xxx，这样就实现了以**所有节点开始的 xxx**

一个典型的加法双递归是这样的：

```java
int dfs_inner(root){
    // 这里写你的逻辑，就是前序遍历
    dfs_inner(root.left);
    dfs_inner(root.right);
    // 或者在这里写你的逻辑，那就是后序遍历
}

    
int dfs_main(root){
        return dfs_inner(root) + dfs_main(root.left) + dfs_main(root.right)
}
```

* [面试题 04.12. 求和路径](https://leetcode-cn.com/problems/paths-with-sum-lcci/)
* [563. 二叉树的坡度](https://leetcode-cn.com/problems/binary-tree-tilt/)

> 其中 xxx 可以替换成任何题目描述，比如路径和等

但是如果递归中有重复计算，则可以使用双递归 + 记忆化 或者 直接单递归。

#### 前后遍历

前后序对于树而言，前后序遍历更形象的说法因为自顶向下(前序)/自底向上(后序)

* **自顶向下**就是在每个递归层级，首先访问节点来计算一些值，并在递归调用函数时将这些值传递到子节点，一般是**通过参数传到子树**中。
* **自底向上**是另一种常见的递归方法，首先对所有子节点递归地调用函数，然后根据**返回值**和**根节点本身**的值得到答案。

题目：

* [1022. 从根到叶的二进制数之和](https://leetcode-cn.com/problems/sum-of-root-to-leaf-binary-numbers/)
* [1448. 统计二叉树中好节点的数目](https://leetcode-cn.com/problems/count-good-nodes-in-binary-tree/)

#### 虚拟节点

和链表专题中的虚拟头类似

* [814. 二叉树剪枝](https://leetcode-cn.com/problems/binary-tree-pruning/)
* [1325. 删除给定值的叶子节点](https://leetcode-cn.com/problems/delete-leaves-with-a-given-value/)

#### 边界

**搜索类**

* 空节点

  ```java
  int dfs(TreeNode root){
  	if(root == null){
  		return 0;
  	}
  }
  ```

* 叶子节点

  ```java
  void dfs(TreeNode root){
  	if(root == null){
  		// 空节点
  	}
      if(root.left == null && root.right == null){
          // 叶子节点
      }
  }
  ```

**构建类**

* 参数扩展的边界：[1008. 前序遍历构造二叉搜索树](https://leetcode-cn.com/problems/construct-binary-search-tree-from-preorder-traversal/)
* 虚拟节点

#### 参数扩展大法

* 若不考虑参数扩展，一个简单的dfs通常如下：

  ```java
  void dfs(TreeNode root){
      // 判空...
  	dfs(root.left);
      dfs(root.right);
  }
  ```

* 携带父节点情况

  ```java
  void dfs(TreeNode root, TreeNode parent){
      // 判空...
  	dfs(root.left, root);
      dfs(root.right, root);
  }
  ```

* 携带路径信息

  ```java
  // 路径和
  void dfs(TreeNode root, int sum){
      // 判空...
  	dfs(root.left, sum+root.val);
      dfs(root.right, sum+root.val);
  }
  
  // 路径
  void dfs(TreeNode root, List<Integer> path){
      // 判空...
      path.add(root.val)
  	dfs(root.left, path);
      dfs(root.right, path);
  }
  ```

* 携带最大最小值

  ```java
  void dfs(TreeNode root, int lower, int upper){
      // 判空...
      dfs(root.left, Math.min(root.val, lower), Math.max(root.val, upper));
      dfs(root.right, Math.min(root.val, lower), Math.max(root.val, upper));
  }
  ```

  > [1026. 节点与其祖先之间的最大差值](https://leetcode-cn.com/problems/maximum-difference-between-node-and-ancestor/)

#### 返回元祖/列表

这个技巧和参数扩展有异曲同工之妙，只不过一个作用于函数参数，一个作用于函数返回值。

**返回元祖**

> Java 中可以创建一个类/一个Map

[865. 具有所有最深节点的最小子树](https://leetcode-cn.com/problems/smallest-subtree-with-all-the-deepest-nodes/)

**返回数组**

[894. 所有可能的满二叉树](https://leetcode-cn.com/problems/all-possible-full-binary-trees/)

[1530. 好叶子节点对的数量](https://leetcode-cn.com/problems/number-of-good-leaf-nodes-pairs/)

## 题目

### 典型题目：

**索引**

* [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)
* [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)
* [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)
* [102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)
* [117. 填充每个节点的下一个右侧节点指针 II](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii/)
* [513. 找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value/)
* [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)
* [105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
* [106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)
* [889. 根据前序和后序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)
* [297. 二叉树的序列化与反序列化](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/)
* [654. 最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)
* [894. 所有可能的满二叉树](https://leetcode-cn.com/problems/all-possible-full-binary-trees/)
* [1008. 前序遍历构造二叉搜索树](https://leetcode-cn.com/problems/construct-binary-search-tree-from-preorder-traversal/)
* [116. 填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)
* [450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)
* [669. 修剪二叉搜索树](https://leetcode-cn.com/problems/trim-a-binary-search-tree/)
* [863. 二叉树中所有距离为 K 的结点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)
* [98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)
* [99. 恢复二叉搜索树](https://leetcode-cn.com/problems/recover-binary-search-tree/)
* [222. 完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes/)
* [662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/)
* [124. 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)
* [834. 树中距离之和](https://leetcode-cn.com/problems/sum-of-distances-in-tree/)【待做】
* [面试题 04.12. 求和路径](https://leetcode-cn.com/problems/paths-with-sum-lcci/)
* [563. 二叉树的坡度](https://leetcode-cn.com/problems/binary-tree-tilt/)
* [1022. 从根到叶的二进制数之和](https://leetcode-cn.com/problems/sum-of-root-to-leaf-binary-numbers/)
* [1448. 统计二叉树中好节点的数目](https://leetcode-cn.com/problems/count-good-nodes-in-binary-tree/)
* [814. 二叉树剪枝](https://leetcode-cn.com/problems/binary-tree-pruning/)
* [1325. 删除给定值的叶子节点](https://leetcode-cn.com/problems/delete-leaves-with-a-given-value/)
* [1026. 节点与其祖先之间的最大差值](https://leetcode-cn.com/problems/maximum-difference-between-node-and-ancestor/)
* [865. 具有所有最深节点的最小子树](https://leetcode-cn.com/problems/smallest-subtree-with-all-the-deepest-nodes/)
* [1530. 好叶子节点对的数量](https://leetcode-cn.com/problems/number-of-good-leaf-nodes-pairs/)

**题目**

#### [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

```java
// 方法1：递归法
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> res = new LinkedList<>();
    inorder(root, res);
    return res;
}

public void inorder(TreeNode root, List res){

    if (root == null){
        return;
    }
	// 中序遍历：
    // 先左子树
    inorder(root.left, res);
    // 再根节点
    res.add(root.val);
    // 最后右子树
    inorder(root.right, res);
}

// 方法2：迭代法
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new LinkedList<>();
    Stack<TreeNode> stack = new Stack<>();

    TreeNode cur = root;
    while(!stack.isEmpty() || cur != null){
        // 将树的左节点压入栈
        while(cur != null){
            stack.push(cur);
            cur = cur.left;
        }
        // 此时栈顶元素为最左侧元素
        TreeNode node = stack.pop();
        // 处理
        result.add(node.val);
        // 存在右节点，进行也要相同的遍历操作
        if(node.right != null){
            cur = node.right;
        }
    }
    return result;
}

// 方法3：标记法——通吃前中后序三种遍历
public List<Integer> inorderTraversal(TreeNode root){

    List<Integer> result = new LinkedList<>();
    Stack<TreeNode> stack = new Stack<>();
    // 标记作用：参数Integer：1-已经被遍历了  0-还没有被遍历
    Map<TreeNode, Integer> map = new HashMap<>(); 

    stack.push(root);
    map.put(root, 0);  

    while(!stack.isEmpty()){
        TreeNode node = stack.pop();
        if (node == null){
            continue;
        }

        if (map.get(node) == 0){
            // 这个节点还没有被遍历过
            stack.push(node.right);
            stack.push(node);
            stack.push(node.left);
            
            map.put(node, 1);
            map.put(node.right, 0);
            map.put(node.left, 0);
        }else{
            // 这个已经被遍历过了，则加入result结果中
            result.add(node.val);
        }
    }
    return result;
}
```

#### [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

```java
// 方法1：递归法
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> result = new LinkedList<>();
    
    preorderTraversal(root, result);
    return result;
}

public void preorderTraversal(TreeNode root, List<Integer> list){

    if(root == null){
        return;
    }
    // 前序遍历：
    // 先根
    list.add(root.val);
    // 再左
    preorderTraversal(root.left, list);
    // 最后右
    preorderTraversal(root.right, list);
}

// 方法2：迭代法
public List<Integer> preorderTraversal(TreeNode root){

    List<Integer> result = new LinkedList<>();
    if (root == null){
        return result;
    }

    // 创建栈
    Stack<TreeNode> stack = new Stack<>();
    // 每个节点都进栈一次，然后再出栈一次，出栈时若有左右节点则令左右节点进栈
    stack.push(root);
    while(!stack.isEmpty()){
        TreeNode node = stack.pop();
        result.add(node.val);
        if (node.right != null){
            stack.push(node.right);
        }
        if (node.left != null){
            stack.push(node.left);
        }
    }
    return result;
}

// 方法3：标记法——通吃前中后序三种遍历
public List<Integer> preorderTraversal(TreeNode root){

    List<Integer> result = new LinkedList<>();
    Stack<TreeNode> stack = new Stack<>();
    // 标记作用：参数Integer：1-已经被遍历了  0-还没有被遍历
    Map<TreeNode, Integer> map = new HashMap<>();  

    stack.push(root);
    map.put(root, 0);  

    while(!stack.isEmpty()){
        TreeNode node = stack.pop();
        if (node == null){
            continue;
        }
		
        if (map.get(node) == 0){
    	    // 这个节点还没有被遍历过
            stack.push(node.right);
            stack.push(node.left);
            stack.push(node);
            map.put(node, 1);
            map.put(node.right, 0);
            map.put(node.left, 0);
        }else{
            // 这个已经被遍历过了，则加入result结果中
            result.add(node.val);
        }
    }
    return result;
}
```

#### [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

```java
// 方法1： 递归解法
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    postorder(root, res);
    return res;
}

public void postorder(TreeNode root, List res){

    if(root == null){
        return;
    }
	// 后续遍历：
    // 先左子树
    postorder(root.left, res);
    // 再右子树
    postorder(root.right, res);
    // 最后跟节点
    res.add(root.val);
}

// 方法2：标记法——通吃前中后序三种遍历
public List<Integer> postorderTraversal(TreeNode root) {
 
    List<Integer> res = new ArrayList<>();
    Stack<TreeNode> stack = new Stack<>();
    // 标记作用：参数Integer：1-已经被遍历了  0-还没有被遍历
    Map<TreeNode, Integer> map = new HashMap();

    stack.push(root);
    map.put(root, 0);
    while(!stack.isEmpty()){
        TreeNode node = stack.pop();

        if (node == null){
            continue;
        }

        if(map.get(node) == 0){
            // 这个节点还没有被遍历过
            stack.push(node);
            stack.push(node.right);
            stack.push(node.left);

            map.put(node, 1);
            map.put(node.left, 0);
            map.put(node.right, 0);
        }else{
            // 这个已经被遍历过了，则加入result结果中
            res.add(node.val);
        }
    }
    return res;
}

// 方法3：先 左右互换的前序遍历，然后转换为后续遍历
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    // 前序遍历：中左右--->中右左
    Stack<TreeNode> stack1 = new Stack<>();
    // 后续遍历：左右中，即为stack1出栈，stack2入栈
    Stack<TreeNode> stack2 = new Stack<>();
	
    // 先进行左右互换的前序遍历：
    stack1.push(root);
    while(!stack1.isEmpty()){
        TreeNode node = stack1.pop();
        if (node == null){
            continue;
        }
        // node处理完成，然后放到2中，等价于之前的result.add(node.val);
        stack2.push(node);
        // 这里相较前序遍历的话就是左右互换
        stack1.push(node.left);
        stack1.push(node.right);
    }
	
    // stack2弹出，放到result中即可
    while(!stack2.isEmpty()){
        TreeNode node = stack2.pop();
        if (node == null){
            continue;
        }
        result.add(node.val);
    }

    return result;
}
```

#### [102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

```java
// 方法1：带一个标记节点的层序遍历
public List<List<Integer>> levelOrder(TreeNode root) {

    List<List<Integer>> result = new ArrayList<>();
    // 先进先出的队列
    Queue<TreeNode> queue = new ArrayDeque<>();

    if (root == null){
        return result;
    }

    // 标记节点，表示每一层的最后一个节点
    TreeNode mark = new TreeNode(-1);
    queue.offer(root);
    queue.offer(mark);

    List<Integer> tmp = new ArrayList<>();
    while (!queue.isEmpty()){
        TreeNode node = queue.poll();
        if (node != mark){
            // 这一层还没有遍历完
            tmp.add(node.val);
            if (node.left != null){
                queue.add(node.left);
            }
            if (node.right != null){
                queue.add(node.right);
            }
        }else{
            // 这一层已经遍历完了
            result.add(tmp);
            tmp = new ArrayList<>();
            if (queue.isEmpty()){
                break;
            }
            // 在队列的最后加上一个标记节点
            queue.add(mark);
        }
    }
    return result;
}

// 方法2：不需要标记节点，利用队列的size
public List<List<Integer>> levelOrder(TreeNode root) {

    List<List<Integer>> result = new ArrayList<>();
    // 先进先出的队列
    Queue<TreeNode> queue = new ArrayDeque<>();

    if (root == null){
        return result;
    }
	
    // 根节点加入队列中
    queue.offer(root);
    while (!queue.isEmpty()){
        List<Integer> tmp = new ArrayList<>();
        // 根据队列中的节点个数去遍历
        int size = queue.size();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();
            tmp.add(node.val);
            if (node.left != null){
                queue.add(node.left);
            }
            if (node.right != null){
                queue.add(node.right);
            }
        }
        result.add(tmp);
    }
    return result;
}
```

#### [117. 填充每个节点的下一个右侧节点指针 II](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii/)

```java
// 方法1：基础的层序遍历（反向）解决：
public Node connect(Node root) {

    if (root == null){
        return root;
    }

    Queue<Node> queue = new ArrayDeque<>();
    Node pre = null;
    queue.offer(root);

    while (!queue.isEmpty()){
        // 根据队列中的节点个数去遍历
        int size = queue.size();
        for(int i=0; i<size; i++){
            Node cur = queue.poll();
            cur.next = pre;

            if (cur.right != null){
                queue.offer(cur.right);
            }
            if (cur.left != null){
                queue.offer(cur.left);
            }
            pre = cur;
        }
        // 前一个节点复位
        pre = null;
    }
    return root;
}

// 方法2：常量级额外空间:参照
public Node connect(Node root) {

    if (root == null){
        return root;
    }

     // cur我们可以把它看做是每一层的链表
     Node cur = root;
     while(cur != null){
        // 遍历当前层的时候，为了方便操作在下一层前面添加一个哑结点
        // （注意这里是访问当前层的节点，然后把下一层的节点串起来）
        Node dummy = new Node(0);
        // pre表示访下一层节点的前一个节点
        Node pre = dummy;
        // 然后开始遍历当前层的链表
        while(cur != null){
            if (cur.left != null){
                // 如果当前节点的左子节点不为空，就让pre节点的next指向他，也就是把它串起来
                pre.next = cur.left;
                // 然后再更新pre
                pre = pre.next;
            }
            // 同理参照左子树
            if (cur.right != null){
                pre.next = cur.right;
                pre = pre.next;
            }
            // 继续访问这一行的下一个节点
            cur = cur.next;
        }
        // 把下一层串联成一个链表之后，让他赋值给cur后续继续循环，直到cur为空为止
        cur = dummy.next;
     }
     return root;
}
```

#### [513. 找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value/)

```java
public int findBottomLeftValue(TreeNode root) {

    // 创建队列
    Queue<TreeNode> queue = new ArrayDeque<>();
    // 将树的根节点放入队列中
    queue.add(root);

    int result = root.val;
    while(!queue.isEmpty()){
        // 多去当前层的节点个数
        int size = queue.size();
        // 遍历当前层的所有节点
        for(int i=0; i < size; i++){
            TreeNode node = queue.remove();
            if (i==0){
                result = node.val;
            }
            if(node.left != null){
                queue.add(node.left);
            }
            if(node.right != null){
                queue.add(node.right);
            }
        }
    }
    return result;
}
```

#### [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

```java
public List<List<Integer>> pathSum(TreeNode root, int sum) {

    if (root == null){
        return new ArrayList<>();
    }

    List<Integer> onePath = new ArrayList<>();
    List<List<Integer>> res = new ArrayList<>();
    dfs(root, 0, onePath, sum, res);
    return res;
}

private void dfs(TreeNode root, int value, List<Integer> path, int sum, List<List<Integer>> res){
	
    // 节点为空，直接返回
    if (root == null){
        return;
    }

    // 为子节点，则需要进行计算是否满足sum
    if(root.left == null && root.right==null){
        if((value + root.val) == sum){
            // value == sum,则为一条完成路径，进行记录，别忘记把该节点添加进path
            path.add(root.val);
            res.add(new ArrayList<>(path));
            // 最终要把节点移除，别忘记
            path.remove(path.size() - 1);
        }
        return;
    }
	
    // 记录值
    value += root.val;
    // 节点添加进路径中
    path.add(root.val);
    // 继续递归遍历
    dfs(root.left, value, path, sum, res);
    dfs(root.right, value, path, sum, res);
	
    // 遍历完一个节点后，需要把节点移除
    path.remove(path.size() - 1);
}
```

#### [105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

```java
// 方法1：直接明了，但是消耗的资源过大
public TreeNode buildTree(int[] preorder, int[] inorder) {

    if(preorder.length == 0){
        return null;
    }
	
    // 前序遍历中，第一个节点必为根节点
    TreeNode root = new TreeNode(preorder[0]);

    // 在中序遍历中，根节点对应的下标，前面为左子节点数目，后面为右子节点的数目
    int index = 0;  
    for(int i=0; i<inorder.length; i++){
        if (inorder[i] == root.val){
            index = i;  // 找到在中序遍历中根节点的下标位置
            break;
        }
    }
    // ☆
    // 根据index来分割 preorder与inorder
    // preorder:[根节点, [左子树的前序遍历结果], [右子树的前序遍历结果] ]
    // inorder:[左子树的中序遍历结果], 根节点(index), [右子树的中序遍历结果] ]
    root.left = buildTree(Arrays.copyOfRange(preorder, 1, index+1), Arrays.copyOfRange(inorder, 0, index));
    root.right = buildTree(Arrays.copyOfRange(preorder, index+1, preorder.length), Arrays.copyOfRange(inorder, index+1, inorder.length));

    return root;
}

// 方法2：加速inorder中对于的根节点的下标寻找
private Map<Integer, Integer> indexMap;

public TreeNode buildTree(int[] preorder, int[] inorder) {
    
    if (preorder.length == 0){
        return null;
    }
	
    // 加入map后，之后的寻找就是O(1)的复杂度了
    indexMap = new HashMap<>();
    for(int i=0; i<preorder.length; i++){
        indexMap.put(inorder[i], i);
    }

    return myBuildTree(preorder, inorder, 0, preorder.length, 0, inorder.length);
}

private TreeNode myBuildTree(int[] preorder, int[] inorder, int p_start, int p_end, int i_start, int i_end){
    // preorder为空，直接返回 null
    if (p_start == p_end){
        return null;
    }

    TreeNode root = new TreeNode(preorder[p_start]);
    int index = indexMap.get(root.val);
    
    // 这部分的传参，画个图比较好理解
    int leaf_num = index - i_start;
    root.left = myBuildTree(preorder, inorder, p_start+1, p_start+leaf_num+1, i_start, index);
    root.right = myBuildTree(preorder, inorder, p_start+leaf_num+1, p_end, index+1, i_end);
    return root;
}
```

#### [106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

```java
// 方法1：简单但是效率低
public TreeNode buildTree(int[] inorder, int[] postorder) {
 
    if (inorder.length == 0){
        return null;
    }

    // 后序遍历中，最后一个节点必为根节点
    TreeNode root = new TreeNode(postorder[postorder.length-1]);
    // 在中序遍历中，根节点对应的下标，前面为左子节点数目，后面为右子节点的数目
    int index = 0;
    for(int i=0; i<inorder.length; i++){
        if (inorder[i] == root.val){
            index = i;
            break;
        }
    }
	
    // ☆
    // 根据index来分割 inorder与postorder
    // inorder:[左子树的中序遍历结果], 根节点(index), [右子树的中序遍历结果] ]
    // postorder:[[左子树的后序遍历结果], [右子树的后序遍历结果], 根节点]
    root.left = buildTree(Arrays.copyOfRange(inorder, 0, index), Arrays.copyOfRange(postorder, 0, index));
    root.right = buildTree(Arrays.copyOfRange(inorder, index+1, inorder.length), Arrays.copyOfRange(postorder, index, postorder.length-1));

    return root;
}


// 方法2：效率高些
private Map<Integer, Integer> hashMap = new HashMap<>();

public TreeNode buildTree(int[] inorder, int[] postorder) {
    if (inorder == null || postorder == null){
        return null;
    }
	
    // 加入map后，之后的寻找就是O(1)的复杂度了
    int len = inorder.length;
    for(int i=0; i<len; i++){
        hashMap.put(inorder[i], i);
    }

    return myBuildTree(inorder, postorder, 0, inorder.length, 0, postorder.length);
}

private TreeNode myBuildTree(int[] inorder, int[] postorder, int i_start, int i_end, int p_start, int p_end){

    if (i_start == i_end){
        return null;
    }

    TreeNode root = new TreeNode(postorder[p_end-1]);
    int index = hashMap.get(root.val);
    // 这部分的传参，画个图比较好理解
    int left_count = index - i_start;
    root.left = myBuildTree(inorder, postorder, i_start, index, p_start, p_start+left_count);
    root.right = myBuildTree(inorder, postorder, index+1, i_end, p_start+left_count, p_end-1);

    return root;
}
```

#### [889. 根据前序和后序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

```java
// 参照这里的说明
// 前序遍历：preorder:[根节点, [左子树的前序遍历结果], [右子树的前序遍历结果]]
// 后续遍历：postorder:[[左子树的后序遍历结果], [右子树的后序遍历结果], 根节点]
// 前序遍历中，第一个节点为root，第二个节点为左子树（若存在）的根节点val，但对于后续遍历来说，这个节点则为左子树后续遍历的结束，因此有如下：
// 在后续遍历中，找到val的下表index，则左子树为0~index，而在前序遍历中，左子树为1~index+1
public TreeNode constructFromPrePost(int[] pre, int[] post) {

    if (pre == null || post == null || pre.length == 0 || post.length == 0){
        return null;
    }
	
    // 在前序遍历中找到根节点
    TreeNode root = new TreeNode(pre[0]);
    if (pre.length == 1){
        return root;
    }
	
    // 前序遍历第二个节点为左子树更节点，在后续遍历中找到该节点的index
    int index = 0;
    for(int i=0; i<post.length; i++){
        if (post[i] == pre[1]){
            index = i;
            break;
        }
    }
	
    // 前序遍历：1~index+1为左子树，后续遍历：0~index为左子树
    root.left = constructFromPrePost(Arrays.copyOfRange(pre, 1, index+2), Arrays.copyOfRange(post, 0, index+1));
    root.right = constructFromPrePost(Arrays.copyOfRange(pre, index+2, pre.length), Arrays.copyOfRange(post, index+1, post.length-1));

    return root;
}
```

#### [297. 二叉树的序列化与反序列化](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/)

```java
// 序列化过程，层序遍历，
// 类似处理完全二叉树，但是还是有区别的，完全二叉树中的部分节点是不会被序列化的！
public String serialize(TreeNode root) {

    if(root == null){
        return null;
    }

    // 创建队列--->层序遍历
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    StringBuffer result = new StringBuffer();
    result.append("[");

    while(!queue.isEmpty()){
        int size = queue.size();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();
            if (node != null){
                // 节点不为空，这里还添加了为空节点的子节点
                result.append(node.val+",");
                queue.offer(node.left);
                queue.offer(node.right);
            }else{
                // 节点为空，直接添加null
                result.append("null,");
            }
        }
    }
    // 完成序列化，输出
    return result.substring(0, result.length()-1) + "]";
}

// 反序列化
public TreeNode deserialize(String data) {
    
    if(data == null){
        return null;
    }

    String subData = data.substring(1, data.length() - 1);
    String[] datas = subData.split(",");
    
    // 还是需要队列进行存储节点的，不然会丢失结点间的关系
    Queue<TreeNode> queue = new ArrayDeque<>();
    TreeNode root = new TreeNode(Integer.parseInt(datas[0]));
    queue.offer(root);
	
    // 关键两点：用三个指针分别指向数组第一项，第二项和第三项，p1,p2,p2
    // p2,p3分别为p1的左右子节点：p1每次移动一位，p2和p3每次移动两位, 
    int left_pos = -1, right_pos = 0;
    for(int i=1; i<datas.length-2; i++){
		
        // 对于null还是需要特殊考虑的
        if(datas[i].equals("null")){
            continue;
        }
		
        // p1每次移动一位，p2和p3每次移动两位, 
        left_pos += 2;
        right_pos += 2;
        
        TreeNode node = queue.poll();

        if(!datas[left_pos].equals("null")){
            node.left = new TreeNode(Integer.parseInt(datas[left_pos]));
            queue.offer(node.left);
        }

        if(!datas[right_pos].equals("null")){
            node.right = new TreeNode(Integer.parseInt(datas[right_pos]));
            queue.offer(node.right);
        }
    }
    return root;
}
```

#### [654. 最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)

```java
public TreeNode constructMaximumBinaryTree(int[] nums) {

    if (nums.length == 0){
        return null;
    }
	
    // 找到最大值，及其下标
    int max = nums[0];
    int index = 0;
    for(int i=0; i<nums.length; i++){
        if(nums[i] > max){
            max = nums[i];
            index = i;
        }
    }
	
    // 根节点就是这个最大值
    TreeNode root = new TreeNode(max);
    // 左子树是最大值的左边部分
    root.left = constructMaximumBinaryTree(Arrays.copyOfRange(nums, 0, index));
    // 右子树是最大值的右边部分
    root.right = constructMaximumBinaryTree(Arrays.copyOfRange(nums, index+1, nums.length));

    return root;
}
```

#### [894. 所有可能的满二叉树](https://leetcode-cn.com/problems/all-possible-full-binary-trees/)

```java
// 我是笨B，这题太难了！！！
public List<TreeNode> allPossibleFBT(int N) {
    List<TreeNode> ans = new ArrayList<>();

    // 偶数是不能构成满二叉树的
    if(N % 2 == 0){
        return ans;
    }

    // 1个node直接返回
    if(N == 1){
        TreeNode head = new TreeNode(0);
        ans.add(head);
        return ans;
    }

    for(int i=1; i<N; i+= 2){
        List<TreeNode> left = allPossibleFBT(i);
        List<TreeNode> right = allPossibleFBT(N-1-i);
        // 每种可能性都加进来
        for(TreeNode l: left){
            for(TreeNode r: right){
                TreeNode head = new TreeNode(0);
                head.left = l;
                head.right = r;
                ans.add(head);                    
            }
        }
    }
    return ans;
}
```

#### [1008. 前序遍历构造二叉搜索树](https://leetcode-cn.com/problems/construct-binary-search-tree-from-preorder-traversal/)

```java
public TreeNode bstFromPreorder(int[] preorder) {

    if(preorder == null || preorder.length == 0){
        return null;
    }
    
    // 前序遍历第一个节点为根节点
    TreeNode root = new TreeNode(preorder[0]);
    if (preorder.length == 1){
        return root;
    }
	
    // 找到左右子树，左子树的值都小于root值，右子树的值都大于root值
    int index = 0;
    for(int i=0; i<preorder.length; i++){
        if (preorder[i] > root.val){
            break;
        }
        // 左子树的结束下标
        index = i;
    }

    root.left = bstFromPreorder(Arrays.copyOfRange(preorder, 1, index+1));
    root.right = bstFromPreorder(Arrays.copyOfRange(preorder, index+1, preorder.length));

    return root;
}
```

#### [116. 填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

```java
// 方法1：
public Node connect(Node root) {
 
    if (root == null){
        return root;
    }
	// 建立双向队列存放节点，每个节点进出队列两次
    Queue<Node> queue = new ArrayDeque<>();
    queue.offer(root);
    Node pre = null;
	
    while(!queue.isEmpty()){
     	// 获取该层节点个数进行遍历
        int size = queue.size();
        for(int i=0; i<size; i++){
            Node node = queue.poll();
            node.next = pre;
			
            // 指向右侧节点的话，改变一下进队的顺序即可
            if(node.right != null){
                queue.offer(node.right);
            }
            if(node.left != null){
                queue.offer(node.left);
            }
            pre = node;
        }
        pre = null;
    }
    return root;
}
```

#### ？[450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

```java
// 我觉得我是对的...88 / 91  淦
public TreeNode deleteNode(TreeNode root, int key) {
 
    return dfs(root, key);
}

private TreeNode dfs(TreeNode root, int key){
 
    if(root == null){
        return null;
    }

    if(root.val == key){
        // 1.为叶子节点
        if(root.left == null && root.right == null){
            return null;
        }

        // 2.只有左/右子节点
        if(root.left == null || root.right == null){
            return root.left == null ? root.right : root.left;
        }

        // 3.最复杂：左右子节点都存在，则找到右子树中最小节点，替换，然后再删除这个节点
        TreeNode parent = root;
        TreeNode current = root.right;
        while(current.left != null){
            parent = current;
            current = current.left;
        }
        if(parent == root){
            // 画图就明白了
            parent.right = current.right;
        }else{
            parent.left = current.right;
        }
        root.val = current.val;
    }else if(root.val > key){
        root.left = dfs(root.left, key);
    }else{
        root.right = dfs(root.right, key);
    }
    return root;
}
```

#### [669. 修剪二叉搜索树](https://leetcode-cn.com/problems/trim-a-binary-search-tree/)

```java
public TreeNode trimBST(TreeNode root, int low, int high) {
    TreeNode ans = new TreeNode(-1);
    ans.left = root;

    dfs(root, low, high);

    // root修改的情况
    if(root.val < low){
        return root.right;
    }

    if(root.val > high){
        return root.left;
    }

    return ans.left;    
}

private TreeNode dfs(TreeNode root, int low, int high){

    if(root == null){
        return null;
    }

    root.left = dfs(root.left, low, high);
    root.right = dfs(root.right, low, high);

    if(root.val >= low && root.val <= high){
        return root;
    }else{
        if(root.val < low){
            return root.right;
        }else{
            return root.left;
        }
    }
}

// 官方 无敌简介代码：
public TreeNode trimBST(TreeNode root, int L, int R) {
    if (root == null) return root;
    if (root.val > R) return trimBST(root.left, L, R);
    if (root.val < L) return trimBST(root.right, L, R);

    root.left = trimBST(root.left, L, R);
    root.right = trimBST(root.right, L, R);
    return root;
}
```

#### [863. 二叉树中所有距离为 K 的结点](https://leetcode-cn.com/problems/all-nodes-distance-k-in-binary-tree/)

```java
// 这道题做的心情很不美丽！！！
// 节点有指向父节点的引用，也就知道了距离该节点1距离的所有节点
Map<TreeNode, TreeNode> parent;

public List<Integer> distanceK(TreeNode root, TreeNode target, int K) {

    parent = new HashMap();
    dfs(root, null);

    List<Integer> result = new ArrayList<>();
    Queue<TreeNode> queue = new LinkedList<>();
    Set<TreeNode> visited = new HashSet<>();
    visited.add(null);   // 这样节点就不用判空了

    int dist = 0;
    queue.add(target);
    visited.add(target);
    while(!queue.isEmpty()){  // 类型进行图的bfs
        if(dist == K){
            for(TreeNode n: queue){
                result.add(n.val);
            }
            return result;
        }else{
            dist++;
        }

        int size = queue.size();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();

            if(!visited.contains(node.left)){
                visited.add(node.left);
                queue.offer(node.left);
            }

            if(!visited.contains(node.right)){
                visited.add(node.right);
                queue.offer(node.right);
            }

            TreeNode par = parent.get(node);
            if(!visited.contains(par)){
                visited.add(par);
                queue.offer(par);
            }
        }
    }
    return result;
}

private void dfs(TreeNode root, TreeNode par){
    if(root == null){
        return;
    }
    parent.put(root, par);
    dfs(root.left, root);
    dfs(root.right, root);
}
```

#### [98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

```java
public boolean isValidBST(TreeNode root) {
	// 遍历结果
    List<Integer> path = new ArrayList<>();
	
    // 中序遍历
    inOrder(root, path);

    for(int i=0; i<path.size()-1; i++){
        if(path.get(i) >= path.get(i+1)){
            return false;
        }
    }

    return true;
}

public void inOrder(TreeNode root, List path) {
    if (root == null){
        return;
    }
    // 中序遍历,左中右
    inOrder(root.left, path);
    list.add(root.val);
    inOrder(root.right, path);
}
```

#### [99. 恢复二叉搜索树](https://leetcode-cn.com/problems/recover-binary-search-tree/)

```java
public void recoverTree(TreeNode root) {
    List<TreeNode> nodes = new LinkedList<>();
    // 需要交换的节点
    TreeNode node1 = null;
    TreeNode node2 = null;

    // 中序遍历
    inorder(root, nodes);

    int index=0;
    for(int i=index; i<nodes.size()-1; i++){
        if(nodes.get(i).val > nodes.get(i+1).val){
            // 只有一个递减
            node1 = nodes.get(i);
            node2 = nodes.get(i+1);
            index = ++i;
            break;
        }
    }

    if(index == 0){
        return;
    }

    for(int i=index; i<nodes.size()-1; i++){
        if(nodes.get(i).val > nodes.get(i+1).val){
            // 还有一个递减出现，更新node2
            node2 = nodes.get(i+1);
            break;
        }
    }

    int tmp = node1.val;
    node1.val = node2.val;
    node2.val = tmp;
}

private void inorder(TreeNode root, List<TreeNode> nodes){
    
    if(root == null){
        return;
    }

    inorder(root.left, nodes);
    nodes.add(root);
    inorder(root.right, nodes);
}
```

#### [222. 完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes/)

```java
// 解法1：但是效率较低
public int countNodes(TreeNode root) {
	// 判空
    if (root == null){
        return 0;
    }
	// 准备好队列，并让root入队
    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);
    int sum = 1;  // 计数

    while(!queue.isEmpty()){
        // 遍历一层
        int size = queue.size();
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();

            if(node.left != null){
                sum += 1;  // 当前节点存在左子节点，sum+1
                queue.offer(node.left);
            }else{
                // 完全二叉树的性质，当前节点左子树为空，说明已经遍历完了
                return sum;  
            }

            if(node.right != null){
                sum += 1;  // 当前节点存在右子节点，sum+1
                queue.offer(node.right);
            }else{
                // 完全二叉树的性质，当前节点右子树为空，说明已经遍历完了
                return sum;
            }
        }
    }

    return sum;
}

// 解法2：根本没用到完成二叉树的性质，所有的树都可以那么做，直接好家伙
public int countNodes(TreeNode root) {
    if(root == null) return 0;
    return countNodes(root.left) + countNodes(root.right) + 1;
}

// 解法3：参考——最优解法，在结题中：Wanglihao的解法
public int countNodes(TreeNode root) {
 
    if(root == null){
        return 0;
    }
	
    int left_height = countLevel(root.left);
    int right_height = countLevel(root.right);

    if(left_height == right_height){
        // return countNodes(root.right) + ((int)Math.pow(2, left_height));
        return countNodes(root.right) + (1<<left_height);
    }else{
        // return countNodes(root.left) + ((int)Math.pow(2, right_height));
        return countNodes(root.left) + (1<<right_height);
    }
}

private int countLevel(TreeNode root){
    int level = 0;
    while(root != null){
        level ++;
        root = root.left;
    }
    return level;
}
```

#### [662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/)

```java
// 给每个节点加个标号
public int widthOfBinaryTree(TreeNode root) {

    if (root == null){
        return 0;
    }
 
    Queue<TreeNode> queue = new LinkedList<>();
    Map<TreeNode, Integer> map = new HashMap<>();
    queue.offer(root);
    map.put(root, 1);
    List<Integer> result = new ArrayList<>();

    while(!queue.isEmpty()){
        int size = queue.size();
        int pos = 0;       // 记录节点的pos
        int pos_base = 0;  // 节点开始pos
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();

            pos = map.get(node);
            if(pos_base == 0){
                pos_base = pos;
            }

            if(node.left != null){
                queue.offer(node.left);
                map.put(node.left, pos*2);
            }

            if(node.right != null){
                queue.offer(node.right);
                map.put(node.right, pos*2+1);
            }
        }
        // 记录下每一层的长度
        result.add(pos - pos_base + 1);
    }

    // 取每一层长度的最大值
    int max = 1;
    for(int i=0; i<result.size(); i++){
        if(result.get(i) > max){
            max = result.get(i);
        }
    }
    return max;
}


// 改进方法1：不需要额外的list来记录每层的长度，然而结果还是一样的....
public int widthOfBinaryTree(TreeNode root) {

    if (root == null){
        return 0;
    }
    
    Queue<TreeNode> queue = new LinkedList<>();
    Map<TreeNode, Integer> map = new HashMap<>();
    queue.offer(root);
    map.put(root, 1);

    int max = 1;
    while(!queue.isEmpty()){
        int size = queue.size();
        int pos = 0;       // 记录节点的pos
        int pos_base = 0;  // 节点开始pos
        for(int i=0; i<size; i++){
            TreeNode node = queue.poll();

            pos = map.get(node);
            if(pos_base == 0){
                pos_base = pos;
            }

            if(node.left != null){
                queue.offer(node.left);
                map.put(node.left, pos*2);
            }

            if(node.right != null){
                queue.offer(node.right);
                map.put(node.right, pos*2+1);
            }
            max = Math.max(max, pos - pos_base + 1);
        }
    }
    return max;
}
```

#### [124. 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

```java
int maxSum = Integer.MIN_VALUE;

public int maxPathSum(TreeNode root) {
    maxGain(root);
    return maxSum;
}

public int maxGain(TreeNode node){
    if(node == null){
        return 0;
    }

    // 递归计算左右子节点的最大贡献值
    // 只有在最大贡献值大于0时，才会选取对应子节点
    int leftGain = Math.max(maxGain(node.left), 0);
    int rightGain = Math.max(maxGain(node.right), 0);

    // 节点的最大路径和取决于该节点的值与该节点的左右子节点的最大贡献值
    int priceNewpath = node.val + leftGain + rightGain;
    // 更新答案
    maxSum = Math.max(maxSum, priceNewpath);

    return node.val + Math.max(leftGain, rightGain);
}
```

#### [面试题 04.12. 求和路径](https://leetcode-cn.com/problems/paths-with-sum-lcci/)

```java
// 方法1：双递归，对每个节点都进行一次dfs
public int pathSum(TreeNode root, int sum) {
    if(root == null){
        return 0;
    }
    // 每个结点都作为root结点，进行递归
    return helper(root, sum) + pathSum(root.left, sum) + pathSum(root.right, sum);
}

private int helper(TreeNode root, int sum){
    
    if(root == null){
        return 0;
    }
    sum -= root.val;

    int count = 0;
    if(sum == 0){
        count ++;
    }

    count += helper(root.left, sum);
    count += helper(root.right, sum);

    return count;
}

// 方法2：前缀和，先放放
```

#### [563. 二叉树的坡度](https://leetcode-cn.com/problems/binary-tree-tilt/)

```java
public int findTilt(TreeNode root) {

    if(root == null){
        return 0;
    }

    return Math.abs(helper(root.left) - helper(root.right)) + findTilt(root.left) + findTilt(root.right);
}

// 返回传入的root结点的所有值
private int helper(TreeNode root){

    if(root == null){
        return 0;
    }

    int left_val = helper(root.left);
    int right_val = helper(root.right);

    return root.val + left_val + right_val;
}
```

#### [1022. 从根到叶的二进制数之和](https://leetcode-cn.com/problems/sum-of-root-to-leaf-binary-numbers/)

```java
// 方法1：自己这个笨B做的[狗头]，直接dfs
int sum = 0;
public int sumRootToLeaf(TreeNode root) {

    if(root == null){
        return 0;
    }
 	// 记录路径
    Stack<Integer> path = new Stack<>();
    helper(root, path);
    return sum;
}

private void helper(TreeNode root, Stack path){
	// 路径添加
    path.push(root.val);
    
    if(root.left != null){
        helper(root.left, path);
    }

    if(root.right != null){
        helper(root.right, path);
    }
	
    // 当叶子节点时，去计算一波结果，加到sum中
    if(root.left == null && root.right == null){
        addSum(path);
    }
	// 回去
    path.pop();
}

private void addSum(Stack<Integer> path){
    Integer result = 0;
    for(int i=0; i<path.size(); i++){
        result += path.get(path.size()-i-1) * (int)Math.pow(2, i);
    }
    sum += result;
}

// 方法2：dfs，别写的dfs
int ans;
public int sumRootToLeaf(TreeNode root) {
    sumbinary(root, 0);
    return ans;
}

public void sumbinary(TreeNode root, int cur){
    if(root == null){
        return;
    }
    if(root.left == null && root.right == null){
        ans += cur * 2 + root.val;
        return;
    }
    sumbinary(root.left, cur * 2 + root.val);
    sumbinary(root.right, cur * 2 + root.val);
}
```

#### [1448. 统计二叉树中好节点的数目](https://leetcode-cn.com/problems/count-good-nodes-in-binary-tree/)

```java
int sum;

public int goodNodes(TreeNode root) {

    if(root == null){
        return 0;
    }

    helper(root, root.val);

    return sum;
}

private void helper(TreeNode root, int max){

    if(root == null){
        return;
    }

    if(root.val >= max){
        sum ++;
        max = root.val;
    }

    helper(root.left, max);
    helper(root.right, max);
}
```

#### [814. 二叉树剪枝](https://leetcode-cn.com/problems/binary-tree-pruning/)

```java
public TreeNode pruneTree(TreeNode root) {
    // 虚拟节点，记录一下根节点
    TreeNode ans = new TreeNode(-1);
    ans.left = root;

    dfs(root);

    // 特殊情况
    if(root.left==null && root.right==null && root.val==0){
        return null;
    }
    return ans.left;
}

public int dfs(TreeNode root){
    if(root == null){
        return 0;
    }

    int left_val = dfs(root.left);
    int right_val = dfs(root.right);

    if(left_val == 0){
        root.left = null;
    }
    if(right_val == 0){
        root.right = null;
    }

    return root.val + left_val + right_val;
}
```

#### [1325. 删除给定值的叶子节点](https://leetcode-cn.com/problems/delete-leaves-with-a-given-value/)

```java
public TreeNode removeLeafNodes(TreeNode root, int target) {
    // 虚拟节点，记录一下根节点
    TreeNode ans = new TreeNode(-1);
    ans.left = root;

    dfs(root, target);

    // 特殊情况
    if(root.val == target && root.left == null && root.right == null){
        return null;
    }

    return ans.left;
}

private int dfs(TreeNode root, int target){

    if(root == null){
        return 0;
    }

    int left_flag = dfs(root.left, target);
    int right_flag = dfs(root.right, target);

    if(left_flag == 1){
        root.left = null;
    }

    if(right_flag == 1){
        root.right = null;
    }

    // 根节点 && 值为target
    if(root.val == target && root.left == null && root.right == null){
        return 1;
    }

    return 0;
}
```

#### [1026. 节点与其祖先之间的最大差值](https://leetcode-cn.com/problems/maximum-difference-between-node-and-ancestor/)

```java
int max = 0;
public int maxAncestorDiff(TreeNode root) {

    dfs(root, root.val, root.val);
    return max;
}

private void dfs(TreeNode root, int lower, int upper){

    if(root == null){
        int tmp = upper - lower;
        if(tmp > max){
            max = tmp;
        }
        return;
    }

    dfs(root.left, Math.min(root.val, lower), Math.max(root.val, upper));
    dfs(root.right, Math.min(root.val, lower), Math.max(root.val, upper));
}
```

#### [865. 具有所有最深节点的最小子树](https://leetcode-cn.com/problems/smallest-subtree-with-all-the-deepest-nodes/)

```java
// 额外创建一个类
class TreeNodeWithLayer{
    TreeNode node;
    int layer;

    TreeNodeWithLayer(TreeNode node, int layer){
        this.node = node;
        this.layer = layer;
    }
}

public TreeNode subtreeWithAllDeepest(TreeNode root) {

    TreeNodeWithLayer ans = dfs(root, 1);

    return ans.node;
}

private TreeNodeWithLayer dfs(TreeNode root, int layer){

    if(root == null){
        return new TreeNodeWithLayer(null, layer-1);
    }

    TreeNodeWithLayer left_node = dfs(root.left, layer + 1);
    TreeNodeWithLayer right_node = dfs(root.right, layer + 1);

    if(left_node.layer == right_node.layer){
        // 注意：left_node.layer
        return new TreeNodeWithLayer(root, left_node.layer);
    }else if(left_node.layer > right_node.layer){
        return left_node;
    }else{
        return right_node;
    }
}
```

#### [1530. 好叶子节点对的数量](https://leetcode-cn.com/problems/number-of-good-leaf-nodes-pairs/)

```java
int ans = 0;

public int countPairs(TreeNode root, int distance) {
    dfs(root, distance);
    return ans;
}

private int[] dfs(TreeNode root, int distance){
    if(root == null){
        return new int[0];
    }

    if(root.left == null && root.right == null){
        int[] ints = {0};  // 叶子节点站数组中的一位
        return ints;
    }

    int[] left_res = dfs(root.left, distance);
    int[] right_res = dfs(root.right, distance);

    int index = 0;
    int length = left_res.length + right_res.length;
    int[] left_right_res = new int[length];

    // 子树的结果计算出来了，那么父节点只需要把子树的每一项加 1 即可
    for(int i=0; i<left_res.length; i++){
        left_res[i] += 1;
        left_right_res[index++] = left_res[i];
    }

    for(int j=0; j<right_res.length; j++){
        right_res[j] += 1;
        left_right_res[index++] = right_res[j];
    }

    // 两个叶子节点的最短路径 = 
    // 其中一个叶子节点到最近公共祖先的距离 + 另外一个叶子节点到最近公共祖先的距离
    for(int i=0; i<left_res.length; i++){
        for(int j=0; j<right_res.length; j++){
            if(left_res[i] + right_res[j] <= distance){
                ans += 1;
            }
        }
    }

    return left_right_res;
}
```

### 其他题目：

**索引**

* [101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/) 

* [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

* [226. 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

* [530. 二叉搜索树的最小绝对差](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/)

* [543. 二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/)

* [783. 二叉搜索树节点最小距离](https://leetcode-cn.com/problems/minimum-distance-between-bst-nodes/)

**题目**

#### [101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

```java
// 方法1：bfs，搞两个队列，直接来
public boolean isSymmetric(TreeNode root) {

    if(root == null){
        return true;
    }

    // 双队列
    Queue<TreeNode> left_queue = new LinkedList<>();
    Queue<TreeNode> right_queue = new LinkedList<>();

    left_queue.offer(root);
    right_queue.offer(root);



    while(!left_queue.isEmpty()){

        int size_left = left_queue.size();
        int size_right = right_queue.size();

        if(size_left != size_right){
            return false;
        }

        for(int i=0; i<size_left; i++){
            TreeNode node_left = left_queue.poll();
            TreeNode node_right = right_queue.poll();

            if(node_left.val != node_right.val){
                return false;
            }else{
                if((node_left.left != null && node_right.right == null) 
                || (node_left.right != null && node_right.left == null)
                || (node_left.left == null && node_right.right != null)
                || (node_left.right == null && node_right.left != null)){
                    return false;
                }
            }

            if(node_left.left != null)
                left_queue.offer(node_left.left);
            if(node_left.right != null)
                left_queue.offer(node_left.right);

            if(node_right.right != null)
                right_queue.offer(node_right.right);
            if(node_right.left != null)
                right_queue.offer(node_right.left);

        }
    }

    return true;
}

// 方法2：dfs
public boolean isSymmetric(TreeNode root) {
    if(root == null){
        return true;
    }

    return dfs(root.left, root.right);

}

private boolean dfs(TreeNode node1, TreeNode node2){

    if(node1 == null && node2 == null){
        // 都为null
        return true;
    }

    if(node1 == null || node2 == null || node1.val != node2.val){
        // 上个判断已经排除了一个不为null的情况
        // 因此还存在为null 或者 值不相等直接返回false
        return false;
    }

    // 精华！注意手法
    return dfs(node1.left, node2.right) && dfs(node1.right, node2.left);
}
```

#### [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

```java
public int maxDepth(TreeNode root) {

    if(root == null){
        return 0;
    }

    int left = maxDepth(root.left);
    int right = maxDepth(root.right);

    return Math.max(left, right) + 1;
}
```

#### [226. 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

```java
// 这种直接dfs
public TreeNode invertTree(TreeNode root) {
    
    dfs(root);

    return root;
}

private void dfs(TreeNode root){

    if(root == null){
        return;
    }

    dfs(root.left);
    dfs(root.right);

    TreeNode tmp = root.left;
    root.left = root.right;
    root.right = tmp;

}
```

#### [530. 二叉搜索树的最小绝对差](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/)

```java
// 根据二叉搜索树的性质，进行中序遍历，最小差一定在相邻两个元素之间
List<Integer> path = new ArrayList<>();

public int getMinimumDifference(TreeNode root) {
    inorder(root);

    int min = Integer.MAX_VALUE;

    for(int i=0; i<path.size()-1; i++){
        int tmp = Math.abs(path.get(i) - path.get(i+1));
        if(tmp < min){
            min = tmp;
        }
    }

    return min;
}

private void inorder(TreeNode root){

    if(root == null){
        return;
    }

    inorder(root.left);
    path.add(root.val);
    inorder(root.right);
}
```

> 同783题

#### [543. 二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/)

```java
// 方法1：双递归，但是效率不高
public int diameterOfBinaryTree(TreeNode root) {

    if(root == null){
        return 0;
    }

    int val1 = dfs(root.left, 1) + dfs(root.right, 1);
    int val2 = diameterOfBinaryTree(root.left);
    int val3 = diameterOfBinaryTree(root.right);
    int max = Math.max(val1, val2);
    max = Math.max(max, val3);
    return max;
}

private int dfs(TreeNode root, int layer){

    if(root == null){
        return layer-1;
    }

    int left = dfs(root.left, layer+1);
    int right = dfs(root.right, layer+1);

    return Math.max(left, right);
}

// 方法2：单递归，参考来的
int max = 0;
public int diameterOfBinaryTree(TreeNode root) {
    if (root == null) {
        return 0;
    }
    dfs(root);
    return max;
}

private int dfs(TreeNode root) {
    if (root.left == null && root.right == null) {
        return 0;
    }
    int leftSize = root.left == null? 0: dfs(root.left) + 1;
    int rightSize = root.right == null? 0: dfs(root.right) + 1;
    max = Math.max(max, leftSize + rightSize);
    return Math.max(leftSize, rightSize);
}  
```

#### [783. 二叉搜索树节点最小距离](https://leetcode-cn.com/problems/minimum-distance-between-bst-nodes/)

```java
// 根据二叉搜索树的性质，进行中序遍历，最小差一定在相邻两个元素之间
List<Integer> path = new ArrayList<>();

public int minDiffInBST(TreeNode root) {

    inorder(root);

    int min = Integer.MAX_VALUE;

    for(int i=0; i<path.size()-1; i++){
        int tmp = Math.abs(path.get(i) - path.get(i+1));
        if(tmp < min){
            min = tmp;
        }
    }

    return min;
}

private void inorder(TreeNode root){

    if(root == null){
        return;
    }

    inorder(root.left);
    path.add(root.val);
    inorder(root.right);
}
```

> 同530题

