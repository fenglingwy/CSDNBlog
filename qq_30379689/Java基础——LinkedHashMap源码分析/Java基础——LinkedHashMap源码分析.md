>更多Java集合文章，请关注我的博客：http://blog.csdn.net/qq_30379689

#Java基础——LinkedHashMap源码分析

>本篇文章包含以下内容，请点击左上角+号展开目录

>* 什么是LinkedHashMap
>* LinkedHashMap的有序性
>* LinkedHashMap的成员变量
>* LinkedHashMap的初始化
>* LinkedHashMap的存储
>* LinkedHashMap的读取
>* LinkedHashMap其他API
>* LinkedHashMap与HashMap的区别


----------


##一、什么是LinkedHashMap

1. LinkedHashMap是基于哈希表的Map接口的非同步实现
1. LinkedHashMap是HashMap的子类
2. LinkedHashMap是有序的
2. LinkedHashMap中元素的key是唯一的、value值可重复
3. LinkedHashMap允许null键和null值

##二、LinkedHashMap的有序性

LinkedHashMap底层使用哈希表与双向链表来保存所有元素，它维护着一个运行于所有条目的双向链表（如果学过双向链表的同学会更好的理解它的源代码），此链表定义了迭代顺序，该迭代顺序可以是插入顺序或者是访问顺序

1. 按插入顺序的链表：在LinkedHashMap调用get方法后，输出的顺序和输入时的相同，这就是按插入顺序的链表，默认是按插入顺序排序
2. 按访问顺序的链表：在LinkedHashMap调用get方法后，会将这次访问的元素移至链表尾部，不断访问可以形成按访问顺序排序的链表。简单的说，按最近最少访问的元素进行排序（类似LRU算法）

我们可以通过例子来理解我们上面所说的LinkedHashMap的插入顺序和访问顺序

```java
public static void main(String[] args) {
    Map<String, String> map = new HashMap<String, String>();
    map.put("apple", "苹果");
    map.put("watermelon", "西瓜");
    map.put("banana", "香蕉");
    map.put("peach", "桃子");

    Iterator iter = map.entrySet().iterator();
    while (iter.hasNext()) {
        Map.Entry entry = (Map.Entry) iter.next();
        System.out.println(entry.getKey() + "=" + entry.getValue());
    }
}
```

上面是简单的HashMap代码，通过控制台的输出，我们可以看到HashMap是没有顺序的

```
banana=香蕉
apple=苹果
peach=桃子
watermelon=西瓜
```
我们现在将HashMap换成LinkedHashMap，其他代码不变
```
Map<String, String> map = new LinkedHashMap<String, String>();
```
看一下控制台的输出
```
apple=苹果
watermelon=西瓜
banana=香蕉
peach=桃子
```

我们可以看到，其输出顺序是完成按照插入顺序的，也就是我们上面所说的保留了插入的顺序。下面我们修改一下代码，通过访问顺序进行排序

```
public static void main(String[] args) {
    Map<String, String> map = new LinkedHashMap<String, String>(16,0.75f,true);
    map.put("apple", "苹果");
    map.put("watermelon", "西瓜");
    map.put("banana", "香蕉");
    map.put("peach", "桃子");

    map.get("banana");
    map.get("apple");

    Iterator iter = map.entrySet().iterator();
    while (iter.hasNext()) {
        Map.Entry entry = (Map.Entry) iter.next();
        System.out.println(entry.getKey() + "=" + entry.getValue());
    }
}
```
代码与之前的相比

1. 替换了LinkedHashMap的构造函数，使用三个参数的构造函数，第三个参数传进true就是表明用访问顺序来排序，默认是false（即插入顺序）
2. 增加了两句LinkedHashMap的get方法，来表示最近已经访问过这两个元素了
```
//修改的代码
Map<String, String> map = new LinkedHashMap<String, String>(16,0.75f,true);
......
map.get("banana");
map.get("apple");
```

看一下控制台的输出结果

```
watermelon=西瓜
peach=桃子
banana=香蕉
apple=苹果
```

我们可以看到，顺序是先从最少访问的元素开始遍历（西瓜、桃子），而香蕉、苹果是因为分别调用了get方法，香蕉是最先访问的，所以它的比苹果更少用一些。这也就是我们之前提到过的，LinkedHashMap可以选择按照访问顺序进行排序

##三、LinkedHashMap的成员变量

LinkedHashMap采用的hash算法和HashMap相同，但是它重新定义了数组中保存的元素Entry，该Entry除了保存当前对象的引用外，还保存了其上一个元素before和下一个元素after的引用，从而在哈希表的基础上又构成了双向链接列表

