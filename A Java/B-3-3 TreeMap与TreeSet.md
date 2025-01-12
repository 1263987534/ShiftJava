[TOC]

### TreeMap与TreeSet

#### 基础

##### 1. 杂记

- TreeMap 是一个**有序的 key-value 集合**，它是通过**红黑树**实现的，红黑树结构天然支持排序，默认情况下通过 Key 值的**自然顺序**进行排序；TreeMap 是按照**键**而不是值**有序**，都是对**键**进行比较。
- TreeMap 继承了 NavigableMap 接口，NavigableMap 接口继承了 SortedMap 接口，可支持一系列的导航定位以及导航操作的方法，当然只是提供了接口，需要 TreeMap 自己去实现。

##### 2. 基本API与使用

###### (1) 构造方法

**TreeMap()**：创建一个空 TreeMap，keys 按照**自然排序**。要求 Map 中的键实现 **Comparable 接口**。

```java
TreeMap<Integer, String> treeMap = new TreeMap<>();
```

**TreeMap(Comparator comparator)**：创建一个空 TreeMap，按照指定的 **comparator** 排序。即创建**自定义**的比较器。

```java
TreeMap<Integer, String> map = new TreeMap<>(Comparator.reverseOrder());
map.put(3, "val");
map.put(2, "val");
map.put(1, "val");
map.put(5, "val");
map.put(4, "val");
System.out.println(map); // {5 = val, 4 = val, 3 = val, 2 = val, 1 = val} 逆序
```

```java
// String.CASE_INSENSITIVE_ORDER是String类中的一个忽略大小写的Comparator
Map<String, String> map = new TreeMap<>(String.CASE_INSENSITIVE_ORDER);
// 逆序并忽略大小写
Map<String, String> map = new TreeMap<>(Collections.reverseOrder(String.CASE_INSENSITIVE_ORDER));
```

**TreeMap(Map m)**：由**给定的 map** 创建一个 TreeMap，keys 按照自然排序。Hutool 中对 Map 的排序就是用的这个方法将原本的 map 转为 TreeMap。

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "val");
...
TreeMap<Integer, String> treeMap = new TreeMap<>(map);
```

###### (2) 使用示例

```java
TreeMap<Integer, String> treeMap = new TreeMap<>();
treeMap.put(1, "a");
treeMap.put(2, "b");
treeMap.put(3, "c");
treeMap.put(4, "d"); 	// treeMap: {1 = a, 2 = b, 3 = c, 4 = d}

treeMap.remove(4); // treeMap: {1 = a, 2 = b, 3 = c}
int sizeOfTreeMap = treeMap.size(); // sizeOfTreeMap: 3

treeMap.replace(2, "e"); // treeMap: {1 = a, 2 = e, 3 = c}

Map.Entry entry = treeMap.firstEntry(); // entry: 1 -> a
Integer key = treeMap.firstKey(); // key: 1
entry = treeMap.lastEntry(); // entry: 3 -> c
key = treeMap.lastKey(); // key: 3
String value = treeMap.get(3); // value: c
SortedMap sortedMap = treeMap.headMap(2); // sortedMap: {1 = a}
sortedMap = treeMap.subMap(1, 3); // sortedMap: {1 = a, 2 = e}

Set setOfEntry = treeMap.entrySet(); // setOfEntry: [1 = a, 2 = e, 3 = c]
Collection<String> values = treeMap.values(); // values: [a, e, c]
treeMap.forEach((integer, s) -> System.out.println(integer + "->" + s)); 
// output：
// 1 -> a
// 2 -> e
// 3 -> c
```

###### (3) 遍历方式

**for 循环**

```java
// 与HashMap迭代类似
for (Map.Entry entry : treeMap.entrySet()) {
      System.out.println(entry);
}
```

**迭代器循环**

```java
Iterator iterator = treeMap.entrySet().iterator();
while (iterator.hasNext()) {
      System.out.println(iterator.next());
}
```



#### 源码解析

##### 1. 红黑树

因为TreeMap的存储结构是**红黑树**，回顾一下红黑树的特点以及基本操作。

下图为典型的红黑树：

<img src="assets/image-20200506220030047.png" alt="image-20200506220030047" style="zoom:43%;" />

**红黑树规则特点：**

1. **节点**分为**红色或者黑色**；
2. **根节点**必为**黑色**；
3. **叶子节点都为黑色**，且为 null；
4. 连接**红色节点**的两个子节点都为黑色（红黑树**不会出现相邻的红色节点**）；
5. 从任意节点出发，到其**每个叶子节点的路径中包含相同数量的黑色节点**。
6. 新插入的结点为**红色**。

**红黑树自平衡基本操作：**

1. **变色**：在不违反上述红黑树规则特点情况下，将红黑树某个 node 节点颜色**由红变黑**，或者**由黑变红**。
2. **左旋**：**逆时针**旋转两个节点，让一个节点被其右子节点取代，而该节点成为右子节点的左子节点。
3. **右旋**：**顺时针**旋转两个节点，让一个节点被其左子节点取代，而该节点成为左子节点的右子节点。

##### 2. 基本属性

先看一下 TreeMap 中主要的成员变量。

```java
/**
 * 自定义比较器,不传就默认使用自然排序
 */
