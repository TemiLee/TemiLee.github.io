---
layout: post
title:  "HashMap 源码解析"
date:   2017-08-05 16:54:13
tags: JAVA
author: Temi Lee
---

HashMap 应该是java程序员应用最多的一中数据结构了，但是对于HashMap 的底层数据结构还是要进行一个深入全面的了解，这样才能更加游刃有余的在工作中使用这一神奇的数据结构

本次源码分析基于的jdk版本：

    java version "1.7.0_80"
    Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
    Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)

**HashMap的数据存储方式:**
> ![数组链表][1]

如图所示，HashMap内部使用一个保存链表节点Entry的数组作为基本的存储结构
其中数组是HashMap的主要存储介质，链表是为了防止Hash冲突，事实上HashMap采用的是对hashcode的一个取余操作，这种冲突是不避免的


HashMap基本的字段结构:

{% highlight java %}

    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     * 保存table数据的数组，可调整大小，大小必须为2的整次幂
     */
    transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

    /**
     * The load factor used when none specified in constructor. 默认的加载因子
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The number of key-value mappings contained in this map.
     * 记录map中保存的Key-Value对数目,用于size()方法的返回
     */
    transient int size;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     *
     * modCount标记HashMap发生结构更改的次数。结构更改指：改变了key-value对的映射数目活着
     * 改变了HashMap的内部结构，比如rehash() 方法。
     * 迭代器使用这个字段实现快速失败机制
     */
    transient int modCount;

    /**
     * The default initial capacity - MUST be a power of two.
     * table 数组的容量 － 大小必须为2的整次幂
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The next size value at which to resize (capacity * load factor).
     * 下次触发resize()方法的 Map大小 值为：数组大小 * 加载因子  默认threshold为16
     * 计算公式：(int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
     * @serial
     */
    // If table == EMPTY_TABLE then this is the initial capacity at which the
    // table will be created when inflated.
    int threshold;

{% endhighlight %}

**HashMap.Entry的定义:**

{% highlight java %}

    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;

        /**
         * Creates new entry.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }

        /**
         * This method is invoked whenever the value in an entry is
         * overwritten by an invocation of put(k,v) for a key k that's already
         * in the HashMap.
         */
        void recordAccess(HashMap<K,V> m) {
        }

        /**
         * This method is invoked whenever the entry is
         * removed from the table.
         */
        void recordRemoval(HashMap<K,V> m) {
        }
    }

{% endhighlight %}

**在了解了HashMap的结构后,看一下这个类内部比较重要的几个方法**

