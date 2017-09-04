>�������Ͻ�+�Ų鿴Ŀ¼�����»᳤�ڲ���ʱ����

#���Ը�ϰ����Android����ʦ֮�㷨����


----------

[toc]

##����

###��������ݽṹ

```
static class Node {
    int data;
    Node next;

    Node(int data) {
        this.data = data;
    }
}
```

###ɾ��������node�ڵ�

```
/**
 * ɾ��������node�ڵ�
 *
 * @param head ͷ�ڵ�
 * @param node node�ڵ�
 * @return ͷ�ڵ�
 */
static Node delete(Node head, Node node) {
    //β�ڵ�
    if (node.next == null) {
        while (head.next != node) {
            head = head.next;
        }
        head.next = null;
    }
    //ͷ�ڵ�
    else if (head == node) {
        head = null;
    }
    //��ͨ�ڵ�
    else {
        Node q = node.next;
        node.next = q.next;
        node.data = q.data;
    }
    return head;
}
```

###��ʽһ��ɾ��������ָ��ֵ�Ľڵ�

```
/**
 * ��ʽһ��ɾ��������ָ��ֵ�Ľڵ�
 *
 * @param head ͷ�ڵ�
 * @param num  ָ��ֵ
 * @return ͷ�ڵ�
 */
static Node delete(Node head, int num) {
    Stack<Node> stack = new Stack<>();
    //��ջ
    while (head != null) {
        if (head.data != num) {
            stack.push(head);
        }
        head = head.next;
    }
    //��ջ���ɱ�
    while (!stack.isEmpty()) {
        stack.peek().next = head;
        head = stack.pop();
    }
    return head;
}
```

###��ʽ����ɾ��������ָ��ֵ�Ľڵ�

```
/**
 * ��ʽ����ɾ��������ָ��ֵ�Ľڵ�
 *
 * @param head ͷ�ڵ�
 * @param num  ָ��ֵ
 * @return ͷ�ڵ�
 */
static Node remove(Node head, int num) {
    //�ҳ�ͷ�ڵ�
    while (head != null) {
        if (head.data != num) {
            break;
        }
        head = head.next;
    }
    //������ָ��
    Node pre = head;
    Node cur = head;
    //��������ɾ��ָ��ֵ�Ľڵ�
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

###ɾ�������в��ظ���ֵ

```
/**
 * ɾ�������в��ظ���ֵ
 *
 * @param head ͷ�ڵ�
 * @return ͷ�ڵ�
 */
static Node deleteDuplication(Node head) {
    //�սڵ�
    if (head == null) {
        return null;
    }
    HashSet<Integer> set = new HashSet<>();
    //ͷ�ڵ�һ���ǲ����ڵ�
    set.add(head.data);
    //������ָ��
    Node per = head;
    Node cur = head.next;
    //��������ɾ���ظ�ֵ�Ľڵ�
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

###���������

```
/**
 * ��������ӣ���1-5-8 + 3-5-1 �����5-0-9
 *
 * @param head1 ����1ͷ�ڵ�
 * @param head2 ����2ͷ�ڵ�
 * @return ��������ͷ�ڵ�
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
    int n1 = 0;//����1����ֵ
    int n2 = 0;//����2����ֵ
    int n = 0;//n = n1 + n2 + ca
    int ca = 0;//���֮��Ľ�λ

    Node node = null;//��ǰ�ڵ�
    Node pnode = null;//node�ڵ��ǰ���ڵ�

    while (stack1.isEmpty() || stack2.isEmpty()) {
        n1 = stack1.isEmpty() ? 0 : stack1.pop();
        n2 = stack2.isEmpty() ? 0 : stack1.pop();
        n = n1 + n2 + ca;
        //����������
        node = new Node(n % 10);
        node.next = pnode;
        pnode = node;
        //�����λ
        ca = n / 10;
    }
    //���ڵ㻹�н�λ
    if (ca == 1) {
        pnode = node;
        node = new Node(n / 10);
        node.next = pnode;
    }
    return node;
}
```

###�ж�һ���������Ƿ�Ϊ���Ľṹ

```
/**
 * �ж�һ���������Ƿ�Ϊ���Ľṹ
 *
 * @param head ͷ�ڵ�
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

###ɾ������������K���ڵ�

```
/**
 * ɾ������������K���ڵ�
 *
 * @param head ͷ�ڵ�
 * @param k
 * @return
 */
static Node removeLastKthNdoe(Node head, int k) {
    if (k <= 0 || head == null) {
        return head;
    }
    //�ƶ�k��λ��
    Node p = head;
    for (int i = 0; i < k; i++) {
        if (p.next != null) {
            p = p.next;
        } else {
            //K�����ܳ���
            return head;
        }
    }
    //���ƶ�ʣ���λ��
    Node q = head;
    while (p.next != null) {
        p = p.next;
        q = q.next;
    }
    //ɾ��
    q.next = q.next.next;
    return head;
}
```

##ջ

###ʹ������ջģ�����

```
/**
 * ʹ������ջģ�����
 */
class imitateQueue {

    Stack<Object> stack1 = new Stack<>();
    Stack<Object> stack2 = new Stack<>();

    void push(Object object) {
        //ѹ���Ԫ��
        stack1.push(object);
    }

    // ��Ϊ����
    // ��һ�������Ԫ���򵯳�
    // �ڶ������û��Ԫ����ȫ����ջ1����ջ2
    void pop() {
        if (!stack2.isEmpty()) {
            //������Ԫ��
            stack2.pop();
        } else {
            if (stack1.isEmpty())
                throw new RuntimeException("����Ϊ��");
            while (!stack1.isEmpty()) {
                Object item = stack1.pop();
                stack2.push(item);
            }
        }
    }
}
```

###��ƺ���С����min()��ջ

```
/**
 * ��ƺ���С����min()��ջ
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


##������

###�����������ݽṹ

```
class TreeNode {
    TreeNode left;
    TreeNode right;
    int data;
}
```

###������ȱ���

```
/**
 * ������ȱ���
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

###������ȱ������������к�

```
/**
 * ������ȱ������������к�
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
    //��һ�α��������һ���ڵ�
    TreeNode last = root;
    //��һ�α��������һ���ڵ�
    TreeNode nLast = root;

    while (!queue.isEmpty()) {
        TreeNode cur = queue.poll();
        System.out.println("data:" + cur.data);
        //������
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
            System.out.println("����");
            last = nLast;
        }
    }
    return list;
}
```

###ǰ�����

```
/**
 * ǰ��������ݹ飩
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
 * ǰ�������������
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

###�������

```
/**
 * ����������ݹ飩
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
 * ���������������
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

###�������

```
/**
 * ����������ݹ飩
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
 * ���������������
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