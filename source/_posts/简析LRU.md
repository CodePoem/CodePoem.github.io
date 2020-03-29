# LRU

最近最少使用算法。

## 基本原理

- -要求查找快，插入快，删除快，有顺序之分。哈希表查找快，但是数据无固定顺序；链表有顺序之分，插入删除快，但是查找慢。所以结合一下，形成一种新的数据结构：哈希链表。

- 按顺序插入 ，所以需要双向链表。

- LinkedHashMap，使用 accessOrder=true 基于顺序访问，元素访问后被移动到末尾。

## 自己动手实现LRU

```java
class Node {
    public int key, val;
    public Node next, prev;
    public Node(int k, int v) {
        this.key = k;
        this.val = v;
    }
}

class DoubleList {  
    // 在链表头部添加节点 x，时间 O(1)
    public void addFirst(Node x);

    // 删除链表中的 x 节点（x 一定存在）
    // 由于是双链表且给的是目标 Node 节点，时间 O(1)
    public void remove(Node x);

    // 删除链表中最后一个节点，并返回该节点，时间 O(1)
    public Node removeLast();

    // 返回链表长度，时间 O(1)
    public int size();
}

class LRUCache {
    // key -> Node(key, val)
    private HashMap<Integer, Node> map;
    // Node(k1, v1) <-> Node(k2, v2)...
    private DoubleList cache;
    // 最大容量
    private int cap;

    public LRUCache(int capacity) {
        this.cap = capacity;
        map = new HashMap<>();
        cache = new DoubleList();
    }

    public int get(int key) {
        if (!map.containsKey(key))
            return -1;
        int val = map.get(key).val;
        // 利用 put 方法把该数据提前
        put(key, val);
        return val;
    }

    public void put(int key, int val) {
        // 先把新节点 x 做出来
        Node x = new Node(key, val);

        if (map.containsKey(key)) {
            // 删除旧的节点，新的插到头部
            cache.remove(map.get(key));
            cache.addFirst(x);
            // 更新 map 中对应的数据
            map.put(key, x);
        } else {
            if (cap == cache.size()) {
                // 删除链表最后一个数据
                Node last = cache.removeLast();
                map.remove(last.key);
            }
            // 直接添加到头部
            cache.addFirst(x);
            map.put(key, x);
        }
    }
}
```

## LruCache

Android SDK里有提供。

## DiskLruCache

Android SDK里没有提供，AOSP源码有。

很多文件操作都采用了事务的处理方式，即修改文件前先写入一个同名的 tmp 文件，当所有内容写完后再将 tmp 文件的扩展名去掉以覆盖原有文件，这样做的好处就是不会因为应用的异常退出或 Crash 而出现数据损坏，保证了原有文件的完整性。

DiskLruCache 在操作文件的时候使用 journal 文件 来记录操作日志。

journal头：

- 第一行是固定的字符串“libcore.io.DiskLruCache”，标志着使用的是DiskLruCache技术。
- 第二行是DiskLruCache的版本号。
- 第三行是应用程序的版本号。
- 第四行是valueCount
- 第五行是一个空行。

DIRTY 脏数据行 正在写入
CLEAN 洗净脏数据行 写入成功 行末尾加上该条缓存数据的大小，以字节为单位。
REMOVE 移除脏数据行 写入失败

READ 读取数据行

每一行DIRTY的key，后面都应该有一行对应的CLEAN或者REMOVE的记录，否则这条数据就是“脏”的。
redundantOpCount变量来记录用户操作的次数，每执行一次写入、读取或移除缓存的操作，这个变量值都会加1，当变量值达到2000的时候就会触发重构journal的事件，这时会自动把journal中一些多余的、不必要的记录全部清除掉，保证journal文件的大小始终保持在一个合理的范围内。
