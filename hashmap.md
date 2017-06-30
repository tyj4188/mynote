```java
/**
 * 默认初始长度.这样做的原因就是因为计算机进行2次幂的运算是非常高效的
 * , 仅通过位移操作就可以完成2的N次幂的运算
 */
static final int DEFAULT_INITIAL_CAPACITY = 16;
```

```java
/**
 * 最大长度. 如果在有参构造方法中指定的长度大于这个值那么将使用这个值.
 * 也就是说长度必须大于0, 小于 1 << 30, 并且是2的倍数.
 */
static final int MAXIMUM_CAPACITY = 1 << 30; // 2的30次方
```

```java
/**
 * 默认负载因子.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

```java
/**
 * Key/Value数组, 真正用来存数据的. 长度总是2的倍数.
 */
transient Entry<K,V>[] table; // 不参与序列化
```

```java
/**
 * table中有效元素的个数.
 */
transient int size; // 不参与序列化
```

```java
/**
 * 触发扩容操作的临界值 (默认达到总长度的75%会触发扩容).
 * @serial
 */
int threshold;
```

```java
/**
 * HashMap的负载因子.
 * @serial
 */
final float loadFactor;
```

```java
/**
 * If {@code true} then perform alternative hashing of String keys to reduce
 * the incidence of collisions due to weak hash code calculation.
 * 是否要对字符串键的HashMap使用备选哈希函数.
 * 备选哈希函数的使用可以减少由于对字符串键进行弱哈希码计算时的碰撞概率
 */
transient boolean useAltHashing;
```

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity
 * (16) and the default load factor (0.75).
 */
public HashMap() {
    // 最常用的构造方法, 长度16、扩容因子0.75.
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}
```

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and the default load factor (0.75).
 *
 * @param  initialCapacity the initial capacity.
 * @throws IllegalArgumentException if the initial capacity is negative.
 */
public HashMap(int initialCapacity) {
    // 指定长度, 但是使用默认的扩容因子.
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // 如果指定长度大于限制最大长度则使用最大长度
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);

    // Find a power of 2 >= initialCapacity
    int capacity = 1;
    // 循环位移, 找到一个最接近指定长度的2的倍数
    while (capacity < initialCapacity)
        capacity <<= 1; // << 1 == * 2

    this.loadFactor = loadFactor;
    // 扩容操作的临界值, 当元素数量大于临界值则触发扩容操作.
    threshold = (int)Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    // 初始化 table
    table = new Entry[capacity]; 
    useAltHashing = sun.misc.VM.isBooted() &&
            (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    init(); // init只在LinkedHashMap中有实现，主要在构造函数初始化和clone、readObject中有调用
}
```

```java
/**
 * Constructs a new <tt>HashMap</tt> with the same mappings as the
 * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
 * default load factor (0.75) and an initial capacity sufficient to
 * hold the mappings in the specified <tt>Map</tt>.
 *
 * @param   m the map whose mappings are to be placed in this map
 * @throws  NullPointerException if the specified map is null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    /**
     * 长度为 (m.size / 0.75) + 1 与 DEFAULT_INITIAL_CAPACITY 中
     * 比较大的那个值.
     */
    this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                  DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
    putAllForCreate(m); // for循环put
}
```

```java
/**
 * Retrieve object hash code and applies a supplemental hash function to the
 * result hash, which defends against poor quality hash functions.  This is
 * critical because HashMap uses power-of-two length hash tables, that
 * otherwise encounter collisions for hashCodes that do not differ
 * in lower bits. Note: Null keys always map to hash 0, thus index 0.
 * 
 * 这个hash算法不懂是什么意思只知道返回一个整数类型的hash值
 */
