---
title: 死磕链表
date: 2020-02-15 20:42:11
tags: 
   - 链表
   - 死磕系列
categories: LeetCode
thumbnail: https://i.loli.net/2020/03/17/k74FDqg1EKwXxTp.png
---

## 前言

LeetCode死磕系列二： 链表

<!--more-->

链表作为基础的数据结构，将链表的操作掌握了，其他的相关结构与算法，理论上来说为题不大了。LeetCode中， 链表的操作是有迹可循的， 当然也可当场理解，然后画图，找解法（奈何我太笨，编码功底还不够），反正链表的所有操作都离不开基础的`CRUD`, 还需要注意的是，当涉及到头结点的操作的时候，最好设置一个`dummy`节点指向头结点，这样方便统一处理。

刷了LeetCode中所有链表有关的题目（34题免费题目）做了如下总结

### CRUD

#### 增

需要注意构造新链表时的`头插法` 和`尾插法`

`头插法`： 逆序

`尾插法`： 正序

#### 删

**根据前驱删除**

```java
pre.next = pre.next.next;
```

**根据当前节点删除**

```java
ListNode next = cur.next;
cur.val = next.val;
cur.next = next.next;
```

#### 改

#### 查

## 链表的常用操作

1. 两两之间的交换
2. 多节点的交换
3. 反转
4. 环

## LeetCode 链表题目整理

* 2 Add Two Numbers
* 445 Add Two Numbers II

## 题解

### 2. [ Add Two Numbers](https://leetcode.com/problems/add-two-numbers)  

**Example:**

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

思路：

链表的反转序列代表一个整数，然后整数的相加之后，结果用`个位-->十位--->百位`的顺序表示。所以可以使用两个队列存储两个链表，然后开始依次弹出，进行相加，使用`尾插法`构造输出链表

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    Queue<ListNode> queue1 = list2Queue(l1);
    Queue<ListNode> queue2 = list2Queue(l2);
    // 应题目要求 使用尾插发
    ListNode dummy = new ListNode(-1);
    ListNode real = dummy;
    int carry = 0;
    while (!queue1.isEmpty() || !queue2.isEmpty()) {
        int p1 = queue1.isEmpty() ? 0 : queue1.poll().val;
        int p2 = queue2.isEmpty() ? 0 : queue2.poll().val;
        int sum = p1 + p2 + carry;
        int i = sum % 10;
        // 尾插法
        ListNode cur = new ListNode(i);
        real.next = cur;
        real = cur;
        carry = sum / 10;
    }
    if (carry > 0) {
        ListNode node = new ListNode(carry);
        real.next = node;
        real = node;
    }
    return dummy.next;
}
public Queue<ListNode> list2Queue(ListNode node) {
    Queue<ListNode> queue = new LinkedList<>();
    while (node != null) {
        queue.offer(node);
        node = node.next;
    }
    return queue;
}
```



### 445 [Add Two Numbers II](https://leetcode.com/problems/add-two-numbers-ii/)

思路

链表代表一个整数，两整数相加之后，结果用`最高位--->....--->十位--->个位`顺序表示。所以可以使用两个栈存储两个链表，然后开始依次弹出，进行相加，使用头插法构造输出链表。

```java
public ListNode addTwoNumbersII(ListNode l1, ListNode l2) {
    if (l1 == null && l2 == null) {
        return null;
    }
    Stack<ListNode> s1 = new Stack();
    while(l1 != null) {
        s1.push(l1);
        l1 = l1.next;
    }
    Stack<ListNode> s2 = new Stack();
    while(l2 != null) {
        s2.push(l2);
        l2 = l2.next;
    }
    int carry = 0;
    ListNode resNode = null;
    while (!s1.isEmpty() || !s2.isEmpty()) {
        int n1 = s1.isEmpty() ? 0 : s1.pop().val;
        int n2 = s2.isEmpty() ? 0 : s2.pop().val;
        int sum = n1 + n2 + carry;
        ListNode n = new ListNode(sum % 10);
        // 头插法
        n.next = resNode;
        resNode = n;
        
        carry = sum / 10;
    }
    // 最高位
    if (carry > 0) {
        ListNode n = new ListNode(carry);
        n.next = resNode;
        resNode = n;
    }
    return resNode;
}
```



链表表示整数，进行加法计算，这是两个链表的操作，没有涉及链表的增删改查，实现方式数组中的`Add Two Numbers`同理。但是需要注意题目要求，上述两个题目思路是一样的，但是需要注意题目说明，**给定的链表是如何代表一个整数的，然后输出结果是如何表示的**，（因为我们的算法是从个位开始计算的）

#### 正序表示整数 && 从个位依次输出

使用栈存储，尾插法构造结果

#### 逆序表示整数 && 从最高位依次输出

使用队列存储，头插法构造结果

### 19. [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list)  

**查、删**

倒数第n个节点删除

先获取第n个节点的前驱节点，然后按照`前驱结点删除法删除`即可

```java
public static ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(-1);
    dummy.next = head;
    ListNode preNode = getPreNode(dummy, n);
    preNode.next = preNode.next.next;
    return head;
}
public static ListNode getPreNode(ListNode head, int n){
    int count = 0;
    ListNode p = head;
    while (p != null){
        count ++;
        p = p.next;
    }
    p = head;
    while ( count-n-1 > 0){
        p = p.next;
        count --;
    }
    return p;
}

