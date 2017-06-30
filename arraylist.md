

##### 成员属性

```java
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer.
 */
private transient Object[] elementData;
```

elementData是ArrayList真正用来存储数据的数组。使用transient修饰表示该属性不参与序列化，因为整个数组并不一定全部存在有效数据，参与序列化很有可能会浪费空间。

---

```java
/**
 * The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;
```

有效元素的个数.

---

```java
/**
 * The maximum size of array to allocate.
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

这是数组的理论最大长度，如果是byte\[Integer.MAX\_VALUE\]，需要的内存是Integer.MAX\_VALUE，如果int\[Integer.MAX\_VALUE\]，那就是 4\*\[Integer.MAX\_VALUE\]，要是复杂对象，需要内存更多。具体为什么会写成 Integer.MAX\_VALUE - 8，是因为有些虚拟机会占用数组的开头几个长度，所以长度可能达不到 Integer.MAX\_VALUE，用 Integer.MAX\_VALUE - 8 比较保险。

---

##### 构造方法

ArrayList的构造方法有三个，一个是默认无参的，第二个是指定长度的，第三个是使用另一个集合\(Collection的子类\)创建。

```java
/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this(10);
}

/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
}

/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    size = elementData.length;
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```

1. 无参构造方法是默认的也是写代码时最常用的，内部调用的则是指定长度的构造方法，默认长度是10。但是这里要说的是使用这种方法效率是最低的，在使用时元素有很大可能会大于10，这会使ArrayList不断的扩容。
2. 指定长度的构造方法会先调用父类的构造方法，然后判断长度是否有效，无效则抛出参数异常，最后再初始化一个长度为 initialCapacity 的数组。

3. 使用另一个集合创建，使用集合的toArray方法，而长度也是形参中集合的长度。这里注释的意思是 toArray 方法不一定会返回一个 Object 数组，所以下面判断类型如果不是Object类型的数组则创建一个Object类型数组并且复制原有数据，最后使elementData指向新的数组。

---



##### 成员方法

```java
/**
 * Increases the capacity of this <tt>ArrayList</tt> instance, if
 * necessary, to ensure that it can hold at least the number of elements
 * specified by the minimum capacity argument.
 *
 * @param   minCapacity   the desired minimum capacity
 * "设置缓冲区大小"，是ArrayList对外提供的扩容方法。
 */
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
```

如果传入的参数大于当前数组的长度则开始扩容。

---

```java
/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
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

扩容 - 每次扩容最小是原有长度的1.5倍，最大不能大于Integer.MAX\_VALUE。

1.  获取现有长度 oldCapacity
2. 计算新长度 newCapacity，oldCapacity + \(oldCapacity &gt;&gt; 1\)就相当于 oldCapacity \* 1.5\(位运算右移一位相当于 / 2，又加上原长度，所以相当于 \* 1.5\)

3. 如果扩容一次的长度 newCapacity 小于传入的扩容数量则使用传入的扩容数\(就是说最小扩容量就是 长度 \* 1.5\)。

4. 如果新的长度大于理论最大值，那么就会调用 hugeCapacity\(\) 方法，这个方法下面会写，简而言之就是 minCapacity 如果比 MAX\_ARRAY\_SIZE 大就返回 MAX\_ARRAY\_SIZE + 8，如果不大于就返回 MAX\_ARRAY\_SIZE。

5. 创建一个长度为 newCapacity 的数组，copy旧数据到新数组，返回新数组，扩容成功。

---

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

大量扩容，如果传入扩容数量大于 MAX\_ARRAY\_SIZE 返回 Integer.MAX\_VALUE\(也就是\) MAX\_ARRAY\_SIZE + 8， 反之则返回 MAX\_ARRAY\_SIZE 。这里判断 minCapacity &lt; 0 会报内存溢出是因为如果 minCapacity &gt; Ineger.MAX\_VALUE 就会变成一个负数，因为二进制的第一位是符号位。

---

