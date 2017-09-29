---
title: java集合框架-ArrayList
date: 2017-09-29 13:50:53
categories: java集合
tags:
  - java
  - ArrayList
  - 集合
---

# ArrayList特点概述
 - 是实现了List接口的变长数组(暗示其内部存储数据结构为数组)
 - 是有序的
 - 允许所有元素为null
 - 允许元素重复
 - 有一个容量，指存储元素的数组大小，至少和列表大小一样，可自动增长；当没有足够空间存储元素时，一次增长1.5倍（扩容太多会导致浪费更多的内存，扩容太少会导致频繁扩容操作，内存数据中的频繁copy，降低了性能）；在使用的过程中，开发者应该先预见可能需要扩容的大小，调用ensureCapacity
方法扩容，避免添加元素频繁的扩容。
 - 非线程安全的，与Vector相反；可以通过Collections.synchronizedList(...)包装成同步List；但是这种方式生成的同步List，通过迭代器(iterator和listIterator)访问时又不是线程安全的，因为Collections.synchronizedList中没有对迭代器的方法加synchronized访问。
 - 实现了[RandomAccess](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/RandomAccess.java#RandomAccess)标记接口（此接口的主要目的是允许一般的算法更改其行为，从而在将其应用到随机或连续访问列表时能提供良好的性能），支持快速随机访问，比使用迭代器遍历的方式高效；相对LinkedList，查找，修改元素高效，添加和删除没有LinkedList高效
 - 实现了[Serializable](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/io/Serializable.java#Serializable)接口，可以序列化

# 源码角度分析
> 文章中分析的源码基于jdk7,我分析的是我本地openjdk的代码，文中给的连接是[grepcodejdk7(7u40-b43)](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java)中的，可能有些地方不一样，但没有大的影响，希望大家谅解。

既然是存储数据元素的容器那么我们就从容器的数据结构；构造方法；元素操作（增、删、改、查）；和附加操作的角度展开分析

## 数据结构

{% codeblock lang:java %}
    private static final int DEFAULT_CAPACITY = 10; //默认初始化容量.

    private static final Object[] EMPTY_ELEMENTDATA = {}; //空的Object数组.

    private transient Object[] elementData; //用户存储数据元素的数组，不可序列化.

    private int size; //包含元素的个数

    /*
    *用于记录列表中的元素发生结构性变化次数(add、remove、clear等操作)；在多线程环境下，在使用迭代器遍历列表时，需要保持单线程的唯一操作，
    *如果另一个线程修改了列表的结构则会抛出ConcurrentModificationException异常
    */
    protected transient int modCount = 0; 
{% endcodeblock %}

## 构造方法

{% codeblock lang:java %}
    public ArrayList(int initialCapacity) { //根据指定的初始容量初始化存储数据元素的数组
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

    public ArrayList() { //初始化容量为空的数据元素存储数组.
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) { //初始化数据元素数组，容量为指定容器包含的数据元素个数，并将指定容器中的数据元素添加到数据元素数组中
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
{% endcodeblock %}

## 增加元素

添加元素到size+1处

{% codeblock lang:java %}
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 确定容量，如果容量不足就扩容
        elementData[size++] = e; //将元素添加到数组中，由此可见ArrayList是有序的。
        return true;
    }
{% endcodeblock %}

添加元素到指定游标处

{% codeblock lang:java %}
    public void add(int index, E element) {
        rangeCheckForAdd(index); //检查指定游标是否在容器的容量范围内

        ensureCapacityInternal(size + 1);  // 确定容量，如果容量不足就扩容
        System.arraycopy(elementData, index, elementData, index + 1, size - index); //将数组中原来index之后的元素拷贝到index+1处
        elementData[index] = element;//将指定元素添加到指定游标处
        size++; //容器元素个数加1
    }
{% endcodeblock %}

添加元素时，对指定的游标进行范围检查如果没有在0到size的范围内抛出IndexOutOfBoundsException异常

{% codeblock lang:java %}
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
{% endcodeblock %}

将指定容器中的元素添加到List中，列表的size+1处

{% codeblock lang:java %}
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // 确定容量(size+c.size())，如果容量不足就扩容
        System.arraycopy(a, 0, elementData, size, numNew); //将元素添加到List的结尾处
        size += numNew; //修改容器元素个数
        return numNew != 0;
    }
{% endcodeblock %}

将指定容器中的元素添加到List的指定游标处

{% codeblock lang:java %}
    public boolean addAll(int index, Collection<? extends E> c) { //将指定容器中的元素添加到指定游标处
        rangeCheckForAdd(index); //检查指定游标是否在容器的容量范围内

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // 确定容量(size+c.size())，如果容量不足就扩容

        int numMoved = size - index; //需要移动元素个数
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew, numMoved); //移动指定游标后的元素到index + numNew

        System.arraycopy(a, 0, elementData, index, numNew); //将指定容器中的元素添加到指定游标处
        size += numNew; //修改容器元素个数
        return numNew != 0;
    }
{% endcodeblock %}

既然ArrayList的元素数据存储结构是数组，那么肯定面临容量不足的问题，同时在添加元素之前就要判断容量

{% codeblock lang:java %}
    private void ensureCapacityInternal(int minCapacity) { //确定需要的容量minCapacity
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity); //默认容量是10
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) { //确保明确容量
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0) //如果需要的容量大于当前容器的容量，进行扩容操作
            grow(minCapacity); //扩容
    }
{% endcodeblock %}

