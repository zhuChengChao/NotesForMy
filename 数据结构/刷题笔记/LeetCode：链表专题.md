# LeetCode：链表专题

> 参考了**力扣加加**对与链表专题的讲解，刷了些 leetcode 题，在此做一些记录，不然没几天就没印象了
>
> 出处：[力扣加加-链表专题](https://leetcode-solution-leetcode-pp.gitbook.io/leetcode-solution/thinkings/linked-list)

## 总结

### leetcode 中对于链表的定义

```java
// 定义方式1：
// Definition for singly-linked list.
public class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

// 定义方式2：
// Definition for singly-linked list.
public class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}
```

### 链表的基本操作：

```java
// 插入： B 插入到 AC之间
temp = A.next;
A.next = B;
B.next = temp

// 删除：ABC节点，删除B节点
A.next = C
// or
A.next = B.next.next

// 遍历
cur = head
while(cur != null){
	// 操作 ...
	cur = cur.next;
}
```

> 如果是头尾节点等特殊情况的话，需要另做考虑

### 一个原则

**画图**，真的很有用

### 两个考点

**修改指针**：反转链表为典型代表，[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

**链表拼接**：如，[21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

### 三个注意

**环**：快慢指针判断环

**边界**：看题，多思考

**前后序：**

* 前序遍历

  ```java
  void function(ListNode head){
  	if (head == null){
          return;
      }    
      // 主逻辑....
      function(head.next);
  }
  ```

* 后续遍历

  ```java
  void function(ListNode head){
  	if (head == null){
          return;
      }    
      function(head.next);
      // 主逻辑....
  }
  ```

* 原话摘录：**如果是前序遍历，那么你可以想象前面的链表都处理好了，怎么处理的不用管**。**如果是后序遍历，那么你可以想象后面的链表都处理好了，怎么处理的不用管**。

### 四个技巧

**虚拟头**

新设立一个节点，用于指向头节点，后续处理完成后，可以返回这个节点

```java
ListNode ans = new ListNode(-1, null);
// ans 不变，后续对于head的操作都不会影响ans，ans.next还是指向最初的head节点
ans.next = head;  
// ...
return ans.next;
```

**快慢指针**：[141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)，[142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

**穿针引线**：[61. 旋转链表](https://leetcode-cn.com/problems/rotate-list/)，[92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

**先穿再排后判空**：

1. 先写逻辑;
2. 再对逻辑排序;
3. 再判断特殊条件

## 题目

### 典型题目：

**索引**

* [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)
* [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)
* [141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)
* [142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)
* [61. 旋转链表](https://leetcode-cn.com/problems/rotate-list/)
* [92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

**题目**

#### [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

```java
// 正向循环
public ListNode reverseList(ListNode head) {
    ListNode pre = null;   // 前一个节点
    ListNode cur = head;   // 当前节点
    ListNode next = null;  // 后一个节点

    while(cur != null){
        // 留下联系方式
        next = cur.next;
        // 修改指针
        cur.next = pre;
        // 继续往下走
        pre = cur;
        cur = next;
    }
    // 反转后的新的头节点返回出去
    return pre;
}

// 递归方式
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null){
        return head;
    }

    ListNode p = reverseList(head.next);
    head.next.next = head;
    head.next = null;
    return p;
}
```

#### [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

```java
// 方法1：遍历法
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    // 虚拟头节点
    ListNode head = new ListNode(-1, null);
    ListNode temp = head;

    // 开始遍历两个链表
    while (l1 != null && l2 != null){
        if (l1.val < l2.val){
            temp.next = l1;
            l1 = l1.next;
        }else{
            temp.next = l2;
            l2 = l2.next;
        }
        temp = temp.next;
    }

	// 剩下的接上
    temp.next = l1 == null ? l2: l1;
    return head.next;
}

// 方法2：递归-后序
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if (l1 == null){
        return l2;
    }else if(l2 == null){
        return l1;
    }else if(l1.val < l2.val){
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    }else{
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    }
}
```

#### [141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

```java
// 快慢指针法
public boolean hasCycle(ListNode head) {
 
    ListNode fast = head;
    ListNode slow = head;

    while(fast != null && fast.next!=null){     
        fast = fast.next.next;  // 快指针走两步
        slow = slow.next;       // 慢指针走一步
        if(fast == slow){
            return true;
        }
    }
    return false;
}


// hash法
public boolean hasCycle(ListNode head) {

    Set<ListNode> hashSet = new HashSet<>();
    while(head != null){
        if(!hashSet.add(head)){
            return true;
        }
        head = head.next;
    }
    return false;
}
```

#### [142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

```java
// hash表
public ListNode detectCycle(ListNode head) {
 
    Set<ListNode> hashSet = new HashSet<>();
    
    while(head != null){
        if(!hashSet.add(head)){
            return head;
        }
        head = head.next;
    }
    return null;
}

// 参考来的：快慢指针法
// 参考地址：https://leetcode-cn.com/problems/linked-list-cycle-ii/solution/linked-list-cycle-ii-kuai-man-zhi-zhen-shuang-zhi-/
public ListNode detectCycle(ListNode head) {
    ListNode fast = head;
    ListNode slow = head;

    while(fast != null && fast.next != null){
        fast = fast.next.next;
        slow = slow.next;
        
        if(fast == slow){
            fast = head;
            while(fast != slow){
                fast = fast.next;
                slow = slow.next;
            }
            return slow;
        }
    }
    return null;
}
```

#### [61. 旋转链表](https://leetcode-cn.com/problems/rotate-list/)

```java
public ListNode rotateRight(ListNode head, int k) {

    // 判断
    if (head == null || head.next == null || k == 0){
        return head;
    }

    // 记录头节点
    ListNode ans = new ListNode(-1);
    ans.next = head;
    ListNode last = null;

    // 获取链表的长度
    int length = 1;
    while (head.next != null){
        head = head.next;
        length ++;
    }
    last = head;

    // 当k为length时，直接返回链表即可
    if (k % length == 0){
        return ans.next;
    }

    // 处理k大于length的情况
    int new_k = k % length;

    // 找到循环后的头节点
    int headIndex = length - new_k;
    ListNode pre = null;
    ListNode cur = ans.next;
    while(headIndex != 0){
        pre = cur;
        cur = cur.next;
        headIndex-=1;
    }

    // 改变指针方向
    last.next = ans.next;
    pre.next = null;

    return cur;
}
```

#### [92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

```java
public ListNode reverseBetween(ListNode head, int m, int n) {

    // 虚拟头
    ListNode ans = new ListNode(-1);
    ans.next = head;

    ListNode pre = null;
    ListNode next = null;

    ListNode m_before = null;
    ListNode m_node = null;

    int count = 0;
    while(head != null){
        count++;

        if(count+1 == m){
            // 找到m位置的前一个节点
            m_before = head;
        }else if(count == m){
            // 找到m节点
            m_node = head;
        }

        if (count >= m && count <= n){
            // 开始反转m节点之后的内容
            next = head.next;
            head.next = pre;
            pre = head;
            head = next;
            if (count == n){
                // 反转到n结点后结束
                m_node.next = head;
                if (m_before != null){
                    m_before.next = pre;
                    pre = ans.next;
                }
                break;
            }
            continue;
        }

        head = head.next;
    }
    return pre;
}
```

### 其他题目：

**索引**

* [25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

* [82. 删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

* [83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

* [86. 分隔链表](https://leetcode-cn.com/problems/partition-list/)

* [138. 复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

* [143. 重排链表](https://leetcode-cn.com/problems/reorder-list/)

* [148. 排序链表](https://leetcode-cn.com/problems/sort-list/)

* [234. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)

* [876. 链表的中间结点](https://leetcode-cn.com/problems/middle-of-the-linked-list/)

* [面试题 02.02. 返回倒数第 k 个节点](https://leetcode-cn.com/problems/kth-node-from-end-of-list-lcci/)

* [面试题 02.03. 删除中间节点](https://leetcode-cn.com/problems/delete-middle-node-lcci/)

**题目**

#### [25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

```java
public ListNode reverseKGroup(ListNode head, int k) {
    if(head == null){
        return head;
    }

    ListNode a = head;
    ListNode b = head;

    for(int i=0; i<k; i++){
        if(b == null){
            return head;
        }
        b = b.next;
    }

    // 反转前 k 个元素
    ListNode newHead = reverse(a, b);
    a.next = reverseKGroup(b, k);

    return newHead;

}

// 反转 [a, b) 之间的元素
ListNode reverse(ListNode a, ListNode b){

    ListNode pre = null;
    ListNode cur = a;
    ListNode next = null;

    while(cur != b){
        next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return pre;
}
```

#### [82. 删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

```java
public ListNode deleteDuplicates(ListNode head) {
    
    // 虚拟头
    ListNode ans = new ListNode(-1);
    ans.next = head;
	
    // set中存了有重复的值
    Set<Integer> set = new HashSet<>();

    while ( head != null){
        // 把存在重复的值都放入set中
        if (head.next != null && head.val == head.next.val){
            set.add(head.val);
        }
        head = head.next;
    }
	
    // set不为空，说明有重复元素存在
    if (!set.isEmpty()){
        head = ans.next;
        ans.next = null;
        ListNode pre;
        pre = ans;
        while (head != null){
            if(!set.contains(head.val)){
                pre.next = new ListNode(head.val);
                pre = pre.next;
            }
            head = head.next;
        }
    }
    return tmp.next;
}

// 参考解法：
public ListNode deleteDuplicates(ListNode head) {
     // 创建虚拟头
    ListNode ans = new ListNode(-1); 
    ans.next = head;
    ListNode cur = ans;

    while(cur.next != null && cur.next.next != null){
        if(cur.next.val == cur.next.next.val){
            ListNode tmp = cur.next;
            while(tmp.next != null && tmp.val == tmp.next.val){
                tmp = tmp.next;
            }
            cur.next = tmp.next;
        }else{
            cur = cur.next;
        }
    }

    return ans.next;
}
```

#### [83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

```java
// 遍历思路
public ListNode deleteDuplicates(ListNode head) {
    ListNode ans = new ListNode(-1);
    ans.next = head;

    while (head != null && head.next != null){
        if (head.val == head.next.val){
            ListNode tmp = head.next;
            while(tmp.next != null && tmp.val == tmp.next.val){
                tmp = tmp.next;
            }
            head.next = tmp.next;
        }
        head = head.next;
    }
    return ans.next;
}

// 官方解法
public ListNode deleteDuplicates(ListNode head) {
    
    ListNode cur = head;
    
    while(cur != null && cur.next!=null){
        if (cur.val == cur.next.val){
            cur.next = cur.next.next;
        }else{
            cur = cur.next;
        }
    }

    return head;
}

// 递归解法：
public ListNode deleteDuplicates(ListNode head) {
    if (head == null || head.next == null){
        return head;
    }

    if(head.val == head.next.val){
        return deleteDuplicates(head.next);
    }else{
        head.next = deleteDuplicates(head.next);
        return head;
    }
}
```

#### [86. 分隔链表](https://leetcode-cn.com/problems/partition-list/)

```java
public ListNode partition(ListNode head, int x) {
    
    // 虚拟头
    ListNode ans = new ListNode(-1);
    ListNode tmp = ans;  // 串联比x小的节点
	
    ListNode ans2 = new ListNode(-2);
    ListNode tmp2 = ans2;  // 串联比x大的节点

    while (head != null){
        if (head.val < x){
            tmp.next = head;
            tmp = tmp.next;
        }else{
            tmp2.next = head;
            tmp2 = tmp2.next;
        }
        head = head.next;
    }

    tmp2.next = null;
    tmp.next = ans2.next;

    return ans.next;
}
```

#### [138. 复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

```java
public Node copyRandomList(Node head) {

    Node ans = new Node(-1);
    ans.next = head;

    Node tmp;
    // node1: 原节点  node2：新节点
    HashMap<Node, Node> hashMap = new HashMap<>();

    while(head != null){
        tmp = new Node(head.val);
        hashMap.put(head, tmp);
        head = head.next;
    }

    head = ans.next;
    while(head != null){

        tmp = hashMap.get(head);
        tmp.next = hashMap.get(head.next);
        tmp.random = hashMap.get(head.random);

        head = head.next;
    }

    return hashMap.get(ans.next);
}
```


#### [143. 重排链表](https://leetcode-cn.com/problems/reorder-list/)

```java
// 效率较低
public void reorderList(ListNode head) {
    ListNode ans = new ListNode(-1);
    ans.next = head;
	
    // 记录链表长度
    int count = 0;
    // 记录每个节点
    HashMap<Integer, ListNode> hashMap = new HashMap<>();

    while(head != null){
        hashMap.put(count++, head);
        head = head.next;
    }

    ListNode tmp = null;
    for(int i=0; i < count/2; i++){
        ListNode first = hashMap.get(i);
        ListNode last = hashMap.get(count-i-1);
        tmp = first.next;

        if(tmp == last){
            last.next = null;
        }else{
            first.next = last;
            last.next = tmp;
        }
    }

    if (tmp!=null && tmp.next != null){
        tmp.next = null;
    }
}

// 官方解法，与我逻辑上相同，但是编程更加简洁
public void reorderList(ListNode head) {
    if(head == null){
        return;
    }

    List<ListNode> list = new ArrayList<>();
    ListNode node = head;
    while(node != null){
        list.add(node);
        node = node.next;
    }

    int i = 0, j = list.size() - 1;
    while(i < j){
        list.get(i).next = list.get(j);
        i++;
        if(i==j){
            break;
        }
        list.get(j).next = list.get(i);
        j--;
    }
    list.get(i).next = null;
}

/***********************************************/
// 官方解法:寻找链表中点 + 链表逆序 + 合并链表
public void reorderList(ListNode head) {
    
    if (head == null){
        return;
    }
    // 找到链表中点
    ListNode mid = middleNode(head);
    ListNode l1 = head; // 左边的链表头
    ListNode l2 = mid.next;  // 右边的链表头
    mid.next = null;

    // 链表逆序
    l2 = reverseList(l2);

    // 合并链表
    mergeList(l1, l2);
}

// 找到链表中点: 调整：相同返回前面的
public ListNode middleNode(ListNode head){
    ListNode fast = head;
    ListNode slow = head;

    while(fast.next != null && fast.next.next != null){
        fast = fast.next.next;
        slow = slow.next;
    }

    return slow;
}

// 链表逆序
public ListNode reverseList(ListNode head){
    ListNode pre = null;
    ListNode cur = head;
    ListNode next = null;

    while(cur != null){
        next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    } 

    return pre;
}

// 合并链表
public void mergeList(ListNode l1, ListNode l2){
    
    ListNode tmp1;
    ListNode tmp2;

    while(l1 != null && l2 != null){
        tmp1 = l1.next;
        tmp2 = l2.next;

        l1.next = l2;
        l2.next = tmp1;

        l1 = tmp1;
        l2 = tmp2;
    }
}
/***********************************************/
```

#### [148. 排序链表](https://leetcode-cn.com/problems/sort-list/)

```java
// 解法1：通过list保存所有节点+对list进行排序实现
public ListNode sortList(ListNode head) {

    if (head == null){
        return null;
    }
 
    List<ListNode> list = new ArrayList<>();

    while(head != null){
        list.add(head);
        head = head.next;
    }

    list.sort(new Comparator<ListNode>() {
        @Override
        public int compare(ListNode o1, ListNode o2) {
            return o1.val - o2.val;
        }
    });

    for(int i=0; i < list.size()-1; i++){
        list.get(i).next = list.get(i+1);
    }

    list.get(list.size() - 1).next = null;

    return list.get(0);
}

// 解法2：归并排序：递归实现
// 时间复杂度：O(nlogn)
// 空间复杂度：O(n)，递归过程中要开辟和原链表同样长度的节点数目
public ListNode sortList(ListNode head) {

    if(head == null || head.next == null){
        return head;
    }

    // 切分:把链表切分成左右两部分
    ListNode fast = head, slow = head;
    while(fast.next != null && fast.next.next != null){
        fast = fast.next.next;
        slow = slow.next;
    }
    // 右边链表的起始点
    ListNode tmp = slow.next;
    // 左右链表之间的连接
    slow.next = null;
    // 递归继续切分：直到切分至只有一个节点为止，进行后续的合并操作
    ListNode left = sortList(head);
    ListNode right = sortList(tmp);

    // 开始进行合并操作
    // 保存left和right合并后的头节点，用于后续的合并
    ListNode ans = new ListNode(-1);
    ListNode tmp_ans = ans;
    while(left != null && right != null){
        if(left.val < right.val){
            tmp_ans.next = left;
            left = left.next;
        }else{
            tmp_ans.next = right;
            right = right.next;
        }
        tmp_ans = tmp_ans.next;
    }
    tmp_ans.next = left != null ? left : right; 
    return ans.next;
}
```

#### [234. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)

```java
// YES:快慢指针找到中间节点+反转后续链表+再进行比较+最好恢复一下链表（没做）
public boolean isPalindrome(ListNode head) {

    if (head == null || head.next == null){
        return true;
    }
 
    // 快慢指针找到中间节点
    ListNode fast = head;
    ListNode slow = head;
    while (fast != null && fast.next !=null){
        fast = fast.next.next;
        slow = slow.next;
    }

    // 反转链表: show指针后的链表将其进行反转
    ListNode pre = null;
    ListNode cur = slow;
    ListNode next = null;
    while(cur != null){
        next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
	
    // 开始比较了
    ListNode cmp_head = pre;

    while(cmp_head != null){
        if (head.val != cmp_head.val){
            return false;
        }
        head = head.next;
        cmp_head = cmp_head.next;
    }
    return true;
}

// 好家伙！直接通过数组：快速解决，但是效率不高
public boolean isPalindrome(ListNode head) {

    if (head == null || head.next == null){
        return true;
    }

    List<Integer> list = new ArrayList<Integer>();

    while(head != null){
        list.add(head.val);
        head = head.next;
    }

    // 使用双指针判断是否回文
    int front = 0;
    int back = list.size()-1;

    while(front < back){
        if (!list.get(front).equals(list.get(back))){
            return false;
        }
        front++;
        back--;
    }
    return true;
}
```

#### [876. 链表的中间结点](https://leetcode-cn.com/problems/middle-of-the-linked-list/)

```java
// 快慢指针解题
public ListNode middleNode(ListNode head) {

    ListNode fast = head;
    ListNode slow = head;

    while(fast != null && fast.next != null){
        slow = slow.next;
        fast = fast.next.next;
    }

    return slow;
}
```

#### [面试题 02.02. 返回倒数第 k 个节点](https://leetcode-cn.com/problems/kth-node-from-end-of-list-lcci/)

```java
// 方法1：暴力法
public int kthToLast(ListNode head, int k) {
 
    ListNode ans = new ListNode(-1);
    ans.next = head;

    if(head == null){
        return 0;
    }

    int size = 0;
    while(head != null){
        size ++;
        head = head.next;
    }

    head = ans.next;
    for(int i=0; i<size; i++){
        if(i == (size-k)){
            return head.val;
        }
        head = head.next;
    }

    return 0;
}

// 方法2：双指针，指针1先走k，然后指针1,2一起走，指针1走到末尾后，返回指针2，指针2的距离即为：size-k
public int kthToLast(ListNode head, int k) {

    if(head == null){
        return 0;
    }

    ListNode node1 = head;
    ListNode node2 = head;
    while(k-- != 0){
        node1 = node1.next;
    }

    while(node1 != null){
        node2 = node2.next;
        node1 = node1.next;
    }

    return node2.val;
}
```

#### [面试题 02.03. 删除中间节点](https://leetcode-cn.com/problems/delete-middle-node-lcci/)

```java
public void deleteNode(ListNode node) {
    node.val = node.next.val;
    node.next = node.next.next;
}
```