private final Comparator<? super K> comparator;

/**
 * TreeMap中红黑树唯一的根节点
 */
private transient Entry<K,V> root;

/**
 * Map中key-val对的数量,也即是红黑树中节点Entry的数量
 */
private transient int size = 0;

/**
 * 红黑树结构性改变次数
 */
private transient int modCount = 0;
```

上面的主要成员变量根节点 root 是 **Entry** 类的实体，来看一下 Entry 类的源码。

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    // key,val是存储的原始数据
    K key;
    V value;
    // 左孩子
    Entry<K,V> left;
    // 右孩子
    Entry<K,V> right;
    // 父节点
    Entry<K,V> parent;
    // 默认情况下为黑色节点，可调整
    boolean color = BLACK;

	// 构造器
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }

    /**
     * 获取节点的key值
     */
    public K getKey() {return key;}

    /**
     * 获取节点的value值
     */
    public V getValue() {return value;}

    /**
     * 用新值替换当前值，并返回当前值
     */
    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }
    
	// 覆写equals方法，这与前面的介绍如何写equals方法的套路是一样的
    public boolean equals(Object o) {
        // 先判断O是不是之定的类型
        if (!(o instanceof Map.Entry))
            return false;
        // 进行类型转换
        Map.Entry<?, ?> e = (Map.Entry<?, ?>)o;
        // 比较键和值
        return valEquals(key, e.getKey()) && valEquals(value, e.getValue());
    }
	
    public int hashCode() {
        int keyHash = (key == null ? 0 : key.hashCode());
        int valueHash = (value == null ? 0 : value.hashCode());
        // 异或求哈希值
        return keyHash ^ valueHash;
    }

    public String toString() {
        return key + "=" + value;
    }
}
```

Entry 静态内部类实现了 Map 的内部接口 **Entry**，提供了**红黑树存储结构**的 Java 实现，通过 left 属性可以建立**左子树**，通过 right 属性可以建立**右子树**，通过 parent 可以往上找到**父节点**。

大体的实现**结构图**如下：

![image-20200507000141908](assets/image-20200507000141908.png)

##### 3. 构造方法

