---
title: 前端面试知识库 Index
type: index
created: 2026-06-03
updated: 2026-06-06
---

# 前端面试知识库

> Karpathy 风格知识库 — 持续编译，知识复利积累

## JavaScript 核心

- [[数据类型]] — 原始类型 vs 引用类型、typeof/instanceof、Symbol、BigInt
- [[作用域与变量提升]] — var/let/const、TDZ、作用域链、循环陷阱
- [[闭包]] — 应用场景、循环陷阱、内存泄漏、stale closure
- [[原型与继承]] — 原型链、new 原理、6种继承方式、ES6 class
- [[事件循环]] — 宏任务/微任务、执行顺序、async/await
- [[this绑定]] — 四种绑定规则、箭头函数、手写 call/apply/bind
- [[Promise与异步编程]] — 链式调用、静态方法、async/await、手写 Promise
- [[深浅拷贝]] — structuredClone、手写深拷贝、循环引用、WeakMap
- [[ES6+核心特性]] — 解构、可选链、模块化、Proxy/Reflect、WeakMap

## 浏览器 & 安全 & 网络

- [[前端安全XSS防护与CSRF攻击]] — Markdown GFM 白名单 + DOMPurify；CSRF Token / SameSite Cookie（百度面试 Q11–Q12）
- [[SSE与实时通信AI流式输出方案]] — SSE vs WebSocket 选型；SSE 自定义事件进度条；UX 等待优化（百度面试 Q13–Q15）

## HTML + CSS & 动画

- [[打字机效果音波动画与WebAudioAPI]] — rAF 帧率控制；Web Audio API 频谱数组；动画库/UI组件库（百度面试 Q9–Q10 + Q17 + Q22）

## 性能优化

- [[前端性能优化与监控指标]] — Core Web Vitals（LCP/FID/CLS）；PerformanceObserver RUM；加载/运行时优化落地方案（百度面试 Q19–Q20 + Q18）

## 前端工程化

- [[AI辅助开发与Agent工作流]] — AI 日常使用方式；Skill 复用；Workflow 稳定执行；System Prompt 配置（百度面试 Q3–Q8 + Q21）
- [[音频播放器全局状态管理]] — 跨页面持久播放；NgRx / Zustand 方案（百度面试 Q16）

## 即将补充

- [ ] 框架：React 核心原理（Fiber、Hooks、虚拟 DOM）
- [ ] Angular 核心原理（变更检测、DI、Zone.js）
- [ ] HTML + CSS 高频题（BFC、Flex、Grid、层叠上下文）
- [ ] 浏览器原理（渲染流程、缓存、跨域）
- [ ] 前端工程化（Webpack/Vite、模块化）
- [ ] 算法高频题

## 面试题库（按公司）

### 百度（前端 Agent 方向，2026-06）
- Q3–Q8, Q21: AI 辅助开发 & Agent Workflow → [[AI辅助开发与Agent工作流]]
- Q9–Q10, Q17, Q22: 动画 & Web Audio → [[打字机效果音波动画与WebAudioAPI]]
- Q11–Q12: 安全 → [[前端安全XSS防护与CSRF攻击]]
- Q13–Q15: SSE & 实时通信 → [[SSE与实时通信AI流式输出方案]]
- Q16: 全局状态 → [[音频播放器全局状态管理]]
- Q18–Q20: 性能 & 框架选型 → [[前端性能优化与监控指标]]

## 统计

- 已有文章：15 篇
- 覆盖主题：JavaScript 核心 + 安全 + SSE + 动画/音频 + 性能优化 + AI 工作流 + 状态管理
- 最后更新：2026-06-06
