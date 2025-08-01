# Day21 · JavaScript 基础（对象、创建对象、调用对象、new、for...in）

## 1. 对象概述

JavaScript 中，对象是由属性（变量）和方法（函数）组成的一组无序集合。几乎所有内容（数组、函数、数值等）都是对象或可视为对象。

- **属性**：用于描述对象特征（如 `name`, `age`）
- **方法**：用于描述对象行为（如 `sayHi()`）

> ✅ JS 是弱类型语言，数组可以混合存储任意类型，包括函数。但数组不能完全替代对象，因为数组依赖索引访问，不具备对象那样的键值语义。

---

## 2. 创建对象的 4 种方式

### 2.1 空对象

```js
var person = {};
person.name = "Tom";
person.sayHi = function () {
  console.log("Hi, I'm " + person.name);
};
```

### 2.2 字面量方式

```js
var student = {
  name: "Tom",
  age: 19,
  sayHi: function () {
    console.log("Hi, I'm " + this.name);
  }
};
```

### 2.3 `new Object()` 构造方式

```js
var obj = new Object();				// 创建一个空对象
obj.name = "Tom";							// 向空对象添加成员
obj.sayHi = function () {			// 向空对象添加成员
  console.log("Hi, I'm " + obj.name);
};
```

### 2.4 构造函数方式（推荐）

```js
function Student(name, age) {
  this.name = name;
  this.age = age;
  this.sayHi = function () {
    console.log("Hi, I'm " + this.name);
  };
}

var s1 = new Student("Tom", 19);	// 使用 new 关键字使用构造函数创建对象
s1.sayHi(); // 输出 Hi, I'm Tom
```

> 构造函数可作为“对象模板”，避免重复书写结构，提升代码标准化和可维护性。

---

## 3. 对象的访问与调用

### 3.1 访问属性

```js
student.name      // 访问属性（点语法）
student["name"]   // 访问属性（方括号语法）
```

### 3.2 调用方法

```js
student.sayHi();  // 调用方法
```

---

## 4. `new` 关键字执行过程

使用 `new 构造函数()` 的过程如下：

1. 在内存中创建一个空对象
2. 将 `this` 指向该空对象
3. 执行构造函数内部代码，为 `this` 添加成员
4. 返回这个对象

---

## 5. 遍历对象属性：`for...in`

`for...in` 用于遍历**对象的所有属性名**（键）：

```js
for (var key in student) {
  console.log(key);        		// 输出属性名
  console.log(student[key]); 	// 输出属性值（必须用中括号）
}
```

> `for...in` 遍历的是键名（属性名），不能使用 `obj.key`，否则会变成访问名为 `"key"` 的属性。

---

## ✅ 总结

- JS 中对象可通过 `{}`、构造函数、`new Object()` 等方式创建
- 属性与方法通过键值对形式组织，可随时动态扩展
- 构造函数用于创建多个结构相同的对象，提升开发效率
- 可通过点语法 / 中括号访问属性，通过 `for...in` 遍历所有属性名