```
/**
 * 如果为true，则按照访问顺序；如果为false，则按照插入顺序。
 */
private final boolean accessOrder;
/**
 * 双向链表的表头元素。
 */
private transient Entry<K,V> header;

/**
 * LinkedHashMap的Entry元素。
 * 继承HashMap的Entry元素，又保存了其上一个元素before和下一个元素after的引用。
 */
private static class Entry<K,V> extends HashMap.Entry<K,V> {
    Entry<K,V> before, after;
    ……
}
```

##四、LinkedHashMap的初始化

对于LinkedHashMap而言，它可以通过重写父类相关的方法，来实现自己的链接列表特性。通过源代码可以看出，在LinkedHashMap的构造方法中，实际调用了父类HashMap的相关构造方法来构造一个底层存放的table数组，但额外可以增加accessOrder这个参数，如果不设置，默认为false，代表按照插入顺序进行迭代，当然可以显式设置为true，代表以访问顺序进行迭代

```
public LinkedHashMap(int initialCapacity, float loadFactor,boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

我们已经知道 LinkedHashMap的Entry元素继承HashMap的Entry，提供了双向链表的功能。在上述HashMap的构造器中，最后会调用init() 方法，进行相关的初始化，这个方法在HashMap的实现中并无意义，只是提供给子类实现相关的初始化调用，但在LinkedHashMap重写了 init() 方法，在调用父类的构造方法完成构造后，进一步实现了对其元素Entry的初始化操作

```
@Override
void init() {
  header = new Entry<>(-1, null, null, null);
  //双向链表的空实现
  header.before = header.after = header;
}
```

##五、LinkedHashMap的存储

LinkedHashMap并未重写父类HashMap的 put 方法，而是重写了父类HashMap的put方法调用的子方法void recordAccess(HashMap m) ，void addEntry(int hash, K key, V value, int bucketIndex) 和void createEntry(int hash, K key, V value, int bucketIndex)，提供了自己特有的双向链接列表的实现

首先先看下HashMap的put方法
```
public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
}
```

重写方法

```
void recordAccess(HashMap<K,V> m) {
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
    if (lm.accessOrder) {
        lm.modCount++;
        remove();
        addBefore(lm.header);
        }
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    // 调用create方法，将新元素以双向链表的的形式加入到映射中。
    createEntry(hash, key, value, bucketIndex);

    // 删除最近最少使用元素的策略定义
    Entry<K,V> eldest = header.after;
    if (removeEldestEntry(eldest)) {
        removeEntryForKey(eldest.key);
    } else {
        if (size >= threshold)
            resize(2 * table.length);
    }
}

void createEntry(int hash, K key, V value, int bucketIndex) {
    HashMap.Entry<K,V> old = table[bucketIndex];
    Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
    table[bucketIndex] = e;
    // 调用元素的addBrefore方法，将元素加入到哈希、双向链接列表。  
    e.addBefore(header);
    size++;
}

private void addBefore(Entry<K,V> existingEntry) {
    after  = existingEntry;
    before = existingEntry.before;
    before.after = this;
    after.before = this;
}
```


##六、LinkedHashMap的读取

LinkedHashMap重写了父类HashMap的get方法，实际在调用父类getEntry()方法取得查找的元素后，再判断当排序模式accessOrder为true时，记录访问顺序，将最新访问的元素添加到双向链表的表头，并从原来的位置删除。由于的链表的增加、删除操作是常量级的，故并不会带来性能的损失

```
public V get(Object key) {
    // 调用父类HashMap的getEntry()方法，取得要查找的元素。
    Entry<K,V> e = (Entry<K,V>)getEntry(key);
    if (e == null)
        return null;
    // 记录访问顺序。
    e.recordAccess(this);
    return e.value;
}

void recordAccess(HashMap<K,V> m) {
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
    // 如果定义了LinkedHashMap的迭代顺序为访问顺序，
    // 则删除以前位置上的元素，并将最新访问的元素添加到链表表头。  
    if (lm.accessOrder) {
        lm.modCount++;
        remove();
        addBefore(lm.header);
    }
}

private void remove() {
    before.after = after;
    after.before = before;
}

/**clear链表，设置header为初始状态*/
public void clear() {
 super.clear();
 header.before = header.after = header;
}
```


##七、LinkedHashMap其他API

1. removeEldestEntry(Map.Entry< K,V> eldest)：该方法可以提供在每次添加新条目时移除最旧条目的实现程序，默认返回 false，即永远不能移除最旧的元素


##八、LinkedHashMap与HashMap的区别

LinkedHashMap | HashMap
--|--|--|
有序的，有插入顺序和访问顺序|无序的
内部维护着一个运行于所有条目的双向链表|内部维护着一个单链表