```java
// 默认构造函数，按照key的自然顺序排列
public TreeMap() {comparator = null;}
// 传递Comparator具体实现，按照该实现规则进行排序
public TreeMap(Comparator<? super K> comparator) {this.comparator = comparator;}
// 传递一个map实体构建TreeMap,按照默认规则排序
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}
// 传递一个map实体构建TreeMap,按照传递的map的排序规则进行排序
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

##### 4. put方法

put 方法为 Map 的核心方法，TreeMap 的 put 方法大概流程如下：

<img src="assets/image-20200507001555169.png" alt="image-20200507001555169" style="zoom:47%;" />

源码如下

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    /**
     * 如果根节点都为null则先创建红黑树的根
     */
    if (t == null) {
        compare(key, key); 
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    /**
     * 如果节点不为null,定义一个cmp，这个变量用来进行二分查找时的比较；定义parent，
     * 是new Entry时必须要的参数
     */
    int cmp;
    Entry<K,V> parent;
    // cpr表示有无自己定义的排序规则，分两种情况遍历执行
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        /**
         * 从root节点开始遍历，通过二分查找逐步向下找
         * 第一次循环：从根节点开始，这个时候parent就是根节点，然后通过自定义的排序算法
         * cpr.compare(key, t.key)比较传入的key和根节点的key值，如果传入的key<root.key，那么
         * 继续在root的左子树中找，从root的左孩子节点（root.left）开始：如果传入的key>root.key,
         * 那么继续在root的右子树中找，从root的右孩子节点（root.right）开始;
         * 如果恰好key==root.key，那么直接根据root节点的value值即可。
         * 后面的循环规则一样，当遍历到的当前节点作为起始节点，逐步往下找
         * 需要注意的是：这里并没有对key是否为null进行判断，建议自定义Comparator时应该要考虑在内
         */
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    else {
        // 从这里看出，当默认排序时，key值是不能为null的
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        // 这里的实现逻辑和上面一样，都是通过二分查找
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    /**
     * 能执行到这里，说明前面并没有找到相同的key,节点已经遍历到最后了，只需要new一个Entry放到
     * parent下面即可，但放到左子节点上还是右子节点上，就需要按照红黑树的规则来。
     */
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    /**
     * 节点加进去了，并不算完，如果加入节点对红黑树的结构造成了
     * 破坏，需要进行一些自平衡操作，如【变色】【左旋】【右旋】
     */
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

put 方法源码中通过 **fixAfterInsertion**(e) 方法来进行**自平衡**处理。

|        |          无需调整          |         【变色】即可实现平衡         |                  【旋转+变色】才可实现平衡                   |
| ------ | :------------------------: | :----------------------------------: | :----------------------------------------------------------: |
| 情况 1 | 当父节点为黑色时插入子节点 | 空树插入根节点，将根节点红色变为黑色 | 父节点为红色左节点，叔父节点为黑色，插入左子节点，那么通过【**左左节点旋转**】 |
| 情况 2 |             -              |       父节点和叔父节点都为红色       | 父节点为红色左节点，叔父节点为黑色，插入右子节点，那么通过【**左右节点旋转**】 |
| 情况3  |             -              |                  -                   | 父节点为红色右节点，叔父节点为黑色，插入左子节点，那么通过【**右左节点旋转**】 |
| 情况4  |             -              |                  -                   | 父节点为红色右节点，叔父节点为黑色，插入右子节点，那么通过【**右右节点旋转**】 |

接下来看一看这个方法：

```java
private void fixAfterInsertion(Entry<K,V> x) {
    // 新插入的节点为红色节点
    x.color = RED;
    // 当父节点为黑色时，并不需要进行树结构调整，只有当父节点为红色时，才需要调整
    while (x != null && x != root && x.parent.color == RED) {
        // 如果父节点是左节点，对应上表中情况1和情况2
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            // 如果叔父节点为红色，对应于“父节点和叔父节点都为红色”，此时通过变色即可实现平衡
            // 此时父节点和叔父节点都设置为黑色，祖父节点设置为红色
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                // 如果插入节点是黑色，插入的是右子节点，通过[左右节点旋转]（这里先进行父节点左旋）
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                // 设置父节点和祖父节点颜色
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                // 进行祖父节点右旋（这里【变色】和【旋转】并没有严格的先后顺序，达成目的就行）
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
            // 父节点是右节点的情况
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            // 对应于“父节点和叔父节点都为红色”，此时通过变色即可实现平衡
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                // 如果插入节点是黑色，插入的是左子节点，通过[右左节点旋转]（这里先进行父节点右旋）
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                // 进行祖父节点左旋（这里【变色】和【旋转】并没有严格的先后顺序，达成目的就行）
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    // 根节点必须为黑色
    root.color = BLACK;
}
```

源码中通过 rotateLeft 进行【**左旋**】，通过 rotateRight 进行【**右旋**】。都非常类似，就看一下【**左旋**】的代码，【左旋】规则如下：“逆时针旋转两个节点，让一个节点被其右子节点取代，而该节点成为右子节点的左子节点”。

```java
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        /**
         * 断开当前节点p与其右子节点的关联，重新将节点p的右子节点的地址指向节点p的右子节点的左子节点
         * 这个时候节点r没有父节点
         */
        Entry<K,V> r = p.right;
        p.right = r.left;
        // 将节点p作为节点r的父节点
        if (r.left != null)
            r.left.parent = p;
        // 将节点p的父节点和r的父节点指向同一处
        r.parent = p.parent;
        // p的父节点为null，则将节点r设置为root
        if (p.parent == null)
            root = r;
        // 如果节点p是左子节点，则将该左子节点替换为节点r
        else if (p.parent.left == p)
            p.parent.left = r;
        // 如果节点p为右子节点，则将该右子节点替换为节点r
        else
            p.parent.right = r;
        // 重新建立p与r的关系
        r.left = p;
        p.parent = r;
    }
}
```

参考下图。

![image-20200507002126750](assets/image-20200507002126750.png)



emmmm，后面的操作太多了，参考一个 NB 的帖子：https://www.cnblogs.com/LiaHon/p/11221634.html



#### TreeSet

- TreeSet 是 Set 的一个**子类**，TreeSet 集合没有重复元素。实现了**去重与有序**。
- 需要传入一个 Comparator 比较器对象进行排序，或者添加的元素实现了 Comparable 接口。
- 内部基于 **TreeMap** 实现。





#### 参考资料

- [TreeMap详解](https://www.cnblogs.com/LiaHon/p/11221634.html)









