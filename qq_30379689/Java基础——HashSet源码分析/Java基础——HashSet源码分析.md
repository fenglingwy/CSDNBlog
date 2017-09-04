>更多Java集合文章，请关注我的博客http://blog.csdn.net/qq_30379689

#Java基础——HashSet源码分析


----------
>本篇文章包含以下内容，请点击左上角+号展开目录

>* 什么是HashSet
>* HashSet的构造函数
>* HashSet的存储
>* HashSet的其他API
>* HashMap与HashSet的区别

##一、什么是HashSet

HashSet是基于HashMap实现的，底层采用HashMap来保存元素，本篇文章需要在HashMap的基础上进行阅读，对于HashMap的工作原理请阅读我上一篇文章：[Java基础——HashMap详细解析及面试题解答](http://blog.csdn.net/qq_30379689/article/details/72582279)

1. HashSet是无序的
2. HashSet将对象存储在key中，且不允许key重复
3. HashSet的Value是固定的

##二、HashSet的构造函数

```
private transient HashMap<E,Object> map;

private static final Object PRESENT = new Object();

/**
 * 默认的无参构造器，构造一个空的HashSet。
 *
 * 实际底层会初始化一个空的HashMap，并使用默认初始容量为16和加载因子0.75。
 */
public HashSet() {
    map = new HashMap<E,Object>();
}

/**
 * 构造一个包含指定collection中的元素的新set。
 *
 * 实际底层使用默认的加载因子0.75和足以包含指定collection中所有元素的初始容量来创建一个HashMap。
 * @param c 其中的元素将存放在此set中的collection。
 */
public HashSet(Collection<? extends E> c) {
    map = new HashMap<E,Object>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

/**
 * 以指定的initialCapacity和loadFactor构造一个空的HashSet。
 *
 * 实际底层以相应的参数构造一个空的HashMap。
 * @param initialCapacity 初始容量。
 * @param loadFactor 加载因子。
 */
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<E,Object>(initialCapacity, loadFactor);
}

/**
 * 以指定的initialCapacity构造一个空的HashSet。
 *
 * 实际底层以相应的参数及加载因子loadFactor为0.75构造一个空的HashMap。
 * @param initialCapacity 初始容量。
 */
public HashSet(int initialCapacity) {
    map = new HashMap<E,Object>(initialCapacity);
}

/**
 * 以指定的initialCapacity和loadFactor构造一个新的空链接哈希集合。此构造函数为包访问权限，不对外公开，
 * 实际只是是对LinkedHashSet的支持。
 *
 * 实际底层会以指定的参数构造一个空LinkedHashMap实例来实现。
 * @param initialCapacity 初始容量。
 * @param loadFactor 加载因子。
 * @param dummy 标记。
 */
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<E,Object>(initialCapacity, loadFactor);
}
```

HashSet的构造方法基于HashMap的构造方法，如果熟悉HashMap的同学，请先学习完HashMap再来看HashSet


##三、HashSet的存储

```
/**
 * @param e 将添加到此set中的元素。
 * @return 如果此set尚未包含指定元素，则返回true。
 */
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

由于HashMap的put方法添加key-value时，当新放入HashMap的Entry中key与集合中原有Entry的key相同（hashCode()返回值相等，通过 equals 比较也返回 true），新添加的Entry的value会将覆盖原来Entry的value（HashSet 中的 value 都是PRESENT），但key不会有任何改变，因此向HashSet中添加一个已经存在的元素时，新添加的集合元素将不会被放入HashMap中，原来的元素也不会有任何改变，这也就满足了 Set 中元素不重复的特性

该方法如果添加的是在HashSet中不存在的，则返回true；如果添加的元素已经存在，返回false。其原因在于我们之前提到的关于HashMap的put方法。该方法在添加key不重复的键值对的时候，会返回null

##四、HashSet的其他API
 
```
/**
 * 如果此set包含指定元素，则返回true。
 * 更确切地讲，当且仅当此set包含一个满足(o==null ? e==null : o.equals(e))的e元素时，返回true。
 *
 * 底层实际调用HashMap的containsKey判断是否包含指定key。
 * @param o 在此set中的存在已得到测试的元素。
 * @return 如果此set包含指定元素，则返回true。
 */
public boolean contains(Object o) {
return map.containsKey(o);
}
/**
 * 如果指定元素存在于此set中，则将其移除。更确切地讲，如果此set包含一个满足(o==null ? e==null : o.equals(e))的元素e，
 * 则将其移除。如果此set已包含该元素，则返回true
 *
 * 底层实际调用HashMap的remove方法删除指定Entry。
 * @param o 如果存在于此set中则需要将其移除的对象。
 * @return 如果set包含指定元素，则返回true。
 */
public boolean remove(Object o) {
return map.remove(o)==PRESENT;
}
/**
 * 返回此HashSet实例的浅表副本：并没有复制这些元素本身。
 *
 * 底层实际调用HashMap的clone()方法，获取HashMap的浅表副本，并设置到HashSet中。
 */
public Object clone() {
    try {
        HashSet<E> newSet = (HashSet<E>) super.clone();
        newSet.map = (HashMap<E, Object>) map.clone();
        return newSet;
    } catch (CloneNotSupportedException e) {
        throw new InternalError();
    }
}
```

##五、HashMap与HashSet的区别
HashMap | HashSet
--|--
HashMap实现了Map接口|	HashSet实现了Set接口
HashMap储存键值对	|HashSet仅仅存储对象
使用put()方法将元素放入map中	|使用add()方法将元素放入set中
HashMap中使用键对象来计算hashcode值|	HashSet使用成员对象来计算hashcode值
HashMap比较快，因为是使用唯一的键来获取对象|	HashSet较HashMap来说比较慢