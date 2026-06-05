---
title: React 高频面试题 - 状态管理与数据获取
type: concept
created: 2026-06-04
updated: 2026-06-04
sources:
  - https://github.com/yangshun/front-end-interview-handbook
tags: [React, 状态管理, 数据获取, Context, TanStack Query, 面试题]
---

# React 高频面试题 - 状态管理与数据获取

> 针对 6 年前端经验候选人，结合 Azure Data Copy 业务场景（大量异步 API 调用）整理。

---

## 一、状态管理选型：Context vs Redux vs Zustand

### 选型指南

| 方案 | 适合场景 | 不适合 |
|------|---------|--------|
| **本地 useState** | 单组件内部状态 | 跨组件共享 |
| **useState + 提升** | 兄弟组件共享，层级浅 | 层级深（prop drilling） |
| **Context** | 低频更新的全局数据（主题、登录用户、语言） | 高频更新（表单、搜索） |
| **Zustand / Jotai** | 中大型应用，需要细粒度订阅，不想引入 Redux 样板 | 极简场景 |
| **Redux Toolkit** | 大型团队项目，需要严格的单向数据流、DevTools、时间旅行调试 | 小项目（太重） |

### Context 的适用场景与陷阱

**适合用 Context 的场景**（来自原文）：
- 主题切换（dark/light mode）
- 当前登录用户信息
- 多语言 / 国际化
- 路由状态
- 通知/Toast 管理
- 全局 Modal 状态

**Context 核心陷阱**：

```tsx
// ❌ 高频更新放进 Context → 全树重渲染
const SearchContext = createContext<{ query: string; setQuery: Dispatch<SetStateAction<string>> } | null>(null);

function App() {
  const [query, setQuery] = useState(''); // 每次输入都更新 context

  return (
    <SearchContext.Provider value={{ query, setQuery }}>
      <Header />   {/* 不关心 query，但每次输入都重渲染 */}
      <Sidebar />  {/* 同上 */}
      <Results />
    </SearchContext.Provider>
  );
}
```

```tsx
// ✅ 拆分 context / 保持 context value 稳定
const ThemeContext = createContext<'light' | 'dark'>('light');   // 低频
const UserContext = createContext<User | null>(null);            // 低频

// 高频搜索状态留在局部组件，不放 Context
```

**useMemo 稳定 Context value**：

```tsx
function TodoListContainer() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [status, setStatus] = useState<string>('in_progress');

  // ✅ memoize，避免每次 render 生成新数组引用导致全体消费者重渲染
  const filteredTodos = useMemo(
    () => todos.filter((t) => t.status === status),
    [todos, status]
  );

  return (
    <TodoListContext.Provider value={filteredTodos}>
      <TodoList />
    </TodoListContext.Provider>
  );
}
```

### Zustand 简洁示例（对比 Redux）

```tsx
// Zustand：极少样板，支持细粒度订阅
import { create } from 'zustand';

interface CopyJobStore {
  jobs: CopyJob[];
  activeJobId: string | null;
  setActiveJob: (id: string) => void;
  updateJobStatus: (id: string, status: CopyJob['status']) => void;
}

const useCopyJobStore = create<CopyJobStore>((set) => ({
  jobs: [],
  activeJobId: null,
  setActiveJob: (id) => set({ activeJobId: id }),
  updateJobStatus: (id, status) =>
    set((state) => ({
      jobs: state.jobs.map((job) =>
        job.id === id ? { ...job, status } : job
      ),
    })),
}));

// 细粒度订阅：只订阅 activeJobId，jobs 变化不触发重渲染
function JobHeader() {
  const activeJobId = useCopyJobStore((state) => state.activeJobId);
  return <h2>Active: {activeJobId}</h2>;
}
```

### 面试问答

**Q: 什么时候用 Context，什么时候用 Zustand/Redux？**

A: 
- Context 是 React 内置，零依赖，适合**低频、树形共享**的数据（主题、用户信息）。它不是性能优化工具，每次 value 变化整棵消费子树重渲染。
- 当数据**更新频繁**，或需要**细粒度订阅**（只重渲染用到特定字段的组件），用 Zustand/Jotai 这类外部 store。
- Redux 适合**大型团队**需要严格约束、历史追踪的场景，现代项目更推荐 Zustand（更轻，同样支持 DevTools）。

---

## 二、TanStack Query (React Query) 核心概念

### 为什么用 TanStack Query

手写 `useEffect + fetch` 的问题（来自原文）：

| 问题 | useEffect + fetch | TanStack Query |
|------|------------------|----------------|
| 缓存 | 无 | 自动缓存 |
| 重复请求去重 | 需手写 | 内置 |
| 竞争条件 | 需手写 AbortController | 内置处理 |
| 错误重试 | 需手写 | 自动重试（可配置） |
| 后台刷新 | 需手写轮询 | `refetchInterval` |
| 乐观更新 | 手写 + 手写回滚 | 内置 `onMutate` / `onError` |
| 分页 | 需手写 | `useInfiniteQuery` |

