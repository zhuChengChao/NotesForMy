# LeetCode：数组专题

> 有关数组的一些 leetcode 题，在此做一些记录，不然没几天就忘光光了

## 二分查找

> 本文内容摘录自公众号labuladong中有关[二分查找](https://mp.weixin.qq.com/s?__biz=MzAxODQxMDM0Mw==&mid=2247485044&idx=1&sn=e6b95782141c17abe206bfe2323a4226&chksm=9bd7f87caca0716aa5add0ddddce0bfe06f1f878aafb35113644ebf0cf0bfe51659da1c1b733&scene=21#wechat_redirect)的文章

### 总结

#### 总体框架

```java
int binarySearch(int[] nums, int target) {
    int left = 0, right = ...;

    while(...) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            ...
        } else if (nums[mid] < target) {
            left = ...
        } else if (nums[mid] > target) {
            right = ...
        }
    }
    return ...;
}
```

#### 寻找一个数（基本的二分搜索）

```java
int binarySearch(int[] nums, int target) {
    int left = 0; 
    int right = nums.length - 1;  // 注意1

    while(left <= right) {       // <= 注意2
        int mid = left + (right - left) / 2;
        if(nums[mid] == target)
            return mid; 
        else if (nums[mid] < target)
            left = mid + 1;      // 注意3
        else if (nums[mid] > target)
            right = mid - 1;     // 注意4
    }
    return -1;
}
```

#### 寻找左侧边界的二分搜索

1. 写法1：

```java
int left_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0;
    int right = nums.length; // 注意

    while (left < right) { // 注意
        int mid = (left + right) / 2;
        if (nums[mid] == target) {
            right = mid; // 注意：找到了后没有返回，而是继续寻找
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid; // 注意
        }
    }
    // 返回方式1：这样返回的意思就是返回了比target小的个数
    return left;
    // 返回方式2：
    // target 比所有数都大
    if (left == nums.length) return -1;
    // 类似之前算法的处理方式
    return nums[left] == target ? left : -1;
    
}
```

2. 写法2：

```java
int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    // 搜索区间为 [left, right]
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            // 搜索区间变为 [mid+1, right]
            left = mid + 1;
        } else if (nums[mid] > target) {
            // 搜索区间变为 [left, mid-1]
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 收缩右侧边界
            right = mid - 1;
        }
    }
    // 检查出界情况
    if (left >= nums.length || nums[left] != target)
        return -1;
    return left;
}
```

#### 寻找右侧边界的二分查找

1. 写法1：

```java
int right_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0, right = nums.length;

    while (left < right) {
        int mid = (left + right) / 2;
        if (nums[mid] == target) {
            left = mid + 1; // 注意：能找到右边界，没有立即返回，而是继续向右搜索
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;
        }
    }
    // 返回方式1：返回了小于等于target的个数
    return left - 1; // 注意
    // 返回方式2：
    if (left == 0) return -1;
	return nums[left-1] == target ? (left-1) : -1;
}
```

2. 写法2：

```java
int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 这里改成收缩左侧边界即可
            left = mid + 1;
        }
    }
    // 这里改为检查 right 越界的情况，见下图
    if (right < 0 || nums[right] != target)
        return -1;
    return right;
}
```

#### 逻辑统一 :exclamation:

```java
// 寻找一个数
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // 直接返回
            return mid;
        }
    }
    // 直接返回
    return -1;
}

// 寻找左侧边界的二分查找：找左边界的时候，由右侧逼近
int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定左侧边界
            right = mid - 1;
        }
    }
    // 最后要检查 left 越界的情况
    if (left >= nums.length || nums[left] != target)
        return -1;
    return left;
}

// 寻找右侧边界的二分查找：找右边界的时候，由左侧逼近
int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
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
        return -1;
    return right;
}
```

### 题目

#### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

```java
// 标准模版法
public int[] searchRange(int[] nums, int target) {

    int[] res = new int[]{-1, -1};

    int left = 0, right = nums.length-1;
    while(left <= right){
        int mid = left + (right - left) / 2;
        if(nums[mid] < target){
            left = mid + 1;
        }else if(nums[mid] > target){
            right = mid - 1;
        }else{
            right = mid - 1;
        }
    }

    if(left >= nums.length || nums[left] != target){
        return res;
    }else{
        // 找到最左边的值后，继续往右边找到右边界
        res[0] = left;
        while(left+1 < nums.length && nums[left+1] == target){
            left++;
        }
        res[1] = left;

        return res;
    }
}
```

#### [33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

```java
// 很巧妙的二分查找法
public int search(int[] nums, int target) {

    int left = 0, right = nums.length-1;

    while(left <= right){

        int mid = left + (right-left)/2;

        if(nums[mid] == target){
            return mid;
        }else if(nums[mid] < nums[right]){
            // 如果中间的数小于最右边的数，则右半段是有序的,但是左边不确定
            if(nums[mid] < target && target <= nums[right]){
                // 是否在有序的范围内
                left = mid+1;
            }else{
                // 不再右边有序的范围内
                right = mid-1;
            }
        }else{
            // 若中间数大于最右边数，则左半段是有序的
            if(nums[left] <= target && nums[mid] > target){
                // 这个数在左边
                right = mid - 1;
            }else{
                // 这个数在右边
                left = mid + 1;
            }
        }
    }

    return -1;
}
```

#### [74. 搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix/)

```java
// 方法1：先找到行，然后对行进行二分查找
public boolean searchMatrix(int[][] matrix, int target) {

    int r = matrix.length;
    int c = matrix[0].length;

    // 1.先找到所在行
    int row = 0;
    for(int i=0; i<r; i++){
        if(matrix[i][0] <= target && matrix[i][c-1] >= target){
            row = i;
        }
    }

    // 2.对确定的行进行一个二分查找
    int left = 0, right = c-1;
    while(left <= right){
        int mid = left + (right-left)/2;
        if(matrix[row][mid] < target){
            left = mid + 1;
        }else if(matrix[row][mid] > target){
            right = mid - 1;
        }else{
            return true;
        }
    }

    return false;
}

// 方法2：当做树进行查找
public boolean searchMatrix(int[][] matrix, int target) {

    int m = matrix.length;
    int n = matrix[0].length;

    // 左上角作为root节点
    int r = 0, c = n-1;

    while(r < m && c >= 0){

        if(matrix[r][c] < target){
            // 找右子树
            r++;
        }else if(matrix[r][c] > target){
            // 找左子树
            c--;
        }else{
            return true;
        }
    }

    return false;
}
```

#### [240. 搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)

```java
// 和上一题的解法2相同嘞，这里就贴下解法
public boolean searchMatrix(int[][] matrix, int target) {

    // 当成二叉查找树来进行解题
    int m = matrix.length;
    int n = matrix[0].length;

    // 把右上角当做root节点
    int r = 0, c = n-1;

    while(r < m && c >= 0){

        if(target > matrix[r][c]){
            // 相当于走右子树
            r++;
        }else if(target < matrix[r][c]){
            // 相当于走左子树
            c--;
        }else{
            return true;
        }
    }

    return false;
}
```

#### [875. 爱吃香蕉的珂珂](https://leetcode-cn.com/problems/koko-eating-bananas/)

```java
public int minEatingSpeed(int[] piles, int H) {
    // piles 数组的最大值
    int max = getMax(piles);

    // 若：for(int k=1; k < max; k++){...}会有案例超时
    // 通过二分法优化：寻找最小速度，就是寻找左侧边界
    int left = 1, right = max;
    while(left <= right){
        // 这样计算mid是为了防止溢出
        int mid = left + (right - left) / 2;
        // 以 speed 是否能在 H 小时内吃完香蕉
        if(canFinish(piles, mid, H)){
            right = mid-1;
        }else{
            left = mid+1;
        }
    }

    // 最终结果就是返回最大值
    return left;
}

private int getMax(int[] piles){
    int max = piles[0];
    for(int i=1; i<piles.length; i++){
        if(max < piles[i]){
            max = piles[i];
        }
    }
    return max;
}

private boolean canFinish(int[] piles, int speed, int H){
    int time = 0;
    for(int n: piles){
        // 计算吃这堆香蕉用的
        time += n/speed + (n%speed == 0 ? 0: 1);
        if(time > H){
            return false;
        }
    }
    return time <= H;
}
```

#### [1011. 在 D 天内送达包裹的能力](https://leetcode-cn.com/problems/capacity-to-ship-packages-within-d-days/)

```java
// 最小的装载量为能装上最大的货物  最大的装载量就是全部都能装上
public int shipWithinDays(int[] weights, int days) {
    int max = 0;
    int total = 0;
    for(int i=0; i<weights.length; i++){
        total += weights[i];
        if(max < weights[i]){
            max = weights[i];
        }
    }

    int left = max, right = total;
    while(left <= right){
        // 目前闲置能装的量
        int mid = left + (right - left)/2;

        // 判断是否能装完
        if(isFinish(weights, mid, D)){
            right = mid - 1;
        }else{
            left = mid + 1;
        }
    }

    return left;
}


private boolean isFinish(int[] weights, int load, int D){

    int one = 0;
    int cost = 1;  // 这里要从1开始
    for(int i=0; i<weights.length; i++){

        one += weights[i];

        if(one > load){
            one = weights[i];
            cost += 1;
        }
    }
    return cost <= D;
}
```

## 双指针

> 这部分内容主要参考的是公众号 labuladong，中的文章[双指针技巧汇总](https://mp.weixin.qq.com/s?__biz=MzAxODQxMDM0Mw==&mid=2247484505&idx=1&sn=0e9517f7c4021df0e6146c6b2b0c4aba&chksm=9bd7fa51aca07347009c591c403b3228f41617806429e738165bd58d60220bf8f15f92ff8a2e&scene=21#wechat_redirect)

### 总结

#### 快慢指针

1. 判断链表中是否有环

   > 在[链表专题](https://www.cnblogs.com/zhuchengchao/p/14290231.html)已经提及过：[141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)
   >
   > ```java
   > // 快慢指针法
   > public boolean hasCycle(ListNode head) {
   >  
   >        ListNode fast = head;
   >        ListNode slow = head;
   > 
   >        while(fast != null && fast.next!=null){     
   >            fast = fast.next.next;  // 快指针走两步
   >            slow = slow.next;       // 慢指针走一步
   >            if(fast == slow){
   >                return true;
   >            }
   >        }
   >        return false;
   > }
   > ```

2. 已知链表中含有环，返回这个环的起始位置

   > 在[链表专题](https://www.cnblogs.com/zhuchengchao/p/14290231.html)已经提及过：[142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)
   >
   > ```java
   > public ListNode detectCycle(ListNode head) {
   >        ListNode fast = head;
   >        ListNode slow = head;
   > 
   >        while(fast != null && fast.next != null){
   >            fast = fast.next.next;
   >            slow = slow.next;
   >    
   >            if(fast == slow){
   >                // 说明此时有环存在，当其中一个节点回到头结点
   >                fast = head;
   >                // 继续一步一步走，直到相遇，此时就是入环的节点
   >                while(fast != slow){
   >                    fast = fast.next;
   >                    slow = slow.next;
   >                }
   >                return slow;
   >            }
   >        }
   >        return null;
   > }
   > ```

3. 寻找链表的中点

   > 类似上面的思路，快指针走两步，慢指针走一步，当快指针指向链表末尾时，慢指针此时就指向链表的中间节点，需要注意的是链表节点的个数为奇数偶数的情况
   >
   > ```java
   > public boolean hasCycle(ListNode head) {
   >  
   >        ListNode fast = head;
   >        ListNode slow = head;
   > 
   >        while(fast != null && fast.next!=null){     
   >            fast = fast.next.next;  // 快指针走两步
   >            slow = slow.next;       // 慢指针走一步
   >        }
   >        // 需要注意的是：
   >        // 当链表节点个数为奇数时，就是中间节点
   >        // 而当为偶数时，slow指向的是考后边的那个节点
   >        return slow;
   > }
   > ```

4. 寻找链表的倒数第 k 个元素

   > 让快指针先走 k 步，然后快慢指针开始同速前进。这样当快指针走到链表末尾 null 时，慢指针所在的位置就是倒数第 k 个链表节点
   >
   > [面试题 02.02. 返回倒数第 k 个节点](https://leetcode-cn.com/problems/kth-node-from-end-of-list-lcci/)
   >
   > ```java
   > // 方法2：双指针，指针1先走k，然后指针1,2一起走，指针1走到末尾后，返回指针2，指针2的距离即为：size-k
   > public int kthToLast(ListNode head, int k) {
   > 
   >        if(head == null){
   >            return 0;
   >        }
   > 
   >        ListNode node1 = head;
   >        ListNode node2 = head;
   >        // 指针1先走k步
   >        while(k-- != 0){
   >            node1 = node1.next;
   >        }
   > 
   >        while(node1 != null){
   >            node2 = node2.next;
   >            node1 = node1.next;
   >        }
   > 
   >        return node2.val;
   > }
   > ```

#### 左右指针

1. [二分查找](在上面)

2. 两数之和

   > [167. 两数之和 II - 输入有序数组](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/)
   >
   > ```java
   > // 二分法来一波
   > public int[] twoSum(int[] numbers, int target) {
   > 
   >        int left = 0; int right = numbers.length - 1;
   > 
   >        while(left < right){
   >            int sum = numbers[left] + numbers[right];
   > 
   >            if(sum == target){
   >                return new int[]{left+1, right+1};
   >            }else if(sum < target){
   >                left ++;
   >            }else if(sum > target){
   >                right --;
   >            }
   >        }
   > 
   >        return new int[]{-1, -1};
   > }
   > ```

3. 反转数组

   ```java
   void reverse(int[] nums) {
       int left = 0;
       int right = nums.length - 1;
       while (left < right) {
           // swap(nums[left], nums[right])
           int temp = nums[left];
           nums[left] = nums[right];
           nums[right] = temp;
           left++; right--;
       }
   }
   ```

4. 滑动窗口算法

   > 直接见下面

### 题目

#### [26. 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

```java
// 快慢指针去重
public int removeDuplicates(int[] nums) {

    if(nums.length <= 1){
        return nums.length;
    }

    int slow = 0;
    int fast = 1;

    while(fast < nums.length){
        if(nums[fast] != nums[slow]){
            nums[++slow] = nums[fast];
        }
        fast++;
    }
    return slow+1;
}
```

#### [83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

```java
// 双指针解题
public ListNode deleteDuplicates(ListNode head) {

    if(head == null){
        return head;
    }

    ListNode ans = new ListNode(-1);
    ans.next = head;
	
    // 双指针，一个走在前一个走在后
    ListNode fast = head.next;
    ListNode slow = head;

    while(fast != null){

        if(fast.val != slow.val){
            slow.next = fast;
            slow = slow.next;
        }
        fast = fast.next;
    }
    // 这里必须断开连接
    slow.next = null;
    return ans.next;
}

// 直接解题：
public ListNode deleteDuplicates(ListNode head) {
	
    if(head == null || head.next == null){
        return head;
    }
	
    // 虚拟节点
    ListNode ans = new ListNode(-1);
    ans.next = head;

    while(head.next != null){
		// head的值等于下一个节点的值，则跳过下一个节点
        if(head.val == head.next.val){
            head.next = head.next.next;
        }else{
            head = head.next;
        }
    }

    return ans.next;
}
```

#### [27. 移除元素](https://leetcode-cn.com/problems/remove-element/)

```java
public int removeElement(int[] nums, int val) {

    if(nums.length == 0){
        return nums.length;
    }

    int slow = 0;
    int fast = 0;

    while(fast < nums.length){

        if(nums[fast] != val){
            nums[slow] = nums[fast];
            slow ++;
        }
        fast ++;
    }

    return slow;
}
```

#### [283. 移动零](https://leetcode-cn.com/problems/move-zeroes/)

```java
public void moveZeroes(int[] nums) {

    if(nums.length == 0){
        return;
    }

    int fast = 0;
    int slow = 0;

    while(fast < nums.length){
        if(nums[fast] != 0){
            nums[slow] = nums[fast];
            slow++;
        }
        fast ++;
    }

    while(slow < nums.length){
        nums[slow] = 0;
        slow++;
    }
}
```

#### [28. 实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

```java
// 这题的最优解法当然是KMP解法，但是...直接暴力指针解了，后续如果学习相关专题会再去做下的吧
public int strStr(String haystack, String needle) {
	
    // 分别是haystack与needle的指针
    int idx1 = 0, idx2 = 0;
	// 长度为0直接返回0
    if(needle.length() == 0){
        return 0;
    }
	
    // 开始暴力匹配
    for(int i=0; i<=haystack.length()-needle.length(); i++){
		// 指向字符1
        idx1 = i;
        while(haystack.charAt(idx1) == needle.charAt(idx2)){
			// 字符相等，则同时向后移动一位
            idx1 ++;
            idx2 ++;
			// needle匹配完了，返回相应结果
            if(idx2 == needle.length()){
                return i;
            }
        }
        // 匹配失败，needle的指针清0，考虑到后续匹配
        idx2 = 0;
    }
	// 匹配失败直接返回-1
    return -1;
}
```

#### [面试题 10.01. 合并排序的数组](https://leetcode-cn.com/problems/sorted-merge-lcci/)

```java
public void merge(int[] A, int m, int[] B, int n) {

    int idx = A.length;
    int idx1 = m-1;
    int idx2 = n-1;

    while(--idx >= 0){

        if(idx1 < 0){
            A[idx] = B[idx2--];
        }else if(idx2 < 0){
            A[idx] = A[idx1--];
        }
        else if(A[idx1] >= B[idx2]){
            A[idx] = A[idx1--];
        }else{
            A[idx] = B[idx2--];
        }
    }
}
```

## 滑动窗口

### 总结

滑动窗口算法：维护一个窗口，不断滑动，然后更新答案

#### 框架

```java
/* 滑动窗口算法框架 */
void slidingWindow(string s, string t) {
    
    Map<Character, Integer> need = new HashMap<>();
    Map<Character, Integer> window = new HashMap<>();
    for(char c: t.toCharArray()){
        need.put(c, need.getOrDefault(c, 0) + 1);
    }

    int left = 0, right = 0;
    int valid = 0; 
    while (right < s.length()) {
        // c 是将移入窗口的字符
        char c = s.charAt(right);
        // 右移窗口
        right++;
        // 进行窗口内数据的一系列更新 [mark：和下面一个mark处的代码基本一致]
        ...

        /*** debug 输出的位置 ***/
        System.out.println("window: [" + left + "," + right +")");
        /********************/

        // 判断左侧窗口是否要收缩
        while (window needs shrink) {
            // d 是将移出窗口的字符
            char d = s.charAt(left);
            // 左移窗口
            left++;
            // 进行窗口内数据的一系列更新 [mark：和上面一个mark处的代码基本一致] 
            ...
        }
    }
}
```

**使用时只需要思考以下四个问题**：

1. 当移动`right`扩大窗口，即加入字符时，应该更新哪些数据？

2. 什么条件下，窗口应该暂停扩大，开始移动`left`缩小窗口？

3. 当移动`left`缩小窗口，即移出字符时，应该更新哪些数据？
4. 我们要的结果应该在扩大窗口时还是缩小窗口时进行更新？

### 题目

套用上述模版给出两个题目，好好体会

#### [76. 最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)

```java
public String minWindow(String s, String t) {

    Map<Character, Integer> need = new HashMap<>();
    Map<Character, Integer> window = new HashMap<>();
    for(char c: t.toCharArray()){
        need.put(c, need.getOrDefault(c, 0) + 1);
    }

    int left = 0, right = 0;
    int valid = 0; 

    // 记录最小覆盖子串的起始索引及长度
    int start = 0, len = s.length()+1;
    while(right < s.length()){
        // c 是将移入窗口的字符
        char c = s.charAt(right);
        // 右移窗口
        right ++;
        // 进行窗口内数据的一系列更新
        if(need.containsKey(c)){
            window.put(c, window.getOrDefault(c, 0) + 1);
            if(window.get(c).equals(need.get(c))){
                valid++;
            }
        }

        // 判断左侧窗口是否要收缩
        while(valid == need.size()){
            // 在这里更新最小覆盖子串
            if(right - left < len){
                start = left;
                len = right - left;
            }

            // d 是将移出窗口的字符
            char d = s.charAt(left);
            left ++;
            // 进行窗口内数据的一系列更新
            if(need.containsKey(d)){
                if(window.get(d).equals(need.get(d))){
                    valid --;
                }
                window.put(d, window.get(d)-1);
            }
        }
    }

    return len == s.length()+1 ? "" : s.substring(start, start+len);
}
```

#### [567. 字符串的排列](https://leetcode-cn.com/problems/permutation-in-string/)

```java
public boolean checkInclusion(String s1, String s2) {
    Map<Character, Integer> need = new HashMap<>();
    Map<Character, Integer> window = new HashMap<>();
    for(char c: s1.toCharArray()){
        need.put(c, need.getOrDefault(c, 0) + 1);
    }

    int left = 0, right = 0;
    int valid = 0;

    while (right < s2.length()) {
        // c 是将移入窗口的字符
        char c = s2.charAt(right);
        // 右移窗口
        right++;
        // 进行窗口内数据的一系列更新
        if(need.containsKey(c)){
            window.put(c, window.getOrDefault(c, 0) + 1);
            if(window.get(c).equals(need.get(c))){
                valid++;
            }
        }
        
        while(valid == need.size()){
			// 当s2的子串长度和s1相同，且包含的字符都一样，则肯定存在这个排列情况
            if(right - left == s1.length()){
                return true;
            }
			// 待移出的字符
            char d = s2.charAt(left);
            // 左窗口移动
            left ++;
            // 进行窗口内数据的更新
            if(need.containsKey(d)){
                if(window.get(d).equals(need.get(d))){
                    valid--;
                }
                window.put(d, window.get(d) - 1);
            }
        }
    }

    return false;
}
```

#### [438. 找到字符串中所有字母异位词](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)

```java
public List<Integer> findAnagrams(String s, String p) {

    Map<Character, Integer> need = new HashMap<>();
    Map<Character, Integer> window = new HashMap<>();
    for(char c: p.toCharArray()){
        need.put(c, need.getOrDefault(c, 0) + 1);
    }

    int left = 0, right = 0;
    int valid = 0; 
    List<Integer> res = new LinkedList<>();

    while (right < s.length()) {
        // c 是将移入窗口的字符
        char c = s.charAt(right);
        // 右移窗口
        right++;
        // 进行窗口内数据的一系列更新
        if(need.containsKey(c)){
            window.put(c, window.getOrDefault(c, 0) + 1);
            if(window.get(c).equals(need.get(c))){
                valid++;
            }
        }

        // 一种判断方式：判断左侧窗口是否要收缩
        while ((right-left) >= p.length()) {
			
            // 第一次进入时长度刚好相等，然后可以做这样的判断
            if(valid == need.size()){
                res.add(left);
            }
            // d 是将移出窗口的字符
            char d = s.charAt(left);
            // 左移窗口
            left++;
            // 进行窗口内数据的一系列更新
            if(need.containsKey(d)){
                if(window.get(d).equals(need.get(d))){
                    valid--;
                }
                window.put(d, window.get(d) - 1);
            }
        }
        
        // 另一种判断方式：更加标准一些，后续单独做时都是采用的这种方式
        /**
        while(valid == need.size()){

            if(p.length() == right - left){
                res.add(left);
            }

            char d = s.charAt(left);
            left ++;

            if(need.containsKey(d)){
                if(window.get(d).equals(need.get(d))){
                    valid--;
                }
                window.put(d, window.get(d) - 1);
            }
        }
        */
    }
    return res;
}
```

#### [3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

```java
// 之前做的...
public int lengthOfLongestSubstring(String s) {

    Map<Character, Integer> window = new HashMap<>();
    int left = 0, right = 0;
    int len = 0;

    while (right < s.length()) {
        // c 是将移入窗口的字符
        char c = s.charAt(right);

        // 进行窗口内数据的一系列更新
        if(window.getOrDefault(c, 0) < 1){
            // 没有重复的时候的情况：右移窗口
            right++;
            window.put(c, window.getOrDefault(c, 0)+1);
            // 窗口扩大，更新子串长度
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

// 第二遍做 nice 更加清晰合理 :-)
public int lengthOfLongestSubstring(String s) {

    Set<Character> window = new HashSet();

    int left = 0, right = 0, len = 0;

    while(right < s.length()){
		// c将移入窗口
        char c = s.charAt(right);
        right ++;
		// 当窗口中已经存在c了，则窗口要缩小，然后缩小后再判断是否存在c
        while(window.contains(c)){
            // 待删除的字符，删除一个窗口减小
            char d = s.charAt(left);
            left++;
			// 将删除的字符移出窗口
            window.remove(d);
        }
		// 窗口中不存在c了，再把c放进窗口中去
        window.add(c);
        // 计算最长长度
        len = Math.max(len, right - left);
    }

    return len;
}
```

#### [239. 滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)

```java
// 这题感觉不应该出现在这里...没有用上面的模版，而是通过维护一个双向有序队列来解决的
public int[] maxSlidingWindow(int[] nums, int k) {

    // 双向队列，队伍首为当前窗口最大值，且队列按照从大到小排列,队列中存的是数组的下标
    Deque<Integer> deque = new LinkedList<>();

    // 最终结果
    int[] res = new int[nums.length - k + 1];

    // 开始遍历数组
    for(int i=0; i<nums.length; i++){

        // 保证队列的特性
        while(!deque.isEmpty() && nums[i] >= nums[deque.peekLast()]){
            deque.pollLast();
        }
        // 把值添加到队列末
        deque.offerLast(i);
        // 判断当前队列中头部元素是否有效
        if(deque.peekFirst() <= i-k){
            deque.pollFirst();
        }
        // 当窗口长度为k时 保存当前窗口中最大值
        if(i+1>=k){
            res[i-k+1] = nums[deque.peekFirst()];
        }
    }

    return res;
}
```

#### [1052. 爱生气的书店老板](https://leetcode-cn.com/problems/grumpy-bookstore-owner/)

```java
public int maxSatisfied(int[] customers, int[] grumpy, int minutes) {

    int n = customers.length;
    int left = 0, right = 0;
    int res = 0;
    // 在minutes中不满意的顾客与最大不满意的情况
    int angry = 0, maxAngry = 0;

    while(right < n){

        if(grumpy[right] == 1){
            // +不满意的顾客
            angry += customers[right];
        }
        // 窗口移动
        right ++;

        // 当满足x时间了
        if(right-left == minutes){
            if(maxAngry < angry){
                maxAngry = angry;
            }
            // 移动左窗口的话要减去那时候不满意的顾客
            if(grumpy[left] == 1){
                angry -= customers[left];
            }
            left++;
        }
    }

    // 最多不满意的顾客时选择不生气
    res += maxAngry;
    // 然后加上所有满意的客户数
    for(int i=0; i<n; i++){
        if(grumpy[i] == 0){
            res += customers[i];
        }
    }

    return res;
}
```

#### [1208. 尽可能使字符串相等](https://leetcode-cn.com/problems/get-equal-substrings-within-budget/)

```java
public int equalSubstring(String s, String t, int maxCost) {

    int n = s.length();
    // 1. 计算所有开销
    int[] costs = new int[n];

    for(int i=0; i<n; i++){
        costs[i] = Math.abs(s.charAt(i)-t.charAt(i));
    }

    // 2.遍历costs数组，计算cost和小于maxCost的最长子串
    int left = 0, right = 0;
    int cost = 0, res = 0;
    while(right < n){

        // 移动右窗口，+cost
        cost += costs[right];
        right++;

        if(cost <= maxCost){
            if(res < right-left){
                res = right - left;
            }
        }else{
            while(cost > maxCost){
                // 移动左窗口-cost
                cost -= costs[left];
                left++;
            }
        }
    }

    return res;
}
```

#### [1004. 最大连续1的个数 III](https://leetcode-cn.com/problems/max-consecutive-ones-iii/)

```java
public int longestOnes(int[] nums, int k) {

    int left = 0, right = 0;
    int used = 0, res = 0;

    while(right < nums.length){

        if(nums[right] == 1){
            // 右边指向的值为1，则可以继续滑动右窗口
            right ++;
            res = Math.max(res, right - left);
        }else{
            // 右边为0了
            if(used < k){
                // uesd使用的修改机会还有
                used++;
                right++;
                res = Math.max(res, right - left);
            }else{
                // 已经没有修改机会了，右边的0无法再修改为1，只能收缩左边窗口
                if(nums[left] == 0){
                    // 发现一个0了，则可以恢复一次使用
                    used--;
                }
                // 收缩左边窗口
                left++;
            }
        }
    }
    return res;
}
```

## 前缀和/差分数组

> 参考子[论那些小而美的算法技巧：差分数组/前缀和](https://mp.weixin.qq.com/s?__biz=MzAxODQxMDM0Mw==&mid=2247487011&idx=1&sn=5e2b00c1c736fd7afbf3ed35edc4aeec&chksm=9bd7f02baca0793d569a9633cc14117e708ccc9eb41b7f0add430ea78f22e4f2443f421c6841&scene=21#wechat_redirect)

### 前缀和

前缀和数组：一个数组的前n个元素的和而组成的数组，**主要适用的场景是原始数组不会被修改的情况下，频繁查询某个区间的累加和**。

```java
// 例如有一个数组 nums:[1,1,1] 其前缀和为：[0,1,2,3]
int n = nums.length;
// 前缀和数组
int[] preSum = new int[n + 1];
preSum[0] = 0;
for (int i = 0; i < n; i++)
    preSum[i + 1] = preSum[i] + nums[i];
```

#### [560. 和为K的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k/)

```java
// 暴力前缀和
public int subarraySum(int[] nums, int k) {

    int n = nums.length;
    // 前缀和数组
    int[] preSum = new int[n+1];
    preSum[0] = 0;

    for(int i=0; i<n; i++){
        preSum[i+1] = preSum[i] + nums[i];
    }

    int ans = 0;
    // 穷举所有情况
    for(int i=1; i<=n; i++){
        for(int j=0; j<i; j++){
            if(preSum[i] - preSum[j] == k){
                ans++;
            }
        }
    }
    return ans;
}

// 优化：k == preSum[j+1]- preSum[i] ---> preSum[j+1] - k = preSum[i]
public int subarraySum(int[] nums, int k) {

    int n = nums.length;
    // map: 前缀和 -> 该前缀和出现的次数
    Map<Integer, Integer> preSum = new HashMap<>();
    // base case
    preSum.put(0, 1);

    int ans=0;
    int sum_0_i = 0;  // 从0到i的和
    for(int i=0; i<n; i++){
        // 每个元素对应一个“前缀和”
        sum_0_i += nums[i];
        // 根据当前“前缀和”，在 map 中寻找「与之相减 == k」的历史前缀和
        // [0...i] - [0...j]  = k  --> [j...i]是满足的
        int sum_0_j = sum_0_i - k;
        if(preSum.containsKey(sum_0_j)){
            ans += preSum.get(sum_0_j);
        }
        preSum.put(sum_0_i, preSum.getOrDefault(sum_0_i, 0) + 1);
    }
    return ans;
}
```

#### [1371. 每个元音包含偶数次的最长子字符串](https://leetcode-cn.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/)

```java
// 这些元音都是出现偶数次
private static String VOWELS = "aeiou";

public int findTheLongestSubstring(String s) {

    Map<Integer, Integer> map = new HashMap<>();

    int size = s.length();
    // 00000：表示都是偶数 ~ 11111：表示都是奇数  aeiou对应着每一个位
    int state = 0; 
    int maxSize = 0;

    // key:当前的状态state  value：在数组中的index
    map.put(0, -1);

    // 遍历所有字符
    for(int i=0; i<size; i++){
        // 遍历需要变成偶数对的VOWELS
        for(int k=0; k<VOWELS.length(); k++){
            if(s.charAt(i) == VOWELS.charAt(k)){
                // 这个状态就要修改了，通过异或操作
                // 1：出现次数为奇数，此时又出现，则变为0  
                // 0: 出现次数为奇数，此时又出现，则变为1
                state ^= (1 << (VOWELS.length() - k - 1));
                break;  // 已经和一个相等了，后续当然不会再相等了
            }
        }

        // 找到状态一致的就是说  偶数-偶数   奇数-奇数
        if(map.containsKey(state)){
            maxSize = Math.max(maxSize, i - map.get(state));
        }else{
            // 要保证相同的状态index最小，则len最长
            map.put(state, i);
        }
    }

    return maxSize;
}
```

#### [1248. 统计「优美子数组」](https://leetcode-cn.com/problems/count-number-of-nice-subarrays/)

```java
// 暴力前缀和：超时
public int numberOfSubarrays(int[] nums, int k) {

    int n = nums.length;
    int res = 0;
    // 用于记录奇数的次数
    int[] preSum = new int[n+1];

    preSum[0] = 0;
    for(int i=0; i<n; i++){
        preSum[i+1] = preSum[i] + (nums[i] % 2 == 1 ? 1 : 0);
    }

    for(int i=k; i<=n; i++){
        for(int j=0; j <= i-k; j++){
            if(preSum[i] - preSum[j] == k){
                res ++;
            }
        }
    }
    return res;
}

// 优化后的前缀和
public int numberOfSubarrays(int[] nums, int k) {

    int n = nums.length;
    int res = 0;
    // 前缀和 key:前缀和(奇数的个数)  value：有这个前缀和的，下标从0开始的子数组个数
    Map<Integer, Integer> preSum = new HashMap<>();
    // 前缀和为0，的子树个数默认为1，空数组
    preSum.put(0, 1); 
    int sum = 0;
    for(int i=0; i<n; i++){
        sum += nums[i] % 2 == 1 ? 1 : 0;
        // 表示当前奇数个数为sum，从0下标开始到i且前缀和为sum的个数
        preSum.put(sum, preSum.getOrDefault(sum, 0)+1);  
        if(sum >= k){
            res += preSum.get(sum - k);
        }
    }

    return res;
}

// 再次优化后的前缀和
public int numberOfSubarrays(int[] nums, int k) {

    int n = nums.length;
    int res = 0;
    // 用数组来代替hashmap，提高效率
    int[] preSum = new int[n+1];
    preSum[0] = 1;
    int sum = 0;
    for(int i=0; i<n; i++){
        sum += nums[i] % 2 == 1 ? 1 : 0;
        preSum[sum]++;
        if(sum >= k){
            res += preSum[sum - k];
        }
    }

    return res;
}
```

### 差分数组

差分数组：就是数组中一个元素和前一个元素的差

```java
int[] diff = new int[nums.length];
// 构造差分数组
diff[0] = nums[0];
for (int i = 1; i < nums.length; i++) {
    diff[i] = nums[i] - nums[i - 1];
}

// 根据差分数组返回到原数组
int[] res = new int[diff.length];
// 根据差分数组构造结果数组
res[0] = diff[0];
for (int i = 1; i < diff.length; i++) {
    res[i] = res[i - 1] + diff[i];
}
```

**这样构造差分数组`diff`，就可以快速进行区间增减的操作**，如果你想对区间`nums[i..j]`的元素全部加 3，那么只需要让`diff[i] += 3`，然后再让`diff[j+1] -= 3`即可：

可以把差分数组抽象成一个类

```java
class Difference {
    // 差分数组
    private int[] diff;

    public Difference(int[] nums) {
        assert nums.length > 0;
        diff = new int[nums.length];
        // 构造差分数组
        diff[0] = nums[0];
        for (int i = 1; i < nums.length; i++) {
            diff[i] = nums[i] - nums[i - 1];
        }
    }

    /* 给闭区间 [i,j] 增加 val（可以是负数）*/
    public void increment(int i, int j, int val) {
        diff[i] += val;
        if (j + 1 < diff.length) {
            diff[j + 1] -= val;
        }
    }

    public int[] result() {
        int[] res = new int[diff.length];
        // 根据差分数组构造结果数组
        res[0] = diff[0];
        for (int i = 1; i < diff.length; i++) {
            res[i] = res[i - 1] + diff[i];
        }
        return res;
    }
}
```

#### [1109. 航班预订统计](https://leetcode-cn.com/problems/corporate-flight-bookings/)

```java
// 暴力解题：cost 1513 ms
public int[] corpFlightBookings(int[][] bookings, int n) {

    int[] res = new int[n];

    for(int i=0; i<bookings.length; i++){
        int j = bookings[i][0];
        int k = bookings[i][1];
        for(int len=j-1; len <=k-1; len++){
            res[len] += bookings[i][2];
        }

    }
    return res;
}

// 差分数组解题：cost 3ms
public int[] corpFlightBookings(int[][] bookings, int n) {

    int[] res = new int[n];
    // 差分数组
    int[] diff = new int[n];

    for(int[] booking: bookings){
        int i = booking[0] - 1;
        int j = booking[1] - 1;
        int val = booking[2];
        // 对区间 nums[i..j] 增加 val
        diff[i] += val;
        if (j + 1 < diff.length) {
            diff[j + 1] -= val;
        }
    }

    // 根据差分数组构造结果数组
    res[0] = diff[0];
    for (int i = 1; i < diff.length; i++) {
        res[i] = res[i - 1] + diff[i];
    }
    return res;
}
```

