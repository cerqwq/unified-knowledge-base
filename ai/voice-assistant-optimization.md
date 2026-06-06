# 语音助手优化知识库

## 架构优化

### 混合模型架构
```
用户语音 → STT → 任务分类 → 简单任务 → 本地模型（快、免费）
                        → 复杂任务 → 云端模型（强、付费）
```

### 关键优化点

#### 1. 响应延迟优化
- 流式输出（streaming）：边生成边播放
- 预测性响应：常见问题预缓存
- 并行处理：STT + 语义理解同时进行

#### 2. Token 优化
- 压缩系统提示：去掉冗余指令
- 上下文裁剪：只保留最近几轮
- 摘要机制：长对话自动总结

#### 3. 工具调用优化
- 预编译工具定义
- 缓存工具结果
- 异步工具执行

### STT 优化

#### Whisper 模型选择
| 模型 | 大小 | 速度 | 准确度 |
|------|------|------|--------|
| tiny | 39MB | 最快 | 低 |
| base | 74MB | 快 | 中 |
| small | 244MB | 中 | 高 |
| medium | 769MB | 慢 | 很高 |

#### VAD（语音活动检测）
- Silero VAD：最准确
- WebRTC VAD：轻量级
- 调参关键：灵敏度、静音时长

### TTS 优化

#### 方案对比
| 方案 | 延迟 | 质量 | 离线 |
|------|------|------|------|
| Edge TTS | 低 | 高 | ❌ |
| Piper TTS | 中 | 中 | ✅ |
| pyttsx3 | 低 | 低 | ✅ |

#### 优化技巧
- 流式合成：边生成边播放
- 缓存常用回复
- 异步合成

### 唤醒词优化

#### 方案对比
| 方案 | 准确度 | 资源占用 |
|------|--------|---------|
| 文本匹配 | 低 | 极低 |
| OpenWakeWord | 中 | 低 |
| Porcupine | 高 | 中 |

#### 最佳实践
- 双重检测：音频模型 + 文本匹配
- 模糊匹配：处理口音差异
- 低功耗模式：降低检测频率

## 代码实现

### 流式对话
```python
def stream_with_tools(message):
    # 1. 检查是否需要工具
    response = llm.chat(message, tools=tools)
    
    # 2. 如果需要工具，先执行
    if response.tool_calls:
        for tool_call in response.tool_calls:
            result = execute_tool(tool_call)
            # 流式输出工具执行状态
            yield f"[工具] {tool_call.name}..."
    
    # 3. 流式输出最终回复
    stream = llm.chat(message, stream=True)
    for chunk in stream:
        yield chunk
```

### 记忆系统
```python
class MemoryManager:
    def add_memory(content, memory_type):
        # 1. 添加到 JSON 存储
        self.memory["facts"].append(content)
        
        # 2. 添加到向量数据库
        if self.chroma_client:
            self.collection.add(
                documents=[content],
                ids=[f"mem_{timestamp}"]
            )
        
        # 3. 保存到文件
        self._save()
```

## 测试策略

### 单元测试
- 工具注册和执行
- 记忆存储和检索
- 模型选择逻辑

### 集成测试
- 完整对话流程
- 工具调用链
- 错误恢复

### 性能测试
- 响应延迟
- 内存占用
- Token 消耗
