#### 头
```
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```  
骨架实现。这样hashMap就不用实现map的所有方法
#### 重要参数
```
 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
 DEFAULT_LOAD_FACTOR
 ```
 1. 容量 ：是哈希表中桶的数量，初始容量只是哈希表在创建时的容量，实际上就是Entry< K,V>[] table的容量 
　　

2. 加载因子 ：是哈希表在其容量自动增加之前可以达到多满的一种尺度。它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。系统默认负载因子为0.75，一般情况下我们是无需修改的。 
　　当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。
 
3.  4个构造函数
4. hashMap是一个链表散列
5. hash冲突？ 链地址法。数据家链表。
#### 方法

![](https://i.loli.net/2019/01/08/5c33f0d1efd65.png)
```
   public V put(K key, V value) {
 2     // 对key的hashCode()做hash
 3     return putVal(hash(key), key, value, false, true);
 4 }
 5
 6 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
 7                boolean evict) {
 8     Node<K,V>[] tab; Node<K,V> p; int n, i;
 9     // 步骤①：tab为空则创建
10     if ((tab = table) == null || (n = tab.length) == 0)
11         n = (tab = resize()).length;
12     // 步骤②：计算index，并对null做处理 
13     if ((p = tab[i = (n - 1) & hash]) == null) 
14         tab[i] = newNode(hash, key, value, null);
15     else {
16         Node<K,V> e; K k;
17         // 步骤③：节点key存在，直接覆盖value
18         if (p.hash == hash &&
19             ((k = p.key) == key || (key != null && key.equals(k))))
20             e = p;
21         // 步骤④：判断该链为红黑树
22         else if (p instanceof TreeNode)
23             e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
24         // 步骤⑤：该链为链表
25         else {
26             for (int binCount = 0; ; ++binCount) {
27                 if ((e = p.next) == null) {
28                     p.next = newNode(hash, key,value,null);
                        //链表长度大于8转换为红黑树进行处理
29                     if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
30                         treeifyBin(tab, hash);
31                     break;
32                 }
                    // key已经存在直接覆盖value
33                 if (e.hash == hash &&
34                     ((k = e.key) == key || (key != null && key.equals(k))))                                            break;
36                 p = e;
37             }
38         }
39        
40         if (e != null) { // existing mapping for key
41             V oldValue = e.value;
42             if (!onlyIfAbsent || oldValue == null)
43                 e.value = value;
44             afterNodeAccess(e);
45             return oldValue;
46         }
47     }
 
48     ++modCount;
49     // 步骤⑥：超过最大容量 就扩容
50     if (++size > threshold)
51         resize();
52     afterNodeInsertion(evict);
53     return null;
54 }
```
### hash算法
```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    //根据 hash 值找到数组种的对象
    //当n为2的指数时，等同于hash%n
    tab[i = (n - 1) & hash]
```
#### 解释
1. 为什么右移16位并且进行异或:

首先，假设有一种情况，对象 A 的 hashCode 为 1000010001110001000001111000000，对象 B 的 hashCode 为 0111011100111000101000010100000。

如果数组长度是16，也就是 15 与运算这两个数，都是0。因为int为32位，右移16位。等于将该二进制数对半切开。再进行亦或运算。这样使数据均匀。
2. 为什么(n-1)&hash：
a % b == (b-1) & a ,当b是2的指数时，等式成立。实际上是希望通过取模进行桶定位，但对于处理器来说，取模运算耗费更多的计算资源。
3. 为什么容量建议为2的n次方
 首先，目的：为了让hash值均匀的分布在桶中，如果不为2的n次方，例子
 ```
 假设我们的数组长度是10,2个不同的hashCode：
 1010 & 101010100101001001000 结果：1000 = 8
 1010 & 101000101101001001001 结果：1000 = 8
 1010 & 101010101101101001010 结果： 1010 = 10
 1010 & 101100100111001101100 结果： 1000 = 8
 这种散列结果，会导致这些不同的key值全部进入到相同的插槽中，形成链表，性能急剧下降。
 所以说，我们一定要保证 & 中的二进制位全为 1，才能最大限度的利用 hash 值，并更好的散列，只有全是1 ，才能有更多的散列结果。如果是 1010，**有的散列结果是永远都不会出现的**，比如 0111，0101，1111，1110…，只要 & 之前的数有 0， 对应的 1 肯定就不会出现（因为只有都是1才会为1）。大大限制了散列的范围

```