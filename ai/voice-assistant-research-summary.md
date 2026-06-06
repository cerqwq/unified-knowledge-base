# 语音助手技术研究摘要

> 研究日期：2026-05-31
> 8 个并行研究任务的综合成果

---

## 📋 目录
1. [推荐技术栈](#推荐技术栈)
2. [GitHub 参考项目](#github-参考项目)
3. [框架对比](#框架对比)
4. [核心架构模式](#核心架构模式)
5. [工具实现](#工具实现)
6. [成本分析](#成本分析)
7. [踩坑记录](#踩坑记录)

---

## 推荐技术栈

```
唤醒词:    OpenWakeWord（免费）或 RealtimeSTT 内置
STT:       faster-whisper（本地，免费，中文好）或 Whisper API
VAD:       Silero VAD（准确率最高）
LLM:       Claude Haiku（快速回复）+ Claude Sonnet（复杂任务）
TTS:       edge-tts（免费微软神经网络语音）+ pyttsx3（离线后备）
框架:      自建 Pipeline 或 Pipecat
```

### 一行安装
```bash
pip install sounddevice numpy silero-vad pygame pydub imageio-ffmpeg scipy torch
pip install anthropic requests pyperclip mss Pillow pycaw comtypes pywin32
pip install pynput keyboard duckduckgo-search screen-brightness-control
pip install edge-tts faster-whisper openwakeword rich pystray winotify
```

---

## GitHub 参考项目

### ⭐ 最值得参考的项目

| 项目 | Stars | 特点 |
|------|-------|------|
| [voicemode](https://github.com/mbailey/voicemode) | 1,199 | Claude Code 语音插件，MCP 服务器 |
| [jarvis](https://github.com/ethanplusai/jarvis) | 598 | macOS JARVIS，Claude Haiku+Opus 双模型 |
| [voice-chat-ai](https://github.com/bigsk1/voice-chat-ai) | 441 | 多供应商支持，含 Claude，Docker 部署 |
| [openclaw-voice](https://github.com/Purple-Horizons/openclaw-voice) | 118 | 浏览器语音聊天，faster-whisper + ElevenLabs |
| [fastapi-claude-voice-agent](https://github.com/mortogo321/fastapi-claude-voice-agent) | 1 | 生产级，Twilio 电话，Claude Opus，PostgreSQL |
| [synthia](https://github.com/markmiddo/synthia) | 6 | 隐私优先，100%本地，Tauri+React 前端 |

### 关键架构模式（从项目中学到）

**1. 双模型路由**（jarvis 项目）
```
简单问题 → Claude Haiku（200ms TTFT）
复杂任务 → Claude Opus（更智能）
```

**2. 流式句子管道**（所有项目通用）
```
Claude 流式输出 → 按句号分割 → 逐句送 TTS → 边想边说
```

**3. Prompt Caching**（fastapi-claude-voice-agent）
```
系统提示 + 工具定义标记为可缓存 → 节省 90% token
```

---

## 框架对比

| 特性 | Pipecat | 自建 Pipeline | RealtimeSTT+TTS |
|------|---------|--------------|-----------------|
| 复杂度 | 中等 | 低 | 低 |
| 灵活性 | 高 | 最高 | 中 |
| Claude 支持 | ✅ 原生 | ✅ 自己接 | ✅ 自己接 |
| 工具调用 | ✅ 装饰器 | ✅ 自己写 | ✅ 自己写 |
| 打断支持 | ✅ 内置 | 自己实现 | ✅ 内置 |
| 唤醒词 | ✅ 内置 | 自己接 | ✅ 内置 |
| 适合场景 | 快速原型 | 完全控制 | STT/TTS 专注 |

---

## 核心架构模式

### 主循环
```python
while True:
    # 1. 等待唤醒词（或快捷键）
    wait_for_wake_word()
    
    # 2. 录音 + VAD 自动停止
    audio = record_until_silence()
    
    # 3. STT（本地 faster-whisper）
    text = transcribe(audio)
    
    # 4. Claude API（流式 + 工具调用）
    for sentence in stream_claude_response(text):
        # 5. TTS 逐句播放
        speak(sentence)
```

### 流式 TTS 关键代码
```python
with client.messages.stream(...) as stream:
    buffer = ""
    for text in stream.text_stream:
        buffer += text
        if any(buffer.rstrip().endswith(p) for p in ['.', '!', '?', '。']):
            tts_engine.speak(buffer.strip())
            buffer = ""
```

---

## 工具实现

已写好代码在 `E:\Claude code work\voice-assistant\tools\`:

| 工具 | 文件 | 依赖 |
|------|------|------|
| 应用启动 | app_launcher.py | 无 |
| 天气查询 | weather.py | requests |
| 系统控制 | system_control.py | pycaw |
| 网络搜索 | web_search.py | duckduckgo-search |
| 文件搜索 | file_search.py | 无 |
| 剪贴板 | clipboard.py | pyperclip |
| 截图 | screenshot.py | mss |
| 媒体控制 | media_control.py | pynput |

工具注册中心：`core/tool_registry.py`
Claude API 循环：`core/assistant.py`

---

## 成本分析

### 混合方案（推荐）
| 组件 | 方案 | 月费（100次/天） |
|------|------|-----------------|
| STT | faster-whisper 本地 | ¥0 |
| LLM | Claude API | ~¥15-30 |
| TTS | edge-tts 免费 | ¥0 |
| **总计** | | **~¥15-30/月** |

### 全云端方案
| 组件 | 方案 | 月费（100次/天） |
|------|------|-----------------|
| STT | Whisper API | ~¥30 |
| LLM | Claude API | ~¥15-30 |
| TTS | OpenAI TTS | ~¥30 |
| **总计** | | **~¥75-90/月** |

---

## 踩坑记录

### 音频相关
1. ❌ **pyaudio 安装失败** → ✅ 用 sounddevice 替代
2. ❌ **音频卡顿** → ✅ buffer 设为 512-2048
3. ❌ **VAD 误触发** → ✅ 用 silero-vad 替代 webrtcvad
4. ❌ **ffmpeg 找不到** → ✅ 用 imageio-ffmpeg（pip 安装自带）

### Claude API 相关
5. ❌ **回复太长不适合语音** → ✅ max_tokens=200 + 系统提示约束
6. ❌ **工具调用延迟** → ✅ Prompt Caching 系统提示
7. ❌ **上下文太长** → ✅ 滑动窗口 + Haiku 摘要旧对话

### TTS 相关
8. ❌ **中文 TTS 效果差** → ✅ edge-tts 的 zh-CN-XiaoxiaoNeural
9. ❌ **TTS 延迟高** → ✅ 用 generator 模式流式输入
10. ❌ **英文缩写读错** → ✅ 预处理替换（AI→人工智能）

### 唤醒词相关
11. ❌ **Porcupine 需要 API key** → ✅ 用 OpenWakeWord 免费替代
12. ❌ **唤醒词不准** → ✅ RealtimeSTT 内置唤醒功能

---

## 详细文档索引

| 主题 | 文件路径 |
|------|---------|
| Pipecat 框架 | `E:\Claude code work\knowledge-base\pipecat-voice-assistant-reference.md` |
| RealtimeSTT/TTS | `E:\Claude code work\voice-assistant\REALTIME_STT_TTS_REFERENCE.md` |
| 音频库参考 | `E:\Claude code work\python-audio-reference\audio_libraries_reference.py` |
| UI/UX 模式 | `E:\Claude code work\voice-assistant-ui-research\voice_assistant_ui_patterns.py` |
| STT/TTS 替代方案 | `C:\Users\99593\voice_assistant_alternatives_research.md` |
| 工具实现 | `E:\Claude code work\voice-assistant\tools\TOOLKIT_REFERENCE.md` |
