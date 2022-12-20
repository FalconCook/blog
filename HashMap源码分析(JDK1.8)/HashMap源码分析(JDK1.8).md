## （一）使用

分析任何源码之前都要先用一遍，HashMap也不例外。
```java
import java.util.HashMap;

public class Demo {
    public static void main(String[] args) {
        HashMap<Integer, String> hashMap = new HashMap<>();
        hashMap.put(1, "Cracker13");
        System.out.println(hashMap.get(1));
    }
}
```
这个段简单的代码，实现了HashMap存、取两个操作。HashMap的结构特点是以键值对的形式。
接下来我们就开始分析源码。

## （二）源码分析
整个代码多达2387行，我从上到下开始分析。
```java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
可以看出HashMap的数据结构是一个数组，这里定义了一个变量，初始化数组大小。
必须是2的次方。
&nbsp;
&nbsp;
```java
/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<30.
 */
static final int MAXIMUM_CAPACITY = 1 << 30;
```
这里定义了最大的容量大小，超过这个值就将threshold修改为Integer.MAX_VALUE。
&nbsp;
&nbsp;
```java
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
负载因子，即hashmap扩容的阈值。设小了浪费空间，设太大会造成频繁的hash冲突。
&nbsp;
&nbsp;
```java
/**
 * The bin count threshold for using a tree rather than list for a
 * bin.  Bins are converted to trees when adding an element to a
 * bin with at least this many nodes. The value must be greater
 * than 2 and should be at least 8 to mesh with assumptions in
 * tree removal about conversion back to plain bins upon
 * shrinkage.
 */
static final int TREEIFY_THRESHOLD = 8;
```
链表大于这个值就会树化，这里牵扯出hashmap的另一个数组结构。即hash冲突的时候，数组的某一点形成一个链表，随着冲突继续链表不断延长，延长到某个长度，整个链表转换成红黑树。
![](https://img-blog.csdnimg.cn/img_convert/057ba2be961e62260bd0e7c2f923434c.png)
![](https://img-blog.csdnimg.cn/img_convert/20631897aac34e8bf2189cdd8d5cc500.png)
&nbsp;
&nbsp;
```java
/**
 * The bin count threshold for untreeifying a (split) bin during a
 * resize operation. Should be less than TREEIFY_THRESHOLD, and at
 * most 6 to mesh with shrinkage detection under removal.
 */
static final int UNTREEIFY_THRESHOLD = 6;
```
小于这个值就会反树化
&nbsp;
&nbsp;
```java
/**
 * The smallest table capacity for which bins may be treeified.
 * (Otherwise the table is resized if too many nodes in a bin.)
 * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
 * between resizing and treeification thresholds.
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```
最小树形化容量阈值，即当哈希表中的容量大于该值时，才允许树形化链表。
&nbsp;
&nbsp;
```java
/**
 * Basic hash bin node, used for most entries.  (See below for
 * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
 */
static class Node<K,V> implements Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Entry<?,?> e = (Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```
数组的节点类
&nbsp;
&nbsp;
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
这里对key进行操作，来确定value存在数组的哪个位置。
先调用key的hashCode()，返回一个int型整数，我们知道int有32位，高16位与低16位做一个异或运算，得到的一个新的int。从而使结果充分地散列。
&nbsp;
&nbsp;
```java
/**
 * Returns x's Class if it is of the form "class C implements
 * Comparable<C>", else null.
 */
static Class<?> comparableClassFor(Object x) {
    if (x instanceof Comparable) {
        Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
        if ((c = x.getClass()) == String.class) // bypass checks
            return c;
        if ((ts = c.getGenericInterfaces()) != null) {
            for (int i = 0; i < ts.length; ++i) {
                if (((t = ts[i]) instanceof ParameterizedType) &&
                    ((p = (ParameterizedType)t).getRawType() ==
                     Comparable.class) &&
                    (as = p.getActualTypeArguments()) != null &&
                    as.length == 1 && as[0] == c) // type arg is c
                    return c;
            }
        }
    }
    return null;
}
```
红黑树相关
&nbsp;
&nbsp;
```java
/**
 * Returns k.compareTo(x) if x matches kc (k's screened comparable
 * class), else 0.
 */
@SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
static int compareComparables(Class<?> kc, Object k, Object x) {
    return (x == null || x.getClass() != kc ? 0 :
            ((Comparable)k).compareTo(x));
}
```
红黑树相关
&nbsp;
&nbsp;
```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
我们可以指定数组初始大小，返回最接近initialCapacity的2的次方。比如我们指定大小为31，结果返回32。
具体原理如图。
![](https://img-blog.csdnimg.cn/img_convert/8c2227a7030435db72e90c3aaaed7e7a.png)
&nbsp;
&nbsp;
```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```
构造方法1，指定初始容量大小，负载因子。
&nbsp;
&nbsp;
```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```
构造方法2， 负载因子用的默认值。
&nbsp;
&nbsp;
```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```
空参构造，均使用默认值。
&nbsp;
&nbsp;
```java
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```
构造方法4
&nbsp;
&nbsp;
```java
/**
 * Implements Map.putAll and Map constructor
 *
 * @param m the map
 * @param evict false when initially constructing this map, else
 * true (relayed to method afterNodeInsertion).
 */
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```
赋值方法。
&nbsp;
&nbsp;
```java
    public int size() {
        return size;
    }
```
返回大小
&nbsp;
&nbsp;
```java
    public boolean isEmpty() {
        return size == 0;
    }
```
判空
&nbsp;
&nbsp;
```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```
查找value
&nbsp;
&nbsp;
```java
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //1、判断当前数组不为空并长度大于0 && 由key的hash值找到对应数组下的桶（可能是红黑树或者链表）
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //2、先判断桶的第一个节点 如果key一致 返回
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //3、再判空下一个节点不为空 && 判断是红黑树还算链表
            if ((e = first.next) != null) {
                //4、如果是红黑树 则按红黑树方式取值
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //否则就是链表，遍历取值
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```
查找value的具体方法。```first = tab[(n - 1) & hash]```是重点。
前面提到为什么数组的大小要为2的次方，就是为了更加均匀分布在数组上，这里比较抽象。
&nbsp;
&nbsp;
```java
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
```
判断是否包含key
&nbsp;
&nbsp;
```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
添加value
&nbsp;
&nbsp;
```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        //数组+链表+红黑树，链表型（Node泛型)数组，每一个元素代表一条链表，则每个元素称为桶
        //HashMap 的每一个元素，都是链表的一个节点（Entry<K,V>）这里也就是Node<K,V>

        //tab:桶 p:桶 n:哈希表数组大小 i:数组下标(桶的位置)
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //1.判断当前桶是否为空，空的就调用resize()方法（resize 中会判断是否进行初始化）
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //2.判断是否有hash冲突，根据入参key与key的hash值找到具体的桶并判空，空则无冲突 直接新建桶
        //？为什么采用(n - 1) & hash计算数组下标，感兴趣的可以深入了解
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //3.以下表示有冲突，处理hash冲突
        else {
            Node<K,V> e; K k;
            //4.判断当前桶的key是否与入参key一致，一致则存在，把当前桶p赋值给e,覆盖原 value 在步骤10进行
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //5.如果当前的桶为红黑树，用putTreeVal()方法写入 赋值给e
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //6.则当前的桶是链表 遍历链表
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        //7.尾插法，链表下一个节点是null（链表末尾），就new一个新节点写入到当前链表节点的后面
                        p.next = newNode(hash, key, value, null);
                        //8.判断是否大于阈值，需要链表转红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            //binCount从0开始的, 所以当binCount为7时，链表长度为8（算上数组槽位开始的那个节点，总长度为9），则需要树化桶
                            treeifyBin(tab, hash);
                        break;
                    }
                    //9.与步骤4一致，如果链表中key存在则直接跳出 步骤10覆盖原值
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //10.存在相同的key的Node节点,则覆盖原value
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                //onlyIfAbsent为true：不改变原来的值 ；false： 改变原来的值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                //LinkedHashMap用到的回调方法
                afterNodeAccess(e);
                return oldValue;
            }
        }
        /*记录修改次数标识
         用于fast-fail，由于HashMap非线程安全，在对HashMap进行迭代时，
         如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），
         需要抛出异常ConcurrentModificationException
         */
        ++modCount;
        //11.容量超过阈值，扩容
        if (++size > threshold)
            resize();
        //LinkedHashMap用到的回调方法
        afterNodeInsertion(evict);
        return null;
    }
```
添加value具体方法。
&nbsp;
&nbsp;
```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        //1、原数组扩容
        if (oldCap > 0) {
            //如果原数组长度大于最大容量，把阈值调最大，return
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //把原数组大小、阈值都扩大一倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        // 初始化指定的初始化容量
        // 使用了指定initialCapacity的构造方法，则用原阈值作为新容量
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        //使用空参构造，用默认值
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //使用了指定initialCapacity的构造方法，新阈值为0，则计算新的阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //2、用新的数组容量大小初始化数组
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //如果仅仅是初始化过程,到此结束 return newTab
        table = newTab;
        //3、开始扩容的主要工作，数据迁移
        if (oldTab != null) {
            //遍历原数组开始复制旧数据
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;//清除旧表引有
                    //原数组中单个元素，直接复制到新表
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果该元素类型是红黑树，按红黑树方式处理
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        //这段代码设计巧妙，环环相扣啊
                        //先定义了两种类型的链表 以及头尾节点 高位链表与低位链表
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        //按顺序遍历原链表的节点
                        do {
                            next = e.next;
                            //这是一个核心的判断条件，感兴趣的可以深入了解？为什么这么做
                            //=0则放到低位链表
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            //否则放到高位链表
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                            //以上实际上就是对原来链表拆分成了两个高低位链表
                        } while ((e = next) != null);
                        //把整个低位链表放到新数组的j位置
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        //把整个高位链表放到新数组的j+oldCap位置上
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```
扩容的触发条件：
扩容的触发条件通过一个叫(threshold)阈值的参数控制的,阈值的计算公式是数组长度threshold(阈值)=capacity(容量)*loafFactor(负债因子)。以后往HashMap填充的数据如果到达一定量，就把旧数组扩大一倍。
经过观测可以发现：
我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。
`e.hash & oldCap`这行代码跟扩容、链表重排列有关系
&nbsp;
&nbsp;
```java
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```
红黑树相关
&nbsp;
&nbsp;
```java
    public void putAll(Map<? extends K, ? extends V> m) {
        putMapEntries(m, true);
    }
```
添加一个map
&nbsp;
&nbsp;
```java
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }
```
删除对应key的value
&nbsp;
&nbsp;
```java
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```
删除对应key的value具体方法。
&nbsp;
&nbsp;
```java
    public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
```
清空数组
&nbsp;
&nbsp;
```java
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```
判断是否包含某个value
&nbsp;
&nbsp;
```java
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }
```
返回key的集合
&nbsp;
&nbsp;
```java
    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super K> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.key);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```



```java
    public Collection<V> values() {
        Collection<V> vs = values;
        if (vs == null) {
            vs = new Values();
            values = vs;
        }
        return vs;
    }
```
返回value集合
&nbsp;
&nbsp;
```java
   final class Values extends AbstractCollection<V> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<V> iterator()     { return new ValueIterator(); }
        public final boolean contains(Object o) { return containsValue(o); }
        public final Spliterator<V> spliterator() {
            return new ValueSpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super V> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.value);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```
```java
    public Set<Entry<K,V>> entrySet() {
        Set<Entry<K,V>> es;
        return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
    }
```
返回键值对集合
&nbsp;
&nbsp;
```java
    final class EntrySet extends AbstractSet<Entry<K,V>> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<Entry<K,V>> iterator() {
            return new EntryIterator();
        }
        public final boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Entry<?,?> e = (Entry<?,?>) o;
            Object key = e.getKey();
            Node<K,V> candidate = getNode(hash(key), key);
            return candidate != null && candidate.equals(e);
        }
        public final boolean remove(Object o) {
            if (o instanceof Map.Entry) {
                Entry<?,?> e = (Entry<?,?>) o;
                Object key = e.getKey();
                Object value = e.getValue();
                return removeNode(hash(key), key, value, true, true) != null;
            }
            return false;
        }
        public final Spliterator<Entry<K,V>> spliterator() {
            return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super Entry<K,V>> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```
&nbsp;
&nbsp;
```java
    /**
     * Save the state of the <tt>HashMap</tt> instance to a stream (i.e.,
     * serialize it).
     *
     * @serialData The <i>capacity</i> of the HashMap (the length of the
     *             bucket array) is emitted (int), followed by the
     *             <i>size</i> (an int, the number of key-value
     *             mappings), followed by the key (Object) and value (Object)
     *             for each key-value mapping.  The key-value mappings are
     *             emitted in no particular order.
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws IOException {
        int buckets = capacity();
        // Write out the threshold, loadfactor, and any hidden stuff
        s.defaultWriteObject();
        s.writeInt(buckets);
        s.writeInt(size);
        internalWriteEntries(s);
    }

    /**
     * Reconstitute the {@code HashMap} instance from a stream (i.e.,
     * deserialize it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws IOException, ClassNotFoundException {
        // Read in the threshold (ignored), loadfactor, and any hidden stuff
        s.defaultReadObject();
        reinitialize();
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new InvalidObjectException("Illegal load factor: " +
                                             loadFactor);
        s.readInt();                // Read and ignore number of buckets
        int mappings = s.readInt(); // Read number of mappings (size)
        if (mappings < 0)
            throw new InvalidObjectException("Illegal mappings count: " +
                                             mappings);
        else if (mappings > 0) { // (if zero, use defaults)
            // Size the table using given load factor only if within
            // range of 0.25...4.0
            float lf = Math.min(Math.max(0.25f, loadFactor), 4.0f);
            float fc = (float)mappings / lf + 1.0f;
            int cap = ((fc < DEFAULT_INITIAL_CAPACITY) ?
                       DEFAULT_INITIAL_CAPACITY :
                       (fc >= MAXIMUM_CAPACITY) ?
                       MAXIMUM_CAPACITY :
                       tableSizeFor((int)fc));
            float ft = (float)cap * lf;
            threshold = ((cap < MAXIMUM_CAPACITY && ft < MAXIMUM_CAPACITY) ?
                         (int)ft : Integer.MAX_VALUE);
            @SuppressWarnings({"rawtypes","unchecked"})
                Node<K,V>[] tab = (Node<K,V>[])new Node[cap];
            table = tab;

            // Read the keys and values, and put the mappings in the HashMap
            for (int i = 0; i < mappings; i++) {
                @SuppressWarnings("unchecked")
                    K key = (K) s.readObject();
                @SuppressWarnings("unchecked")
                    V value = (V) s.readObject();
                putVal(hash(key), key, value, false, false);
            }
        }
    }
```
非默认序列化：
HashMap 并没有使用默认的序列化机制，table变量被transient修饰，无法序列化，而是通过实现`readObject/writeObject`两个方法自定义了序列化的内容。
因为序列化 talbe 存在着两个问题：
- table 多数情况下是无法被存满的，序列化未使用的部分，浪费空间
- 同一个键值对在不同 JVM 下，所处的桶位置可能是不同的，在不同的 JVM 下反序列化 table 可能会发生错误

## （三）参考资料
https://www.bilibili.com/video/BV1cW411g7RH