// 也可以直接使用第二种删除方法
public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(-1);
        dummy.next = head;
        ListNode fast = dummy;
        ListNode slow = dummy;
        while(n > 0){
            fast = fast.next;
            n--;
        }
        while(fast.next != null){
            fast = fast.next;
            slow = slow.next;
        }
        
        slow.next = slow.next.next;
 
        return dummy.next;
}
```



### 21. [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists)  

增、查

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(-1);
    ListNode real = dummy;
    while (l1 != null && l2 != null){
        if(l1.val <= l2.val){
            real.next = l1;
            l1 = l1.next;
        }else {
            real.next = l2;
            l2 = l2.next;
        }
        real= real.next;
    }
    if (l1 != null){
        real.next = l1;
    }
    if (l2 != null) {
        real.next = l2;
    }
    return dummy.next;
}
```



### 23. [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists)  

增，查

#### 方法一

多链表之间按照递增顺序进行整合。（根据上一题的思路，先两两合并，最后合并成一个链表）

```java
public ListNode mergeKLists(ListNode[] lists) {
        if (lists == null) {
            return null;
        }
        int length = lists.length;
        while (length > 1) {
            int k = (length + 1) / 2;
            for (int i = 0; i < (length / 2); i++) {
                // mergeTwo为21题函数
                lists[i]  = mergeTwo(lists[i], lists[i+k]);
            }
            length = k;
        }
        return lists[0];
    }
```

#### 方法二

使用小根堆

```java
public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> heap = new PriorityQueue<>(new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                return o1.val - o2.val;
            }
        });
        // 尾插法
        ListNode dummy = new ListNode(-1);
        ListNode real = dummy;
        for (ListNode node : lists) {
            if (node != null) {
                heap.offer(node);
            }
        }
        while (!heap.isEmpty()) {
            real.next = heap.poll();
            real = real.next;
            if (real.next != null) {
                heap.offer(real.next);
            }
        }
        return dummy.next;
    }
```



### 24. [ Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs)  

**改**

两两交换，

两个节点的交换， 必然牵引到四个指针的改变

具体步骤见图：

