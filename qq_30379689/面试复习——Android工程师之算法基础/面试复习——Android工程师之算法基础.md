>请点击左上角+号查看目录，文章会长期不定时更新

#面试复习——Android工程师之算法基础


----------

[toc]

##链表

###链表的数据结构

```
static class Node {
    int data;
    Node next;

    Node(int data) {
        this.data = data;
    }
}
```

###删除链表中node节点

```
/**
 * 删除链表中node节点
 *
 * @param head 头节点
 * @param node node节点
 * @return 头节点
 */
static Node delete(Node head, Node node) {
    //尾节点
    if (node.next == null) {
        while (head.next != node) {
            head = head.next;
        }
        head.next = null;
    }
    //头节点
    else if (head == node) {
        head = null;
    }
    //普通节点
    else {
        Node q = node.next;
        node.next = q.next;
        node.data = q.data;
    }
    return head;
}
```

###方式一：删除链表中指定值的节点

```
/**
 * 方式一：删除链表中指定值的节点
 *
 * @param head 头节点
 * @param num  指定值
 * @return 头节点
 */
static Node delete(Node head, int num) {
    Stack<Node> stack = new Stack<>();
    //入栈
    while (head != null) {
        if (head.data != num) {
            stack.push(head);
        }
        head = head.next;
    }
    //出栈链成表
    while (!stack.isEmpty()) {
        stack.peek().next = head;
        head = stack.pop();
    }
    return head;
}
```

###方式二：删除链表中指定值的节点

```
/**
 * 方式二：删除链表中指定值的节点
 *
 * @param head 头节点
 * @param num  指定值
 * @return 头节点
 */
static Node remove(Node head, int num) {
    //找出头节点
    while (head != null) {
        if (head.data != num) {
            break;
        }
        head = head.next;
    }
    //定义两指针
    Node pre = head;
    Node cur = head;
    //遍历链表，删除指定值的节点
    while (cur != null) {
        if (cur.data != num) {
            pre = cur;
        } else {
            pre.next = cur.next;
        }
        cur = cur.next;
    }
    return head;
}
```

###删除链表中不重复的值

```
/**
 * 删除链表中不重复的值
 *
 * @param head 头节点
 * @return 头节点
 */
static Node deleteDuplication(Node head) {
    //空节点
    if (head == null) {
        return null;
    }
    HashSet<Integer> set = new HashSet<>();
    //头节点一定是不存在的
    set.add(head.data);
    //定义两指针
    Node per = head;
    Node cur = head.next;
    //遍历链表，删除重复值的节点
    while (cur != null) {
        if (set.contains(cur.data)) {
            per.next = cur.next;
        } else {
            set.add(cur.data);
            per = cur;
        }
        cur = cur.next;
    }
    return head;
}
```

###两链表相加

```
/**
 * 两链表相加，如1-5-8 + 3-5-1 输出：5-0-9
 *
 * @param head1 链表1头节点
 * @param head2 链表2头节点
 * @return 相加链表的头节点
 */
static Node add(Node head1, Node head2) {
    Stack<Integer> stack1 = new Stack<>();
    Stack<Integer> stack2 = new Stack<>();
    while (head1 != null) {
        stack1.push(head1.data);
        head1 = head1.next;
    }
    while (head2 != null) {
        stack2.push(head2.data);
        head2 = head2.next;
    }
    int n1 = 0;//链表1的数值
    int n2 = 0;//链表2的数值
    int n = 0;//n = n1 + n2 + ca
    int ca = 0;//相加之后的进位

    Node node = null;//当前节点
    Node pnode = null;//node节点的前驱节点

    while (stack1.isEmpty() || stack2.isEmpty()) {
        n1 = stack1.isEmpty() ? 0 : stack1.pop();
        n2 = stack2.isEmpty() ? 0 : stack1.pop();
        n = n1 + n2 + ca;
        //链成新链表
        node = new Node(n % 10);
        node.next = pnode;
        pnode = node;
        //计算进位
        ca = n / 10;
    }
    //最后节点还有进位
    if (ca == 1) {
        pnode = node;
        node = new Node(n / 10);
        node.next = pnode;
    }
    return node;
}
```

###判断一个单链表是否为回文结构

```
/**
 * 判断一个单链表是否为回文结构
 *
 * @param head 头节点
 * @return
 */
static boolean isPalindrome(Node head) {
    if (head == null) {
        return false;
    }
    Stack<Node> stack = new Stack<>();
    Node cur = head;
    while (cur != null) {
        stack.push(cur);
        cur = cur.next;
    }
    while (head != null) {
        if (head.data != stack.pop().data) {
            return false;
        }
        head = head.next;
    }
    return true;
}
```

###删除单链表倒数第K个节点

```
/**
 * 删除单链表倒数第K个节点
 *
 * @param head 头节点
 * @param k
 * @return
 */
static Node removeLastKthNdoe(Node head, int k) {
    if (k <= 0 || head == null) {
        return head;
    }
    //移动k个位置
    Node p = head;
    for (int i = 0; i < k; i++) {
        if (p.next != null) {
            p = p.next;
        } else {
            //K超出总长度
            return head;
        }
    }
    //再移动剩余的位置
    Node q = head;
    while (p.next != null) {
        p = p.next;
        q = q.next;
    }
    //删除
    q.next = q.next.next;
    return head;
}
```

##栈

###使用两个栈模拟队列

