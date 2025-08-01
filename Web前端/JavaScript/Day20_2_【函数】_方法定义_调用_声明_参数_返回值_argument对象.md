# Day20 · JavaScript 基础（方法定义、调用、声明、参数、返回值、argument对象）

## 1. 方法的作用

JS 中的方法（函数）封装了一段可被重复调用执行的代码块。通过调用方法可以实现逻辑复用、参数化传递与结果返回。

---

## 2. 方法调用与定义位置

- 使用 **命名函数**声明的方法会被 **提升（hoisting）**，可以在声明前调用
- 使用 **函数表达式** 或 **箭头函数** 声明的方法不会被提升，必须在定义后调用

---

## 3. 方法声明方式

### 3.1 命名函数（Function 声明）

```js
function sayHi(name) {
  console.log("Hi " + name);
}
sayHi("Alice");
```

- 使用 `function` 关键字声明
- **支持在JS代码任何位置调用（**提升）

---

### 3.2 匿名函数（函数表达式）

```js
const sayHi = function(name) {
  console.log("Hi " + name);
};
sayHi("Alice");
```

- 将匿名函数赋值给变量
- **不能在定义前调用**（未提升）

---

## 4. 参数机制

- **形参**：方法定义时括号中的变量（`name`，`age`）
- **实参**：方法调用时传入的具体值（`‘Tom’`，`18`）
- 参数之间用逗号 `,` 分隔

```js
function greet(name, age) {	// name，age 为型参
  console.log(name + " is " + age + " years old.");
}
greet("Tom", 18);	// "Tom"，18 为实参
```

---

## 5. 实参与形参不匹配的行为

| 情况                 | 说明                                                                 |
|----------------------|----------------------------------------------------------------------|
| 实参与形参数量一致   | 正常运行                                                              |
| 实参数量多于形参     | 多出的参数会被忽略（但可以通过 `arguments` 对象访问）                |
| 实参数量少于形参     | 缺失的形参为 `undefined`，参与运算时可能变为 `NaN`、`"undefined"` 等 |

> 建议：尽量保持实参与形参数量一致，防止出现不可预期结果

---

## 6. 返回值 `return`

- 使用 `return` 返回方法结果，并中断方法执行
- 无 `return` 则默认返回 `undefined`

```js
function add(a, b) {
  return a + b;
}
```

> `return` 类似 `break`，但作用于方法本身，是方法的“出口语句”

---

## 7. `arguments` 对象

- 所有函数中都内置 `arguments` 对象
- 是伪数组，包含**所有传入的实参**
- 常用于**参数个数不确定**的场景

```js
function combine() {
  let result = "";
  for (let i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
}
console.log(combine("A", "B", "C")); // "ABC"
```

---

## ✅ 总结

- JS 方法支持多种声明方式，命名函数支持提前调用，表达式函数不支持
- 参数不匹配会导致 `undefined`，进而产生 `NaN` 或 `"undefined"` 等结果
- `arguments` 对象可用于接收所有传入参数，但推荐使用与形参匹配的调用方式
