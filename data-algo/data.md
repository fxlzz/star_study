> 我只能说，那句话真的没说错，*程序 = 数据结构 + 算法*。原来我也认为这些不重要，后来才发现这些恰恰是最最最重要的！！！ 
> --- 所有的东西都是数据结构，无非是以一种特殊的方式进行了处理

# 链表

## 单链表
```js
function ListNode(val) {
	this.val = val;
	this.next = null;
}
```

## 判断链表是否相交
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

## 双向链表
>  一般做链表的题目，涉及到增删，那么设置虚拟结点操作会简单一些

>  还有注意一下，箭头函数没有 this !!

```js
class DoubleLinkedListNode {
    constructor(val = 0) {
        this.val = val;
        this.prev = null; // 前驱
        this.next = null; // 后继
    }
}

// 删除结点
// 删除尾部结点，在有 tail dummy结点时，只需要 removeNode(tail.prev); 
const removeNode = function(node) {
	node.prev.next = node.next;
	node.next.prev = node.prev;
}

// 插入头结点
const addToHead = function(node) {
	// 先操作 node ，最后操作 head.next 指向
	node.prev = head;
	node.next = head.next;
	head.next.prev = node;
	head.next = node;
}
```

## LRU 

>  LRU 缓存： 最近最少使用缓存，看时间*长短*
>  这个题目就非常适合用 *双向链表* + *哈希表* 来做

理解 LRU：
怎么理解呢? 将整一个 LRU 看成一摞书
- `get(key)`： 
	- 如果一摞书中存在要看的书 `key`，那么将其移动到书的最上面，方便看；
	- 如果没有，那么告诉自己下次要记得买 `return -1`
- `put(key, value)`： 
	- 如果一摞书中存在要找的书，这  `key` 这一版本我看过了，将其拿出来放置书摞最上面，并且更新为 `value` 这个版本的；
	- 如果没找到这本书，那么将新书 `{key: value}` 放到书摞上；
	- 如果超过书摞能累计的最大容量，不能放书了，那么将最长时间没有看过的书（最后一本书）丢掉

[146. LRU 缓存 - 力扣（LeetCode）](https://leetcode.cn/problems/lru-cache/description/?envType=study-plan-v2&envId=top-100-liked)

```js
/**
 * 双向链表
 */
class DoubleLinkedListNode {
    constructor(key = 0, val = 0) {
        this.key = key; // 方便在缓存中，删除尾部key
        this.val = val;
        this.prev = null;
        this.next = null;
    }
}

/**
 * @param {number} capacity
 */
var LRUCache = function (capacity) {
    this.capacity = capacity;
    this.cache = new Map();
    
    // dummy head & tail
    this.head = new DoubleLinkedListNode();
    this.tail = new DoubleLinkedListNode();
    this.head.next = this.tail;
    this.tail.prev = this.head;
};

/**
 * @param {number} key
 * @return {number}
 */
LRUCache.prototype.get = function (key) {
    if (!this.cache.has(key)) {
        return -1;
    }
    
    const node = this.cache.get(key); // 存入双向链表结点
    this.moveToHead(node);
    return node.val;
};

/**
 * @param {number} key
 * @param {number} value
 * @return {void}
 */
LRUCache.prototype.put = function (key, value) {
    if(!this.cache.has(key)) {
        const node = new DoubleLinkedListNode(key, value);
        this.addToHead(node);
        this.cache.set(key, node);

        if (this.cache.size > this.capacity) {
            const lastNode = this.tail.prev;
            this.removeNode(lastNode);
            this.cache.delete(lastNode.key);
        }
    } 
        
    const node = this.cache.get(key);
    this.moveToHead(node);
    node.val = value;
    this.cache.set(key, node);
};

/**
 * 在头结点处插入结点
 */
LRUCache.prototype.addToHead = function (node) {
    node.next = this.head.next;
    node.prev = this.head;
    this.head.next.prev = node;
    this.head.next = node;
}

/**
 * 删除结点
 */
LRUCache.prototype.removeNode = function (node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
}

/**
 * 已存在的结点移动到头结点
 */
LRUCache.prototype.moveToHead = function (node) {
    this.removeNode(node);
    this.addToHead(node);
}
```

## LFU
[460. LFU 缓存 - 力扣（LeetCode）](https://leetcode.cn/problems/lfu-cache/description/)

>  LFU 缓存： 最不经常使用，看使用*次数*

# 栈
# 队列
# 树
>  二叉树节点描述
```js
class TreeNode {
	constructor(val = 0, left = null, right = null) {
		this.val = val;
		this.left = left; // 左孩子
		this.right = right; // 右孩子
	}
}
```

## 二叉树遍历 
二叉树的遍历，一般采用 *递归* 会简单一点；由于在计算机底层，*递归* 的实现是借助于 *栈* 来实现的，所以，也可用 *栈* 来模拟 *递归* 遍历

### 先序
>  根左右
### 中序
>  左根右

#### 递归
[94. 二叉树的中序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-inorder-traversal/description/?envType=study-plan-v2&envId=top-100-liked)
```js
/**
 * @param {TreeNode} root
 * @return {number[]}
 */
var inorderTraversal = function(root) {
    const ans = [];

    const dfs = (root) => {
        if(!root) return null;

        dfs(root?.left);
        ans.push(root.val);
        dfs(root?.right);
    }
    dfs(root);
    return ans;
};
```

#### 迭代
*用栈来模拟*
```js
/**
 * @param {TreeNode} root
 * @return {number[]}
 */
var inorderTraversal = function(root) {
    if(!root) return [];
    const stack = [];
    const ans = [];
    
   let cur = root;
   while(cur || stack.length > 0) {
   	// 如果不加 stack.length > 0 的话，只会处理一颗子树，不会回退到root
   	while(cur) {
   		stack.push(cur);
   		cur = cur.left;
   	}
   	
   	// 因为存储时是存的引用，那么回退就简单了，简直就是神之一手!!!
   	cur = stack.pop();
   	ans.push(cur.val);
   	cur = cur.right;
   }
   return ans;
};
```

*线索二叉树*

### 后序
>  左右根


## 二叉树的高度
[104. 二叉树的最大深度 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-depth-of-binary-tree/?envType=study-plan-v2&envId=top-100-liked)

一般有关于二叉树的题目，都可以用"递归"来实现，这题也不例外，但是有两种写法

### 自底向上
>  其实就是递归中的 "归" 这个阶段，从树的最下面，一直返回到根
>  想象成，要求整棵的高度，只要求 `max = Math.max(左子树高度，右子树高度) + 1` 即可

```js
/**
 * @param {TreeNode} root
 * @return {number}
 */
var maxDepth = function(root) {
	if(!root) return 0;
	
	const left = maxDepth(root.left); // 左子树的高度
	const right = maxDepth(root.right); // 右子树的高度
	
	return Math.max(left, right) + 1;
};
```

### 自顶向上
>  这个就是反着的，是递归中的 "递" 这个阶段，从根结点一直遍历到叶子结点

```js
/**
 * @param {TreeNode} root
 * @return {number}
 */
var maxDepth = function(root) {
	let ans = 0;
	
	const dfs = (root, cnt) => {
		if(!root) return;
		
		cnt += 1; // 记录路径上的结点个数
		ans = Math.max(ans, cnt);
		dfs(root.left, cnt);
		dfs(root.right, cnt);
	}
	return ans;
};
```

# 图
