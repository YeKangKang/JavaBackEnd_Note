# Day5 · 面向对象进阶（接口、多态、类型转换）

本节内容涵盖 Java 接口的结构与实现、多接口冲突解决规则、接口与类的继承优先级、多态的定义与前提、向上转型与向下转型等内容。

---

## 1. 接口（interface）

### 1.1 接口定义与本质

- 接口是 Java 中一种引用类型，是方法的集合，属于一种**行为规范抽象**，接口定义了一组应被实现的行为
- 类通过 `implements` 实现接口，**必须实现接口中所有抽象方法**
- 接口无法被实例化，但可以被类实现

```java
public interface Flyable {
    void fly(); // public abstract 可省略
}
```

---

### 1.2 接口中允许的成员类型

| 成员类型       | 修饰符             | 特点说明 |
|----------------|--------------------|----------|
| 抽象方法 | `public abstract`（默认） | 实现类必须重写 |
| 默认方法       | `default`          | 实现类可继承或选择重写；重写时不能保留 `default` |
| 静态方法       | `static`           | 接口本身调用：`接口名.方法名()` |
| 私有方法       | `private`          | 供默认方法内部调用 |
| 静态私有方法   | `private static`   | 供静态方法内部调用 |
| 常量           | `public static final`（默认） | 接口中变量只能是常量 |

> 接口中**只有“抽象方法”和“默认方法”是提供给实现类的**；其他成员如“静态方法”、“私有方法”、“常量”等仅属于接口自身，不能被实现类继承或重写。

---

### 1.3 接口的实现与语法

```java
public class Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("Bird flies");
    }
}
```

- 抽象方法：必须实现  
- 默认方法：可选择重写；**重写时不能保留** `default`  
- 静态方法：不会被继承，只能通过接口名调用

---

## 2. 接口的多实现与冲突处理

### 2.1 接口多实现

```java
public class Bat extends Animal implements Flyable, Soundable {
    // 重写抽象方法（必须）
    // 重写默认方法（可选）
}
```

- Java 支持**类实现多个接口**
- 可同时继承一个类（单继承）+ 实现多个接口（多实现）

---

### 2.2 多接口方法冲突处理规则

| 冲突类型 | 是否必须重写 | 说明 |
|----------|---------------|------|
| 抽象方法重名 | ✅ 必须重写 | 编译器自动合并，c重写一份 |
| 默认方法重名 | ✅ 必须重写 | 实现类需显式重写解决冲突，重写一份 |
| 静态方法重名 | ❌ 不冲突 | 必须通过接口名调用，不参与多态 |

> 实现类重写默认方法时不能保留 `default` 关键字；但子接口重写时可以保留

---

### 2.3 父类 vs 接口 方法冲突时优先级

> 若一个类同时继承父类并实现接口，且方法签名重复，**父类方法优先**

```java
class Parent {
    public void sayHi() { System.out.println("Parent"); }
}
interface A {
    default void sayHi() { System.out.println("Interface A"); }
}
class Child extends Parent implements A {
    // 不重写也没关系，默认调用 Parent 的 sayHi()
}
```

---

### 2.4 接口多继承（了解）

- 接口可通过 `extends` 继承多个父接口
- 子接口**必须处理父接口中的默认方法冲突**
- 子接口**重写默认方法时可以保留 `default`**

```java
interface A {
    default void method() { System.out.println("A"); }
}
interface B {
    default void method() { System.out.println("B"); }
}
interface C extends A, B {
    @Override
    default void method() { System.out.println("C"); }
}
```

> 父接口中同名抽象方法不会冲突，子接口无需处理，编译器会自动合并为一个方法声明。

---

## 3. 多态（Polymorphism）

### 3.1 多态的定义与前提

> 多态：**同一种行为在不同对象中表现不同的形式**

### ✅ 多态成立的三个前提：

1. 必须有继承或实现关系（类继承 or 实现接口）
2. 必须发生方法重写（Override）
3. 必须使用父类/接口类型指向子类对象

```java
Animal a = new Dog();   // 向上转型，父类/接口类型指向子类对象
a.eat();                // 调用 Dog 中重写的 eat()
```

---

### 3.2 多态的使用优势

- **统一方法定义**，提升扩展性
- 不必为每个子类写一套方法
- 例如定义一个 `animalEat(Animal a)` 方法，可以同时接收 `Cat` 和 `Dog`

---

## 4. 类型转换（引用类型）

### 4.1 向上转型（自动）

```java
Animal a = new Cat(); // 子类 → 父类
```

- 安全、自动发生
- 子类看作父类处理，只能访问父类成员；**但方法如果被重写，调用的是子类版本（多态）**

---

### 4.2 向下转型（强制）

```java
Cat c = (Cat) a;
```

- 父类引用 → 强转为子类引用
- 可访问子类特有方法

---

### 4.3 类型检查：`instanceof`

> 强制转型前建议使用 `instanceof` 判断是否安全

```java
if (a instanceof Cat) {
    Cat c = (Cat) a;
    c.catchMouse();
}
```

- 防止 `ClassCastException`
- 常用于多态结合分支逻辑处理

---

## ✅ 小结

- 接口是一种彻底抽象的“规范”结构，支持多实现、多继承
- 接口可包含多种方法（默认、静态、私有），但只允许常量成员变量
- 多态提供统一编程接口，配合方法重写、向上转型、向下转型等机制
- 使用 `instanceof` 保证转型安全，避免运行时异常
