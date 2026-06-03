# 前端面试知识库 Knowledge Base

> Schema document — read at the start of every session together with `wiki/index.md`.
> Update after every major compile, ingest batch, or structural change.

## Scope

What this wiki covers:
- JavaScript 核心：基础语法、ES6+、闭包、原型链、异步（Promise/async-await/EventLoop）
- 框架：React、Angular 常见面试题与原理
- HTML + CSS：语义化、布局（Flexbox/Grid）、动画、BFC、层叠规则
- 浏览器原理：渲染流程、缓存机制、跨域、安全（XSS/CSRF）
- 性能优化：加载性能、运行时性能、Lighthouse 指标
- 前端工程化：Webpack/Vite、模块化、CI/CD、TypeScript
- 算法与数据结构：前端高频算法题（排序、链表、树、DP）
- 综合问题：系统设计、团队协作、前端架构

What this wiki deliberately excludes:
- 后端、数据库、DevOps（非前端相关）
- 纯理论计算机科学（超出前端面试范围的部分）

## Operations

This wiki follows the llm-wiki skill's five operations: `compile`, `ingest`, `query`, `lint`, `audit`.
Every operation appends an entry to `log/YYYYMMDD.md`.

## Naming conventions

- **Concept pages** (`wiki/concepts/`): Title Case noun phrases.
- **Folder-split concepts** (`wiki/concepts/<topic>/`): used when a topic exceeds ~1200 words. Contains `index.md` + one file per aspect.
- **Entity pages** (`wiki/entities/`): Proper names（框架名、工具名）.
- **Summary pages** (`wiki/summaries/`): kebab-case source slug.

All pages require YAML frontmatter: `title`, `type`, `created`, `updated`, `sources`, `tags`.

### Diagrams and formulas
- All diagrams are **mermaid**. No ASCII art.
- All formulas are **KaTeX** (inline `$...$` or block `$$...$$`).

### Raw file policy
- Small text sources → copy into `raw/<subfolder>/`.
- Large binaries → create a pointer file at `raw/refs/<slug>.md` with `kind: ref` and `external_path` fields. Do not copy the binary.

## Current articles

*None yet — update this list after every compile.*

### Concepts
*(none)*

### Entities
*(none)*

### Summaries
*(none)*

## Open research questions

- JavaScript 事件循环与微任务/宏任务的考察重点？
- React Fiber 架构在面试中需要讲到什么深度？
- 前端性能优化中哪些指标是大厂最关注的？

## Research gaps

Sources to ingest:
- [ ] 常见前端面试题集合（待收集）
- [ ] MDN 核心文档精华
- [ ] React 官方文档重点章节

## Audit backlog

*(none — run `python3 scripts/audit_review.py <wiki-root> --open` to refresh)*

## Notes for the LLM

- Language: bilingual（中文优先，代码和技术术语保留英文）
- Tone: conversational，像懂行的朋友在讲解
- Depth: deep technical（面试级别，要能直接用于回答面试官）
- Handling contradictions: state both, cite each, add to Open Research Questions.