```java
/**
 * Returns the index of the first occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the lowest index <tt>i</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
 */
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

遍历查询元素位置。

---

```java
/**
 * Returns the index of the last occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the highest index <tt>i</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
 */
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

跟 indexOf 方法相同, 不过是反向遍历。

---

```java
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```

只返回有效元素

---

```java
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

数组转换\(指定类型\)，如果传入数组的长度小于elementData有效元素的个数，则返回一个新的数组。反之则调用System.arraycopy方法\(这里介绍一下五个参数的意思 arg1 - 源数组; arg2 - 源起始位置; arg3 - 目标数组; arg4 - 目标起始位置 arg5 - copy元素的个数\)，如果发现目标数组的长度比源数组要长，那么就把大于原数组长度部分的第一个元素置为null\(这里为什么这么写不懂\)。

---

```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

set方法。

---

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

add方法。使用ensureCapacityInternal方法判断当前有效元素数量size + 1是不是已经超过 elementData.length，如果超过就扩容，不超过就什么都不做。在下标为size的位置添加元素e, 之后size++。

---

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

1. add到指定位置\(index是下标值，是从0开始的\)。首先判断index是否小于0或者大于当前size，也就是说只能在下标0到下标size之间的位置进行add操作；
2. 之后使用ensureCapacityInternal方法判断是否需要扩容；
3. 使用arraycopy方法从源数组的index位置复制到源数组的index + 1的位置，复制size - index个元素，这样就相当于把下标为index的元素空了出来；
4. 把下标为index的元素覆盖为 element ；
5. size++。

---

```java
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
```

1. 根据下标删除元素\(index为下标\), 使用rangeCheck判断index是否有效\(这个rangeCheck没有判断小于0，所以可以传负数\)；
2. 因为删除不需要调用ensureCapacityInternal方法判断扩容所以这里写了modCount++；
3. 先把要删除的元素拿到用于返回；
4. 计算需要移动元素的个数\(这里需要写成 size - index - 1是因为size是个数而index是下标，两个值相差1，所以需要减1\)；
5. 判断如果需要移动元素则调用 arraycopy 方法，把从index + 1开始移动numMoved个元素，copy到index位置；先 --size，把删除前最后一个有效元素手动置为null，让gc更快的回收；
6. 返回删除的元素。

---

```java
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

/**
 * 快速删除, 简化版的 remove 方法
 */
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // Let gc do its work
}
```

比较元素内容删除。先判断传入的元素是否是 null ，如果为 null 则使用 == 比较，如果不为 null 则使用 equals 比较；遍历出 index；使用 fastRemove 方法进行删除。

---

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```

1. 添加一个集合。转换集合为数组；获取新集合的长度；
2. 判断当前有效元素个数+新集合的长度是否达到扩容标准\(ensureCapacityInternal内部会把modCount++\)；

3. 使用 arraycopy 合并内容；有效元素个数增加；返回结果\(这个方法为什么不在开始就校验新数组的长度并返回flase呢，不明白\)。

---

```java
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

1. 在指定位置添加集合。集合转换成数组；
2. 获取新数组的长度；

3. 是否需要扩容；

4. 计算需要移动元素的个数\(这里不像remove，删除的时候需要重合一个位置，所以这里不用减1\)；

5. 需要移动元素的话就调用 arraycopy 进行copy；

6. 然后再空出的位置中copy进新的数组；

7. 有效元素增加；

8. 返回结果\(同上面addAll的问题\)。

---

```java
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                     numMoved);

    // Let gc do its work
    int newSize = size - (toIndex-fromIndex);
    while (size != newSize)
        elementData[--size] = null;
}
```

1. 批量删除\(删除从fromIndex到toIndex中间的元素，fromIndex必须小于toIndex\)。
2. 先把操作数+1；
3. 计算需要移动的元素个数\(没有再减1是删除包含fromIndex但不包含toIndex\)；
4. 计算新的长度；
5. 依次置null尾部元素让GC尽快释放内存。

![](/assets/clipboard.png)

