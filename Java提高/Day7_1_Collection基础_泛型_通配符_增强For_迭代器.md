# Day7 · Java 提高（Collection、迭代器、增强 for、泛型、通配符）

---

## 1. Collection 集合概述

集合是 Java 提供的容器类，用于存储多个对象，具备以下特性：

- **长度可变**，适用于元素数量频繁变化的场景
- **仅支持引用类型**（如对象、包装类等），不支持基本类型
- **元素类型可不一致**，但推荐使用泛型统一规范元素类型

---

## 2. 集合框架结构

### 2.1 按功能分类

- **单列集合**：`java.util.Collection`
- **双列集合**：`java.util.Map`

### 2.2 单列集合体系

```text
Collection<E>  ← 根接口
 ├─ List<E>    ← 有序、可重复：ArrayList、LinkedList
 └─ Set<E>     ← 无序、不可重复：HashSet、TreeSet
```

### 2.3 Collection 接口常用方法

| 方法                           | 说明                         |
|--------------------------------|------------------------------|
| `boolean add(E e)`             | 添加元素                     |
| `void clear()`                 | 清空集合                     |
| `boolean remove(E e)`          | 移除指定元素                 |
| `boolean contains(E e)`        | 是否包含某元素               |
| `boolean isEmpty()`            | 判断集合是否为空             |
| `int size()`                   | 获取元素个数                 |
| `Object[] toArray()`           | 转为数组（只能为引用类型数组）|

> 集合常通过多态声明为 `Collection<String> list = new ArrayList<>()`，但此时无法调用 `ArrayList` 独有方法（如按索引访问等）。

> 打印集合会自动调用 `toString()`，因 `ArrayList` 未重写该方法，实际调用的是 `AbstractCollection` 中的实现。

---

## 3. Iterator 接口与迭代器

`Iterator<E>` 是集合迭代器接口，可通过 `Collection` 接口或其子类（如 `ArrayList`）的 `iterator()` 方法获取：

```java
Collection<String> c = new ArrayList<>();
Iterator<String> it = c.iterator();
while (it.hasNext()) {
    String s = it.next();
}
```

### 常用方法

| 方法              | 说明                         |
|-------------------|------------------------------|
| `boolean hasNext()`| 是否还有可迭代元素           |
| `E next()`         | 返回下一个元素               |

> 注意：`isEmpty()` 是集合的方法，`Iterator` 没有该方法。

### 原理说明

1. `iterator()` 初始索引在第一个元素之前
2. 每次调用 `next()` 前需先 `hasNext()` 判断
3. 每次调用 `next()` 会将迭代器位置向后推进一位，并返回当前位置所指的元素
4. 遍历完后 `hasNext()` 返回 false
5. **迭代器是一次性的**，遍历完后需重新获取迭代器

---

## 4. 增强 for 循环（for-each）

JDK 1.5 引入，用于简化数组或集合的遍历：

```java
for (元素类型 变量 : 数组/集合) {
    // 使用变量
}
```

本质是使用迭代器遍历，不能在遍历过程中修改集合结构。

---

## 5. 泛型（Generic）

### 5.1 泛型的作用

泛型是一种**在编译阶段约束和确保类型一致性的机制**，允许方法或类在逻辑上处理任意类型的数据，但每个类型参数一旦确定，就在其作用域内保持一致，确保类型安全。

- 编译期检查元素类型，避免运行期类型错误
- 减少类型强转
- 明确集合中存储的元素类型

```java
ArrayList<String> list = new ArrayList<>(); // 推荐写法
```

> 泛型是类型的一部分，例如：`ArrayList<String>` 整体是类型声明。

---

### 5.2 自定义泛型类

```java
// 泛型类定义示例：通过 <T> 声明类中成员变量、方法参数与返回值的类型都为泛型
public class Box<T> {
    private T value;
    public void set(T v) { this.value = v; }
    public T get() { return value; }
}
```

使用方式：

```java
// 创建泛型类对象时指定具体类型，此后该对象中的 T 都会被替换为指定类型
Box<String> b = new Box<>();
b.set("abc");
```

> 类的泛型声明写在 **类名后**，表示整个类中都可以使用 `T` 这个类型变量（字段、方法参数、返回值等）。

---

### 5.3 自定义泛型方法

```java
public static <T> void printArray(T[] arr) {
    for (T t : arr) System.out.println(t);
}

public static <T> T returnGeneric(T value) {
    return value;
}
```

> 方法的泛型声明写在 **返回值类型前**

---

### 5.4 自定义泛型接口

```java
interface MyInterface<E> {
    void add(E e);
    E get();
}
```

可在实现类中确定泛型类型，或在创建对象时指定：

```java
class MyImpl implements MyInterface<String> { ... }
```

> 若不在实现类中确定泛型类型，可保留 `<E>`，在创建对象时再指定。此时实现类需声明为 `class MyImpl<E> implements MyInterface<E>`。

---

## 6. 泛型通配符（Wildcard `<?>`）

### 6.1 通配符基本语法

通配符用于在**不确定具体类型的情况下接收任意类型的泛型参数**，它允许不同类型在同一方法中共存，增强了泛型代码在运行时的灵活性，但**限制了类型写操作**。

```java
public void print(Collection<?> c) { ... }
```

可接受 `Collection<String>`、`Collection<Integer>` 等所有泛型版本。

> 通配符下集合元素被视为 `Object`，只能调用 `Object` 的共性方法（如 `toString()`）

---

### 6.2 有界通配符

#### ✅ 上限通配符：`<? extends T>`

- 表示该集合的泛型类型必须是 **T 或 T 的子类**
- 可读不可写（只读安全）

```java
public void carry(Collection<? extends Number> c) { ... }
```

接受：`List<Number>`、`List<Integer>`，不接受 `List<Object>`、`List<String>`

---

#### ✅ 下限通配符：`<? super T>`

- 表示集合类型是 **T 或 T 的父类**
- 可写但读取不安全（只能当 Object）

```java
public void insert(Collection<? super Integer> c) { ... }
```

接受：`List<Integer>`、`List<Number>`、`List<Object>`

---

## ✅ 小结：Day7 核心知识点回顾

- Collection 是单列集合体系的根接口，常用子接口有 List 和 Set
- `Iterator` 提供统一遍历方式，`hasNext()/next()` 是核心方法，迭代器为一次性对象
- 增强 for 实际底层使用迭代器，但写法更简洁，不能增删元素
- 泛型用于限制集合元素类型，提升类型安全与可读性
- 泛型通配符 `<?>` 适用于传参和数据中转，类型未知但不操作元素内容时使用
- 有界通配符允许灵活控制“子类可传入”、“父类可赋值”等行为，增强泛型表达能力