HashMap的构造方法:
HashMap的其他三个构造方法最终都调用的此方法：
{% highlight java %}

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * (使用制定的初始化大小和加载因子初始化空的HashMap)
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +loadFactor);
        this.loadFactor = loadFactor;
        threshold = initialCapacity;
        init();  //这是一个模版方法在HashMap中是空的方法体，见下：
    }

     /**
      * Initialization hook for subclasses. This method is called
      * in all constructors and pseudo-constructors (clone, readObject)
      * after HashMap has been initialized but before any entries have
      * been inserted.  (In the absence of this method, readObject would
      * require explicit knowledge of subclasses.)
      * 子类的初始化钩子方法。这个方法在对象被初始化后，插入任何数据之前，被构造方法，
      * 或者其他类构造方法(eg:clone(),readObject())调用.这个方法的一个负面作用
      * 是：在反序列化的时候，readObject方法需要明确知道子类的类型
      *
      * 多说一句：－－－－－－－－
      * 这中模版方法其实体现了面向对象设计原则中的 开闭原则:对扩展开放，对修改关闭
      */
     void init() {

     }


    /**
     * Retrieve object hash code and applies a supplemental hash function to the
     * result hash, which defends against poor quality hash functions.  This is
     * critical because HashMap uses power-of-two length hash tables, that
     * otherwise encounter collisions for hashCodes that do not differ
     * in lower bits. Note: Null keys always map to hash 0, thus index 0.
     *
     * 考虑到劣质的hash函数带来的hash分布不均匀, 采用一个经过扩展的hash方法获得对象的hash code
     * TODO(中间这句话还不是特别明白)
     * 空的键总是被转换成 hashcode 为0 ，也就是放在table数组的0下标位置
     */
    final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    /**
     * Returns index for hash code h.
     * 返回特定hash code 在table数组中的下表
     */
    static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";

        /**
        * 想了很久才明白这里的处理方法,其实这句话就是返回 h 除 length 的余数
        * 只有当 length 是2的整次幂数的时候，这样做才是具有幂等性的
        * 比如我们计算 12％8 <=> 1100 & 111 = 4
        */

        return h & (length-1);
    }

    /**
     * Returns the value to which the specified key is mapped,
     * or {@code null} if this map contains no mapping for the key.
     *
     * 返回指定key对应的value值，如果没有这个key对应的k-v对，返回空
     *
     * <p>More formally, if this map contains a mapping from a key
     * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
     * key.equals(k))}, then this method returns {@code v}; otherwise
     * it returns {@code null}.  (There can be at most one such mapping.)
     *
     * 更进一步的，如果hashmap包含一个类似这样的键值对：key==null ? k==null : key.equals(k)
     * 即：如果key为空，判断链表中是否有值为null的key，否则 执行key的equals方法去判断
     * 不管我们传入的key是否为空，都至多会返回一个值
     *
     * <p>A return value of {@code null} does not <i>necessarily</i>
     * indicate that the map contains no mapping for the key; it's also
     * possible that the map explicitly maps the key to {@code null}.
     * The {@link #containsKey containsKey} operation may be used to
     * distinguish these two cases.
     * 查询返回空值是有歧义的：我们没有办法区分 1，hashmap中不存在这个key；
     * 2，这个key本身对应的value为null 这两种情况
     * 要做到这一点 我们可以使用:containsKey()去判断
     *
     * @see #put(Object, Object)
     */
    public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }

    /**
     * Returns the entry associated with the specified key in the
     * HashMap.  Returns null if the HashMap contains no mapping
     * for the key.
     * 返回特定key所对应的entry
     * 如果hashmap不包含这个key的k-v映射 则返回null
     */
    final Entry<K,V> getEntry(Object key) {
        // 如果map不包含任何映射关系，直接返回null
        if (size == 0) {
            return null;
        }

        //获取key的hashcode
        int hash = (key == null) ? 0 : hash(key);

        //获取hash所对应的table的数组下表中的entry，执行链表的遍历
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;

            // 判断依据：hash值相等，并且(key是同一个对象，或者调用equals方法返回true)
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }

        //改hash对应的table的数组value为null
        return null;
    }


    /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * 将指定的key-value映射关系放入map中，如果这个map已经包含了这个key的映射，
     * 那么将会用新值取代旧值
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V put(K key, V value) {
        //如果表还没进行初始化的操作，则初始化。－－>> 延迟初始化的设计
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        //null 值key的处理
        if (key == null)
            return putForNullKey(value);

        int hash = hash(key);
        int i = indexFor(hash, table.length);

        //取得hash对应的数组下表的entry进行遍历
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;

            //判断是否有相等的key存在，如果有，执行值替换，返回
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                // LinkedHashMap 实现了这个模版方法
                e.recordAccess(this);
                //返回旧值
                return oldValue;
            }
        }

        //map中不存在指定的key ，结构改变计数modCount + 1
        modCount++;
        //添加key-value对
        addEntry(hash, key, value, i);
        return null;
    }

    /**
     * Adds a new entry with the specified key, value and hash code to
     * the specified bucket.  It is the responsibility of this
     * method to resize the table if appropriate.
     *
     * 在数组指定的下表位置增加一个拥有对应的key,value,hash code的entry
     * 这个方法还负责在适当的时机进行table数组的扩容
     *
     * Subclass overrides this to alter the behavior of put method.
     * 子类应该覆盖此方法以达到更改put方法表的的目的
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
     * 和 addEntry 很像的一个方法，不过这个方法不进行table数组的扩容(适用初始容量已知的场景
     * 比如clone、序列化
     *
     * Subclass overrides this to alter the behavior of HashMap(Map),
     * clone, and readObject.
     */
    void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }

{% endhighlight %}


[1]: /img/blog/hashmap/hashmap.jpeg