final int hash(Object k) {
    int h = 0;
    /**
     * 如果 useAltHashing == true 并且 key 类型为 String 
     * 则使用备用 hash 算法.
     */
    if (useAltHashing) {
        if (k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }
        h = hashSeed;
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

hash方法中的二次hash感觉应该是为了让hash值能更均匀的分布在除符号位的另外31bit中进而减少hash碰撞的几率, 比如说下图\(转自[http://www.cnblogs.com/rogerluo1986/p/5851300.html\)：假设](http://www.cnblogs.com/rogerluo1986/p/5851300.html%29：假设) k.hashCode\(\) == 0x7FFFFFFF

![](http://images2015.cnblogs.com/blog/1020081/201609/1020081-20160907220904519-303306849.png)

```java
/**
 * Returns index for hash code h.
 */
static int indexFor(int h, int length) {
    /**
     * 根据hash值计算在table中的下标值, 使用位与的写法效果跟取余相同
     * , 但并不完全等同于 '%' 运算
     */
    return h & (length-1);
}

/**
 * Returns the number of key-value mappings in this map.
 *
 * @return the number of key-value mappings in this map
 */
public int size() {
    // 返回有效元素个数
    return size;
}

/**
 * Returns <tt>true</tt> if this map contains no key-value mappings.
 *
 * @return <tt>true</tt> if this map contains no key-value mappings
 */
public boolean isEmpty() {
    // 是否空
    return size == 0;
}
```

```java
/**
 * HashMap 的 get 方法, 根据 key 获取 value.
 */
public V get(Object key) {
    // HashMap是允许 null 作为key的, 位置是 table 下标为 0 的位置
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}

/**
 * 存在返回ture, 否则返回false.
 */
public boolean containsKey(Object key) {
    return getEntry(key) != null;
}

/**
 * 获取 key == null 的元素
 */
private V getForNullKey() {
    // 找到下标为0的bucket, 如果里面有元素则顺着链表往后找, 直到末尾或者找到 key == null 的元素
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}

/**
 * Returns the entry associated with the specified key in the
 * HashMap.  Returns null if the HashMap contains no mapping
 * for the key.
 */
final Entry<K,V> getEntry(Object key) {
    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

get\(\)和contains\(\)方法都使用了getEntry\(\)方法, getEntry\(\)是final方法, 不能继承和重写。看一下执行的逻辑 :

1. 首先判断 key == null时 hash 值为0 否则调用上面的hash方法计算hash值.

2. 根据上一步返回的hash码, 使用indexFor\(\)方法计算出在table中的下标值, 如果上面因为key == null导致hash值为0, 那么indexFor\(上面有介绍\)方法也会返回下标0, 这样方法的逻辑就会跟getForNullKey\(\)相同了, 保证了如果key == null时逻辑的准确性。如果key != null那么就会返回index &gt;= 0 && index &lt; table.length。

3. 然后看当前下标中是否有有效元素, 每次循环都顺着链表往后找 : e = e.next。

4. 如果有有效元素则先判断当前元素的hash值, 然后用"=="判断, 这是为了配合第2步中如果key == null的情况, 最后才用 equals 判断是否相等, 如果匹配到了就返回当前的元素, 否则返回null。

```java
/**
 * 虽说是看HashMap的源码主要是put方法, 但是突然感觉没啥好写的
 */
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
            e.recordAccess(this); // 这个方法什么也没做
            return oldValue;
        }
    }

    modCount++; // 替换是不加操作数的
    addEntry(hash, key, value, i);
    return null;
}

/**
 * null键的put, 这个方法没什么好说的, 把key == null的元素放到下标0的位
 * 置, 如果下标0的bucket中已经有数据就顺着链表找, 找到就更新value返回之前的
 * value, 如果没找到就增加新的节点, 并返回null.
 */
private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this); // 这个方法什么也没做
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
```

1. put的时候先判断Key是否为null, 如果是null则调用 nullKey的put方法.
2. 如果key != null, 使用hash方法\(hash方法上面有介绍,反正就是把对象映射成一个int值\)计算出key的hash值, 通过indexFor方法计算下标\(类似求余\).
3. 剩下的就简单了, 如果当前下标 i 中已有数据就顺着链表结构往下找, 一边找一边比较\(蝌蚪找妈妈?\), 找到就替换新的value返回旧的value, 找不到就增加新的节点, 并且操作数加1.

```java
/**
 * Adds a new entry with the specified key, value and hash code to
 * the specified bucket.  It is the responsibility of this
 * method to resize the table if appropriate.
 *
 * Subclass overrides this to alter the behavior of put method.
 */
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    createEntry(hash, key, value, bucketIndex);
}

