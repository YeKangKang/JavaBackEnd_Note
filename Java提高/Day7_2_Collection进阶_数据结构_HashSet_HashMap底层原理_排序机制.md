# Day7 · Java 提高（Collection 进阶、常用数据结构、ArrayList、LinkedList、HashSet、LinkedHashSet、可变参数、Collections工具类、Comparable、Comparator比较器

本日内容围绕 Collection 接口下的重要实现类和数据结构，包括：常见线性结构（栈、队列、数组、链表）、自平衡树（红黑树）、List 与 Set 体系结构、HashSet/HashMap 判重机制、Collections 工具类的使用，以及 Comparable / Comparator 两种排序策略的实现方法。

---

## 1. 常见数据结构与特点概览

### ✅ 栈（Stack）

- 特点：**后进先出（LIFO）**
- 操作限制：只允许在**栈顶**进行 `push`（压栈）与 `pop`（弹栈）
- 常见场景：方法调用栈、撤销操作、浏览器历史记录

### ✅ 队列（Queue）

- 特点：**先进先出（FIFO）**
- 操作限制：在一端入队（尾），另一端出队（头）
- 常见场景：排队系统、消息中间件、线程池任务调度

### ✅ 数组（Array）

- 特点：**连续内存块，支持随机访问**
- 查询效率高：通过索引快速访问
- 插入/删除代价高：涉及大量元素移动
- 适合：频繁查询场景

### ✅ 链表（Linked List）

- 特点：节点通过引用连接，**非连续内存**
- 增删效率高：只需修改前后引用指向
- 查询效率低：访问某个 index 时需从头遍历
- Java 中 `LinkedList` 为**双向链表**

### ✅ 红黑树（Red-Black Tree）

- 特点：**自平衡二叉搜索树**
- 解决普通 BST 插入有序数据时退化为链表的问题
- 保证操作时间复杂度为 `O(log n)`
- Java 中 `TreeMap`、`TreeSet` 的底层结构
- HashMap 桶链表转红黑树（当链表元素 > 8）

---

## 2. List 接口与实现类

### ✅ List 接口特点

- 继承自 `Collection`，是**有序可重复集合**
- **具备索引**，支持通过索引访问、插入、删除

### ✅ 常用实现类

#### 🔸 `ArrayList`

- 底层为数组结构
- 查询快，增删慢（涉及数组移动）
- 适合：查询频繁、写操作少的场景

#### 🔸 `LinkedList`

- 底层为双向链表结构
- 查询慢，增删快
- 提供头尾专用方法：
  - `addFirst()` / `addLast()`
  - `removeFirst()` / `removeLast()`
  - `push()` / `pop()`：视为栈使用
- 适合：频繁插入/删除、模拟队列/栈结构的场景

---

## 3. Set 接口与 HashSet 机制

### ✅ Set 接口特点

- 继承自 `Collection`
- **不允许重复元素**，且**元素无顺序**（逻辑上无序，无法用索引访问）

### ✅ HashSet 实现原理

- 基于 `HashMap` 实现，集合中每个元素作为 `HashMap` 的 key 存储，value 使用一个统一的占位对象（如 `PRESENT`）
- 插入流程：
  1. 通过 `hashCode()` 计算哈希值
  2. 计算索引决定桶位置（`hash % capacity`）
  3. 若哈希冲突，链表/红黑树方式追加
  4. 若 `equals()` 判断为已存在，拒绝存入

### ✅ 判重机制

- `hashCode()` 决定存入位置
- `equals()` 决定是否重复
- **两者必须一致重写**，才能保证判重逻辑有效

### ✅ 自定义对象判重示例（需重写两个方法）

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Student s = (Student) o;
    return age == s.age && Objects.equals(name, s.name);
}

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

---

## 4. HashMap 底层原理

- 底层为**数组 + 链表/红黑树结构**
- 每个元素的 key 通过 `hashCode()` 定位桶位置
- 若冲突，链表追加；链长 > 8 时转为红黑树
- key 重复则使用 `equals()` 判重并**覆盖旧值**

### ✅ 理想哈希函数目标