数组分配的最大大小，可参考文末资料，为什么数组的最大容量是Integer.MAX_VALUE - 8;

{% codeblock lang:java %}
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8; 
{% endcodeblock %}

数组的内存大小在请求的时候是固定的，所以要扩容就需要重新申请更大的数组空间，并将原来数组空间中的元素copy到新的数组空间中

{% codeblock lang:java %}
    private void grow(int minCapacity) { //扩容
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); //扩容1.5倍
        if (newCapacity - minCapacity < 0) //扩容后的容量小于需要的容量
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)  //扩容的容量大于数组分配的最大大小
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity); //扩容后内存中数据元素从老的数组中拷贝到新的数组中
    }

    private static int hugeCapacity(int minCapacity) { //超大容量
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
{% endcodeblock %}

## 删除元素

删除指定游标处的元素

{% codeblock lang:java %}
    public E remove(int index) {
        rangeCheck(index); //游标范围检查

        modCount++;
        E oldValue = elementData(index); //取出游标处的元素

        int numMoved = size - index - 1; //需要移动的元素个数
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved); //移动index之后的元素到index处，这时要删除的元素是数组中的最后一个
        elementData[--size] = null; // 非常重要，将最后一个要删除的元素赋值为空，便于gc

        return oldValue;
    }
{% endcodeblock %}

检查游标是否在size的范围内，如果没有抛出IndexOutOfBoundsException异常

{% codeblock lang:java %}
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
{% endcodeblock %}

删除指定的Object，因为列表允许插入null，所以比较元素时为了防止空指针错误，<span style="color:orange;">同时ArrayList删除元素是通过equals比较删除的，而不是==比较</span>

{% codeblock lang:java %}
    public boolean remove(Object o) { //删除指定的Object
        if (o == null) { //object为空时
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index); //删除元素
                    return true;
                }
        } else { //object不为空
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index); //删除元素
                    return true;
                }
        }
        return false;
    }

    private void fastRemove(int index) { //该方法类似于上述的public E remove(int index) {...}
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
{% endcodeblock %}

删除所有元素

{% codeblock lang:java %}
    public void clear() { 
        modCount++;

        // clear时将所有元素设置为空，方便gc
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0; //容器元素个数修改为0
    }
{% endcodeblock %}

删除一定范围内的元素

{% codeblock lang:java %}
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex; //移动元素个数
        System.arraycopy(elementData, toIndex, elementData, fromIndex, numMoved); //移动元素

        // 同上，需要将容器中已经删除的元素引用置为空，方便gc
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }
{% endcodeblock %}

删除指定容器中包含的元素

{% codeblock lang:java %}
    public boolean removeAll(Collection<?> c) {
        return batchRemove(c, false);
    }
{% endcodeblock %}

删除指定容器中不包含的元素，与removeAll相反，补集

{% codeblock lang:java %}
    public boolean retainAll(Collection<?> c) {
        return batchRemove(c, true);
    }
{% endcodeblock %}

删除或不删除指定容器中的元素，complement为false就保留不包含指定集合的元素(removeAll)，反之包含(retainAll)

