- List ：`ArrayList`、`LinkedList`、`Vector`、`CopyOnWriteArrayList`
- Set：`HashSet`、`TreeSet`、`LinkedHashSet`
- Queue：`PriorityQueue`
- Map：`HashMap`，`TreeMap`，`LinkedHashMap`
- fast-fail，fast-safe 机制
- 源码分析（底层数据结构，插入、扩容过程）、线程安全。



### ArrayList

> 构造函数

```java
// 第一种
    public ArrayList(int initialCapacity) {
        // 初始化容量大于 0 时，创建 Object 数组
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        // 初始化容量等于 0 时，使用 EMPTY_ELEMENTDATA 对象
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        // 初始化容量小于 0 时，抛出 IllegalArgumentException 异常
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

// 第二种
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 第三种
  public ArrayList(Collection<? extends E> c) {
        // 将 c 转换成 Object 数组
        elementData = c.toArray();
        // 如果数组大小大于 0
        if ((size = elementData.length) != 0) {
            // defend against c.toArray (incorrectly) not returning Object[]
            // (see e.g. https://bugs.openjdk.java.net/browse/JDK-6260652)
            // 如果集合元素不是 Object[] 类型，则会创建新的 Object[] 数组，并将 elementData 赋值到其中，最后赋值给 elementData 。
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        // 如果数组大小等于 0 ，则使用 EMPTY_ELEMENTDATA 。
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```



> 扩容

谈到扩容，首先看看扩容函数。

```java
    private Object[] grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // 如果原容量大于 0 ，或者数组不是 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时，计算新的数组大小，并创建扩容
        if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            int newCapacity = ArraysSupport.newLength(oldCapacity,
                    minCapacity - oldCapacity, /* minimum growth */
                    oldCapacity >> 1           /* preferred growth */);
// 上面这个newCapacity就是说，oldCapacity + Math.max(minimum growth,preferred growth)
            return elementData = Arrays.copyOf(elementData, newCapacity);
        // 如果是 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 数组，直接创建新的数组即可。
        } else {
            return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
        }
    }

    private Object[] grow() {
        return grow(size + 1);
    }
```

从以上函数可以很清晰的看出来。如果没有指定初始化ArrayList的大小。那么就会被赋值为 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 数组。那么在插入第一个元素的时候，就会直接扩容为10.之后的扩容就是1.5倍扩容。如果指定了大小0，那么前面几次扩容都是加一加一的扩容。到preferred growth大于1的时候，就是1.5倍的扩容了。

总的来说，要记住以下几点

- 如果提前知道要插入元素的数量，尽量指定大小创建ArrayList，避免扩容。
- 如果没有指定大小，我们用的最多的就是这种方式。然后在插入第一个元素的时候的扩容，会直接给一个默认大小10。
- 数组的扩容默认是1.5倍的，有一点点例外就是，数组如果长度是小于4的话是，加一加一扩容的。具体看代码。

> 移除元素

- 移除单个元素
  - 首先判断index的值是否合法
  - 然后保存将要被移除的那个位置的值
  - 然后调用fastRemove进行移除（就是把index+1到末尾的值往前移动一个位置）。再把最后一个位置置位null。在将size减一。
- 移除首次出现的那个元素
  - 如果没有找到就返回false。找到了返回true。这里要注意的就是，是通过传入元素的equals方法来进行判断元素是否相等的。
- removeAll批量移除
  - 用一个类似于快慢指针的东西。找到第一个需要移除的元素，然后快指针继续往后移动，如果被移除集合中存在元素在要移除的集合，那么快指针继续往后。直到结束。再把慢指针到结尾的位置的元素置位null。

> 查找单个元素

`#indexOf(Object o)` 方法，查找首个为指定元素的位置。代码如下：

```
// ArrayList.java

public int indexOf(Object o) {
    return indexOfRange(o, 0, size);
}

int indexOfRange(Object o, int start, int end) {
    Object[] es = elementData;
    // o 为 null 的情况
    if (o == null) {
        for (int i = start; i < end; i++) {
            if (es[i] == null) {
                return i;
            }
        }
    // o 非 null 的情况
    } else {
        for (int i = start; i < end; i++) {
            if (o.equals(es[i])) {
                return i;
            }
        }
    }
    // 找不到，返回 -1
    return -1;
  }
```

> 序列化数组

注意点：我们可以看到elementData是一个`transient` 修饰的属性，但是为啥序列化的时候还是会保留到ArrayList中的元素呢？因为在ArrayList中重写了writeObject,然后逐个写入elementData里面的数据，目的是为了

节省空间，因为原来elementData中的数组不一定是满的。

