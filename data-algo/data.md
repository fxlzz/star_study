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
*双指针* -> 类似于，将所有的 next 指向全部反转
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
- 找到链表中心，将中心后续的链表节点反转，比较两条链表（起点 -> 中心点 或 中心点 -> 终点）
### 链表中心
*双指针*


# 栈
# 队列
# 树
# 图