### 核心概念

**queryKey**：唯一标识一个查询，也是缓存的 key。

```tsx
// queryKey 建议：[资源类型, 过滤参数]
useQuery({ queryKey: ['copy-jobs', { projectId, status }], queryFn: ... })
```

**staleTime vs cacheTime（gcTime）**：

```
请求完成
    │
    ├── staleTime 内：数据是 fresh，不会自动重新请求
    │
    └── staleTime 到期：数据变 stale，下次访问时后台重新请求（同时展示缓存旧数据）
         │
         └── 所有 observer 卸载后，gcTime（默认 5 min）后从内存清除
```

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// 基础查询
function CopyJobList({ projectId }: { projectId: string }) {
  const { data: jobs, isLoading, error } = useQuery({
    queryKey: ['copy-jobs', projectId],
    queryFn: async () => {
      const res = await fetch(`/api/projects/${projectId}/jobs`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res.json() as Promise<CopyJob[]>;
    },
    staleTime: 10_000,        // 10秒内数据视为新鲜，不重新请求
    refetchInterval: 5_000,  // 每5秒后台刷新（轮询 running 状态）
    refetchIntervalInBackground: false, // tab 不在前台时暂停轮询
  });

  if (isLoading) return <p>Loading jobs...</p>;
  if (error) return <p>Error: {(error as Error).message}</p>;

  return (
    <ul>
      {jobs?.map((job) => <li key={job.id}>{job.id}: {job.status}</li>)}
    </ul>
  );
}
```

**条件查询（enabled）**：

```tsx
// 只有 projectId 存在时才发请求
const { data } = useQuery({
  queryKey: ['jobs', projectId],
  queryFn: () => fetchJobs(projectId!),
  enabled: !!projectId, // projectId 为空时不请求
});
```

---

## 三、竞争条件（Race Condition）处理方案

### 问题描述

用户快速切换 projectId，旧请求（慢）可能在新请求（快）之后返回，导致界面显示过时数据。

### 方案对比

**方案一：isMounted flag（简单场景）**

```tsx
useEffect(() => {
  let isMounted = true;

  fetch(`/api/projects/${projectId}/jobs`)
    .then((r) => r.json())
    .then((data) => {
      if (isMounted) setJobs(data); // 只有还挂载时才更新
    });

  return () => { isMounted = false; };
}, [projectId]);
```

**方案二：AbortController（推荐，真正取消请求）**

```tsx
useEffect(() => {
  const controller = new AbortController();

  async function fetchJobs() {
    try {
      const res = await fetch(`/api/projects/${projectId}/jobs`, {
        signal: controller.signal,
      });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      setJobs(data);
    } catch (err) {
      if ((err as Error).name === 'AbortError') return; // 正常取消，忽略
      setError(err as Error);
    }
  }

  fetchJobs();

  return () => controller.abort(); // 切换 projectId 时取消旧请求
}, [projectId]);
```

**方案三：TanStack Query（生产推荐）**

TanStack Query 内部已经处理竞争条件，旧 queryKey 的响应不会覆盖新 queryKey 的数据。

```tsx
// 直接用 TanStack Query，不需要手写任何 race condition 处理
const { data } = useQuery({
  queryKey: ['jobs', projectId],
  queryFn: () => fetchJobs(projectId),
});
```

### 面试问答

**Q: 防抖（debounce）能解决竞争条件吗？**

A: 不能完全解决。防抖减少了请求数量，降低了竞争概率，但如果防抖窗口内最后一次请求延迟比下一次请求更长，仍然会有竞争。必须配合 AbortController 或 ignore flag 才能彻底解决。

---

## 四、乐观更新（Optimistic Update）实现

### 原理

不等服务器响应，立即更新 UI；如果请求失败，回滚到之前状态。

**适用场景**：点赞、收藏、标记完成、评论提交——成功率高、用户体验要求快。

### 手写实现

```tsx
// Azure Data Copy 场景：标记 job 为已取消
function JobActions({ job }: { job: CopyJob }) {
  const [localStatus, setLocalStatus] = useState(job.status);

  const handleCancel = async () => {
    const previousStatus = localStatus;

    // 1. 乐观更新 UI
    setLocalStatus('cancelled');

    try {
      // 2. 发送请求
      await fetch(`/api/copy-jobs/${job.id}/cancel`, { method: 'POST' });
      // 3. 成功：保持 UI 不变
    } catch (err) {
      // 4. 失败：回滚
      setLocalStatus(previousStatus);
      alert('Cancel failed, please retry.');
    }
  };

  return (
    <div>
      <span>Status: {localStatus}</span>
      {localStatus === 'running' && (
        <button onClick={handleCancel}>Cancel</button>
      )}
    </div>
  );
}
```

### TanStack Query 乐观更新

```tsx
function useCancelJob() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (jobId: string) =>
      fetch(`/api/copy-jobs/${jobId}/cancel`, { method: 'POST' }).then((r) => r.json()),

    onMutate: async (jobId) => {
      // 取消正在进行的 refetch，避免覆盖乐观更新
      await queryClient.cancelQueries({ queryKey: ['copy-jobs'] });

      // 保存快照（用于回滚）
      const previousJobs = queryClient.getQueryData<CopyJob[]>(['copy-jobs']);

      // 乐观更新缓存
      queryClient.setQueryData<CopyJob[]>(['copy-jobs'], (old) =>
        old?.map((job) =>
          job.id === jobId ? { ...job, status: 'cancelled' } : job
        ) ?? []
      );

      return { previousJobs }; // 返回 context 供 onError 使用
    },

    onError: (_err, _jobId, context) => {
      // 失败时回滚
      if (context?.previousJobs) {
        queryClient.setQueryData(['copy-jobs'], context.previousJobs);
      }
    },

    onSettled: () => {
      // 无论成功失败，最终重新拉取服务端真实数据
      queryClient.invalidateQueries({ queryKey: ['copy-jobs'] });
    },
  });
}
```

---

## 五、Azure Data Copy 业务场景综合示例

结合以上所有概念，一个完整的数据复制任务列表组件：

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

interface CopyJob {
  id: string;
  name: string;
  status: 'pending' | 'running' | 'completed' | 'failed' | 'cancelled';
  progress: number;
  sizeMB: number;
}

// ========== API 函数 ==========
const api = {
  getJobs: async (projectId: string): Promise<CopyJob[]> => {
    const res = await fetch(`/api/projects/${projectId}/copy-jobs`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  },
  retryJob: async (jobId: string): Promise<CopyJob> => {
    const res = await fetch(`/api/copy-jobs/${jobId}/retry`, { method: 'POST' });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  },
};

// ========== 自定义 Hook ==========
function useCopyJobs(projectId: string) {
  return useQuery({
    queryKey: ['copy-jobs', projectId],
    queryFn: () => api.getJobs(projectId),
    staleTime: 5_000,
    refetchInterval: (query) => {
      // 只有有 running 任务时才轮询
      const jobs = query.state.data;
      return jobs?.some((j) => j.status === 'running') ? 3_000 : false;
    },
  });
}

function useRetryJob(projectId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: api.retryJob,
    onMutate: async (jobId) => {
      await queryClient.cancelQueries({ queryKey: ['copy-jobs', projectId] });
      const prev = queryClient.getQueryData<CopyJob[]>(['copy-jobs', projectId]);

      queryClient.setQueryData<CopyJob[]>(['copy-jobs', projectId], (old) =>
        old?.map((j) => (j.id === jobId ? { ...j, status: 'pending' } : j)) ?? []
      );

      return { prev };
    },
    onError: (_err, _jobId, ctx) => {
      if (ctx?.prev) {
        queryClient.setQueryData(['copy-jobs', projectId], ctx.prev);
      }
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['copy-jobs', projectId] });
    },
  });
}

// ========== 组件 ==========
function CopyJobDashboard({ projectId }: { projectId: string }) {
  const { data: jobs, isLoading, error } = useCopyJobs(projectId);
  const retryMutation = useRetryJob(projectId);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {(error as Error).message}</div>;

  return (
    <table>
      <tbody>
        {jobs?.map((job) => (
          <tr key={job.id}>
            <td>{job.name}</td>
            <td>{job.status}</td>
            <td>{job.sizeMB} MB</td>
            <td>
              {job.status === 'failed' && (
                <button
                  onClick={() => retryMutation.mutate(job.id)}
                  disabled={retryMutation.isPending}
                >
                  {retryMutation.isPending ? 'Retrying...' : 'Retry'}
                </button>
              )}
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

---

## 六、面试高频问答汇总

| 问题 | 回答要点 |
|-----|---------|
| Context 的缺点？ | 每次 value 变化整棵消费树重渲染；不可静态分析；一旦加入难以移除；不适合高频更新 |
| TanStack Query 的 staleTime 作用？ | 指定多长时间内数据视为新鲜，窗口内不发新请求，窗口后变 stale 下次访问时后台重新请求 |
| 如何避免 useEffect 数据请求的竞争条件？ | AbortController 在 cleanup 中取消旧请求；或用 isMounted flag 忽略旧响应；生产建议直接用 TanStack Query |
| 乐观更新失败怎么办？ | 在 catch/onError 中用保存的快照 rollback，同时 toast 提示用户操作失败 |
| 何时用 useReducer 而非 useState？ | 多个相关 state 字段、复杂状态转换、状态间有约束关系（如 isSubmitting/isSubmitted 改成 status 枚举） |
| 如何防止 Context 引发不必要重渲染？ | 拆分 context（高频/低频分开）；用 useMemo 稳定 value；考虑换用 Zustand |
