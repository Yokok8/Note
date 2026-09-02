# JavaScript 学习笔记

---

## 目录

1. [JavaScript 是什么](#一javascript-是什么)
2. [JS 引入方式](#二js-引入方式)
3. [JS 基础语法](#三js-基础语法)
   - 输出语句 / 变量 / 常量 / 数据类型 / 函数 / 自定义对象 / JSON / 流程控制 / DOM
4. [JS 事件监听](#四js-事件监听)
5. [重点速查表](#五重点速查表)

---

## 一、JavaScript 是什么

- **JavaScript（JS）** 是一门跨平台、面向对象的**脚本语言**，用来控制网页行为、实现人机交互。
- ⚠️ JS 和 Java **完全是两门语言**，只是基础语法类似。
- 由三部分组成：

| 组成 | 全称 | 作用 |
|------|------|------|
| **ECMAScript** | — | JS 核心语法（变量、数据类型、流程控制、函数、对象等） |
| **BOM** | 浏览器对象模型 | 操作浏览器本身（弹窗、地址栏、关闭窗口等） |
| **DOM** | 文档对象模型 | 操作 HTML 文档（改内容、改样式等） |

> 💡 ECMAScript 由 ECMA 国际制定标准，JS 遵循该标准（最新 ES2024）。

---

## 二、JS 引入方式

### 1. 内部脚本（写在 HTML 页面里）

- JS 代码必须写在 `<script></script>` 标签之间。
- 可以放任意位置、放任意多个 `<script>`。
- ✅ 一般放在 `<body>` 底部，可加快显示速度。

```html
<body>
  <script>
    alert('Hello JS')
  </script>
</body>
```

### 2. 外部脚本（独立的 .js 文件）

- 外部 JS 文件里**只有 JS 代码，没有 `<script>` 标签**。
- 引入时 `<script>` 必须是**双标签**，不能自闭合。

```html
<script src="js/demo.js"></script>
```

```javascript
// demo.js 文件内容
alert('Hello JS')
```

### 3. JS 书写规范

- **结束符**：每行以分号结尾；分号可有可无，但一个项目里要保持一致（都加或都不加）。
- **注释**：与 Java 一致 —— 单行 `//`、多行 `/* ... */`。

---

## 三、JS 基础语法

### 1. 输出语句（3 种）

| 方式 | 作用 | 说明 |
|------|------|------|
| `document.write(...)` | 写入 body 区域 | 直接显示在页面上 |
| `window.alert(...)` | 弹出警示框 | 阻塞，需点确定 |
| `console.log(...)` | 输出到控制台 | 调试用，F12 查看 |

```javascript
document.write("Hello JS (document.write)");
window.alert("Hello JS (window.alert)");
console.log("Hello JS (console.log)");
```

### 2. 变量

- 用 **`let`** 关键字声明变量。
- **JS 是弱类型语言**：同一个变量可以存不同类型的值。
- 命名规则：
  - 由字母、数字、下划线 `_`、美元符 `$` 组成，**数字不能开头**。
  - **严格区分大小写**（`name` 和 `Name` 是不同变量）。
  - 不能用关键字（`let`、`if`、`for` 等）。

```javascript
let a = 20;
a = "Hello";   // 可以，因为是弱类型
alert(a);
```

- ⚠️ 早期 JS 用 **`var`** 声明变量，`var` 允许重复声明（容易出错），现在推荐用 `let`。

### 3. 常量

- 用 **`const`** 声明，一旦赋值**不能改变**（不能重新赋值）。

```javascript
const PI = 3.14;
PI = 3.15;   // 报错：常量不可重新赋值
```

### 4. 数据类型

JS 数据类型分 **原始类型** 和 **引用类型**，先学原始类型：

| 类型 | 说明 | `typeof` 返回值 |
|------|------|----------------|
| number | 数字（整数、小数） | `"number"` |
| string | 字符串 | `"string"` |
| boolean | 布尔 true / false | `"boolean"` |
| null | 空值 | `"object"`（⚠️ 历史遗留 bug） |
| undefined | 声明但未赋值 | `"undefined"` |

```javascript
alert(typeof 3);        // number
alert(typeof "A");      // string
alert(typeof true);     // boolean
alert(typeof null);     // object  （⚠️ 注意！）
var a;
alert(typeof a);        // undefined
```

#### 模板字符串（反引号）

- 字符串可用双引号、单引号，也可用 **反引号 `` ` `` **（键盘 tab 上方波浪键）。
- 反引号字符串叫**模板字符串**，用 `${变量}` 拼接字符串和变量。

```javascript
let name = 'Tom';
let age = 18;
// 原始方式（手动拼接，麻烦）
console.log('大家好, 我是' + name + ', 今年' + age + '岁');
// 模板字符串（推荐）
console.log(`大家好, 我是${name}, 今年${age}岁`);
```

### 5. 函数

#### 方式一：具名函数

```javascript
function 函数名(参数1, 参数2, ...) {
    要执行的代码
}
```

```javascript
function add(a, b) {
    return a + b;
}
let result = add(10, 20);   // 调用
alert(result);              // 30
```

⚠️ 注意（弱类型导致）：
- **形参不需要声明类型**，返回值也不需声明类型。
- **实参个数可以和形参不一致**（多传的会被忽略，少传的为 undefined），但建议保持一致。

#### 方式二：匿名函数

匿名函数没有名字，通过变量/表达式调用，有两种写法：

```javascript
// ① 函数表达式
var add = function (a, b) { return a + b; };

// ② 箭头函数（前端开发更常用）
var add = (a, b) => { return a + b; };
```

```javascript
let result = add(10, 20);
alert(result);   // 30
```

### 6. 自定义对象

```javascript
let 对象名 = {
    属性名1: 属性值1,
    属性名2: 属性值2,
    方法名称: function(形参列表){ }
};
```

```javascript
let user = {
    name: "Tom",
    age: 10,
    sing: function() { console.log("悠悠的唱着最炫的民族风~"); }
};

console.log(user.name);   // 调用属性
user.sing();              // 调用方法
```

✅ 方法可以简写：

```javascript
let user = {
    name: "Tom",
    sing() { console.log("..."); }   // 简写，省略 : function
};
```

### 7. JSON

- **JSON** = JavaScript Object Notation（JS 对象标记法），用 JS 标记法书写的文本。
- 格式：`{"key": value, ...}`，**key 必须用双引号**，value 任意类型。
- 常用于**网络数据传输**（作为数据载体）。

| API | 作用 |
|-----|------|
| `JSON.stringify(obj)` | JS 对象 → JSON 字符串 |
| `JSON.parse(str)` | JSON 字符串 → JS 对象 |

```javascript
let person = { name: 'itcast', age: 18, gender: '男' };
alert(JSON.stringify(person));   // {"name":"itcast","age":18,"gender":"男"}

let personJson = '{"name": "heima", "age": 18}';
alert(JSON.parse(personJson).name);   // heima
```

### 8. 流程控制

与 Java 完全一致，机制相同：

- `if ... else if ... else`
- `switch`
- `for`
- `while`
- `do ... while`

### 9. DOM 操作

#### DOM 介绍

- **DOM** = Document Object Model 文档对象模型，浏览器把 HTML 各部分封装成对象：

| 对象 | 含义 |
|------|------|
| Document | 整个文档对象 |
| Element | 元素对象 |
| Attribute | 属性对象 |
| Text | 文本对象 |
| Comment | 注释对象 |

- DOM 的作用：改变元素内容、改变样式（CSS）、对事件作出反应、增删元素。

#### DOM 操作

- **核心思想**：把网页内容当对象处理，改对象属性就自动映射到标签上。
- 所有内容都封装在 `document` 对象里。
- 操作两步：① 获取 DOM 元素对象 → ② 操作属性/方法。

**获取元素（推荐，用 CSS 选择器）：**

| 方法 | 作用 |
|------|------|
| `document.querySelector('选择器')` | 获取匹配的**第一个**元素 |
| `document.querySelectorAll('选择器')` | 获取匹配的**所有**元素 |

> ⚠️ `querySelectorAll` 返回 NodeList（伪数组：有长度、有索引，但没有 push/pop 等方法）。

```javascript
let hs = document.querySelectorAll('h1');
hs[0].innerHTML = '修改后的文本内容';   // 改第一个 h1 的内容
```

> 早期获取方式（了解即可）：`getElementById` / `getElementsByTagName` / `getElementsByName` / `getElementsByClassName`。

---

## 四、JS 事件监听

### 1. 事件介绍

- **事件** = 发生在 HTML 元素上的"事情"：按钮被点击、鼠标移入、输入框失去焦点、键盘按下……
- **事件监听** = 给事件绑定函数，事件触发时自动执行。

### 2. 事件监听语法（addEventListener）

```javascript
事件源.addEventListener('事件类型', 要执行的函数);
```

**三个要素：**

| 要素 | 含义 | 例子 |
|------|------|------|
| 事件源 | 哪个 DOM 元素触发了事件 | `#btn1` |
| 事件类型 | 用什么方式触发 | `click`、`mouseover` |
| 执行函数 | 要做什么事 | `() => { ... }` |

```javascript
document.querySelector("#btn1").addEventListener('click', () => {
    alert("按钮被点击了...");
});
```

**另外两种（早期写法，了解）：**

1. HTML 标签事件属性绑定：`onclick="on()"`
2. DOM 元素事件属性绑定：`元素.onclick = function(){}`

> ✅ **区别**：`on` 方式会被覆盖；`addEventListener` 可绑定多次、特性更多，**推荐用 addEventListener**。

### 3. 常见事件

| 事件 | 触发时机 |
|------|----------|
| `click` | 鼠标单击 |
| `mouseenter` | 鼠标移入 |
| `mouseleave` | 鼠标移出 |
| `keydown` | 键盘键按下 |
| `keyup` | 键盘键抬起 |
| `blur` | 失去焦点 |
| `focus` | 获得焦点 |
| `input` | 用户输入时 |
| `submit` | 表单提交 |

### 4. 隔行换色案例（核心代码）

```javascript
// 鼠标移入背景变 #f2e2e2，移出变白
const trs = document.querySelectorAll('tr');
for (let i = 1; i < trs.length; i++) {
    trs[i].addEventListener('mouseenter', function () {
        this.style.backgroundColor = '#f2e2e2';
    });
    trs[i].addEventListener('mouseleave', function () {
        this.style.backgroundColor = '#fff';
    });
}
```

---

## 五、重点速查表

### 变量声明三兄弟

| 关键字 | 可否重复声明 | 可否重新赋值 | 使用场景 |
|--------|:---:|:---:|------|
| `var` | ✅ 可以 | ✅ 可以 | 旧语法，少用 |
| `let` | ❌ 不行 | ✅ 可以 | **声明变量（推荐）** |
| `const` | ❌ 不行 | ❌ 不行 | **声明常量** |

### 类型速记

| 值 | typeof | 备注 |
|----|--------|------|
| `3` / `3.14` | number | 数字 |
| `"A"` | string | 字符串 |
| `true` | boolean | 布尔 |
| `null` | object | ⚠️ 历史 bug |
| 声明未赋值 | undefined | 未定义 |

### DOM 核心速记

| 需求 | 代码 |
|------|------|
| 获取第一个匹配元素 | `document.querySelector('选择器')` |
| 获取所有匹配元素 | `document.querySelectorAll('选择器')` |
| 修改元素内容 | `元素.innerHTML = '新内容'` |
| 修改元素样式 | `元素.style.背景属性 = '值'` |

### 事件绑定速记

```javascript
事件源.addEventListener('事件类型', () => { 要做什么 });
```

### JSON 转换速记

```javascript
JSON.stringify(obj)   // 对象 → 字符串
JSON.parse(str)       // 字符串 → 对象
```

---


