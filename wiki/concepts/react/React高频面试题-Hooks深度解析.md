---
title: React 高频面试题 - Hooks 深度解析
type: concept
created: 2026-06-04
updated: 2026-06-04
sources:
  - https://github.com/yangshun/front-end-interview-handbook
tags: [React, Hooks, useState, useEffect, useCallback, useMemo, 面试题]
---

# React 高频面试题 - Hooks 深度解析

> 针对 6 年前端经验（Angular/React/TypeScript）候选人，结合 Azure Data Copy 业务场景整理。

---

## 一、useCallback vs useMemo 区别

### 核心区别

| Hook | 缓存的是 | 适用场景 |
|------|---------|---------|
| `useMemo` | **计算结果**（值） | 缓存昂贵的计算，避免重复运算 |
| `useCallback` | **函数引用** | 缓存事件处理函数，避免子组件不必要重渲染 |

`useCallback(fn, deps)` 等价于 `useMemo(() => fn, deps)`。

### 代码示例（TypeScript）

```tsx
import { useState, useCallback, useMemo } from 'react';

interface CopyJob {
  id: string;
  status: 'pending' | 'running' | 'completed' | 'failed';
  sizeMB: number;
}

function DataCopyDashboard({ jobs }: { jobs: CopyJob[] }) {
  const [filter, setFilter] = useState<CopyJob['status']>('running');

  // useMemo：缓存过滤后的列表（避免每次 render 重新 filter）
  const filteredJobs = useMemo(
    () => jobs.filter((job) => job.status === filter),
    [jobs, filter]
  );

  // useMemo：缓存统计数值
  const totalSizeMB = useMemo(
    () => filteredJobs.reduce((sum, job) => sum + job.sizeMB, 0),
    [filteredJobs]
  );

  // useCallback：缓存传给子组件的回调，保持引用稳定
  const handleRetry = useCallback((jobId: string) => {
    console.log('Retrying job:', jobId);
    // 调用 API 重试
  }, []); // 无外部依赖，空数组

  return (
    <div>
      <p>Total size: {totalSizeMB} MB</p>
      {filteredJobs.map((job) => (
        <JobRow key={job.id} job={job} onRetry={handleRetry} />
      ))}
    </div>
  );
}

// 用 React.memo 包裹，useCallback 才有意义
const JobRow = React.memo(({ job, onRetry }: { job: CopyJob; onRetry: (id: string) => void }) => {
  console.log('JobRow render:', job.id);
  return (
    <div>
      {job.id} - {job.status}
      {job.status === 'failed' && (
        <button onClick={() => onRetry(job.id)}>Retry</button>
      )}
    </div>
  );
});
```

### 面试问答

**Q: useCallback 和 useMemo 的使用时机？**

A:
- `useMemo` 用于**昂贵计算**（如大数组过滤、聚合统计），避免每次渲染都重算。
- `useCallback` 用于**传给子组件的函数**，配合 `React.memo` 防止子组件因函数引用变化而重渲染。
- ⚠️ 不要滥用：对简单值/函数加 memo 反而有开销（比较依赖数组本身也有成本）。React Compiler 未来会自动处理这些优化。

**Q: 如果不用 React.memo，useCallback 有意义吗？**

A: 基本没意义。`useCallback` 只是稳定函数引用，但如果子组件没有用 `React.memo`，父组件重渲染时子组件无论如何都会重渲染，稳定的引用也无济于事。

---

## 二、useEffect 深度解析

### 核心语法

```ts
useEffect(effectFunction, dependencyArray);
```

| 依赖数组 | 执行时机 |
|---------|---------|
| 省略 | 每次渲染后 |
| `[]` | 只在挂载时 |
| `[a, b]` | a 或 b 变化时 |
| 返回 cleanup | 卸载或下次 effect 运行前 |

### 陷阱一：缺少依赖 → Stale Closure

```tsx
// ❌ 错误：fetchJobs 依赖了 projectId，但未加入依赖数组
useEffect(() => {
  fetchJobs(projectId); // 永远读到初始的 projectId
}, []);

// ✅ 正确
useEffect(() => {
  fetchJobs(projectId);
}, [projectId]);
```

