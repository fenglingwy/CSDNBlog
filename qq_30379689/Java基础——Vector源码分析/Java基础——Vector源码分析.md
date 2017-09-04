>更多Java集合文章，请关注我的博客：http://blog.csdn.net/qq_30379689

#Java基础——Vector源码分析

----------
>本篇文章包含以下内容，请点击左上角+号展开目录

>* 什么是Vector
>* Vector的成员变量
>* Vector的构造方法
>* Vector的存储
>* Vector的获取
>* Vector的删除
>* Vector调整数组容量
>* Vector的遍历方式
>* Vector和ArrayList的区别
>* 总结

##一、什么是Vector

1. Vector是基于可变数组的List接口的同步实现
2. Vector是有序的
3. Vector允许null键和null值
4. Vector已经不建议使用了

```
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
    
}
```
1. Vector实现了List接口、底层使用数组保存所有元素，其操作基本上是对数组的操作
1. Vector继承了AbstractList抽象类，它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能
1. Vector实现了RandmoAccess接口，即提供了随机访问功能，RandmoAccess是java中用来被List实现，为List提供快速访问功能的，我们可以通过元素的序号快速获取元素对象，这就是快速随机访问
1. Vector实现了Cloneable接口，即覆盖了函数clone()，能被克隆
1. Vector实现了java.io.Serializable接口，意味着ArrayList支持序列化

##二、Vector的成员变量

```
// 保存Vector中数据的数组  
protected Object[] elementData;  
// 实际数据的数量  
protected int elementCount;  
// 容量增长系数  
protected int capacityIncrement;  
// Vector的序列版本号  
private static final long serialVersionUID = -2767605614048989439L;  
```


##三、Vector的构造方法

Vector共有4个构造函数 

1. Vector()：默认构造函数
2. Vector(int capacity)：capacity是Vector的默认容量大小，当由于增加数据导致容量增加时，每次容量会增加一倍
3. Vector(int capacity, int capacityIncrement)：capacity是Vector的默认容量大小，capacityIncrement是每次Vector容量增加时的增量值
4. Vector(Collection< ? extends E> collection)：创建一个包含collection的Vector
```
// Vector构造函数。默认容量是10。  
public Vector() {  
    this(10);  
}  

// 指定Vector容量大小的构造函数  
public Vector(int initialCapacity) {  
    this(initialCapacity, 0);  
}  

// 指定Vector"容量大小"和"增长系数"的构造函数  
public Vector(int initialCapacity, int capacityIncrement) {  
    super();  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal Capacity: "+  
                                           initialCapacity);  
    // 新建一个数组，数组容量是initialCapacity  
    this.elementData = new Object[initialCapacity];  
    // 设置容量增长系数  
    this.capacityIncrement = capacityIncrement;  
}  

// 指定集合的Vector构造函数。  
public Vector(Collection<? extends E> c) {  
    // 获取“集合(c)”的数组，并将其赋值给elementData  
    elementData = c.toArray();  
    // 设置数组长度  
    elementCount = elementData.length;  
    // c.toArray might (incorrectly) not return Object[] (see 6260652)  
    if (elementData.getClass() != Object[].class)  
        elementData = Arrays.copyOf(elementData, elementCount, Object[].class);  
} 
```

##四、Vector的存储

1、setElementAt(E obj, int index)或add(int index, E element)

更新index位置元素，此方法无返回值
```
public synchronized void setElementAt(E obj, int index) {
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                 elementCount);
    }
    elementData[index] = obj;
}
//另一种写法
public void add(int index, E element) {
    insertElementAt(element, index);
}
```
2、set(int index, E element)

