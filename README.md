# Java-JMM-STUDY
hashtable
一、继承关系
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
}
继承Dictionary，实现Cloneable、Serializable
Dictionary是抽象类，是声明了操作"键值对"函数接口的抽象类
Enumeration（枚举）接口的作用和Iterator类似，只提供了遍历Vector和HashTable类型集合元素的功能，不支持元素的移除操作。

2、构造函数
    
    public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        table = new Entry<?,?>[initialCapacity];
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    }

    
    public Hashtable(int initialCapacity) {
        this(initialCapacity, 0.75f);
    }
    
    public Hashtable() {
        this(11, 0.75f);
    }

    public Hashtable(Map<? extends K, ? extends V> t) {
        this(Math.max(2*t.size(), 11), 0.75f);
        putAll(t);
    }

3、成员变量

    /**
     * 一个单向链表,哈希表的"key-value键值对"都是存储在这个数组中
     */
    private transient Entry<?,?>[] table;

    /**
     * Hashtable中元素的实际数量 
     */
    private transient int count;

    /**
     * 阈值，用于判断是否需要调整Hashtable的容量（threshold = 容量*加载因子）  
     * @serial
     */
    private int threshold;

    /**
     * 加载因子
     * @serial
     */
    private float loadFactor;

    /**
     * Hashtable被改变的次数,用来实现fail-fast机制的
     */
    private transient int modCount = 0;
    
    transient关键字能实现什么？
    当对象被序列化时（写入字节序列到目标文件）时，transient阻止实例中那些用此关键字声明的变量持久化；
    当对象被反序列化时（从源文件读取字节序列进行重构），这样的实例变量值不会被持久化和恢复。
    例如，当反序列化对象——数据流（例如，文件）可能不存在时，
    原因是你的对象中存在类型为java.io.InputStream的变量，序列化时这些变量引用的输入流无法被打开
    
    2.什么是 fail-fast 机制?
    fail-fast机制在遍历一个集合时，当集合结构被修改，会抛出Concurrent Modification Exception。
    fail-fast会在以下两种情况下抛出ConcurrentModificationException
    （1）单线程环境
    集合被创建后，在遍历它的过程中修改了结构。
    注意 remove()方法会让expectModcount和modcount 相等，所以是不会抛出这个异常。
    （2）多线程环境
    当一个线程在遍历这个集合，而另一个线程对这个集合的结构进行了修改。
    
    3. fail-fast机制是如何检测的？
      迭代器在遍历过程中是直接访问内部数据的，因此内部的数据在遍历的过程中无法被修改。
      为了保证不被修改，迭代器内部维护了一个标记 “mode” ，当集合结构改变（添加删除或者修改），
      标记"mode"会被修改，而迭代器每次的hasNext()和next()方法都会检查该"mode"是否被改变，
      当检测到被修改时，抛出Concurrent Modification Exception
      
      4. fail-safe机制
      fail-safe任何对集合结构的修改都会在一个复制的集合上进行修改，因此不会抛出ConcurrentModificationException
      fail-safe机制有两个问题
      （1）需要复制集合，产生大量的无效对象，开销大
      （2）无法保证读取的数据是目前原始数据结构中的数据。
      
      4、put和get方法
      public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
    
    private void addEntry(int hash, K key, V value, int index) {
        modCount++;

        Entry<?,?> tab[] = table;
        if (count >= threshold) {
            // Rehash the table if the threshold is exceeded
            rehash();

            tab = table;
            hash = key.hashCode();
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // Creates the new entry.
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>) tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }
   
