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

[144. 二叉树的前序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-preorder-traversal/)
#### 递归
```js
/**
 * @param {TreeNode} root
 * @return {number[]}
 */

var preorderTraversal = function(root) {
	const ans = [];
	const dfs = root => {
		if(!root) return null;
		
		ans.push(root.val);
		dfs(root.left);
		dfs(root.right);
	}
	dfs(root);
	return ans;
};
```

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

[145. 二叉树的后序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-postorder-traversal/description/)

#### 递归
```js
/**
 * @param {TreeNode} root
 * @return {number[]}
 */
var postorderTraversal = function(root) {
	const ans = [];
	const dfs = root => {
		if(!root) return null;
		
		dfs(root.left);
		dfs(root.right);
		ans.push(root.val);
	}
	dfs(root);
	return ans;
};
```


### 层次遍历
>  如果说，上面的递归都可以用 *栈* 来模拟，那么层次递归，就可以用 *队列* 来模拟

[102. 二叉树的层序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-level-order-traversal/?envType=study-plan-v2&envId=top-100-liked)

```js
/**
 * @param {TreeNode} root
 * @return {number[][]}
 */
var levelOrder = function(root) {
	if(!root) return []; // 记得越界判断，不然会报空指针异常
	
	const queue = [root]; // 初始化队列，根结点
	const ans = [];
	
	// 只要队列中有值，说明结点未遍历完
	while(queue.length > 0) { 
		let size = queue.length;
		const res = [];
		while(size--) { // 删除那一层的元素
			const node = queue.shift();
			res.push(node.val);
			
			if(node.left) queue.push(node.left);
			if(node.right) queue.push(node.right);
		}
		ans.push(res);
	}
	return ans;
};
```

## 二叉树的高度
[104. 二叉树的最大深度 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-depth-of-binary-tree/?envType=study-plan-v2&envId=top-100-liked)

一般有关于二叉树的题目，都可以用"递归"来实现，这题也不例外，但是有两种写法

### 自底向上 -> 归（后序遍历）
>  其实就是递归中的 "归" 这个阶段，*先"递"到叶子结点，在返上来*
>  "归" 需要*返回值*，一般都是先"递"（调自己），在处理 root 的情况

>  对于这个题目，"归" 的视角：给我一个结点，我将求出左子树和右子树的高度，对于 root 来说，还需要处理一步，也就是 `Math.max(left, right) + 1`

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

### 自顶向上 -> 递（先序遍历）
>  这个就是反着的，是递归中的 "递" 这个阶段，从*根结点一直遍历到叶子结点*
>  "递" 处理*当前这一层的数据*，在让孩子们在去处理孩子的数据
>  没有返回值，先处理本层结点 -> 再递归

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


## 二叉搜索树
[98. 验证二叉搜索树 - 力扣（LeetCode）](https://leetcode.cn/problems/validate-binary-search-tree/description/?envType=study-plan-v2&envId=top-100-liked)

>  定义：
>  - 节点的左子树所有节点必须 *严格小于* 当前节点
>  - 节点的右子树所有节点必须 *严格大于* 当前节点
>  - 左右子树必须都是二叉搜索树

在一次体会到，"递归" 这个词的深意，递归的写法一般都有两种【 自底向上（归） | 自顶向下（递）】

### 自底向上
递归到叶子节点，在"归"的时候，判断当前节点是否在范围内（所以，这需要返回当前节点能够允许的范围）

```js
/**
 * @param {TreeNode} root
 * @return {boolean}
 */
var isValidBST = function(root) {
	const dfs = (root) => {
		if(!root) return {valid: true, min: Infinity, max: -Infinity};
		
		// 返回当前节点能够满足的范围
		const left = dfs(root.left);
		const right = dfs(root.right);
		
		if(!left.valid || !right.valid || root.val <= left.max || root.val >= right.min) {
			return {valid: false};
		}
		
		return {
			valid: true,
			min: Math.min(root.val, left.min),
			max: Math.max(root.val, right.max)
		}
	}
	
	return dfs(root).valid;
};
```

### 自顶向下
需要判断每一个值，它的范围有没有超过祖先定下来的范围 `[min, max]`

```js
/**
 * @param {TreeNode} root
 * @return {boolean}
 */
var isValidBST = function(root) {
	const dfs = (root, min, max) => {
		// 如果到了null，说明到了判断到了叶子节点，说明这棵子树是一个棵二叉搜索树
		if(!root) return true;
		
		// 判断节点是否在规定范围内
		if(root.val >= min || root.val <= max) return false;
		
		return dfs(root.left, min, root.val) && dfs(root.right, root.val, max);
	}
	
	// 对于根节点的范围当然无所谓 [-Infinity, Infinity]
	return dfs(root, -Infinity, Infinity);
};
```

### 中序遍历
利用二叉搜索树的特性： *中序遍历*后的结果一定是*升序*的


# 图
