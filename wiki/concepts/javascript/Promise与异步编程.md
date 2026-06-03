---
title: Promise 与异步编程
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - https://juejin.cn/post/7628895336028307456
tags: [javascript, Promise, async/await, 异步, 手写, 高频面试题]
---

# Promise 与异步编程

## Promise 基础

```js
const p = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => resolve('成功'), 1000);
});

p.then(val => console.log(val))   // '成功'
 .catch(err => console.error(err))
 .finally(() => console.log('结束'));
```

**三种状态（不可逆）：**
- `pending` → 初始态
- `fulfilled` → resolve 后
- `rejected` → reject 后

## Promise 链式调用

```js
fetch('/api/user')
  .then(res => res.json())          // 返回新 Promise
  .then(user => fetch(`/api/posts?userId=${user.id}`))
  .then(res => res.json())
  .catch(err => console.error(err)) // 捕获链上所有错误
```

> **关键**：`.then()` 返回新 Promise，链式调用每次都是新的异步层。

## 常用静态方法

```js
// Promise.all：全部成功才 resolve，任一失败即 reject
const results = await Promise.all([
  fetch('/api/a'),
  fetch('/api/b'),
  fetch('/api/c'),
]);
// 实际开发：页面初始化并行请求多个接口，比串行快 2-3 倍

// Promise.allSettled：全部结束（无论成败），返回状态数组
const results = await Promise.allSettled([p1, p2, p3]);
results.forEach(r => {
  if (r.status === 'fulfilled') console.log(r.value);
  else console.error(r.reason);
});
// 实际开发：批量操作（如批量上传），不因单个失败中断

// Promise.race：最快的那个决定结果
const result = await Promise.race([
  fetch('/api/data'),
  new Promise((_, reject) => setTimeout(() => reject('超时'), 3000))
]);
// 实际开发：请求超时控制

// Promise.any：任一成功即 resolve，全部失败才 reject
const result = await Promise.any([mirror1, mirror2, mirror3]);
// 实际开发：多镜像源，取最快响应的
```

## async/await

```js
// async 函数总是返回 Promise
async function fetchUser(id) {
  try {
    const res = await fetch(`/api/user/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.error('请求失败:', err);
    throw err; // 重新抛出，让调用方处理
  }
}

// 并行执行（不要串行！）
async function fetchAll() {
  // ❌ 串行：总时间 = t1 + t2（浪费）
  const a = await fetch('/api/a');
  const b = await fetch('/api/b');

  // ✅ 并行：总时间 = max(t1, t2)
  const [a, b] = await Promise.all([
    fetch('/api/a'),
    fetch('/api/b')
  ]);
}
```

## 手写 Promise（核心版）

```js
class MyPromise {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];

    const resolve = (value) => {
      if (this.state !== 'pending') return;
      this.state = 'fulfilled';
      this.value = value;
      this.onFulfilledCallbacks.forEach(fn => fn(value));
    };

    const reject = (reason) => {
      if (this.state !== 'pending') return;
      this.state = 'rejected';
      this.reason = reason;
      this.onRejectedCallbacks.forEach(fn => fn(reason));
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v;
    onRejected = typeof onRejected === 'function' ? onRejected : e => { throw e; };

    return new MyPromise((resolve, reject) => {
      const handle = (fn, val) => {
        try {
          const result = fn(val);
          result instanceof MyPromise ? result.then(resolve, reject) : resolve(result);
        } catch (e) {
          reject(e);
        }
      };

      if (this.state === 'fulfilled') queueMicrotask(() => handle(onFulfilled, this.value));
      if (this.state === 'rejected') queueMicrotask(() => handle(onRejected, this.reason));
      if (this.state === 'pending') {
        this.onFulfilledCallbacks.push(v => handle(onFulfilled, v));
        this.onRejectedCallbacks.push(r => handle(onRejected, r));
      }
    });
  }

  catch(onRejected) { return this.then(null, onRejected); }

  static resolve(value) {
    return value instanceof MyPromise ? value : new MyPromise(r => r(value));
  }

  static reject(reason) {
    return new MyPromise((_, r) => r(reason));
  }

  static all(promises) {
    return new MyPromise((resolve, reject) => {
      const results = [];
      let count = 0;
      promises.forEach((p, i) => {
        MyPromise.resolve(p).then(val => {
          results[i] = val;
          if (++count === promises.length) resolve(results);
        }, reject);
      });
    });
  }
}
```

## 面试高频题

**Q: Promise 解决了什么问题？**
> 解决了回调地狱（Callback Hell）——嵌套回调难以阅读和维护。Promise 将异步操作的结果链式处理，async/await 让异步代码看起来像同步。

**Q: async/await 的错误处理最佳实践？**
```js
// 方案1：try/catch（推荐，清晰）
async function fn() {
  try {
    const data = await fetchData();
  } catch (err) {
    handleError(err);
  }
}

// 方案2：await-to-js 风格（Go 风格错误处理）
const [err, data] = await to(fetchData());
if (err) return handleError(err);
```

**Q: Promise.all 和 Promise.allSettled 什么时候用哪个？**
> - `Promise.all`：所有请求必须成功，一个失败全部失败（如初始化必要数据）
> - `Promise.allSettled`：关心每个的结果，不因单个失败中断（如批量操作、统计成功率）
