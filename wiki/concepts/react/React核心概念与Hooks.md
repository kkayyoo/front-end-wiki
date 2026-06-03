---
title: React核心概念与Hooks
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - https://github.com/wxlvip/Interviewer
tags: [React, Hooks, 组件通信, 生命周期, Fiber, 面试题]
---

# React 核心概念与 Hooks

## Class 组件 vs 函数组件

| 对比项 | Class 组件 | 函数组件（+Hooks）|
|---|---|---|
| 语法 | ES6 class，需要 this | 普通函数，更简洁 |
| 状态 | this.state | useState |
| 生命周期 | componentDidMount 等 | useEffect |
| 性能 | 需要实例化 | 直接执行，更轻 |
| 复用逻辑 | HOC / render props | 自定义 Hook |
| 官方推荐 | 不推荐新项目使用 | ✅ 推荐 |

**Class 组件的主要缺点**：
- 业务逻辑分散在各个生命周期方法中
- `this` 的指向问题令人困惑（事件需要手动 bind）
- 难以拆分和复用状态逻辑

> **实际开发**：新的 Angular 项目使用 Signals，React 项目全面转向 Hooks。理解两者差异是大厂面试必考。

## 常用 Hooks

### useState

```typescript
const [count, setCount] = useState<number>(0);

// 函数式更新（基于前一个状态时使用）
setCount(prev => prev + 1);
```

### useEffect

```typescript
// 模拟 componentDidMount + componentDidUpdate
useEffect(() => {
  // Azure Data Copy 场景：订阅数据复制进度
  const subscription = subscribeToJobStatus(jobId, setStatus);

  // 返回清理函数（模拟 componentWillUnmount）
  return () => subscription.unsubscribe();
}, [jobId]); // 依赖数组：jobId 变化时重新执行
```

### useCallback 和 useMemo

```typescript
// useCallback：记忆函数，防止子组件不必要重渲染
const handleSubmit = useCallback((data: FormData) => {
  submitJob(data);
}, [submitJob]);

// useMemo：记忆计算结果
const filteredJobs = useMemo(
  () => jobs.filter(job => job.status === activeFilter),
  [jobs, activeFilter]
);
```

**区别**：
- `useCallback(fn, deps)` → 返回记忆后的**函数**
- `useMemo(() => value, deps)` → 返回记忆后的**值**（会执行函数）

### useRef

```typescript
// 访问 DOM
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();

// 保存不触发重渲染的值（如定时器 ID）
const timerRef = useRef<number>();
timerRef.current = setTimeout(() => {}, 1000);
```

### useContext

```typescript
const ThemeContext = createContext<'light' | 'dark'>('light');

// 父组件提供
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>

// 子组件消费（无需逐层传 props）
const theme = useContext(ThemeContext);
```

## React 组件通信

```
父→子：props
子→父：props + 回调函数
跨层级：Context
兄弟/全局：Redux / Zustand / EventEmitter
```

```typescript
// 父 → 子
<JobList jobs={jobs} onSelect={handleSelect} />

// 子 → 父（回调）
const JobItem = ({ onDelete }: { onDelete: (id: string) => void }) => (
  <button onClick={() => onDelete(job.id)}>删除</button>
);

// 跨层级（Context）
const JobContext = createContext<JobStore | null>(null);
```

## 生命周期（Class 组件）

```
挂载：constructor → getDerivedStateFromProps → render → componentDidMount
更新：getDerivedStateFromProps → shouldComponentUpdate → render → getSnapshotBeforeUpdate → componentDidUpdate
卸载：componentWillUnmount
```

**Hooks 等价关系**：
| 生命周期 | Hooks 等价 |
|---|---|
| componentDidMount | `useEffect(() => {}, [])` |
| componentDidUpdate | `useEffect(() => {}, [deps])` |
| componentWillUnmount | `useEffect(() => { return () => {} }, [])` |
| shouldComponentUpdate | `React.memo` / `useMemo` |

## setState 的同步/异步问题

⚠️ **注意版本差异**：

- **React 18 之前**：
  - React 合成事件中：**异步**（批处理）
  - setTimeout / 原生 DOM 事件中：**同步**
- **React 18 及之后（Concurrent Mode）**：
  - 全部**批处理（自动批量更新）**，包括 setTimeout 和 Promise 中
  - 如需强制同步刷新：使用 `flushSync`

```typescript
// React 18：全部批处理
setTimeout(() => {
  setA(1); // 不立即触发渲染
  setB(2); // 合并，只触发一次渲染
});

// 强制同步（React 18）
import { flushSync } from 'react-dom';
flushSync(() => setCount(1)); // 立即更新 DOM
```

## React Fiber

**背景**：大量同步计算任务阻塞 UI 渲染（主线程互斥），页面元素多时超过 16ms 导致掉帧。

**解决方案**：将同步递归的 Diff 改为**可中断的链表遍历**。

```js
// Fiber 节点是一个纯 JS 对象
const fiber = {
  stateNode,  // DOM 实例
  child,      // 第一个子节点
  sibling,    // 下一个兄弟节点
  return,     // 父节点
};
```

**Fiber 执行分两阶段**：
- **阶段一（可中断）**：遍历 Fiber 树，找出需要更新的节点，可被高优先级任务打断
- **阶段二（不可中断）**：批量提交更新到 DOM

## React 事件机制

⚠️ **版本差异**：
- **React 17 之前**：所有事件委托到 `document`
- **React 17 及之后**：事件委托到 **root 容器**（`ReactDOM.createRoot` 的挂载点），解决了多个 React 版本共存时的冲突问题

**合成事件**：React 封装了跨浏览器统一的合成事件（SyntheticEvent），性能更好（事件池）。

```typescript
// 阻止冒泡
const handleClick = (e: React.MouseEvent) => {
  e.stopPropagation();  // 阻止 React 合成事件冒泡
};
// 注意：event.preventDefault() 阻止默认行为，不是 stopPropagation
```

## React.lazy 懒加载

```typescript
import React, { Suspense, lazy } from 'react';

// 懒加载大型组件（如数据表格、图表等）
const DataTable = lazy(() => import('./components/DataTable'));

function App() {
  return (
    <Suspense fallback={<div>加载中...</div>}>
      <DataTable />
    </Suspense>
  );
}
```

**原理**：
1. `React.lazy` 返回一个特殊对象（_status: -1）
2. 首次渲染时触发 `import()`，并 throw 一个 Promise
3. `Suspense` 捕获这个 Promise，显示 fallback
4. Promise resolve 后，重新渲染子组件

## 面试高频题

**Q: useEffect 的依赖数组有什么作用？**

> - 空数组 `[]`：只在挂载和卸载时执行（等同于 componentDidMount + componentWillUnmount）
> - 有依赖 `[a, b]`：a 或 b 变化时重新执行
> - 不传：每次渲染都执行（慎用，容易无限循环）

**Q: React.memo 和 useMemo 的区别？**

> - `React.memo`：包裹**组件**，props 不变则跳过重渲染（浅比较）
> - `useMemo`：在组件内部缓存**计算结果**
> - `useCallback`：在组件内部缓存**函数引用**，常配合 `React.memo` 使用

**Q: React 为什么要用 key？**

> key 帮助 React 识别哪些 DOM 可以复用，哪些需要重建。key 应该是稳定唯一的（如 id），不要用 index——因为列表重排时 index 变化会导致状态混乱。