---

```java
// 删除集合
public boolean removeAll(Collection<?> c) {
    return batchRemove(c, false);
}

// 保留形参集合内的元素
public boolean retainAll(Collection<?> c) {
    return batchRemove(c, true);
}

// 批量删除
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```

批量删除\(删除集合和保留集合的实现逻辑\)

先来看看删除一个集合的运行过程\(这里只讨论正常过程，异常逻辑还没搞清楚\)：

1. 创建一个引用指向当前elementData;
2. 遍历elementData，判断如果当前元素包含在集合c中就什么也不做 w不变 r++；这时 w还指向第一个下标，r 已经指向下一个下标；第二次遍历如果还同第一次遍历那么w不动 r 继续前移；当发现有元素不存在于集合c当中，那么执行赋值语句，使下标为 r 的元素覆盖下标为 w 的元素；这样就完成了一次删除；重复上面的流程遍历结束后 elementData 中的数据应该是 "src1，src2，src3，src2，src3"；这说明删除的元素有两个\(这时 w == 3，r == size == 5\)。

3. finally中，正常情况应该是 r == size的，第一个if 不讨论。从第二个开始看 这时 w == 3 && w != size ；循环 i = 3 , elementData\[3\] = null 这里是把上一步覆盖后的遗留数据进行置 null 变为不可达对象让GC可以回收；操作数增加；size变更；返回true。

这样removeAll的流程就看完了，而retainAll的流程只有在上面判断是否包含的地方不一样，所以一个传 true，一个传false。

---

```java
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out array length
    s.writeInt(elementData.length);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++)
        s.writeObject(elementData[i]);

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }

}
```

重写writeObject方法。defaultWriteObject\(\)方法把非静态字段和非瞬时字段写入流，之后写入数组长度和有效元素，最后判断modCount是否变化，如果有变化代表有另外的线程修改了 elementData 中的数据。

---

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in array length and allocate array
    int arrayLength = s.readInt();
    Object[] a = elementData = new Object[arrayLength];

    // Read in all elements in the proper order.
    for (int i=0; i<size; i++)
        a[i] = s.readObject();
}
```

从流中读取。读取的arrayLength是elementData.lenth，创建新的 elementData ，依次读取元素赋值到数组中，多余的长度为 null。

---

```java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}
```

截取子串, 返回一个子串。

---

```java
private class SubList extends AbstractList<E> implements RandomAccess {
    private final AbstractList<E> parent;
    private final int parentOffset;
    private final int offset;
    int size;

    SubList(AbstractList<E> parent,
            int offset, int fromIndex, int toIndex) {
        this.parent = parent;
        this.parentOffset = fromIndex;
        this.offset = offset + fromIndex;
        this.size = toIndex - fromIndex;
        this.modCount = ArrayList.this.modCount;
    }
    ...
    ...
}
```

SubList内部类。从构造方法就可以看出在操作这个SubList其实就是在操作源List。

除了这一个内部类还有两个迭代器的内部类分别是："Itr" 和 “ListItr”，其中ListItr继承Itr。

```java
public E set(int index, E e) {
    rangeCheck(index);
    checkForComodification();
    E oldValue = ArrayList.this.elementData(offset + index);
    ArrayList.this.elementData[offset + index] = e;
    return oldValue;
}

public E get(int index) {
    rangeCheck(index);
    checkForComodification();
    return ArrayList.this.elementData(offset + index);
}

public int size() {
    checkForComodification();
    return this.size;
}

public void add(int index, E e) {
    rangeCheckForAdd(index);
    checkForComodification();
    parent.add(parentOffset + index, e);
    this.modCount = parent.modCount;
    this.size++;
}

public E remove(int index) {
    rangeCheck(index);
    checkForComodification();
    E result = parent.remove(parentOffset + index);
    this.modCount = parent.modCount;
    this.size--;
    return result;
}
```

这里除了size属性返回的是 subList 原有的，其余的操作都是在操作 parent.XXX。

