>更多Java集合文章，请关注我的博客：http://blog.csdn.net/qq_30379689

#Java基础——ArrayList源码分析


----------
>本篇文章包含以下内容，请点击左上角+号展开目录

>* 什么是ArrayList
>* ArrayList的构造函数
>* ArrayList的存储
>* ArrayList的读取
>* ArrayList的删除
>* ArrayList调整数组容量
>* ArrayListFail-Fast机制
>* 总结


##一、什么是ArrayList

1. ArrayList可以理解为动态数组，它的容量能动态增长，该容量是指用来存储列表元素的数组的大小，随着向ArrayList中不断添加元素，其容量也自动增长
2. ArrayList允许包括null在内的所有元素
3. ArrayList是List接口的非同步实现
4. ArrayList是有序的


注意：自动增长会带来数据向新数组的重新拷贝，因此，如果可预知数据量的多少，可在构造 ArrayList 时指定其容量。在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量


```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{

}
```

1. ArrayList实现了List接口、底层使用数组保存所有元素，其操作基本上是对数组的操作
2. ArrayList继承了AbstractList抽象类，它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能
3. ArrayList实现了RandmoAccess接口，即提供了随机访问功能，RandmoAccess是java中用来被List实现，为List提供快速访问功能的，我们可以通过元素的序号快速获取元素对象，这就是快速随机访问
4. ArrayList实现了Cloneable接口，即覆盖了函数clone()，能被克隆
5. ArrayList实现了java.io.Serializable接口，意味着ArrayList支持序列化

##二、ArrayList的构造函数

ArrayList 提供了三种方式的构造器

1. public ArrayList()：可以构造一个默认初始容量为10的空列表
1. public ArrayList(int initialCapacity)：构造一个指定初始容量的空列表
1. public ArrayList(Collection< ? extends E> c)：构造一个包含指定collection的元素的列表，这些元素按照该collection的迭代器返回它们的顺序排列的

```
private transient Object[] elementData;

public ArrayList() {
    this(10);
}

public ArrayList(int initialCapacity) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
}

public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    size = elementData.length;
    
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```


##三、ArrayList的存储

1、set(int index, E element)

该方法首先调用rangeCheck(index)来校验index变量是否超出数组范围，超出则抛出异常如果没有超出，则取出原index位置的值，并且将新的element放入index位置，返回oldValue

```
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
    
private void rangeCheck(int index) {
  if (index >= size)
  throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

2、add(E e)

该方法是将指定的元素添加到列表的尾部，当容量不足时，会调用grow方法增长容量

```
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
private void ensureCapacityInternal(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

3、add(int index, E element)

在 index 位置插入 element

```
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

4、addAll(Collection< ? extends E> c) 和 addAll(int index, Collection< ? extends E> c)

将特定Collection中的元素添加到Arraylist末尾

```
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```



##四、ArrayList的读取

ArrayList 能够支持随机访问的原因也是很显然的，因为它内部的数据结构是数组，而数组本身就是支持随机访问，该方法首先会判断输入的index值是否越界，然后将数组的index位置的元素返回即可

```
public E get(int index) {
    rangeCheck(index);
    return (E) elementData[index];
}
private void rangeCheck(int index) {
    if (index >= size)
    throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```


##五、ArrayList的删除

ArrayList 提供了根据下标或者指定对象两种方式的删除功能，需要注意点是：从数组中移除元素的操作，也会导致被移除的元素以后的所有元素的向左移动一个位置

1. 删除数组中index位置下的元素
2. 删除数组中相同对象的元素

```
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // Let gc do its work

    return oldValue;
}

public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```



##六、ArrayList调整数组容量

从上面介绍的向ArrayList中存储元素的代码中，我们看到，每当向数组中添加元素时，都要去检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求

数组扩容有两个方法，其中开发者可以通过一个public的方法ensureCapacity(int minCapacity)来增加ArrayList的容量，而在存储元素等操作过程中，如果遇到容量不足，会调用priavte方法private void ensureCapacityInternal(int minCapacity)实现


```
public void ensureCapacity(int minCapacity) {
    if (minCapacity > 0)
        ensureCapacityInternal(minCapacity);
}

private void ensureCapacityInternal(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

从上述代码中可以看出，数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长是其原容量的 1.5倍（从int newCapacity = oldCapacity + (oldCapacity >> 1)这行代码得出），这种操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。当我们可预知要保存的元素的多少时，要在构造 ArrayList 实例时，就指定其容量，以避免数组扩容的发生，或者根据实际需求，通过调用ensureCapacity方法来手动增加ArrayList实例的容量


##七、ArrayListFail-Fast机制

ArrayList 也采用了快速失败的机制，通过记录modCount参数来实现，在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。 关于Fail-Fast的介绍，可以查看我之前的HashMap的文章[Java基础——HashMap详细解析及面试题解答](http://blog.csdn.net/qq_30379689/article/details/72582279)

##八、总结

复习一下之前所学的内容和今天所学的内容，用图说话

![这里写图片描述](http://img.blog.csdn.net/20170525173204767?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
