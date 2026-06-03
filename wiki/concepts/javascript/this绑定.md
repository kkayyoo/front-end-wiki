---
title: this 绑定
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - https://juejin.cn/post/7628895336028307456
tags: [javascript, this, call, apply, bind, 箭头函数, 高频面试题]
---

# this 绑定

## 核心规则（优先级从高到低）

| 规则 | 场景 | this 指向 |
|------|------|----------|
| new 绑定 | `new Fn()` | 新创建的对象 |
| 显式绑定 | `call/apply/bind` | 指定的对象 |
| 隐式绑定 | `obj.fn()` | 调用的对象 obj |
| 默认绑定 | `fn()` 直接调用 | 全局对象（严格模式下 undefined） |
| 箭头函数 | `() => {}` | 词法上下文（外层 this，不可改变） |

## 四种绑定方式

```js
function greet(greeting) {
  return `${greeting}, ${this.name}`;
}

const user = { name: '张三' };

// 隐式绑定
user.greet = greet;
user.greet('Hello'); // 'Hello, 张三'

// call：立即调用，参数逐个传
greet.call(user, 'Hi');      // 'Hi, 张三'

// apply：立即调用，参数数组传
greet.apply(user, ['Hey']);  // 'Hey, 张三'

// bind：返回新函数，不立即调用
const boundGreet = greet.bind(user);
boundGreet('Yo');            // 'Yo, 张三'
```

## 隐式绑定丢失（经典陷阱）

```js
const obj = {
  name: '张三',
  greet() { console.log(this.name); }
};

obj.greet(); // '张三' ✅

// ❌ 赋值给变量后，this 丢失
const fn = obj.greet;
fn(); // undefined（严格模式 TypeError）

// ❌ 传入回调，this 丢失
setTimeout(obj.greet, 100); // undefined

// ✅ 修复方案1：bind
setTimeout(obj.greet.bind(obj), 100);

// ✅ 修复方案2：箭头函数包裹
setTimeout(() => obj.greet(), 100);
```

> **实际踩坑**：React class 组件中，事件处理函数必须在 constructor 里 `.bind(this)` 或用箭头函数，否则 `this` 是 undefined。这也是 React Hooks 出现的原因之一。

## 箭头函数的 this

```js
const obj = {
  name: '张三',
  // 普通函数：this 是调用时决定的
  regular() {
    setTimeout(function() {
      console.log(this.name); // undefined（this 指向 window/undefined）
    }, 100);
  },
  // 箭头函数：this 是定义时的词法上下文
  arrow() {
    setTimeout(() => {
      console.log(this.name); // '张三' ✅
    }, 100);
  }
};
```

**箭头函数 this 特性：**
- 没有自己的 `this`，继承外层词法环境的 `this`
- `call/apply/bind` 无法改变箭头函数的 `this`
- 不能作为构造函数（`new` 箭头函数会报错）
- 没有 `arguments` 对象

## 手写 call / apply / bind

```js
// 手写 call
Function.prototype.myCall = function(ctx, ...args) {
  ctx = ctx ?? globalThis;
  const key = Symbol(); // 避免属性名冲突
  ctx[key] = this;      // this 是被调用的函数
  const result = ctx[key](...args);
  delete ctx[key];
  return result;
};

// 手写 bind
Function.prototype.myBind = function(ctx, ...args) {
  const fn = this;
  return function(...innerArgs) {
    // 注意：new 调用时，ctx 失效，this 指向新对象
    if (new.target) {
      return new fn(...args, ...innerArgs);
    }
    return fn.apply(ctx, [...args, ...innerArgs]);
  };
};
```

## 面试高频题

**Q: call、apply、bind 的区别？**
> - `call`：立即调用，参数逐个传
> - `apply`：立即调用，参数数组传（记忆：apply → array）
> - `bind`：返回新函数，不立即调用，可预设参数（柯里化）

**Q: 箭头函数和普通函数的区别？**
> 1. `this`：箭头函数继承词法上下文，普通函数由调用决定
> 2. `arguments`：箭头函数没有，用 `...rest` 替代
> 3. 构造函数：箭头函数不能 `new`
> 4. `prototype`：箭头函数没有 `prototype` 属性
> 5. `yield`：箭头函数不能作为 Generator

**Q: 如何确定 this 指向？（口诀）**
> 1. new 调用 → 新对象
> 2. call/apply/bind → 指定对象
> 3. 对象方法调用 `obj.fn()` → obj
> 4. 直接调用 `fn()` → 全局/undefined（严格模式）
> 5. 箭头函数 → 外层 this（看定义位置）