{% codeblock lang:java %}
    private boolean batchRemove(Collection<?> c, boolean complement) { 
        final Object[] elementData = this.elementData; //一个局部容器数组
        int r = 0, w = 0; //r是遍历容器的游标，w是包含指定容器的元素游标
        boolean modified = false;
        try {
            for (; r < size; r++) //将包含于指定容器中的元素放置在局部容器数组的头部
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {  //兼容处理，如果上述步骤的contains方法抛出异常时，需要将未处理的元素添加进来
                System.arraycopy(elementData, r, elementData, w, size - r);
                w += size - r;
            }
            if (w != size) { //如果遍历完列表，则说明需要保留的元素是w个，如果w小于size就要将之后的数据引用置为null，便于gc
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
{% endcodeblock %}

## 修改元素

修改指定游标处的元素，并返回游标处的旧元素

{% codeblock lang:java %}
public E set(int index, E element) {
        rangeCheck(index); //游标范围校验

        E oldValue = elementData(index); //查找到指定游标处的旧元素
        elementData[index] = element; 
        return oldValue;
    }
{% endcodeblock %}

## 查找元素

查找指定游标处的元素

{% codeblock lang:java %}
    public E get(int index) {
        rangeCheck(index); //游标范围校验

        return elementData(index);
    }
{% endcodeblock %}

查找指定元素第一次出现的游标，如果没有找到返回-1

{% codeblock lang:java %}
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
{% endcodeblock %}

查找指定元素最后一次出现的游标，如果没有找到返回-1

{% codeblock lang:java %}
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
{% endcodeblock %}

是否包含指定元素

{% codeblock lang:java %}
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
{% endcodeblock %}

## 转换成数组

{% codeblock lang:java %}
    public Object[] toArray() { //直接返回一个新数组
        return Arrays.copyOf(elementData, size);
    }

    public <T> T[] toArray(T[] a) { //将列表中的元素存储到指定的数组中
        if (a.length < size) //当指定数组的大小小于列表元素的个数时，会创建一个新的参数a的对应类型的数组，并添加列表元素到数组中
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
{% endcodeblock %}

## 附加方法

确保容量ensureCapacity，该方法一般使用在开发者已知有多少元素添加，如果使用add方法会导致频繁扩容，降低性能，所以提前预判扩容

{% codeblock lang:java %}
public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != EMPTY_ELEMENTDATA)
            // any size if real element table
            ? 0
            // larger than default for empty table. It's already supposed to be
            // at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
{% endcodeblock %}

trimToSize方法，当列表中存储的元素个数小于容量的时候，就可以使用这个方法，释放多余的内存空间

{% codeblock lang:java %}
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = Arrays.copyOf(elementData, size); //创建一个和存储元素个数大小的数组，并将数据拷贝
        }
    }
{% endcodeblock %}

## subList方法

subList方法，返回一个和List关联的“映像”列表，如果fromIndex等于toIndex，则返回一个empty的列表，子列表和父列表的非结构性修改都会相互反映出来。

{% codeblock lang:java %}
    public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size); //检查sublist的范围是否合法
        return new SubList(this, 0, fromIndex, toIndex);
    }
{% endcodeblock %}

内部类SubList分析

