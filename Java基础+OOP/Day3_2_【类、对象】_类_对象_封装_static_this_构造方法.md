# Day3 · Java 面向对象基础（类、对象、封装、static、this、构造方法）

---

## 1. 类（Class）与对象（Object）

### 1.1 类的概念

- 类是一种抽象的“模板”或“蓝图”，用于描述一类对象的共同特征（属性）与行为（方法）。
- 类是静态定义的，不占用实际内存，只有实例化为对象后才存在于内存中。

```java
public class Employee {
    public String name;
    public int id;
    public double salary;
    public static boolean isAdult = true; // 所有员工共用的类变量
}
```

> 命名规范：类名采用大驼峰命名法（如 `StudentInfo`）

---

### 1.2 对象的概念

- 对象是类的一个具体实例，具有真实的内存空间。
- 类是“车的图纸”，对象是“实际造出的一辆车”。

```java
Employee e1 = new Employee();
e1.name = "Yuheng Ye";
e1.id = 10010;
e1.salary = 4000;
```

---

### 1.3 成员变量默认值

| 类型分类   | 示例类型           | 默认值    |
|------------|--------------------|-----------|
| 整型       | `int`, `long`      | `0`       |
| 浮点型     | `float`, `double`  | `0.0`     |
| 字符型     | `char`             | `\u0000`（空字符） |
| 布尔型     | `boolean`          | `false`   |
| 引用类型   | `String`, `数组`等 | `null`（表示空，不存在） |

---

## 2. 封装与访问控制

### 2.1 封装的意义

- **目的**：隐藏类内部实现细节，只暴露必要接口，提高安全性与灵活性。
- **实现方式**：将成员变量设为 `private`，通过 `public` 的 `getter/setter` 访问。

```java
public class Employee {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        if (name != null && !name.equals("")) { // null表示对象不存在，null和""是有区别的
            this.name = name;
        }
    }
}
```

---

### 2.2 常见访问修饰符

| 修饰符     | 同类 | 同包 | 子类 | 其他类 | 示例              |
|------------|------|------|------|--------|-------------------|
| `private`  | ✅   | ❌   | ❌   | ❌     | 仅类内部访问       |
| `default`  | ✅   | ✅   | ❌   | ❌     | 同包可访问（无修饰）|
| `protected`| ✅   | ✅   | ✅   | ❌     | 子类可访问         |
| `public`   | ✅   | ✅   | ✅   | ✅     | 所有地方都能访问   |

---

## 3. static 关键字

### 3.1 static 修饰变量与方法

| 对象级（非static） | 类级（static） |
|--------|----------------|
| 每个对象一份 | 所有对象共享一份 |

- **静态变量**：归属于类，内存中只有一份，所有对象共享。
- **静态方法**：可直接使用类名调用；**不能访问非 static 成员**。

```java
public class Util {
    public static int square(int x) {
        return x * x;
    }
}
Util.square(5); // 直接通过类调用
```

---

## 4. this 关键字

### 4.1 this 的作用

- `this` 代表“当前对象本身”
- 解决成员变量与参数重名时的冲突

```java
public void setName(String name) {
    this.name = name; // this.name 是成员变量，name 是参数
}
```

---

## 5. 构造方法（构造器）

### 5.1 定义与作用

- 构造方法用于创建对象时初始化成员变量。
- 与类名相同、无返回值（不写 `void`）。

```java
public class Employee {
    public Employee() {
        // 默认构造方法
    }

    public Employee(String name, int id) {
        this.name = name;
        this.id = id;
    }
}
```

---

### 5.2 构造方法重载

- 构造方法可以有多个，只要**参数列表不同**即可。

```java
Employee e1 = new Employee();                     // 调用无参构造
Employee e2 = new Employee("Alice", 10101);       // 调用有参构造
```

> 若类中定义了有参构造，**Java 不会再自动提供默认无参构造器**，需要手动添加。

---

## 6. JavaBean 标准类规范

| 要素           | 要求                           |
|----------------|--------------------------------|
| 类修饰符       | `public`                       |
| 类类型         | 具体类，非 `abstract`          |
| 成员变量       | 使用 `private` 封装            |
| Getter/Setter | 提供 `getXxx()`、`setXxx()`    |
| 构造方法       | 必须有无参构造方法             |
| 有参构造方法（**不是必须**） | 可选，视需求添加                |

---

## ✅ 小结：类与对象核心回顾

- 类：定义抽象模型
- 对象：类的实例，独立存在
- 封装：隐藏细节，通过方法暴露接口
- static：类级共享，适合工具函数和全局配置
- this：明确引用当前对象
- 构造器：负责对象初始化，可重载

---