更新index位置元素，此方法返回index位置上的旧值
```
 public synchronized E set(int index, E element) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

3、add(E e)或addElement(E obj)

尾部添加元素
```
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public synchronized void addElement(E obj) {
    modCount++;
    ensureCapacityHelper(elementCount + 1); // 确保可以插入元素，当到达最大容量，进行扩容
    elementData[elementCount++] = obj;
}
```

4、insertElementAt(E obj, int index)

index位置插入元素 o
```
public synchronized void insertElementAt(E obj, int index) {
    modCount++;
    if (index > elementCount) { // 越界
        throw new ArrayIndexOutOfBoundsException(index
                                                 + " > " + elementCount);
    }
    ensureCapacityHelper(elementCount + 1); // 确保可以插入元素
    // index - elementCount 之间的元素 后移一位
    System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
    elementData[index] = obj; // 插入元素
    elementCount++; // 元素个数 + 1 
}
```

##五、Vector的获取

1、firstElement()

获取第一个元素
```
public synchronized E firstElement() {
    if (elementCount == 0) {
        throw new NoSuchElementException();
    }
    return elementData(0);
}
```
2、lastElement()

获取最后一个元素
```
public synchronized E lastElement() {
    if (elementCount == 0) {
        throw new NoSuchElementException();
    }
    return elementData(elementCount - 1);
}
```
3、elementData(int index)或get(int index)

获取index位置元素，两个方式最大区别于是否抛出异常
```
@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
//另一种方式
public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
```


##六、Vector的删除

1、remove(int index)或removeElementAt(int index)

删除index位置元素，两种方式区别于有无返回值
```
public synchronized E remove(int index) {
    modCount++;
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    E oldValue = elementData(index);

    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--elementCount] = null; // Let gc do its work

    return oldValue;
}
//另一种方式
public synchronized void removeElementAt(int index) {
    modCount++;
    if (index >= elementCount) { // 越界
        throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                 elementCount);
    }
    else if (index < 0) { // 越界
        throw new ArrayIndexOutOfBoundsException(index);
    }
    int j = elementCount - index - 1; // 需要移动元素个数
    if (j > 0) { // elementData数组，从index+1开始复制j个元素，到该数组的index开始位置
        System.arraycopy(elementData, index + 1, elementData, index, j);
    }
    elementCount--; // 元素个数-1 
    elementData[elementCount] = null; // 空，有利于垃圾回收
}
```

2、removeElement(Object obj)或remove(Object o)

删除元素obj

```
public synchronized boolean removeElement(Object obj) {
    modCount++;
    int i = indexOf(obj);
    if (i >= 0) {
        removeElementAt(i);
        return true;
    }
    return false;
}
//另一种方式
public boolean remove(Object o) {
    return removeElement(o);
}
```
3、removeAllElements()

删除所有元素
```
public synchronized void removeAllElements() {
    modCount++;
    // Let gc do its work
    for (int i = 0; i < elementCount; i++)
        elementData[i] = null;

    elementCount = 0;
}
```

##七、Vector调整数组容量

数组扩容有两个方法，其中开发者可以通过一个public的方法ensureCapacity(int minCapacity)来增加Vector的容量，而在存储元素等操作过程中，如果遇到容量不足，会调用priavte方法private void ensureCapacityHelper(int minCapacity)实现，且每次数组容量的增长是其原容量的2倍
```
public synchronized void ensureCapacity(int minCapacity) {
    if (minCapacity > 0) {
        modCount++;
        ensureCapacityHelper(minCapacity);//是否需要增加容量
    }
}

private void ensureCapacityHelper(int minCapacity) {
   // overflow-conscious code
    if (minCapacity - elementData.length > 0)// true 增加容量
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

##八、Vector的遍历方式


1. 通过迭代器遍历，即通过Iterator去遍历
2. 随机访问，通过索引值去遍历
3. Enumeration遍历 


##九、Vector和ArrayList的区别

Vector|ArrayList
--|--|
同步、线程安全的|异步、线程不安全
需要额外开销来维持同步锁，性能慢|性能快
可以使用Iterator、foreach、Enumeration输出|只能使用Iterator、foreach输出


##十、总结

复习一下之前所学的内容和今天所学的内容，用图说话

![这里写图片描述](http://img.blog.csdn.net/20170528121843320?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)