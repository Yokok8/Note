# Vue 知识点笔记

> 依据课程文档《02-前端Web开发(JS+Vue+Ajax)》整理，**只保留 Vue 框架部分**，适合课后快速复习。

---

## 目录

1. [Vue 是什么](#一vue-是什么)
2. [Vue 快速上手](#二vue-快速上手)
3. [Vue 指令](#三vue-指令)
4. [Vue 生命周期](#四vue-生命周期)
5. [重点速查表](#五重点速查表)

---

## 一、Vue 是什么

- **Vue**（读 /vjuː/）是一款用于**构建用户界面**的**渐进式** JavaScript **框架**。官网：https://cn.vuejs.org

### 三个关键词

| 关键词 | 含义 |
|--------|------|
| 构建用户界面 | 基于**数据**渲染出用户看到的界面（数据驱动视图） |
| 渐进式 | 学一点就能用一点，不必全学会才能开发（声明式渲染 → 组件 → 路由 → 状态管理） |
| 框架 | 一套完整的项目解决方案，用于快速构建项目 |

- **优点**：大幅提升前端开发效率。
- **缺点**：需理解记忆使用规则（对照官网）。

---

## 二、Vue 快速上手

### 1. 入门程序三步走（固定步骤）

1. 准备 HTML 文件，引入 Vue 模块（模块化 JS 要加 `type="module"`）
2. 创建 Vue 应用实例，控制视图元素
3. 准备元素（`<div id="app">`）交给 Vue 控制

### 2. 数据驱动视图

- 用 `data()` 方法定义数据（返回值是对象）
- 用插值表达式 `{{...}}` 渲染数据

```html
<div id="app">
  {{ message }}
</div>

<script type="module">
  import { createApp } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'
  createApp({
    data() {
      return {
        message: 'Hello Vue'
      }
    }
  }).mount('#app')
</script>
```

⚠️ 注意：

- 数据必须通过 `data()` 方法定义。
- 插值表达式里的变量必须是 Vue 中定义过的数据，否则报错。
- 插值表达式要写在 `#app`（Vue 接管区域）**里面**。

---

## 三、Vue 指令

### 1. 概述

- **指令** = HTML 标签上带 `v-` 前缀的特殊属性，不同指令有不同功能。

```html
<p v-xxx="....">.....</p>
```

### 2. 五大常用指令

| 指令 | 简写 | 作用 |
|------|:---:|------|
| `v-for` | — | 列表渲染（遍历数组/对象） |
| `v-bind` | `:` | 动态绑定属性值（href、src、style 等） |
| `v-if` / `v-show` | — | 控制元素的显示与隐藏 |
| `v-model` | — | 表单双向数据绑定 |
| `v-on` | `@` | 绑定事件 |

#### ① v-for（列表渲染）

```html
<tr v-for="(item, index) in items" :key="item.id">{{ item }}</tr>
```

- `items`：遍历的数组（必须在 data 中定义）
- `item`：遍历出的元素
- `index`：下标，从 0 开始（可省略：`v-for="item in items"`）
- `:key`：元素的唯一标识，**推荐用 id，不推荐用 index**（会变化）。

#### ② v-bind（动态绑定属性）

```html
<img v-bind:src="item.image" width="30px">
<img :src="item.image" width="30px">   <!-- 简写 -->
```

- 绑定的数据必须在 data 中定义（或由 data 数据推导而来）。

#### ③ v-if 与 v-show（显示隐藏）

| 对比 | v-if | v-show |
|------|------|--------|
| 原理 | 条件判断，**创建/移除**元素节点 | 控制 CSS 的 **display** |
| 场景 | 不频繁切换 | 频繁切换 |
| 配套 | 可配合 `v-else-if` / `v-else` | 无 |

```html
<span v-if="emp.job === '1'">班主任</span>
<span v-else-if="emp.job === '2'">讲师</span>
<span v-else>其他</span>

<span v-show="emp.job === '1'">班主任</span>
```

⚠️ `v-else-if` 必须在 `v-if` 之后；`v-else` 必须在 `v-if`/`v-else-if` 之后。

#### ④ v-model（双向数据绑定）

- 用于**表单元素**，方便获取或设置表单项数据。
- **双向**：Vue 数据变化 → 视图更新；视图输入变化 → Vue 数据更新。

```html
<input type="text" v-model="searchEmp.name">
```

⚠️ `v-model` 绑定的变量必须在 data 中定义。

#### ⑤ v-on（绑定事件）

```html
<input type="button" value="点我" v-on:click="handle">
<input type="button" value="点我" @click="handle">   <!-- 简写 -->
```

- `handle` 函数要定义在 Vue 实例的 **`methods`** 中。
- ⚠️ `methods` 中的 `this` 指向 Vue 实例，可通过 `this.xxx` 获取 data 中的数据。

```javascript
createApp({
  data() { return { searchEmp: { name: '' } } },
  methods: {
    search() { console.log(this.searchEmp) },
    clear() { this.searchEmp = { name: '' } }
  }
}).mount('#app')
```

---

## 四、Vue 生命周期

- **生命周期**：Vue 对象从创建到销毁的过程，共 **8 个阶段**，每阶段自动执行对应的钩子方法。

| 阶段 | 钩子方法 | 说明 |
|------|---------|------|
| 创建前 | beforeCreate | 实例初始化前 |
| 创建后 | created | 实例创建完成 |
| 挂载前 | beforeMount | 渲染到页面之前 |
| 挂载后 | **mounted** | ✅ 挂载完成，页面渲染成功 |
| 更新前 | beforeUpdate | 数据变化、DOM 更新前 |
| 更新后 | updated | DOM 更新完成 |
| 卸载前 | beforeUnmount | 组件销毁前 |
| 卸载后 | unmounted | 组件销毁完成 |

> ✅ **重点记住 `mounted`**：挂载完成，页面渲染成功，常用于**页面初始化时自动执行操作（如加载数据）**，其余了解即可。

```javascript
createApp({
  data() { return { message: 'Hello' } },
  methods: {
    init() {
      // 页面加载后要执行的初始化逻辑
    }
  },
  mounted() {
    this.init()   // 页面加载完自动执行
  }
}).mount('#app')
```

---

## 五、重点速查表

### Vue 指令速记

| 需求 | 写法 |
|------|------|
| 遍历列表 | `v-for="(item, index) in list" :key="item.id"` |
| 绑定属性 | `:src="item.image"` |
| 条件渲染 | `v-if="条件"` / `v-else-if` / `v-else` |
| 切换显示 | `v-show="条件"`（频繁切换用） |
| 双向绑定 | `v-model="变量"` |
| 绑定事件 | `@click="方法"` |

### Vue 实例结构速记

```javascript
createApp({
  data() { return { /* 数据 */ } },
  methods: { /* 方法 */ },
  mounted() { /* 初始化逻辑 */ }
}).mount('#app')
```

### 一句话总结

- **Vue** = 数据驱动视图的渐进式框架；**指令**是 `v-` 前缀的特殊属性。
- **v-if** 删除节点（不频繁切换），**v-show** 控制 display（频繁切换）。
- **v-for** 遍历列表要配 `:key`；**v-model** 双向绑定表单；**v-on/@** 绑定事件到 `methods`。
- **mounted** 钩子 = 页面渲染完成后自动执行，常用于页面初始化。
