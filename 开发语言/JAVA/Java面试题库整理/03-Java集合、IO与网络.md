# Java 集合、IO 与网络

返回索引：[[00-Java面试题库索引]]

## Java 集合体系

- 两大顶层：`Collection` 和 `Map`
- 常见分类：
- `List`：有序、可重复
- `Set`：通常无序、不可重复
- `Queue`：队列
- `Map`：键值对映射

## 常见集合

- `ArrayList`
- `LinkedList`
- `HashSet`
- `TreeSet`
- `ArrayDeque`
- `HashMap`
- `TreeMap`

## 线程安全集合

- 老牌线程安全集合：`Vector`、`Hashtable`
- 包装方式：`Collections.synchronizedXxx()`
- 并发容器：`ConcurrentHashMap`、`CopyOnWriteArrayList`、阻塞队列等

## HashMap

### 底层结构

- 数组 + 链表 + 红黑树

### 关键点

- 哈希定位桶位。
- 冲突时进入链表或红黑树。
- 负载因子默认 `0.75`。
- 扩容通常按 `2^n` 进行。

## HashMap 和 Hashtable

- `HashMap` 线程不安全，性能较好。
- `Hashtable` 线程安全，但性能较差，已不推荐。
- `HashMap` 允许 `null` 键和 `null` 值。
- `Hashtable` 不允许 `null`。

## ConcurrentHashMap

- 线程安全。
- JDK 8 之后核心结构也是数组 + 链表 + 红黑树。
- 初始化和部分更新依赖 CAS。
- 写入时锁桶位或节点，读操作尽量无锁。

## ArrayList

- 底层是 `Object[]`。
- 首次添加后扩容到默认容量。
- 后续常见扩容规则为原容量的 `1.5` 倍。

## ArrayList 和 LinkedList

- `ArrayList`：随机访问快。
- `LinkedList`：插入删除更灵活，但节点额外开销更大。

## List 和 Set

- `List`：有序、可重复。
- `Set`：不可重复。
- `TreeSet` 可排序，底层依赖有序结构。

## BIO、NIO、AIO

- BIO：同步阻塞 IO。
- NIO：同步非阻塞 IO。
- AIO：异步非阻塞 IO。

## IO 多路复用

- 一个线程监听多个文件描述符。
- Linux 常见实现：`select`、`poll`、`epoll`

### select

- 底层常用数组思路。
- 文件描述符数量有限。

### poll

- 类似 select。
- 监听数量理论上不受固定上限约束。

### epoll

- 更适合高并发场景。
- 减少无效轮询。
- 常见触发模式：
- LT：水平触发
- ET：边沿触发

## Java NIO

- 核心组件：
- `Buffer`
- `Channel`
- `Selector`

### Buffer

- 负责数据读写缓冲。

### Channel

- 双向数据通道。

### Selector

- 负责多路复用，监听多个 Channel 的就绪事件。
