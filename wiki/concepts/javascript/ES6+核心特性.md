---
title: ES6+ 核心特性
type: concept
created: 2026-06-03
updated: 2026-06-03
tags: [javascript, ES6, 解构, 展开, 模板字符串, 模块化, 高频面试题]
---

# ES6+ 核心特性

## 解构赋值

```js
// 数组解构
const [a, b, ...rest] = [1, 2, 3, 4, 5];
// a=1, b=2, rest=[3,4,5]

// 对象解构（最常用）
const { name, age = 18, address: { city } } = user;
// 默认值、重命名、嵌套解构

// 函数参数解构
function render({ title, content, footer = '默认底部' }) {
  // 避免 options.title 的写法
}

// 交换变量
let x = 1, y = 2;
[x, y] = [y, x]; // x=2, y=1
```

## 展开运算符 vs 剩余参数

```js
// 展开（spread）：数组/对象展开
const arr = [...arr1, ...arr2];     // 合并数组
const obj = { ...defaults, ...override }; // 合并对象（后者覆盖前者）
Math.max(...numbers);               // 展开为参数

// 剩余参数（rest）：收集多余参数
function sum(first, ...rest) {
  return rest.reduce((acc, n) => acc + n, first);
}
sum(1, 2, 3, 4); // 10
```

## 模板字符串

```js
const name = '张三';
const age = 25;

// 基本用法
`Hello, ${name}! 你今年 ${age} 岁。`

// 多行字符串
const html = `
  <div>
    <h1>${title}</h1>
    <p>${content}</p>
  </div>
`;

// 标签模板（高级）：国际化、SQL 防注入
const query = sql`SELECT * FROM users WHERE id = ${userId}`;
// sql 函数可以对 userId 做转义，防 SQL 注入
```

## 可选链 ?. 和 空值合并 ??

```js
// 可选链：避免 "Cannot read property of undefined"
const city = user?.address?.city;         // 不存在时返回 undefined，不报错
const firstItem = arr?.[0];               // 数组
const result = obj?.method?.();           // 方法调用

// 空值合并：只有 null/undefined 时才用默认值
const name = user.name ?? '匿名';
// vs || 的区别：|| 会把 0、''、false 也视为假值
const count = data.count ?? 0;  // ✅ count=0 时不触发
const count = data.count || 0;  // ❌ count=0 时也触发，变成 0

// 实际开发：配置项处理
const config = {
  timeout: userConfig.timeout ?? 3000,
  retries: userConfig.retries ?? 3,
};
```

> **实际踩坑**：`?.` 和 `??` 是 ES2020，老项目需要 Babel 转译。不要在需要明确报错的地方滥用 `?.`（会掩盖真正的 bug）。

## 模块化（ESM）

```js
// 命名导出
export const PI = 3.14;
export function add(a, b) { return a + b; }
export class Calculator { ... }

// 默认导出（一个模块只能有一个）
export default function main() { ... }

// 导入
import React, { useState, useEffect } from 'react';
import * as utils from './utils';
import defaultExport from './module';

// 动态导入（懒加载）
const module = await import('./heavy-module.js');
// React 中：
const LazyComponent = React.lazy(() => import('./LazyComponent'));
```

## 迭代器与 Generator

```js
// 可迭代协议：实现 Symbol.iterator
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    return {
      next() {
        return current <= end
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      }
    };
  }
}
for (const n of new Range(1, 5)) console.log(n); // 1 2 3 4 5

// Generator：可暂停的函数
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++;
  }
}
const gen = idGenerator();
gen.next().value; // 1
gen.next().value; // 2

// 实际应用：Redux-Saga 的核心就是 Generator
```

## Proxy 与 Reflect

```js
// Proxy：拦截对象操作（Vue3 响应式原理）
const reactive = (obj) => new Proxy(obj, {
  get(target, key, receiver) {
    console.log(`读取 ${key}`);
    return Reflect.get(target, key, receiver);
  },
  set(target, key, value, receiver) {
    console.log(`设置 ${key} = ${value}`);
    return Reflect.set(target, key, value, receiver);
  }
});

const state = reactive({ count: 0 });
state.count; // 读取 count
state.count = 1; // 设置 count = 1
```

> **面试要点**：Vue2 用 `Object.defineProperty`，只能拦截已有属性，新增/删除属性需要 `$set/$delete`。Vue3 改用 `Proxy`，可拦截任何操作，性能更好，无需特殊 API。

## 面试高频题

**Q: `??` 和 `||` 的区别？**
> `||` 左侧为任何假值（`0, '', false, null, undefined, NaN`）时取右侧；`??` 只有 `null` 或 `undefined` 时才取右侧。处理数字、布尔值时推荐用 `??`。

**Q: ESM 和 CommonJS 的区别？**
| | ESM | CommonJS |
|--|-----|----------|
| 语法 | `import/export` | `require/module.exports` |
| 加载 | 静态分析，编译时 | 运行时动态加载 |
| 异步 | 支持 | 不支持 |
| Tree-shaking | 支持 | 不支持 |
| 循环依赖 | 引用绑定（live binding） | 值拷贝 |

**Q: WeakMap 和 Map 的区别？**
> - `WeakMap` 的键只能是对象，弱引用（不阻止 GC），不可迭代
> - `Map` 键可以是任意类型，强引用，可迭代
> - 用途：`WeakMap` 适合缓存对象相关数据（对象被 GC 后自动清理），如手写深拷贝中缓存已拷贝对象