> **规则**：effect 内读取的所有响应式值（props、state、派生变量）都必须在依赖数组中列出。

### 陷阱二：对象/数组作为依赖 → 无限循环

```tsx
// ❌ 每次渲染都创建新数组，effect 无限触发
function TodoList({ todos }: { todos: Todo[] }) {
  const [status, setStatus] = useState('in_progress');
  const filteredTodos = todos.filter((t) => t.status === status); // 新引用

  useEffect(() => {
    console.log('changed');
  }, [filteredTodos]); // ⚠️ 每次都是新引用
}

// ✅ 用 useMemo 稳定引用
const filteredTodos = useMemo(
  () => todos.filter((t) => t.status === status),
  [todos, status]
);
```

### 陷阱三：忘记 cleanup → 内存泄漏

```tsx
// ❌ interval 永远不会被清除
useEffect(() => {
  const interval = setInterval(() => pollJobStatus(), 3000);
}, []);

// ✅ 返回 cleanup 函数
useEffect(() => {
  const interval = setInterval(() => pollJobStatus(), 3000);
  return () => clearInterval(interval); // 组件卸载时清除
}, []);
```

### 陷阱四：竞争条件（Race Condition）

在 Azure Data Copy 场景中，用户切换不同 projectId 时，旧请求可能比新请求更晚返回：

```tsx
// ❌ 有竞争条件
useEffect(() => {
  fetch(`/api/projects/${projectId}/jobs`)
    .then((r) => r.json())
    .then(setJobs); // 旧请求的结果可能覆盖新数据
}, [projectId]);

// ✅ 方案一：isMounted flag
useEffect(() => {
  let isMounted = true;

  fetch(`/api/projects/${projectId}/jobs`)
    .then((r) => r.json())
    .then((data) => {
      if (isMounted) setJobs(data); // 只更新当前有效的请求结果
    });

  return () => { isMounted = false; };
}, [projectId]);

// ✅ 方案二：AbortController（推荐，同时取消网络请求）
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/projects/${projectId}/jobs`, { signal: controller.signal })
    .then((r) => r.json())
    .then(setJobs)
    .catch((err) => {
      if (err.name !== 'AbortError') throw err; // 忽略取消错误
    });

  return () => controller.abort(); // cleanup 时取消请求
}, [projectId]);
```

> ⚠️ 防抖（debounce）并不能完全解决竞争条件，仍需配合 AbortController 或 isMounted flag。

---

## 三、useRef 的多种使用场景

### 场景一：访问 DOM 元素

```tsx
import { useRef } from 'react';

function SearchInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => inputRef.current?.focus();

  return (
    <>
      <input ref={inputRef} type="text" placeholder="Search jobs..." />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

### 场景二：保存不触发重渲染的值（如 timer ID、上一次的值）

```tsx
function PollingComponent({ jobId }: { jobId: string }) {
  const intervalRef = useRef<ReturnType<typeof setInterval> | null>(null);
  const [status, setStatus] = useState<string>('');

  useEffect(() => {
    intervalRef.current = setInterval(async () => {
      const result = await fetchJobStatus(jobId);
      setStatus(result.status);
      if (result.status === 'completed') {
        clearInterval(intervalRef.current!); // 任务完成，停止轮询
      }
    }, 5000);

    return () => clearInterval(intervalRef.current!);
  }, [jobId]);

  return <div>Status: {status}</div>;
}
```

### 场景三：保存最新值，避免 effect 中的 stale closure

```tsx
// Azure Data Copy：回调里需要访问最新的 onComplete prop
function CopyTask({ onComplete }: { onComplete: (result: CopyResult) => void }) {
  // 用 ref 保存最新的 callback，避免在 effect 依赖数组中添加 onComplete
  const onCompleteRef = useRef(onComplete);
  useEffect(() => {
    onCompleteRef.current = onComplete;
  });

  useEffect(() => {
    const ws = new WebSocket('/api/copy-status');
    ws.onmessage = (event) => {
      const result = JSON.parse(event.data) as CopyResult;
      if (result.done) {
        onCompleteRef.current(result); // 始终调用最新版本的回调
      }
    };
    return () => ws.close();
  }, []); // 不需要把 onComplete 加入依赖
}
```

### 面试问答

**Q: useRef 和 useState 的区别？**

A:
- `useState` 更新会触发重渲染；`useRef` 修改 `.current` **不会触发重渲染**。
- `useRef` 适合存储"对 UI 没有影响的数据"，如 timer ID、WebSocket 实例、上一次的值等。
- 不要用 `useRef` 替代 `useState` 管理需要反映在 UI 上的数据。

---

## 四、自定义 Hook 设计原则与实例

### 设计原则

1. **以 `use` 开头**：linter 才能识别并执行 Hooks 调用规则
2. **单一职责**：一个 hook 只做一件事
3. **不暴露内部 setter**：通过封装控制操作边界
4. **可复用性优先**：把重复的 state + effect 逻辑提取出来

### 实例一：`useCopyJobPoller` — Azure Data Copy 场景

```tsx
interface CopyJob {
  id: string;
  status: 'pending' | 'running' | 'completed' | 'failed';
  progress: number;
}

function useCopyJobPoller(jobId: string, intervalMs = 3000) {
  const [job, setJob] = useState<CopyJob | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [isPolling, setIsPolling] = useState(true);

  useEffect(() => {
    if (!isPolling) return;

    const controller = new AbortController();

    const poll = async () => {
      try {
        const res = await fetch(`/api/copy-jobs/${jobId}`, {
          signal: controller.signal,
        });
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const data: CopyJob = await res.json();
        setJob(data);

        // 终态时停止轮询
        if (data.status === 'completed' || data.status === 'failed') {
          setIsPolling(false);
        }
      } catch (err) {
        if ((err as Error).name !== 'AbortError') {
          setError(err as Error);
          setIsPolling(false);
        }
      }
    };

    poll(); // 立即执行一次
    const timer = setInterval(poll, intervalMs);

    return () => {
      controller.abort();
      clearInterval(timer);
    };
  }, [jobId, intervalMs, isPolling]);

  return { job, error, isPolling };
}

// 使用示例
function CopyJobCard({ jobId }: { jobId: string }) {
  const { job, error, isPolling } = useCopyJobPoller(jobId);

  if (error) return <div>Error: {error.message}</div>;
  if (!job) return <div>Loading...</div>;

  return (
    <div>
      <p>Status: {job.status}</p>
      <progress value={job.progress} max={100} />
      {isPolling && <span>🔄 Polling...</span>}
    </div>
  );
}
```

### 实例二：`useDebounce` — 搜索防抖

```tsx
function useDebounce<T>(value: T, delayMs: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delayMs);
    return () => clearTimeout(timer);
  }, [value, delayMs]);

  return debouncedValue;
}

// 使用
function JobSearch() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery) fetchJobs(debouncedQuery);
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

---

## 五、Hooks 规则（Rules of Hooks）

React 通过**调用顺序**追踪每个 Hook 的 state，违反规则会导致状态错乱。

1. **只在顶层调用 Hooks**：不能在条件、循环、嵌套函数中调用
2. **只在 React 函数组件或自定义 Hook 中调用**

```tsx
// ❌ 条件内调用
if (isLoggedIn) {
  const [user, setUser] = useState(null); // 破坏调用顺序
}

// ✅ 正确：始终在顶层
const [user, setUser] = useState(null);
if (isLoggedIn) { /* 使用 user */ }
```

---

## 六、面试常见问题汇总

| 问题 | 要点 |
|-----|-----|
| Hooks 解决了什么问题？ | 函数组件复用状态逻辑；取代 class 生命周期；逻辑复用比 HOC/renderProps 更干净 |
| useState vs useReducer | 简单独立状态用 useState；多个相关状态、复杂转换逻辑用 useReducer |
| useEffect cleanup 何时执行？ | 组件卸载时 + 下次 effect 运行前（依赖变化时） |
| 依赖数组为空 vs 省略？ | `[]` 只运行一次；省略则每次渲染后运行 |
| useMemo 的开销？ | 有存储和比较成本，不是所有计算都值得缓存，简单运算直接算更快 |