```
/**
 * 使用两个栈模拟队列
 */
class imitateQueue {

    Stack<Object> stack1 = new Stack<>();
    Stack<Object> stack2 = new Stack<>();

    void push(Object object) {
        //压入的元素
        stack1.push(object);
    }

    // 分为两步
    // 第一步如果有元素则弹出
    // 第二步如果没有元素则全部从栈1进入栈2
    void pop() {
        if (!stack2.isEmpty()) {
            //弹出的元素
            stack2.pop();
        } else {
            if (stack1.isEmpty())
                throw new RuntimeException("队列为空");
            while (!stack1.isEmpty()) {
                Object item = stack1.pop();
                stack2.push(item);
            }
        }
    }
}
```

###设计含最小函数min()的栈

```
/**
 * 设计含最小函数min()的栈
 */
class min {

    Stack<Integer> stack = new Stack<>();
    Stack<Integer> minStack = new Stack<>();

    void push(int node) {
        stack.push(node);
        if (minStack.isEmpty() || node <= minStack.peek()) {
            minStack.push(node);
        }
    }

    void pop() {
        if (stack.isEmpty())
            return;
        int data = stack.peek();
        stack.pop();
        if (data == minStack.peek()) {
            minStack.pop();
        }
    }

    int min() {
        return minStack.peek();
    }
}
```


##二叉树

###二叉树的数据结构

```
class TreeNode {
    TreeNode left;
    TreeNode right;
    int data;
}
```

###广度优先遍历

```
/**
 * 广度优先遍历
 *
 * @param root
 */
void levelTraversal(TreeNode root) {
    if (root == null) {
        return;
    }
    LinkedList<TreeNode> queue = new LinkedList<>();
    queue.add(root);
    while (!queue.isEmpty()) {
        TreeNode head = queue.removeFirst();
        System.out.print("data:" + head.data);
        if (head.left != null) {
            queue.add(head.left);
        }
        if (head.right != null) {
            queue.add(head.right);
        }
    }
}
```

###广度优先遍历，并带有行号

```
/**
 * 广度优先遍历，并带有行号
 *
 * @param root
 * @return
 */
LinkedList<TreeNode> levelTraversalWithLineNum(TreeNode root) {
    if (root == null) {
        return null;
    }
    LinkedList<TreeNode> list = new LinkedList<>();
    Queue<TreeNode> queue = new ArrayBlockingQueue<>(100);
    //上一次遍历的最后一个节点
    TreeNode last = root;
    //这一次遍历的最后一个节点
    TreeNode nLast = root;

    while (!queue.isEmpty()) {
        TreeNode cur = queue.poll();
        System.out.println("data:" + cur.data);
        //存起来
        list.add(cur);

        if (cur.left != null) {
            queue.add(cur.left);
            nLast = cur.left;
        }
        if (cur.right != null) {
            queue.add(cur.right);
            nLast = cur.right;
        }
        if (cur == last) {
            System.out.println("换行");
            last = nLast;
        }
    }
    return list;
}
```

###前序遍历

```
/**
 * 前序遍历（递归）
 *
 * @param root
 */
void preorderTraversalRec(TreeNode root) {
    if (root == null) {
        return;
    }
    System.out.println("data:" + root.data);
    preorderTraversal(root.left);
    preorderTraversal(root.right);
}

/**
 * 前序遍历（迭代）
 *
 * @param root
 */
void preorderTraversal(TreeNode root) {
    if (root == null) {
        return;
    }
    Stack<TreeNode> stack = new Stack<>();
    stack.add(root);

    while (!stack.isEmpty()) {
        TreeNode cur = stack.pop();
        System.out.print("data:" + cur.data);
        if (cur.right != null) {
            stack.add(cur.right);
        }
        if (cur.left != null) {
            stack.add(cur.left);
        }
    }
}
```

###中序遍历

```
/**
 * 中序遍历（递归）
 *
 * @param root
 */
void inorderTraversalRec(TreeNode root) {
    if (root == null) {
        return;
    }
    preorderTraversal(root.left);
    System.out.println("data:" + root.data);
    preorderTraversal(root.right);
}

/**
 * 中序遍历（迭代）
 *
 * @param root
 */
void inorderTraversal(TreeNode root) {
    if (root == null) {
        return;
    }
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur = root;
    while (!stack.isEmpty() || cur != null) {
        if (cur != null) {
            stack.push(cur);
            cur = cur.left;
        } else {
            TreeNode node = stack.pop();
            System.out.println("data:" + node.data);
            cur = cur.right;
        }
    }
}
```

###后序遍历

```
/**
 * 后序遍历（递归）
 *
 * @param root
 */
void postorderTraversalRec(TreeNode root) {
    if (root == null) {
        return;
    }
    preorderTraversal(root.left);
    preorderTraversal(root.right);
    System.out.println("data:" + root.data);
}

/**
 * 后序遍历（迭代）
 *
 * @param root
 */
void postorderTraversal(TreeNode root) {
    if (root == null) {
        return;
    }
    Stack<TreeNode> s = new Stack<>();
    Stack<TreeNode> output = new Stack<>();
    s.push(root);

    while (!s.isEmpty()) {
        TreeNode cur = s.pop();
        output.push(cur);

        if (cur.left != null) {
            s.push(cur.left);
        }
        if (cur.right != null) {
            s.push(cur.right);
        }
    }

    while (!output.isEmpty()) {
        System.out.print("data:" + output.pop().data);
    }
}
```