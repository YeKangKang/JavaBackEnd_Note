# Day8 · Java 提高（Map接口、Entry、键值对、LinkedHashMap、JDK9创建不可变集合、IDEA Debug）

本节内容涵盖 Java 中 `Map` 接口及其常见实现类（如 `HashMap`、`LinkedHashMap`），包括双列集合的结构特点、常用方法、遍历方式、JDK9 新特性及断点调试简介。

---

## 1. Map 接口与双列集合概述

- `Map` 接口表示**双列集合**，每个元素由一个 `key` 映射一个 `value`，称为“键值对”
- 相比 `Collection` 下的单列集合（如 `List`、`Set`），`Map` 元素**成对存在**，结构如下：

```text
key1 -> value1
key2 -> value2
...
```

### ✅ 特性回顾

- `Map` 中的每个 `key` 必须唯一，不允许重复
- `value` 可重复
- `key` 必须为引用类型（因需重写 `hashCode()` 和 `equals()`）
- `value` 可以是任意类型（包括基本类型，会自动装箱）

---

## 2. Map 的常用实现类

### 2.1 HashMap

- 最常用的 `Map` 实现类，底层基于 **哈希表结构（数组 + 链表 / 红黑树）**
- 不保证元素的存取顺序
- 插入效率高，允许 `null` 键和多个 `null` 值
- 插入时会自动调用 `key` 的 `hashCode()` 方法决定桶位置，再用 `equals()` 判重

### 2.2 LinkedHashMap

- `HashMap` 的子类，底层为**哈希表 + 双向链表结构**
- 保留元素的插入顺序
- 插入、查询效率与 `HashMap` 相近

> `LinkedHashMap` 双向链表仅作为顺序辅助，不参与哈希定位与查找。

---

## 3. Map 接口常用方法

| 方法签名                                      | 说明                                      |
|-----------------------------------------------|-------------------------------------------|
| `V put(K key, V value)`                       | 添加键值对；若键存在则覆盖旧值            |
| `V get(Object key)`                           | 根据键获取对应的值                        |
| `V remove(Object key)`                        | 删除指定键的键值对，返回被删的值          |
| `boolean containsKey(Object key)`             | 判断是否包含指定的键                      |
| `Set<K> keySet()`                             | 获取所有键的集合（`Set` 类型）            |
| `Set<Map.Entry<K,V>> entrySet()`              | 获取所有键值对组成的 `Entry` 集合        |

---

## 4. 遍历 Map 的两种方式

### 4.1 使用 `keySet()`（常用于“键找值”）

```java
Map<String, String> map = new HashMap<>();
for (String key : map.keySet()) {
    String value = map.get(key);
    System.out.println(key + " = " + value);
}
```

> `keySet()` 实际返回一个内部类实现的 `Set` 集合，该集合只包含键。通过 `map.get(key)` 获取对应值。

---

### 4.2 使用 `entrySet()`（推荐方式，性能更好）

```java
for (Map.Entry<String, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
```

> `Entry<K, V>` 是键值对的封装对象，调用 `getKey()` / `getValue()` 分别获取键和值。

---

## 5. HashMap 存储自定义对象（作为 key）

- 若使用自定义类对象作为 `key`，**必须重写其 `hashCode()` 和 `equals()` 方法**，否则无法正确判重
- 典型例子：以 `Student` 对象作为键，地址作为值

```java
@Override
public boolean equals(Object obj) {
    // 判断姓名和年龄是否相同
}

@Override
public int hashCode() {
    // 结合姓名和年龄生成哈希码
}
```

---

## 6. JDK 9：创建不可变集合的简洁语法

```java
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map = Map.of("a", 1, "b", 2);
List<String> list = List.of("a", "b");
```

> ✅ 优点：简洁，便于快速构建集合。
> ⚠️ 缺点：返回的是**只读集合**，**无法添加/修改元素**，否则运行时报错。

---

## 7. IDEA 调试功能简介（Debug）

- 可在代码行左侧添加“断点”，程序执行至该处暂停
- 常见调试操作：
  - `Step Over (F8)`：逐行执行
  - `Step Into (F7)`：进入方法体内部调试
  - 变量面板：查看当前作用域内变量的值
- Debug 常用于分析代码执行流程、变量值变化、异常发生位置等

---

## ✅ 总结

- `Map` 是双列集合核心接口，常见实现有 `HashMap`、`LinkedHashMap`
- `Map` 中 `key` 不可重复，需实现判重逻辑（依赖 `hashCode()` + `equals()`）
- 遍历推荐使用 `entrySet()`；JDK9 提供了 `of()` 创建只读集合的便捷方法
- 初步掌握 IDEA 的调试手段是代码排错的高效利器
