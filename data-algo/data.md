> 我只能说，那句话真的没说错，*程序 = 数据结构 + 算法*。原来我也认为这些不重要，后来才发现这些恰恰是最最最重要的！！！ 
> --- 所有的东西都是数据结构，无非是以一种特殊的方式进行了处理

# 链表
```js
function ListNode(val) {
	this.val = val;
	this.next = null;
}
```

## 判断链表是否相交：
[leetcode:160](https://leetcode.cn/problems/intersection-of-two-linked-lists/?envType=study-plan-v2&envId=top-100-liked)
链表 A 、链表B
*双指针* -> 两个指针分别从 headA 、 headB 出发，如果不相等，移动指针。如果其中一个指针遍历到链表结尾，那么从另一条链表重新遍历

```js
const getIntersectionNode = (headA, headB) => {
	const a = headA;
	const b = headB;
	while (a !== b) {
		a = a !== null ? a.next : headB;
		b = b !== null ? b.next : headA;
	}
	return a;
};
```

## 反转链表 
[leetcode:206](https://leetcode.cn/problems/reverse-linked-list/?envType=study-plan-v2&envId=top-100-liked)
*双指针* -> 将所有的 next 指向全部反转
*pre* 就是反转后链表的头指针
```js
const reverseList = function(head) {
	// head.next 为null，说明链表只有一个元素直接返回即可
	if(!head || !head.next) return head;
	let tmp = pre = null;
	let cur = head;
	while(cur) {
		tmp = cur.next;
		cur.next = pre;
		pre = cur;
		cur = tmp;
	}
	return pre;
};
```

## 判断是否为回文链表
[leetcode:234](https://leetcode.cn/problems/palindrome-linked-list/description/?envType=study-plan-v2&envId=top-100-liked)
两种方法：
- 遍历链表，获取 val 的值，用数组使用双指针来判断是否为回文的，从而推断，链表是否为回文
- 找到链表中心，将中心后续的链表节点反转，比较两条链表（*起点 -> 中心点* 或 *中心点 -> 终点*）

### 快慢指针 -> 查找链表中点
> *快慢指针* 定义两个一个快指针、一个慢指针。快指针一次走两步，慢指针一次走一步。

慢指针所指位置就是链表中心位置

```js
const findMidListNode = (head) => {
	let slow = fast = head;
	while(fast && fast.next && fast.next.next) {
		slow = slow.next;
		fast = fast.next.next;
	}
	return slow;
}
```

### 快慢指针 -> 判断链表是否有环
快慢指针相遇时，即有环
```js
const hasCycle = (head) => {
	if (!head || !head.next) return false;
	let slow = fast = head;
	while(fast && fast.next) {
		slow = slow.next;
		fast = fast.next.next;
		
		if(slow === fast) {
			return true;
		}	
	}
	return false;
}
```

#### 环的入口
当*快慢指针在环内相遇*时，如果此时把*慢指针*放回*链表起点*，*快指针留在相遇点*，然后在以“*每次1步*”的速度同时出发，它们下一次相遇的地方，刚好就是“环的入口”

这里有数学逻辑的证明，真的太强了呀！！！
```js
const findCyclePoint = (head) => {
	if (!head || !head.next) return null;
	let slow = fast = head;
	
	while (fast && fast.next) {
		slow = slow.next;
		fast = fast.next.next;
		
		// 相遇点 -> 有环
		if (slow === fast) {
			slow = head; // 慢指针回到链表起点，快指针在相遇点
			while (slow !== fast) {
				// 每次一步
				slow = slow.next;
				fast = fast.next;
			}
			// 慢指针所在位置即是环的入口
			return slow;
		} 
	}
	return null;
}
```

# 栈
# 队列
# 树
# 图
