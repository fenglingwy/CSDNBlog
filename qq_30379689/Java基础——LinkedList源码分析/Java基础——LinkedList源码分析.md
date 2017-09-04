>更多Java集合文章，请关注我的博客：http://blog.csdn.net/qq_30379689

#Java基础——LinkedList源码分析


----------

>本篇文章包含以下内容，请点击左上角+号展开目录

>* 什么是LinkedList
>* LinkedList的数据结构
>* LinkedList的存储
>* LinkedList的其他API
>* LinkedList与ArrayList的区别
>* 总结

#一、什么是LinkedList

1. LinkedList基于链表的List接口的非同步实现
2. LinkedList允许包括null在内的所有元素
3. LinkedList是有序的
4. LinkedList是fail-fast的

```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{

}
```
1. LinkedList实现了 List 接口、底层基于链表实现的，所以它的插入和删除操作比ArrayList更加高效，但链表的随机访问的效率要比ArrayList差
2. LinkedList继承自AbstractSequenceList抽象类，提供了List接口骨干性的实现以减少实现 List 接口的复杂度
3. LinkedList实现了Deque接口，定义了双端队列的操作，双端队列是一种具有队列和栈的性质的数据结构，双端队列中的元素可以从两端弹出，其限定插入和删除操作在表的两端进行
4. LinkedList实现了Cloneable接口，即覆盖了函数clone()，能被克隆
5. LinkedList实现了java.io.Serializable接口，意味着ArrayList支持序列化


#二、LinkedList的数据结构


LinkedList是基于链表结构实现，在类中包含了first和last两个指针，表示上一个节点和下一个节点的引用，这样就构成了双向的链表

```
transient int size = 0;
transient Node<E> first; //链表的头指针
transient Node<E> last; //尾指针
//存储对象的结构 Node, LinkedList的内部类
private static class Node<E> {
    E item;
    Node<E> next; // 指向下一个节点
    Node<E> prev; //指向上一个节点

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

#三、LinkedList的存储

1、add(E e)

该方法是在链表的末尾添加元素，其调用了自己的方法linkLast(E e)，linkLast(E e)将last的Node引用指向了一个新的Node(l)，然后根据l新建了一个newNode，其中的元素就为要添加的e，而后，我们让 last 指向了 newNode。简单的说就是双向链表的添加操作

```
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

2、add(int index, E element)

该方法是在指定index位置插入元素，如果 index 位置正好等于 size，则调用linkLast(element)将其插入末尾，否则调用linkBefore(element, node(index))方法进行插入。简单的说就是双向链表的添加删除操作


```
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
  
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```

#四、LinkedList的其他API

1. void addFirst(E e)：添加元素到链头
1. void addLast(E e)：添加元素到链尾
1. E removeFirst()：移除链头元素
1. E removeLast()：移除链尾元素

>这里个人经验告诉你们。当问到贪吃蛇是怎么实现的，LinkedList绝对是首选


#五、LinkedList与ArrayList的区别

LinkedList|ArrayList
--|--|
底层是双向链表|底层是可变数组
不允许随机访问，即查询效率低|允许随机访问，即查询效率高
插入和删除效率快|插入和删除效率低
<br>
解释一下：

1. 对于随机访问的两个方法，get和set，ArrayList优于LinkedList，因为LinkedList要移动指针
2. 对于新增和删除两个方法，add和remove，LinedList比较占优势，因为ArrayList要移动数据


#六、总结

对目前学习到的知识进行总结，看图说话

![这里写图片描述](http://img.blog.csdn.net/20170526122108632?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)