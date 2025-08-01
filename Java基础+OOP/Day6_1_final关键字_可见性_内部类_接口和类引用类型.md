# Day6 · 面向对象进阶（final关键字、可见性、内部类、与引用类型）

---

## 1. `final` 关键字使用总结

`final` 表示“不可更改”，可修饰类、方法、变量。

### 1.1 修饰类
```java
final class A {}
```
- 表示该类不能被继承。  
- 常用于 API 中如 `String`、`Integer` 等安全性要求高的类。

### 1.2 修饰方法
```java
public final void print() {}
```
- 表示该方法不能被子类重写。

### 1.3 修饰变量（常量）
#### 基本类型
```java
final int a = 10;
a = 5; // 报错
```

#### 引用类型
```java
final Cat c = new Cat();
c = new Cat(); // 报错
c.name = "黑猫"; // 成员属性可改
```
- 被 `final` 修饰的变量**只能初始化一次，之后不可重新赋值**
- `final` 引用不能指向新对象，但对象内容可变。

#### 成员变量初始化方式
- 显式初始化：`final String NAME = "白猫";`
- 构造方法内初始化：
```java
final String NAME;
public Cat(String name) {
    this.NAME = name;
}
```

---

## 2. 成员的访问权限（public / protected / default / private）

| 修饰符     | 同类 | 同包 | 子类 | 不同包无关类 |
|------------|------|------|------|--------------|
| `public`   | ✅   | ✅   | ✅   | ✅           |
| `protected`| ✅   | ✅   | ✅   | ❌           |
| `default`（无修饰时默认） | ✅   | ✅   | ❌   | ❌           |
| `private`  | ✅   | ❌   | ❌   | ❌           |

### ⚠️ 类的访问权限
- 只能是 `public` 或默认（`default`）
- 不能使用 `private` 或 `protected`

### 编码建议
- 成员变量：`private`（封装）
- 构造方法：`public`（便于创建对象）
- 成员方法：`public`（便于外部调用）

---

## 3. 成员内部类

```java
class Car {
    class Engine {
        // 内部类
    }
}
```

### 特性说明
- 内部类可访问外部类的所有成员（包括 `private`）
- 内部类会单独编译成 `.class` 文件：如 `Car$Engine.class`
- 外部类需通过外部对象创建内部类对象：
```java
Car car = new Car();
Car.Engine engine = car.new Engine(); // 通过外部对象创建内部类对象
```
> 写法 `new Car().new Engine()` 虽然合法，但不推荐（外部对象无法复用）

---

## 4. 匿名内部类

匿名内部类是实现了父类/接口的**匿名子类对象**，用于**快速定义并创建一次性使用的子类实例**。

### 语法形式
```java
ParentType obj = new ParentType() {
    @Override
    public void method() {
        // 实现/重写方法
    }
};
```

### 编译机制说明
- 匿名类继承你指定的类/接口（本质是 `extends` / `implements`）
- 编译器会自动生成一个无名类，类似 `ParentType$1.class`

### 使用场景
- 常用于接口/抽象类作为方法参数时
```java
makeSound(new Animal() {
    @Override
    public void sound() {
        System.out.println("Meow");
    }
});
```

---

## 5. 引用类型用法总结

### 5.1 类作为成员变量
类可以作为其他类的成员，体现“has-a”关系，例如：
```java
class Player {
    Weapon weapon;
    Armor armor;
}
```

### 5.2 接口作为成员变量
适用于“角色可持有某种能力”，例如法术技能接口：
```java
interface FaShuSkill {
    void attack();
}

class Player {
    private FaShuSkill skill;
    public void setSkill(FaShuSkill s) { this.skill = s; }
    public void release() {
        System.out.println("释放技能：");
        skill.attack();
    }
}
```

- 实际传入的是实现了接口的子类对象（多态）
- 接口变量引用的依旧是具体类实例，符合“面向接口编程”

### 5.3 接口作为方法参数与返回值
接口可用于解耦调用者与实现者：

```java
public List<Integer> getEvenNumbers(List<Integer> list) {
    List<Integer> result = new ArrayList<>(); // ArrayList 是 List 接口的实现类，可用接口声明提高灵活性
    for (int i : list) {
        if (i % 2 == 0) result.add(i);
    }
    return result; // 返回的是 List 类型，具体是 ArrayList 实现
}
```

- 参数与返回值为接口类型
- 实际传入/返回的是接口实现类对象

---

## ✅ 小结：Day6 知识回顾

- `final` 关键字用于定义不可更改的类、方法、变量（包括引用类型的引用不可变但内容可变）
- 访问修饰符决定类与成员的可见性，`private` 最封闭，`public` 最开放
- 成员内部类可访问外部类成员，需通过外部对象实例化，编译生成独立 `.class`
- 匿名内部类用于快速实现父类/接口的子类对象，常用于一次性传参
- 引用类型（类或接口）可作为成员变量、方法参数和返回值，体现组合与多态
- 接口作为引用类型时，实际操作的都是其实现类的对象，符合“面向接口编程”的解耦理念