```java
// ArrayList.java

@java.io.Serial
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out element count, and any hidden stuff
    // 获得当前的数组修改次数
    int expectedModCount = modCount;

    // <1> 写入非静态属性、非 transient 属性
    s.defaultWriteObject();

    // Write out size as capacity for behavioral compatibility with clone()
    // <2> 写入 size ，主要为了与 clone 方法的兼容
    s.writeInt(size);

    // Write out all elements in the proper order.
    // <3> 逐个写入 elementData 数组的元素
    for (int i = 0; i < size; i++) {
        s.writeObject(elementData[i]);
    }

    // 如果 other 修改次数发生改变，则抛出 ConcurrentModificationException 异常
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

Serializable原理 https://juejin.cn/post/6844903718538739720

>  创建 Iterator 迭代器

`#iterator()` 方法，创建迭代器。一般情况下，我们使用迭代器遍历 ArrayList、LinkedList 等等 List 的实现类。代码如下：

```
// ArrayList.java

public Iterator<E> iterator() {
    return new Itr();
}
```

- 创建 Itr 迭代器。

  - Itr 实现 [`java.util.Iterator`](https://github.com/YunaiV/openjdk/blob/master/src/java.base/share/classes/java/util/List.java) 接口，是 ArrayList 的内部类。虽然说 AbstractList 也提供了一个 Itr 的实现，但是 ArrayList 为了更好的性能，所以自己实现了，在其类上也有注释“An optimized version of AbstractList.Itr”。

- `#next()` 方法，下一个元素。

  - 简单来说就是获取下一个元素，如果下一个元素是越界了，直接抛出异常，所以使用前一般先使用一个hasnext()方法。

- `#remove()` 方法，移除当前元素。代码如下：

  ```java
  // ArrayList.java#Itr
  
  public void remove() {
      // 如果 lastRet 小于 0 ，说明没有指向任何元素，抛出 IllegalStateException 异常
      if (lastRet < 0)
          throw new IllegalStateException();
      // 校验是否数组发生了变化
      checkForComodification();
  
      try {
          // <1> 移除 lastRet 位置的元素
          ArrayList.this.remove(lastRet);
          // <2> cursor 指向 lastRet 位置，因为被移了，所以需要后退下
          cursor = lastRet;
          // <3> lastRet 标记为 -1 ，因为当前元素被移除了
          lastRet = -1;
          // <4> 记录新的数组的修改次数
          expectedModCount = modCount;
      } catch (IndexOutOfBoundsException ex) {
          throw new ConcurrentModificationException();
      }
  }
  ```

**这里就要提一下那个在遍历时修改ArrayList的问题了，怎样一边遍历一边修改呢。**

当我们调用itr的next()方法的时候，就会一开始就检查

```
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

当不相等的时候，就会抛出异常。当我们遍历的时候，如果删除了元素或者是干了其它改变modCount的事情，那么必然是会抛出异常的。因为，我们没有把迭代器中的expectedModCount 一起改变。所以，我们得用itr自己的删除方法。我们可以看到，它自己提供的删除方法，在删除成功之后，会去修改 expectedModCount = modCount;所以，这样子的话，就不用担心会有异常的抛出。

 ### HashMap

在hashmap中底层存储的是用数组+链表/红黑树的形式。

![点击查看源网页](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com%2Fjpg%2F74959f466f27ab1109af604b71312524.jpg%3Fx-oss-process%3Dimage%2Fresize%2Cp_100%2Fauto-orient%2C1%2Fquality%2Cq_90%2Fformat%2Cjpg%2Fwatermark%2Cimage_eXVuY2VzaGk%3D%2Ct_100&refer=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1613725098&t=1788609d285352b726a9f43ed74c8bfd)

底层数组是用属性table来表示的。

```java
   /**
     * 底层存储的数组
     *
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;
```

node节点，是链表的节点。node也是hashmap的一个类。

> 构造函数

```java
 public HashMap(int initialCapacity, float loadFactor) {
        // 校验 initialCapacity 参数
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        // 避免 initialCapacity 超过 MAXIMUM_CAPACITY
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        // 校验 loadFactor 参数
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        // 设置 loadFactor 属性
        this.loadFactor = loadFactor;
        // 计算 threshold 阀值
        this.threshold = tableSizeFor(initialCapacity);
    }
```

tableSizeFor(initialCapacity); 就是会输出比你传入的那个数字大的二的整数次幂。比如说传入7，输出就是8.传入与12输出就是16.一定是最小的大于传入值的一个二的整数次幂。

为什么一定得是2的n次方呢？

- 提高性能，用&操作。



**需要说明的一点是，hashmap用的是延迟初始化的方式，底层数组只有在put的时候才去真的创建。**

> hash()函数

https://www.zhihu.com/question/20733617/answer/111577937

```java
// HashMap.java

static final int hash(Object key) {
    int h;
    // h = key.hashCode() 计算哈希值
    // ^ (h >>> 16) 高 16 位与自身进行异或计算，保证计算出来的 hash 更加离散
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

简单说明：

1. 为什么需要与高16位进行混合异或？

   这个问题需要结合hashmap的数组长度来说。第一，因为key的hashcode（）返回的是一个int型的数字。为32位。那么将高位和低位进行异或混合得出来的值会将高位一起参与运算。

   第二，我们的hashmap的长度一定是2的n次方的长度。然后，需要根据hash值确定在数组中哪个位置进行存放。那么到底是哪个位置呢？就是调用 h & (length-1);此时，length - 1的二进制是全为1的，就会保留hash值的低位。那么，如果上面的那一点中不进行高低位混合运算的话，就会大大增加碰撞几率。

> put()方法

put方法实际上会调用putVal方法。这个方法大体上分为四个部分。

1. 首先，判断是否需要扩容。
2. 得到将会被放入数组中的那个位置的下标，然后判断这个下标处是否有值。如果没有值，直接创建一个node节点放入。如果有值，代表可能发生冲突。那么，这个有值的点，可能挂载的是一个链表，也有可能是一棵树。
3. 进行链表遍历，如果找到了key是相同的，并且调用equals方法也是返回true的话，看做是要插入的节点的key已经存在于hashmap中了。则执行更新操作。如果链表中没有，则创建新节点，插入链表。（尾插法？）
4. 当然，如果这个节点上挂载的已经是树了，那么会转化为操作红黑树。

> 扩容

仅仅是代码长了点，逻辑很明确，就两步：1）计算新的容量和扩容阀值，并创建新的 `table` 数组；2）将老的 `table` 复制到新的 `table` 数组中。

如果是默认构造方法。即没有传入初始化容量的话，就是16.

> 树化和退化？

当table容量小于64时，只会调用resize方法进行扩容。当大于64的时候，并且链表长度大于等于8，则会进行树化。为什么是8？根据 [泊松概率函数(Poisson distribution)](http://en.wikipedia.org/wiki/Poisson_distribution) ，当链表长度到达 8 的概率是 0.00000006 ，不到千万分之一。

当树内部节点小于等于6 的时候，进行退化。为什么是6？因为如果是7的话，会频繁的树化和退化。影响性能。

### LinkHashMap

LinkedHashMap 就是对hashmap的一种扩充。提供了基于插入顺序的访问。

众所周知，HashMap 提供的访问，是**无序**的。而在一些业务场景下，我们希望能够提供**有序**访问的 HashMap 。那么此时，我们就有两种选择：

- TreeMap ：按照 key 的顺序。
- LinkedHashMap ：按照 key 的插入和访问的顺序。

LinkedHashMap ，在 HashMap 的基础之上，提供了**顺序**访问的特性。而这里的顺序，包括两种：

- 按照 key-value 的**插入**顺序进行访问。关于这一点，相信大多数人都知道。

- 按照 key-value 的**访问**顺序进行访问。

  > 面试中，有些面试官会喜欢问你，如何实现一个 LRU 的缓存。

实际上，LinkedHashMap 可以理解成是 LinkedList + HashMap 的组合。

![类图](http://static.iocoder.cn/images/JDK/2019_12_10/01.png)

![image-20210120192947353](https://gitee.com/qiyuejie/qiyue/raw/master/img/image-20210120192947353.png)

> Entry 属性

LinkedHashMap可以看成是链表和hashmap的结合。

其实在LinkedHashMap中，有以下的实现类。Node是hashmap中的数组的数组类型。那么，可以很直观的看到，其实就是把node添加了一个前指针和一个后指针。形成了一个类似于链表的形式。

```java
   /**
     * HashMap.Node subclass for normal LinkedHashMap entries.
     */
    static class Entry<K,V> extends HashMap.Node<K,V> {

        Entry<K,V> before, // 前一个节点
                after; // 后一个节点

        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }

    }
