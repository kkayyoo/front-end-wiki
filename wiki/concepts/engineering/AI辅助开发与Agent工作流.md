---
title: AI 辅助开发与 Agent 工作流
type: concept
created: 2026-06-06
updated: 2026-06-06
sources:
  - raw/articles/baidu-frontend-agent-interview-2026-06-06.md
tags: [AI, Agent, Workflow, Skill, system-prompt, 百度面试]
companies: [百度]
---

# AI 辅助开发与 Agent 工作流

> 百度前端 Agent 面试高频考点（Q3–Q8）。考察对 AI 工具的实际使用深度，以及 Agent/Workflow 的工程化落地能力。

## 1. 日常开发中如何使用 AI

### 高分答法框架（结合 Angular/TypeScript/React 背景）

```
代码生成 → 代码审查 → 调试诊断 → 文档生成 → 测试辅助
```

**实战案例（贴近 Azure Data Copy 背景）**：
- **代码生成**：用 Copilot / Cursor 快速生成 Angular Service 骨架、TypeScript interface、RxJS operator 链
- **代码审查**：粘贴 PR diff 给 GPT，让其指出潜在 null 风险、异步竞态
- **调试诊断**：报错粘贴 + 上下文 → 快速定位 Angular 依赖注入失败原因
- **测试辅助**：描述组件行为 → 生成 Jasmine/Jest 测试用例骨架

**面试官想听到的**：不只是"我用 Copilot 补全代码"，而是说出**哪类任务**交给 AI、**哪类**必须自己写（业务逻辑/架构决策）。

---

## 2. Skill 复用机制

**百度追问（Q5）**：已有现成 Skill，如何使用？

### 核心思路

Skill = 预封装的结构化提示词 + 工具调用链，解决"每次都要手动描述上下文"的问题。

```
无 Skill 方式：
用户 → 粘贴背景 → 描述任务 → 等待输出 → 手动执行
               ↓
有 Skill 方式：
用户 → 触发 Skill → Agent 自动读取上下文 → 自动执行 → 输出结果
```

**实际回答要点**：
1. 找到对应 Skill（如 `testing-skill`）
2. 调用时传入必要参数（测试目标文件路径、框架类型）
3. Skill 内部自动处理：读代码 → 分析依赖 → 生成测试 → 写文件

---

## 3. Workflow 稳定执行（Q6）

**核心答案**（面试官期望方向）：

> 固定 **主流程节点** 用确定性代码/配置控制，细分的**判断/生成节点**交给 LLM 处理。

```mermaid
flowchart LR
    A[触发条件] --> B[固定节点: 读取文件列表]
    B --> C[LLM节点: 分析代码意图]
    C --> D[固定节点: 执行测试命令]
    D --> E[LLM节点: 解析失败原因]
    E --> F[固定节点: 写入报告]
```

**为什么这样设计**：
- LLM 处理非结构化判断（理解代码意图、解释报错）
- 固定节点处理确定性操作（文件 IO、命令执行、格式化输出）
- 主流程可 review、可回放、可 debug

**稳定性保障**：
- 关键节点加 retry + timeout
- LLM 输出加 schema 校验（JSON Schema / Zod）
- 失败节点有 fallback 或人工介入

---

## 4. Workflow 写入 System Prompt（Q7）

**面试官提示的方案**（用户未能答出，记录备用）：

将 workflow 配置写入 system prompt，让 LLM 在每次对话时都遵循固定执行链路。

```markdown
# System Prompt 示例

你是一个前端测试生成 Agent，遵循以下 workflow：

1. 读取用户提供的文件路径，理解组件功能
2. 识别测试类型（单元测试/集成测试）
3. 根据框架（Angular/React）选择对应测试模板
4. 生成测试代码，覆盖正常路径 + 边界路径
5. 输出为 spec 文件格式，不包含任何解释文字

约束：
- 使用 TypeScript 严格模式
- 覆盖率目标 ≥ 80%
- 必须包含 mock 外部依赖
```

**优点**：
- 无需每次重复描述上下文
- LLM 行为可预期、可版本控制（system prompt 存 git）
- 等效于"代码即规范"

**类比**：ESLint config = 写入 config 文件的代码规范；system prompt = 写入 prompt 文件的 Agent 行为规范。

---

## 5. AI 会不会取代前端开发（Q21）

**高分答法思路**：

| 会被替代的 | 不会被替代的 |
|----------|------------|
| 模板代码生成 | 产品需求理解与拆解 |
| 简单 CRUD 页面 | 系统架构设计 |
| 格式转换、boilerplate | 用户体验决策 |
| 基础测试用例 | 复杂交互设计与实现 |
| 文档生成 | 跨团队技术对齐 |

**核心观点**：AI 是倍增器，不是替代品。懂得用 AI 的前端 > 不懂用 AI 的前端 >> 没有 AI 的前端。  
自身背景加分点：6年经验 + 已在日常开发中深度使用 AI 工具，是懂得如何让 AI 有效工作的人。

---

## 面试 Q&A 速查（百度原题）

**Q: 日常开发如何使用 AI？**  
A: 代码补全（Copilot）+ 代码审查（粘贴 diff 分析）+ 调试诊断 + 测试辅助生成，核心是知道哪类任务适合 AI、哪类需要人工判断。

**Q: 已有 Skill 如何使用？**  
A: 找到对应 Skill → 传入参数触发 → Agent 自动执行封装好的工具调用链，无需重复描述上下文。

**Q: 如何保证 Skill 稳定执行？**  
A: 主流程节点用确定性代码控制，细分判断节点交给 LLM。关键节点加 retry/timeout/schema 校验。

**Q: Workflow 怎么落地？**  
A: 将 workflow 配置写入 system prompt，LLM 在每次对话时遵循固定执行链路，等效于"可版本控制的行为规范"。

**Q: AI 会取代前端吗？**  
A: AI 是倍增器。模板代码会被替代，架构设计和用户体验决策不会。懂用 AI 的工程师价值更高。
