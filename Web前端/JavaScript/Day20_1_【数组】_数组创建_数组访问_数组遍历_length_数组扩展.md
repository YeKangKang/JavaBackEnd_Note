# Day20 · JavaScript 基础（数组创建、数组访问、数组遍历、length、数组扩展）

## 1. 数组概念与特性

- 数组是多个数据的集合，是将一组元素存储在单一变量名下的结构
- JS 是弱类型语言，**数组可包含不同类型的元素**
- JS 数组长度可变，**支持稀疏结构（存在“空槽”）**

---

## 2. 数组创建

#### 字面量创建（推荐）

```js
let arr1 = [];           // 创建空数组
let arr2 = [1, 2, 3];    // 创建含有 3 个元素的数组
```

- 简洁直观，是 JS 中最常用的创建方式

#### 构造函数 `new Array()`

```js
let arr1 = new Array(5);        // 创建长度为 5 的空数组
let arr2 = new Array(1, 2, 3);  // 创建并初始化数组
```

- 若只传一个参数且为非负整数 → 创建对应长度的空数组
- 若传多个参数 → 创建并初始化数组

>  `new Array(value)` 行为：
> - 若 `value` 是单个非负整数 → 创建指定长度数组
> - 否则 → 创建包含该参数的数组

---

## 3. 元素索引与访问

- JS 数组索引从 `0` 开始，访问越界索引返回 `undefined`，**不会抛出异常**

```js
let arr = [1, 2, 3];
console.log(arr[2]); // 输出 3
console.log(arr[5]); // 输出 undefined
```

---

## 4. 数组长度与扩容机制

### 4.1 读取数组长度

```js
let arr = [1, 2, 3];
console.log(arr.length); // 3
```

- 使用 `length` 属性可获取当前数组长度

### 4.2 修改数组长度

#### `length` 属性扩容

```js
let arr = [1, 2, 3];
arr.length = 5;
console.log(arr); // [1, 2, 3, <2 empty items>]
```

- **若 `length` 大于原长度：**扩容，新增部分为 empty slot（不是 0、不是 undefined）
- **若 `length` 小于原长度：**直接截断数组，删除多余元素

#### 高位索引赋值扩容

```js
let arr = [1, 2, 3];
arr[5] = 99;
console.log(arr); // [1, 2, 3, <2 empty items>, 99]
```

- 跳过的索引位置变为空槽（`empty slot`）
- 通过 `arr[arr.length] = 值` 可直接在末尾新增元素

---

## 5. 数组遍历

数组可通过 `for` 循环遍历：

```js
let arr = [1, 2, 3];
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
```

- 每次循环通过索引访问元素，从头到尾依次访问所有有效位置

---

## ✅ 总结

- JS 数组支持字面量与构造器两种创建方式，推荐使用字面量
- 数组长度可动态变化，可通过修改 `length` 或高位赋值实现扩容
- 数组支持稀疏结构，未赋值位置为 empty slot，而非 `undefined`
- `length` 既可读也可写，直接影响数组结构