```

![Node 类图](http://static.iocoder.cn/images/JDK/2019_12_07/04.jpg)

> 访问顺序

在这个类中有一个`accessOrder` 属性，决定了 LinkedHashMap 的顺序。也就是说:

- `true` 时，当 Entry 节点被访问时，放置到链表的结尾，被 `tail` 指向。
- `false` 时，当 Entry 节点被添加时，放置到链表的结尾，被 `tail` 指向。如果插入的 key 对应的 Entry 节点已经存在，也会被放到结尾。

值得注意的是，在hashmap中，我们的节点访问方法是 `get(Object key)`。那么在这个get方法中并没有对访问过后的回调。那么，我们在LinkedHashMap 就需要自己去重写这个get方法，使得我们在访问了这个节点之后，就把这个刚刚被访问的节点放到最后去。

> 遍历

遍历推荐使用forEach（）方法。



来对 LinkedHashMap 做一个简单的小结：

- LinkedHashMap 是 HashMap 的子类，增加了

  顺序

  访问的特性。

  - 【默认】当 `accessOrder = false` 时，按照 key-value 的**插入**顺序进行访问。
  - 当 `accessOrder = true` 时，按照 key-value 的**读取**顺序进行访问。

- LinkedHashMap 的**顺序**特性，通过内部的双向链表实现，所以我们把它看成是 LinkedList + LinkedHashMap 的组合。

- LinkedHashMap 通过重写 HashMap 提供的回调方法，从而实现其对**顺序**的特性的处理。同时，因为 LinkedHashMap 的**顺序**特性，需要重写 `#keysToArray(T[] a)` 等**遍历**相关的方法。

