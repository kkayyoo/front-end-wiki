---
title: TypeScript核心概念
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - https://github.com/wxlvip/Interviewer
tags: [TypeScript, 泛型, 接口, 类型, 面试题]
---

# TypeScript 核心概念

## TS 与 JS 的区别

TypeScript 是 JavaScript 的**超集**，编译后产生标准 JS 代码。

| 对比项 | TypeScript | JavaScript |
|---|---|---|
| 类型系统 | 静态类型，编译时检查 | 动态类型，运行时发现 |
| 接口 | 支持 `interface` | 不支持 |
| 泛型 | 支持 | 不支持 |
| 枚举 | 支持 `enum` | 不支持 |
| 错误发现 | 开发时暴露 | 运行时暴露 |
| 面向对象 | 完整 OOP 支持 | 基于原型，语法较弱 |

> **实际价值**：在 Azure Data Copy 这类数据密集型项目中，TS 的类型系统能避免大量因数据结构错误导致的 bug，接口定义让 API 返回数据有明确约束。

## 为什么用 TypeScript？

1. **开发时就能发现错误**，而非运行时
2. **代码可读性强**，接口和类型注解即文档
3. **重构安全**：修改接口/类型，IDE 会提示所有受影响的地方
4. **大型团队协作**：明确的类型约定减少沟通成本

## 泛型（Generics）

泛型是指定义函数、接口或类时，不预先指定具体类型，使用时再指定。

```typescript
// ❌ 不用泛型：要么 any（失去类型检查），要么重复写多个版本
function createArray(length: number, value: any): any[] {
  return Array(length).fill(value);
}

// ✅ 用泛型：一个函数处理所有类型，同时保留类型安全
function createArray<T>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

// 使用时指定类型，或让 TS 自动推导
const strArr = createArray<string>(3, 'x');  // string[]
const numArr = createArray(3, 0);             // number[]（自动推导）
```

**实际使用场景（Azure 项目）**：

```typescript
// API 响应包装类型
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface CopyJob {
  id: string;
  source: string;
  destination: string;
  status: 'pending' | 'running' | 'completed' | 'failed';
}

// 泛型约束 API 返回
async function fetchCopyJob(id: string): Promise<ApiResponse<CopyJob>> {
  const res = await fetch(`/api/jobs/${id}`);
  return res.json();
}
// 调用时 data 字段类型是 CopyJob，有完整的 IDE 提示
```

## 接口（Interface）

```typescript
// 基本接口
interface User {
  name: string;
  age?: number;        // 可选属性
  readonly id: string; // 只读属性
}

// 函数类型接口
interface Comparator {
  (a: number, b: number): boolean;
}

// 类实现接口
interface Serializable {
  serialize(): string;
  deserialize(data: string): void;
}

class DataCopyConfig implements Serializable {
  serialize() { return JSON.stringify(this); }
  deserialize(data: string) { Object.assign(this, JSON.parse(data)); }
}

// 接口继承
interface AdminUser extends User {
  role: 'admin' | 'superadmin';
  permissions: string[];
}
```

## 类（Class）与访问修饰符

```typescript
class DataPipeline {
  public name: string;         // 公有，外部可访问
  private _status: string;     // 私有，仅类内访问
  protected config: object;    // 受保护，子类可访问
  readonly id: string;         // 只读

  constructor(name: string) {
    this.name = name;
    this._status = 'idle';
    this.id = Math.random().toString(36);
  }

  get status() { return this._status; }
  set status(val: string) {
    if (['idle', 'running', 'done'].includes(val)) {
      this._status = val;
    }
  }
}
```

## never 和 void 的区别

```typescript
// void：函数没有返回值（或返回 undefined/null）
function logMessage(msg: string): void {
  console.log(msg);
}

// never：函数永远不会正常返回（抛出异常或无限循环）
function throwError(msg: string): never {
  throw new Error(msg);  // 永远不会正常返回
}

function infiniteLoop(): never {
  while (true) {}       // 无限循环
}
```

## 实用工具类型（Utility Types）

```typescript
interface Job {
  id: string;
  name: string;
  status: 'pending' | 'running' | 'done';
  priority: number;
}

// Partial<T>：所有属性变为可选
type JobUpdate = Partial<Job>;

// Required<T>：所有属性变为必填
type FullJob = Required<Job>;

// Pick<T, K>：选取指定属性
type JobPreview = Pick<Job, 'id' | 'name'>;

// Omit<T, K>：排除指定属性
type JobWithoutId = Omit<Job, 'id'>;

// Record<K, V>：构造键值对类型
type StatusMap = Record<string, Job[]>;

// Readonly<T>：所有属性变为只读
type ImmutableJob = Readonly<Job>;
```

> **实际使用**：在 Angular + TypeScript 项目中，Partial 常用于表单更新（只传变化的字段），Pick/Omit 常用于接口返回数据裁剪。

## 类型断言与类型守卫

```typescript
// 类型断言（当你比 TS 更了解类型时）
const input = document.getElementById('search') as HTMLInputElement;
input.value = 'hello'; // 不断言的话 TS 会报错

// 类型守卫
function processValue(val: string | number) {
  if (typeof val === 'string') {
    return val.toUpperCase(); // 这里 TS 知道 val 是 string
  }
  return val.toFixed(2);     // 这里 TS 知道 val 是 number
}

// 自定义类型守卫
interface CopyJob { kind: 'copy'; source: string }
interface MoveJob { kind: 'move'; source: string; deleteSource: boolean }
type Job = CopyJob | MoveJob;

function isMoveJob(job: Job): job is MoveJob {
  return job.kind === 'move';
}
```

## 面试高频题

**Q: interface 和 type 的区别？**

> - `interface`：可以被 extends 继承，可以被 implements 实现，同名接口会合并
> - `type`：更灵活，支持联合类型、交叉类型、条件类型，同名 type 不能重复声明
> - 优先用 `interface` 定义对象结构，用 `type` 定义复杂类型或联合类型

**Q: any、unknown、never 的区别？**

> - `any`：关闭类型检查，可赋值给任何类型，不推荐使用
> - `unknown`：类型安全的 any，必须先做类型检查才能操作，推荐替代 any
> - `never`：永不出现的值类型，函数抛异常/无限循环的返回类型，常用于穷尽检查