/**
 * Like addEntry except that this version is used when creating entries
 * as part of Map construction or "pseudo-construction" (cloning,
 * deserialization).  This version needn't worry about resizing the table.
 *
 * Subclass overrides this to alter the behavior of HashMap(Map),
 * clone, and readObject.
 */
void createEntry(int hash, K key, V value, int bucketIndex) {
    // 下面两句代码完成了头结点的替换
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}

/**
 * Creates new entry.
 * Entry的构造方法
 */
Entry(int h, K k, V v, Entry<K,V> n) {
    value = v;
    next = n; // next指向下一个节点
    key = k;
    hash = h;
}
```

增加节点 :

1. 判断当前size是否大于threshold\(临界值\) 并且当前bucketIndex位置已经有有效元素, 则触发扩容操作\(扩容后面写\)。
2. 把当前下标内的头结点赋给一个临时变量, 创建一个新的Entry让Entry.next-&gt;Old\_Head\_Node, 新的节点的next引用指向之前的头结点, 这样新的Entry就变成了头结点, 之前的头结点变成了头结点的下一个节点, 再把新的头结点放到table中. 这样就完成了新元素的添加.
3. 最后 size 加 1 .

```java
/**
 * Rehashes the contents of this map into a new array with a
 * larger capacity.  This method is called automatically when the
 * number of keys in this map reaches its threshold.
 *
 * If current capacity is MAXIMUM_CAPACITY, this method does not
 * resize the map, but sets threshold to Integer.MAX_VALUE.
 * This has the effect of preventing future calls.
 *
 * @param newCapacity the new capacity, MUST be a power of two;
 *        must be greater than current capacity unless current
 *        capacity is MAXIMUM_CAPACITY (in which case value
 *        is irrelevant).
 */
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    boolean oldAltHashing = useAltHashing;
    useAltHashing |= sun.misc.VM.isBooted() &&
            (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    // 是否需要重新hash
    boolean rehash = oldAltHashing ^ useAltHashing; 
    transfer(newTable, rehash);
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

/**
 * Transfers all entries from current table to newTable.
 */
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            // 是否重新hash
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i]; // 当前下标中的元素赋给插入节点的下一个
            newTable[i] = e; // 这样把当前元素变成链表头
            e = next; // 循环下个节点
        }
    }
}
```

1. 如果table.length == MAXIMUM\_CAPACITY 就不做扩容, 把 threshold 设置成 Integer.MAX\_VALUE 然后直接 return.
2. 执行后面操作假设是第一次扩容, 初始的table.length = 16, 那么newCapacity == 32\(调用resize时length \* 2\).
3. 创建一个长度为那么newCapacity的新数组.
4. 调用transfer方法把table中的数据转到newTable中, 外面的for循环遍历table中的元素, 如果当前下标中元素不为null, 则while循环Entry的链表结构, 并且重新计算下标值. 这样循环完了之后会把链表尾变成了链表头. 
5. table指向newTable.
6. 计算新的threshold.

```java
/**
 * Removes and returns the entry associated with the specified key
 * in the HashMap.  Returns null if the HashMap contains no mapping
 * for this key.
 */
final Entry<K,V> removeEntryForKey(Object key) {
    int hash = (key == null) ? 0 : hash(key); // key的hash值
    int i = indexFor(hash, table.length); // 计算下标
    Entry<K,V> prev = table[i]; // 前驱节点
    Entry<K,V> e = prev; // 当前节点

    while (e != null) {
        Entry<K,V> next = e.next; // 后续节点
        Object k; // 当前元素的key
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            if (prev == e) // 如果第一个节点就中了
                table[i] = next; // 直接把头结点的后续节点当头结点
            else
                prev.next = next; // 前驱节点直接指向后续节点
            e.recordRemoval(this); // 方法什么也没做
            return e; // 返回匹配到的节点
        }
        prev = e; // 当前节点变前驱
        e = next; // 遍历下个节点
    }

    return e; // 没匹配到返回 null
}
```