- 尽可能减少 `hashCode()` 的碰撞概率
- 保证哈希值分布均匀，降低桶中元素密度
- 最大程度提升插入、查找、删除操作效率

### ✅ 哈希冲突 vs 哈希碰撞

- **哈希碰撞**：指不同对象计算出的 `hashCode()` 值相同（算法层面的冲突）
- **哈希冲突**：指多个对象最终映射到了同一个桶（存储结构层面的冲突）
- **哈希碰撞一定会导致哈希冲突，但哈希冲突不一定来自哈希碰撞**
  - 例如：不同 `hashCode()` 经过扰动函数后可能落入同一数组索引（桶），这也是冲突
- HashMap 中使用链表/红黑树结构正是为了在冲突时保持良好的性能

### 🔍 哈希表的结构职责

- HashMap 的数组结构是用于定位桶索引的第一层结构，数组中的每个元素指向一个桶（链表或红黑树）
- `hashCode()` 相等只是定位桶的依据，真正判断是否为重复元素，还需进一步通过 `equals()` 判断
- 即使两个对象内容相同，`hashCode()` 相同，也可能由于其他对象的加入而发生冲突
- 因此桶结构的存在是为了解决多个元素可能落入同一索引（哈希冲突）的问题，是哈希表不可或缺的一部分

---

## 5. LinkedHashSet

- 结构：**HashSet + 双向链表**
- 特点：在保证元素唯一性的同时，记录插入顺序
- 适合：既要判重、又要顺序可控的场景

---

## 6. 可变参数（Varargs）

### ✅ 使用说明

```java
public void print(String... args) {
    for (String arg : args) {
        System.out.println(arg);
    }
}
```

等价于：

```java
public void print(String[] args) { ... }
```

### ✅ 语法规则

- 一个方法**最多只能有一个**可变参数
- 可变参数**必须是参数列表最后一个**
- 可变参数方法**不能和“参数类型相同的数组方法”**同时重载，否则签名冲突。**（可变参数不适用于一般的重载规则，因为可能会引起歧义）**

---

## 7. Collections 工具类

来自 `java.util.Collections`，用于简化集合常用操作。

| 方法                                       | 说明                     |
|--------------------------------------------|--------------------------|
| `addAll(Collection<T>, T... elements)`     | 向集合批量添加元素        |
| `shuffle(List<?>)`                         | 打乱列表元素顺序         |
| `sort(List<T>)`                            | 使用元素自身排序规则      |
| `sort(List<T>, Comparator<? super T>)`     | 使用指定比较器排序        |

> 除了前面提到的 `sort()`、`shuffle()` 等依赖顺序的方法外，`Collections` 工具类中还有大量方法的参数类型是 `Collection`，因此可以同时用于 `List` 和 `Set`。这类方法通常不依赖元素顺序，仅关注集合内容本身。

---

## 8. 排序机制：`Comparable` vs `Comparator`

### ✅ `Comparable<T>` 接口

- 适用于**类自身具备固定排序规则**
- 实现 `compareTo(T o)` 方法
- 可直接用于 `Collections.sort(list)`

```java
public class Student implements Comparable<Student> {
    public int compareTo(Student o) {
        return this.age - o.age; // 升序
    }
}
```

### ✅ `Comparator<T>` 接口

- 适用于**临时指定排序规则**
- 实现 `compare(T o1, T o2)` 方法
- 用法：传入 `Collections.sort(list, comparator)`

```java
Collections.sort(list, new Comparator<Student>() {
    public int compare(Student o1, Student o2) {
        return o2.age - o1.age; // 降序
    }
});
```

---

### 🧠 多字段排序模板（常用写法）

```java
int result = o2.getAge() - o1.getAge(); // 年龄降序
if (result == 0) {
    result = o1.getName().charAt(0) - o2.getName().charAt(0); // 姓名升序
}
return result;
```

---

## ✅ 小结

- List 有序可重复，Set 无序不可重复
- HashSet 和 HashMap 底层都用 `hashCode` + `equals` 判重
- 自定义类型如需存入 Set/Map，必须重写这两个方法
- 排序可使用类内 `compareTo` 或外部 `Comparator` 策略，灵活切换
