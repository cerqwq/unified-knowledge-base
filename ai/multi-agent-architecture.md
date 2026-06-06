# 多 Agent 协作架构知识库

## 核心概念

### Agent 定义
Agent = LLM + 工具 + 记忆 + 规划能力

### 多 Agent 模式

#### 1. 编排器模式（Orchestrator）
```
用户 → 编排器 → Agent A
                → Agent B
                → Agent C
                → 汇总结果
```
- 优点：简单、可控
- 缺点：编排器是瓶颈
- 适用：任务明确、Agent 数量少

#### 2. 管道模式（Pipeline）
```
用户 → Agent A → Agent B → Agent C → 结果
```
- 优点：流程清晰
- 缺点：不灵活
- 适用：固定流程任务

#### 3. 协作模式（Collaboration）
```
用户 → Agent A ←→ Agent B
            ↕
         Agent C
```
- 优点：灵活、可互相帮助
- 缺点：复杂、难调试
- 适用：复杂创意任务

#### 4. 辩论模式（Debate）
```
用户 → Agent A (正方)
     → Agent B (反方)
     → 裁判 → 最终答案
```
- 优点：减少幻觉
- 缺点：成本高
- 适用：需要准确性

### 关键技术

#### 工具调用（Tool Use）
```python
tools = [
    {"type": "function", "function": {"name": "search", "parameters": {...}}},
    {"type": "function", "function": {"name": "code_exec", "parameters": {...}}},
]
```

#### 记忆系统
- 短期记忆：对话上下文
- 长期记忆：向量数据库（ChromaDB、FAISS）
- 工作记忆：当前任务状态

#### 规划能力
- ReAct：思考 → 行动 → 观察 → 循环
- Plan-and-Execute：先规划，再执行
- Reflexion：执行后反思，改进

## 主流框架

### LangChain
- 最成熟的 Agent 框架
- 支持多种 LLM、工具、记忆
- LangGraph 用于复杂工作流

### CrewAI
- 多 Agent 协作框架
- 角色定义、任务分配
- 简单易用

### AutoGen (Microsoft)
- 多 Agent 对话框架
- 支持人机协作
- 企业级

### Claude Agent SDK
- Anthropic 官方 SDK
- 原生工具调用
- 最佳实践

## 实现要点

### 1. Agent 接口设计
```python
class BaseAgent:
    name: str
    capabilities: list[str]
    
    def can_handle(task) -> float  # 置信度
    def run(task, context) -> AgentResult
```

### 2. 任务路由
- 关键词匹配（简单）
- LLM 评估（准确）
- 向量相似度（快速）

### 3. 结果汇总
- 直接拼接
- LLM 总结
- 投票机制

### 4. 错误处理
- 超时机制
- 重试策略
- 降级方案