![链表.png](https://i.loli.net/2020/03/18/J98jkA5oNBZEtpm.png)

```java
public static ListNode swapPairs(ListNode head){
        if(head == null){
            return head;
        }
        ListNode dummy = new ListNode(-1);
        dummy.next = head;
        ListNode pre = dummy;
        while (head!=null && head.next!=null){
            ListNode temp = head.next;
            head.next = temp.next;
            temp.next = pre.next;
            pre.next = temp;
            pre = head;
            head = head.next;
        }
        return dummy.next;
    }
```

### 25.  [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group) 

思路和上一题一样。但是需要注意`pre`的`cur`



### 92. [Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii)  

和上述同理

```java
public static ListNode reverseBetween(ListNode head, int m, int n) {
        if (head == null || n - m == 0) {
            return head;
        }
        ListNode dummy = new ListNode(-1);
        dummy.next = head;
        ListNode pre  = dummy;
        for (int i = 0; i < m-1; i++) {
            pre = pre.next;
        }
        ListNode cur = pre.next;
        for (int i = m; i < n; i++) {
            ListNode temp = cur.next;
            cur.next = temp.next;
            temp.next = pre.next;
            pre.next = temp;
        }
        return dummy.next;

    }
```

### 206. [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list)  

头插法解决

```````java
public ListNode reverseList(ListNode head) {
        ListNode dummy = new ListNode(-1);
        while(head != null){
            ListNode curr = head.next;
            head.next = dummy.next;
            dummy.next = head;
            head = curr;
        }
        return dummy.next;
        
    }
```````

### 61. [Rotate List](https://leetcode.com/problems/rotate-list)

**查**

快慢指针法，`fast`指针先走`k`步，之后`slow`和`fast`一起走，`fast`到链表尾部，然后将链表头尾相连。此时`slow`指向新的头结点的前驱节点，然后重新设置头指针，断开链表即可。

```java
// 快慢指针
public static ListNode roateRightII(ListNode head, int k){
    if(head == null){
        return null;
    }
    int length = getLength(head);
    k %= length;
    ListNode endNode = getEndNode(head);
    ListNode fast = head, slow = head;
    while (k > 0){
        fast = fast.next;
        k--;
    }
    while (fast.next != null){
        fast = fast.next;
        slow = slow.next;
    }
    // 构成环
    fast.next = head;
    // 重新设置
    head = slow.next;
    slow.next = null;
    return head;
}
```



### 83. [Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list)  

**删**

删除有重复的节点

使用`前驱节点`删除法，所以需要一个`pre`指针，和`cur`指针。

```java
public ListNode deleteDuplicates(ListNode head) {
    ListNode pre = head;
    ListNode cur;
    while (pre.next != null){
        cur = pre.next;
        // 因为是和cur的前驱比较， 所以不用判断cur.next!=null, 而是使用cur!=null
        while (cur!=null && cur.val == pre.val){
            cur = cur.next;
        }
        if(pre.next != cur){
            pre.next = cur;
        }
        else {
            pre = pre.next;
        }
    }
    return head;
}
```



### 82.  [Remove Duplicates from Sorted List II](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii)  

**删**

删光重复的节点

使用`前驱节点`删除法，所以需要一个`pre`指针，和`cur`指针。

可能存在头结点会变换, 所以设置一个空的头结点

```java
public ListNode deleteDuplicates(ListNode head) {
    if (head == null) {
        return null;
    }
    ListNode dummy = new ListNode(-1);
    dummy.next = head;
    ListNode cur = null;
    ListNode pre = dummy;
    while (pre.next != null) {
        cur = pre.next;
        while (cur.next != null && cur.next.val == cur.val) {
            cur = cur.next;
        }
        if(cur != pre.next){
            pre.next = cur.next;
        }
        else {
            pre = pre.next;
        }
    }
    return dummy.next;
}
```

### 86. [ Partition List](https://leetcode.com/problems/partition-list)  

### 109. [Convert Sorted List to Binary Search Tree](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree) 

### 138. [ Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer)  

### 141. [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle)  

### 142. [ Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii)  

### 143. [Reorder List](https://leetcode.com/problems/reorder-list)  

### 147. [Insertion Sort List](https://leetcode.com/problems/insertion-sort-list)  

### 148. [Sort List](https://leetcode.com/problems/sort-list)  

### 160.  [Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists)  

解法1：暴力求解

解法2： 构造环

### 203. [Remove Linked List Elements](https://leetcode.com/problems/remove-linked-list-elements)  

### 234. [Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list)  

### 237. [Delete Node in a Linked List](https://leetcode.com/problems/delete-node-in-a-linked-list)  

### 328. [Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list)  

### 369. [ Plus One Linked List](https://leetcode.com/problems/plus-one-linked-list)  

### 379. [Design Phone Directory](https://leetcode.com/problems/design-phone-directory)  

### 426. [ Convert Binary Search Tree to Sorted Doubly Linked List](https://leetcode.com/problems/convert-binary-search-tree-to-sorted-doubly-linked-list)  

### 430. [ Flatten a Multilevel Doubly Linked List](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list)  

### 445. [ Add Two Numbers II](https://leetcode.com/problems/add-two-numbers-ii)  

### 707. [ Design Linked List](https://leetcode.com/problems/design-linked-list)  

### 708. [Insert into a Sorted Circular Linked List](https://leetcode.com/problems/insert-into-a-sorted-circular-linked-list)  

### 725. [ Split Linked List in Parts](https://leetcode.com/problems/split-linked-list-in-parts)  

### 817. [Linked List Components](https://leetcode.com/problems/linked-list-components)  

### 876. [Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list)  

### 1019. [ Next Greater Node In Linked List](https://leetcode.com/problems/next-greater-node-in-linked-list)  

### 1171. [Remove Zero Sum Consecutive Nodes from Linked List](https://leetcode.com/problems/remove-zero-sum-consecutive-nodes-from-linked-list)  

删

快慢指针

其实是和计算一个链表中是否存在和为n的子链表是一样的意思。

slow表示当前的元素，使用fast依次遍历其后续的元素，同时值相加，如果等于0，则`slow.next ` = `fast.next`,如果`fast`遍历完了所有的元素了，没有等于0， 则`slow`移动到下一个元素 

### 1290. [Convert Binary Number in a Linked List to Integer](https://leetcode.com/problems/convert-binary-number-in-a-linked-list-to-integer)  

查



### 1369. [ Linked List in Binary Tree](https://leetcode.com/problems/linked-list-in-binary-tree)  