类定义，继承了AbstractList，实现RandomAccess接口[参阅源码](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/ArrayList.java#ArrayList.SubList)

{% codeblock lang:java %}
private class SubList extends AbstractList<E> implements RandomAccess {
{% endcodeblock %}

构造方法

{% codeblock lang:java %}
    SubList(AbstractList<E> parent, int offset, int fromIndex, int toIndex) {
            this.parent = parent;  //父列表
            this.parentOffset = fromIndex; //子列表对应父列表的开始位置
            this.offset = offset + fromIndex; 
            this.size = toIndex - fromIndex; //子列表大小
            this.modCount = ArrayList.this.modCount; //列表结构修改性次数
    }
{% endcodeblock %}

列表方法，我们以add方法为例，其他都相似就不详细分析了

{% codeblock lang:java %}
        public void add(int index, E e) {
            rangeCheckForAdd(index);  //范围是否合法
            checkForComodification(); //父列表的结构性修改和自列表的结构性修改是否相同，不同抛出ConcurrentModificationException
            parent.add(parentOffset + index, e); //父列表添加元素
            this.modCount = parent.modCount; //同步父列表结构性修改次数到子列表
            this.size++;
        }

        private void checkForComodification() {
            if (ArrayList.this.modCount != this.modCount)
                throw new ConcurrentModificationException();
        }
{% endcodeblock %}

由此可以看出，子列表的修改都是对父列表的修改，结构性修改也一样，但是如果我们先对父列表进行结构性修改，再对子列表进行访问就会抛出ConcurrentModificationException异常，下面我们举例验证，调用如下方法

{% codeblock lang:java %}
    private void testSublist() {
        List<String> list = new ArrayList<String>() {
            {
                add("a");
                add("b");
            }
        };
        List<String> subList = list.subList(0, list.size());

        list.add("c");

        System.out.println("list.size = " + list.size());

        System.out.println("subList.size = " + subList.size());
    }
{% endcodeblock %}

执行结果

{% codeblock lang:java %}
list.size = 3
Exception in thread "main" java.util.ConcurrentModificationException
    at java.util.ArrayList$SubList.checkForComodification(ArrayList.java:1169)
    at java.util.ArrayList$SubList.size(ArrayList.java:998)
{% endcodeblock %}

<span style="color:orange;">因此在使用subList方法时要特别注意，获得子列表之后不要再对父列表进行结构性修改了</span>

## iterator和listIterator方法

iterator和listIterator方法返回一个List的迭代器，之前在[java集合框架](http://thinkdevos.net/blog/20160817/java-collections-framework/)我们讲过，迭代器应该不属于集合，只是集合框架提供的一个公共遍历集合的方法

{% codeblock lang:java %}
    public ListIterator<E> listIterator() {
        return new ListItr(0); 
    }

    public Iterator<E> iterator() {
        return new Itr();
    }
{% endcodeblock %}

下面我们就分别对内部类Itr和ListItr进行分析

内部类Itr分析
Itr实现了[Iterator](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/Iterator.java)接口,是[AbstractList.Itr](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/AbstractList.java#AbstractList.Itr)一个优化版本[参阅源码](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/ArrayList.java#ArrayList.Itr)

{% codeblock lang:java %}
    private class Itr implements Iterator<E> {
        int cursor;       // 下一个元素的游标
        int lastRet = -1; // 最后返回元素的游标，即最后一次调用next时元素的游标
        int expectedModCount = modCount; //初始结构化修改次数

        public boolean hasNext() { //是否有由下一个元素
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification(); //校验List是否有结构修改性操作
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1; //修改游标指向下一个元素
            return (E) elementData[lastRet = i]; //返回元素，并给lastRet赋值
        }

        public void remove() {
            if (lastRet < 0) //调用remove之前必须要调用过next方法
                throw new IllegalStateException();
            checkForComodification(); //校验List是否有结构修改性操作

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet; //删除元素后，游标不变
                lastRet = -1;
                expectedModCount = modCount; //同步List结构修改次数
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {  //校验List是否有结构修改性操作
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
{% endcodeblock %}

通过上面的代码我们发现，<span style="color:orange;">使用迭代器的同时不能对List进行结构性操作</span>，在调用remove之前要确掉用过next方法下面我们test这些内容

{% codeblock lang:java %}
    @org.junit.Test
    public void testIterator_loop_remove() {
        List<String> list = new ArrayList<String>() { {
                add("a"); add("b"); add("c"); add("d");
            } };

        System.out.println("before : " + list);

        for (String s : list) {
            System.out.println(list.remove(s));
        }
    }
    /*
    输出：
    before : [a, b, c, d]
    true

    java.util.ConcurrentModificationException
        at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:859)
        at java.util.ArrayList$Itr.next(ArrayList.java:831)
        at com.thinkdevos.java.collections.ArrayListTests.testIterator_loop_remove(ArrayListTests.java:44)
    */
{% endcodeblock %}

{% codeblock lang:java %}
    @org.junit.Test
    public void testIterator_iterator_remove_no_next() {
        List<String> list = new ArrayList<String>() { {
                add("a"); add("b"); add("c"); add("d");
            } };

        System.out.println("before : " + list);
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            iterator.remove();
        }
    }
    /*
    输出：
    before : [a, b, c, d]

    java.lang.IllegalStateException
        at java.util.ArrayList$Itr.remove(ArrayList.java:844)
        at com.thinkdevos.java.collections.ArrayListTests.testIterator_iterator_remove_no_next(ArrayListTests.java:62)
    */
{% endcodeblock %}

内部类ListItr分析
ListItr集成了Itr类，实现了[ListIterator](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/ListIterator.java)接口，是[AbstractList.ListItr](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/AbstractList.java#AbstractList.ListItr)的优化版本，其实现和Itr类同，就不做详细分析了，其只是一系列类似链表操作的的接口实现，详情请[参阅源码](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/ArrayList.java#ArrayList.ListItr)<br>

SubList的iterator、listIterator和subList大家就可以照猫画虎的分析了，也不再赘述<br>

至此，我们就彻底撸通了ArrayList的原理机制，文章中代码比较多，但注释比较详细，希望对大家研读jdk源码有所帮助.

---
参考资料
1、[为什么数组最大容量是Integer.MAX_VALUE - 8](http://stackoverflow.com/questions/35756277/why-the-maximum-array-size-of-arraylist-is-integer-max-value-8)