- LinkedHashMap 可以方便实现 LRU 算法的缓存。

### HashSet 

HashSet 是基于 HashMap 的 Set 实现类。

属性

HashSet 只有一个属性，那就是 `map` 。代码如下：

```java 
// HashSet.java

private transient HashMap<E, Object> map;
```

- `map` 的 key ，存储 HashSet 的每个 key 。

- `map` 的 value ，因为 HashSet 没有 value 的需要，所以使用一个统一的 `PRESENT` 即可。代码如下：

  ```java
  // HashSet.java
  
  // Dummy value to associate with an Object in the backing Map
  private static final Object PRESENT = new Object();
  ```

### TreeMap![类图](http://static.iocoder.cn/images/JDK/2019_12_13_02/01.png)

treemap就是一棵红黑树。

treemap是按照里面的`private final Comparator<? super K> comparator;`进行排序的。我们可以在创建的时候指定一个comparator。那么，如果没有指定，则会用key的comparator。所以说，一般来说，输出的顺序和你存入的顺序是不一样的。但是非得要按存入顺序给输出的话。可以自定义一个comparator传入。

```java
public static void findCircleNum() {
        TreeMap<String, String> mp = new TreeMap<>((k1,k2)->{
            return 1;
        });
        mp.put("k1","v1");
        mp.put("k4","v1");
        mp.put("k2","v1");
        mp.put("k3","v1");
        mp.put("k7","v1");

        mp.forEach((k,v)->{
            System.out.println("k:" + k );
            System.out.println("v:" + v );
        });

    }

    public static void main(String[] args) {
        findCircleNum();
    }

//////////////////////////////
k:k1
v:v1
k:k4
v:v1
k:k2
v:v1
k:k3
v:v1
k:k7
v:v1

Process finished with exit code 0

```

- TreeMap 按照 key 的**顺序**的 Map 实现类，底层采用**红黑树**来实现存储。
- TreeMap 因为采用树结构，所以无需初始考虑像 HashMap 考虑**容量**问题，也不存在扩容问题。
- TreeMap 的 **key** 不允许为空( `null` )，可能是因为红黑树是一颗二叉查找树，需要对 key 进行排序。
- TreeMap 的查找、添加、删除 key-value 键值对的**平均**时间复杂度为 `O(logN)` 。原因是，TreeMap 采用红黑树，操作都需要经过二分查找，而二分查找的时间复杂度是 `O(logN)` 。
- 相比 HashMap 来说，TreeMap 不仅仅支持指定 key 的查找，也支持 key **范围**的查找。当然，这也得益于 TreeMap 数据结构能够提供的有序特性。

### fast-fail，fast-safe 机制

https://blog.csdn.net/zymx14/article/details/78394464?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control

#### fail-fast ( 快速失败 )

fail-fast:直接在容器上进行遍历，在遍历过程中，一旦发现容器中的数据被修改了，会立刻抛出ConcurrentModificationException异常导致遍历失败。java.util包下的集合类都是快速失败机制的, 常见的的使用fail-fast方式遍历的容器有HashMap和ArrayList等。

在使用迭代器遍历一个集合对象时,比如增强for,如果遍历过程中对集合对象的内容进行了修改(增删改),会抛出ConcurrentModificationException 异常.

#### fail-safe ( 安全失败 )

fail-safe:这种遍历基于容器的一个克隆。因此，对容器内容的修改不影响遍历。java.util.concurrent包下的容器都是安全失败的,可以在多线程下并发使用,并发修改。常见的的使用fail-safe方式遍历的容器有ConcerrentHashMap和CopyOnWriteArrayList等。

**原理：**

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

**缺点**：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。



 