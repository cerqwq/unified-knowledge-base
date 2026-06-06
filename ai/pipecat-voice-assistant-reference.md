# Pipecat Framework - Voice AI Assistant Technical Reference

> **Framework**: Pipecat (by Daily.co)
> **GitHub**: https://github.com/pipecat-ai/pipecat
> **Docs**: https://docs.pipecat.ai
> **PyPI**: `pipecat-ai`
> **Last Updated**: 2026-05-31

---

## 1. Overview

Pipecat is an open-source Python framework for building **real-time voice and multimodal AI applications**. It provides a composable pipeline architecture where audio flows through a chain of processors: STT -> LLM -> TTS, with support for interruptions, function calling, wake words, and multiple transport layers.

### Core Architecture

```
Microphone Input
    |
    v
[Transport Input] -- WebRTC / WebSocket / Local
    |
    v
[VAD] -- Silero Voice Activity Detection
    |
    v
[STT] -- Deepgram / Whisper / AssemblyAI
    |
    v
[Context Aggregator (user)] -- Manages conversation history
    |
    v
[LLM] -- Claude / GPT / Gemini
    |
    v
[Context Aggregator (assistant)]
    |
    v
[TTS] -- ElevenLabs / Cartesia / OpenAI / Deepgram
    |
    v
[Transport Output]
    |
    v
Speaker Output
```

---

## 2. Installation

```bash
# Base installation
pip install pipecat-ai

# With specific provider extras
pip install "pipecat-ai[anthropic]"           # Anthropic Claude
pip install "pipecat-ai[openai]"              # OpenAI (GPT, Whisper, TTS)
pip install "pipecat-ai[deepgram]"            # Deepgram STT/TTS
pip install "pipecat-ai[elevenlabs]"          # ElevenLabs TTS
pip install "pipecat-ai[cartesia]"            # Cartesia TTS
pip install "pipecat-ai[daily]"               # Daily.co WebRTC transport
pip install "pipecat-ai[silero]"              # Silero VAD

# Combined install for a full voice assistant
pip install "pipecat-ai[anthropic,deepgram,elevenlabs,daily,silero]"

# For local Whisper
pip install "pipecat-ai[whisper]"
```

---

## 3. Basic Pipeline Structure

### 3.1 Core Components

```python
from pipecat.pipeline.pipeline import Pipeline
from pipecat.pipeline.runner import PipelineRunner
from pipecat.pipeline.task import PipelineParams, PipelineTask
```

- **Pipeline**: An ordered list of processors that frames flow through
- **PipelineRunner**: Orchestrates execution of the pipeline
- **PipelineTask**: Wraps a Pipeline and manages its lifecycle
- **PipelineParams**: Configuration (e.g., `allow_interruptions=True`)

### 3.2 Minimal Pipeline

```python
from pipecat.pipeline.pipeline import Pipeline
from pipecat.pipeline.runner import PipelineRunner
from pipecat.pipeline.task import PipelineParams, PipelineTask

pipeline = Pipeline([
    transport.input(),      # Audio input from transport
    stt,                    # Speech-to-text
    context_aggregator.user(),  # Add user message to context
    llm,                    # LLM generates response
    context_aggregator.assistant(),  # Add assistant message to context
    tts,                    # Text-to-speech
    transport.output(),     # Audio output to transport
])

task = PipelineTask(pipeline, PipelineParams(allow_interruptions=True))
runner = PipelineRunner()
await runner.run(task)
```

---

## 4. Transport Layers

### 4.1 DailyTransport (WebRTC - Production)

```python
from pipecat.transports.services.daily import DailyParams, DailyTransport

transport = DailyTransport(
    room_url="https://your-domain.daily.co/room-name",
    token="your-participant-token",
    bot_name="Voice Assistant",
    params=DailyParams(
        audio_out_enabled=True,
        vad_enabled=True,
        vad_analyzer=SileroVADAnalyzer(),
        vad_audio_passthrough=True,
    ),
)
```

### 4.2 LocalTransport (Development/Testing)

For local development without cloud infrastructure -- uses the machine's microphone and speakers directly.

### 4.3 FastAPIWebsocketTransport

For WebSocket-based communication, useful with FastAPI backends or Twilio integration.

### 4.4 SmallWebRTCTransport

Lightweight WebRTC option for browser-based clients.

---

## 5. STT (Speech-to-Text) Configuration

### 5.1 Deepgram (Recommended for Real-time)

```python
from pipecat.services.deepgram.stt import DeepgramSTTService

stt = DeepgramSTTService(
    api_key="YOUR_DEEPGRAM_API_KEY",
    live_options={
        "model": "nova-2",           # or "nova", "nova-2-general"
        "language": "en-US",
        "encoding": "linear16",
        "sample_rate": 16000,
        "channels": 1,
        "interim_results": True,     # Partial transcription results
        "smart_format": True,
        "punctuate": True,
        "endpointing": 300,          # Silence (ms) to detect end of speech
    }
)
```

### 5.2 OpenAI Whisper (Cloud API)

```python
from pipecat.services.whisper import WhisperSTTService

stt = WhisperSTTService(
    model="whisper-1",
    language="en",
    api_key="YOUR_OPENAI_API_KEY",
)
```

### 5.3 Local Whisper (faster-whisper)

```python
# Requires: pip install faster-whisper
from pipecat.services.whisper import LocalWhisperSTTService

stt = LocalWhisperSTTService(
    model_size="large-v3",    # "base", "small", "medium", "large-v3"
    device="cuda",            # "cpu" or "cuda"
    compute_type="float16",   # "int8", "float16", "float32"
    language="en",
)
```

---

## 6. LLM Configuration

### 6.1 Anthropic Claude (AnthropicLLMService)

```python
from pipecat.services.anthropic import AnthropicLLMService

llm = AnthropicLLMService(
    api_key="YOUR_ANTHROPIC_API_KEY",
    model="claude-sonnet-4-20250514",  # or claude-3-5-sonnet-20241022, etc.
)
```

**Note**: The system prompt is set via the `OpenAILLMContext` messages, not directly on the service.

### 6.2 OpenAI GPT

```python
from pipecat.services.openai import OpenAILLMService

llm = OpenAILLMService(
    api_key="YOUR_OPENAI_API_KEY",
    model="gpt-4o",
)
```

### 6.3 Context Management (OpenAILLMContext)

```python
from pipecat.processors.aggregators.openai_llm_context import OpenAILLMContext

messages = [
    {
        "role": "system",
        "content": (
            "You are a helpful voice assistant. "
            "Keep responses concise and conversational. "
            "Avoid using markdown formatting since responses will be spoken aloud."
        ),
    }
]

context = OpenAILLMContext(messages)
context_aggregator = llm.create_context_aggregator(context)
```

The `context_aggregator` provides two sub-processors:
- `context_aggregator.user()` -- placed before the LLM, adds user messages
- `context_aggregator.assistant()` -- placed after the LLM, adds assistant messages

---

## 7. TTS (Text-to-Speech) Configuration

### 7.1 ElevenLabs

```python
from pipecat.services.elevenlabs import ElevenLabsTTSService

tts = ElevenLabsTTSService(
    api_key="YOUR_ELEVENLABS_API_KEY",
    voice_id="your-voice-id",
    model="eleven_turbo_v2",         # or "eleven_monolingual_v1"
    params=ElevenLabsTTSService.InputParams(
        stability=0.5,
        similarity_boost=0.75,
        style=0.0,
        use_speaker_boost=True,
    ),
)
```

### 7.2 OpenAI TTS

```python
from pipecat.services.openai import OpenAITTSService

tts = OpenAITTSService(
    api_key="YOUR_OPENAI_API_KEY",
    model="tts-1",                   # or "tts-1-hd" for higher quality
    voice="alloy",                   # alloy, echo, fable, onyx, nova, shimmer
)
```

### 7.3 Cartesia (Low Latency)

```python
from pipecat.services.cartesia import CartesiaTTSService

tts = CartesiaTTSService(
    api_key="YOUR_CARTESIA_API_KEY",
    voice_id="your-voice-id",
    model="sonic-english",
)
```

### 7.4 Deepgram TTS

```python
from pipecat.services.deepgram.tts import DeepgramTTSService

tts = DeepgramTTSService(
    api_key="YOUR_DEEPGRAM_API_KEY",
    model="aura-asteria-en",         # Various voice models available
)
```

---

## 8. Function Calling / Tool Use

### 8.1 Defining Tools

Tools are defined in OpenAI-compatible format (works with both OpenAI and Anthropic):

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name, e.g. 'San Francisco, CA'"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "Temperature unit"
                    }
                },
                "required": ["location"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "set_timer",
            "description": "Set a countdown timer",
            "parameters": {
                "type": "object",
                "properties": {
                    "duration_seconds": {
                        "type": "integer",
                        "description": "Timer duration in seconds"
                    },
                    "label": {
                        "type": "string",
                        "description": "Optional label for the timer"
                    }
                },
                "required": ["duration_seconds"]
            }
        }
    }
]
```

For Anthropic's native tool format:

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get the current weather",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City name"}
            },
            "required": ["location"]
        }
    }
]
```

### 8.2 Registering Function Handlers

```python
from pipecat.services.openai import OpenAILLMService

llm = OpenAILLMService(
    api_key="YOUR_API_KEY",
    model="gpt-4o",
)

# Register function handlers using decorator
@llm.register_function("get_weather")
async def get_weather(params: FunctionCallParams):
    location = params.arguments.get("location")
    unit = params.arguments.get("unit", "celsius")

    # Call your actual weather API
    weather_data = await fetch_weather_api(location, unit)

    # Push result back to the LLM
    await params.llm.push_result(
        FunctionCallResult(
            status="success",
            result=weather_data
        )
    )

@llm.register_function("set_timer")
async def set_timer(params: FunctionCallParams):
    duration = params.arguments.get("duration_seconds")
    label = params.arguments.get("label", "Timer")

    # Start your timer logic
    await start_countdown(duration, label)

    await params.llm.push_result(
        FunctionCallResult(
            status="success",
            result=f"Timer set for {duration} seconds: {label}"
        )
    )
```

### 8.3 Context with Tools

```python
context = OpenAILLMContext(
    messages=[
        {"role": "system", "content": "You are a helpful assistant with access to tools."},
    ],
    tools=tools  # Pass tool definitions to the context
)
context_aggregator = llm.create_context_aggregator(context)
```

---

## 9. Voice Activity Detection (VAD)

### 9.1 Silero VAD

```python
from pipecat.vad.silero import SileroVADAnalyzer

vad = SileroVADAnalyzer()

# Used in transport params
transport = DailyTransport(
    # ...
    params=DailyParams(
        vad_enabled=True,
        vad_analyzer=vad,
        vad_audio_passthrough=True,
    ),
)
```

### 9.2 Interruption Handling

When `allow_interruptions=True` is set in `PipelineParams`, the bot will stop speaking when the user starts talking (detected by VAD):

```python
task = PipelineTask(
    pipeline,
    PipelineParams(
        allow_interruptions=True,
        # enable_metrics=True,  # Optional: track pipeline metrics
    )
)
```

---

## 10. Wake Word Detection

### 10.1 Using Porcupine (Picovoice)

Porcupine is a lightweight, on-device wake word engine:

```python
# Conceptual integration pattern
from pipecat.processors.wake.picovoice import PicovoiceWakeWord

wake_word = PicovoiceWakeWord(
    access_key="YOUR_PICOVOICE_ACCESS_KEY",
    keywords=["hey jarvis"],  # or custom wake words
)

pipeline = Pipeline([
    transport.input(),
    wake_word,                  # Only passes audio after wake word detected
    stt,
    context_aggregator.user(),
    llm,
    context_aggregator.assistant(),
    tts,
    transport.output(),
])
```

### 10.2 Using openWakeWord

Open-source alternative for wake word detection:

```python
from pipecat.processors.wake.openwakeword import OpenWakeWord

wake_word = OpenWakeWord(
    model_name="hey_mycroft",  # or other supported wake words
    threshold=0.5,
)
```

### 10.3 Manual Wake Word via STT

An alternative approach is to use STT + keyword matching instead of a dedicated wake word engine:

```python
# In your system prompt, instruct the LLM to only respond when addressed
messages = [
    {
        "role": "system",
        "content": (
            "You are 'Jarvis', a voice assistant. "
            "Only respond when the user addresses you as 'Jarvis'. "
            "If the user's message doesn't start with 'Jarvis', respond with '[IGNORE]'."
        ),
    }
]
```

---

## 11. Event Handlers

```python
@transport.event_handler("on_first_participant_joined")
async def on_first_participant_joined(transport, participant):
    """Called when the first participant joins the room."""
    # Start the pipeline / greet the user
    await task.queue_frames([context_aggregator.user().get_context_frame()])
    await transport.capture_participant_transcription(participant["id"])

@transport.event_handler("on_participant_left")
async def on_participant_left(transport, participant, reason):
    """Called when a participant leaves."""
    await task.cancel()

@transport.event_handler("on_participant_joined")
async def on_participant_joined(transport, participant):
    """Called when any participant joins."""
    pass

@transport.event_handler("on_participant_updated")
async def on_participant_updated(transport, participant):
    """Called when participant state changes."""
    pass
```

---

## 12. Complete Examples

### 12.1 Full Voice Assistant with Claude + Deepgram + ElevenLabs

```python
import asyncio
import logging
import os

from pipecat.pipeline.pipeline import Pipeline
from pipecat.pipeline.runner import PipelineRunner
from pipecat.pipeline.task import PipelineParams, PipelineTask
from pipecat.transports.services.daily import DailyParams, DailyTransport
from pipecat.services.deepgram.stt import DeepgramSTTService
from pipecat.services.anthropic import AnthropicLLMService
from pipecat.services.elevenlabs import ElevenLabsTTSService
from pipecat.processors.aggregators.openai_llm_context import OpenAILLMContext
from pipecat.vad.silero import SileroVADAnalyzer

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


async def main():
    # --- Transport (Daily WebRTC) ---
    transport = DailyTransport(
        room_url=os.getenv("DAILY_ROOM_URL"),
        token=os.getenv("DAILY_TOKEN"),
        bot_name="Claude Assistant",
        params=DailyParams(
            audio_out_enabled=True,
            vad_enabled=True,
            vad_analyzer=SileroVADAnalyzer(),
            vad_audio_passthrough=True,
        ),
    )

    # --- STT (Deepgram) ---
    stt = DeepgramSTTService(
        api_key=os.getenv("DEEPGRAM_API_KEY"),
        live_options={
            "model": "nova-2",
            "language": "en-US",
            "interim_results": True,
            "smart_format": True,
            "punctuate": True,
            "endpointing": 300,
        },
    )

    # --- LLM (Anthropic Claude) ---
    llm = AnthropicLLMService(
        api_key=os.getenv("ANTHROPIC_API_KEY"),
        model="claude-sonnet-4-20250514",
    )

    # --- TTS (ElevenLabs) ---
    tts = ElevenLabsTTSService(
        api_key=os.getenv("ELEVENLABS_API_KEY"),
        voice_id=os.getenv("ELEVENLABS_VOICE_ID", "EXAVITQu4vr4xnSDxMaL"),
        model="eleven_turbo_v2",
    )

    # --- Context ---
    messages = [
        {
            "role": "system",
            "content": (
                "You are a helpful voice assistant powered by Claude. "
                "Keep your responses concise and conversational. "
                "Avoid using markdown formatting, bullet points, or numbered lists "
                "since your responses will be spoken aloud. "
                "If you don't know something, say so honestly."
            ),
        }
    ]
    context = OpenAILLMContext(messages)
    context_aggregator = llm.create_context_aggregator(context)

    # --- Pipeline ---
    pipeline = Pipeline([
        transport.input(),
        stt,
        context_aggregator.user(),
        llm,
        context_aggregator.assistant(),
        tts,
        transport.output(),
    ])

    task = PipelineTask(pipeline, PipelineParams(allow_interruptions=True))

    # --- Event Handlers ---
    @transport.event_handler("on_first_participant_joined")
    async def on_first_participant_joined(transport, participant):
        logger.info("Participant joined, starting conversation")
        await task.queue_frames([context_aggregator.user().get_context_frame()])

    @transport.event_handler("on_participant_left")
    async def on_participant_left(transport, participant, reason):
        logger.info("Participant left, stopping pipeline")
        await task.cancel()

    # --- Run ---
    runner = PipelineRunner()
    await runner.run(task)


if __name__ == "__main__":
    asyncio.run(main())
```

### 12.2 Voice Assistant with Function Calling (Weather Bot)

```python
import asyncio
import logging
import os
import aiohttp

from pipecat.pipeline.pipeline import Pipeline
from pipecat.pipeline.runner import PipelineRunner
from pipecat.pipeline.task import PipelineParams, PipelineTask
from pipecat.transports.services.daily import DailyParams, DailyTransport
from pipecat.services.deepgram.stt import DeepgramSTTService
from pipecat.services.openai import OpenAILLMService
from pipecat.services.cartesia import CartesiaTTSService
from pipecat.processors.aggregators.openai_llm_context import OpenAILLMContext
from pipecat.vad.silero import SileroVADAnalyzer

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# --- Tool Definitions ---
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City and state/country, e.g. 'London, UK'"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "Temperature unit (default: celsius)"
                    }
                },
                "required": ["location"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "Search the web for current information",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "The search query"
                    }
                },
                "required": ["query"]
            }
        }
    }
]


async def main():
    # --- Transport ---
    transport = DailyTransport(
        room_url=os.getenv("DAILY_ROOM_URL"),
        token=os.getenv("DAILY_TOKEN"),
        bot_name="Weather Bot",
        params=DailyParams(
            audio_out_enabled=True,
            vad_enabled=True,
            vad_analyzer=SileroVADAnalyzer(),
            vad_audio_passthrough=True,
        ),
    )

    # --- STT ---
    stt = DeepgramSTTService(
        api_key=os.getenv("DEEPGRAM_API_KEY"),
        live_options={"model": "nova-2", "language": "en-US"},
    )

    # --- LLM (OpenAI with tools) ---
    llm = OpenAILLMService(
        api_key=os.getenv("OPENAI_API_KEY"),
        model="gpt-4o",
    )

    # --- Register Function Handlers ---
    @llm.register_function("get_weather")
    async def get_weather(params):
        location = params.arguments.get("location")
        unit = params.arguments.get("unit", "celsius")

        # Example: call a real weather API
        async with aiohttp.ClientSession() as session:
            url = f"https://wttr.in/{location}?format=j1"
            async with session.get(url) as resp:
                data = await resp.json()
                current = data["current_condition"][0]
                temp = current["temp_C"] if unit == "celsius" else current["temp_F"]
                desc = current["weatherDesc"][0]["value"]
                result = f"In {location}, it's currently {temp}{'C' if unit == 'celsius' else 'F'} and {desc}."

        await params.result_callback(result)

    @llm.register_function("search_web")
    async def search_web(params):
        query = params.arguments.get("query")
        # Implement your web search logic here
        result = f"Search results for '{query}': [implement actual search]"
        await params.result_callback(result)

    # --- TTS ---
    tts = CartesiaTTSService(
        api_key=os.getenv("CARTESIA_API_KEY"),
        voice_id=os.getenv("CARTESIA_VOICE_ID"),
        model="sonic-english",
    )

    # --- Context ---
    messages = [
        {
            "role": "system",
            "content": (
                "You are a helpful weather assistant. You can check weather "
                "for any location and search the web. Keep responses concise "
                "and spoken-friendly. Don't use markdown."
            ),
        }
    ]
    context = OpenAILLMContext(messages, tools=tools)
    context_aggregator = llm.create_context_aggregator(context)

    # --- Pipeline ---
    pipeline = Pipeline([
        transport.input(),
        stt,
        context_aggregator.user(),
        llm,
        context_aggregator.assistant(),
        tts,
        transport.output(),
    ])

    task = PipelineTask(pipeline, PipelineParams(allow_interruptions=True))

    @transport.event_handler("on_first_participant_joined")
    async def on_first_participant_joined(transport, participant):
        await task.queue_frames([context_aggregator.user().get_context_frame()])

    @transport.event_handler("on_participant_left")
    async def on_participant_left(transport, participant, reason):
        await task.cancel()

    runner = PipelineRunner()
    await runner.run(task)


if __name__ == "__main__":
    asyncio.run(main())
```

### 12.3 Local Development Setup (No Cloud Transport)

```python
import asyncio
import logging

from pipecat.pipeline.pipeline import Pipeline
from pipecat.pipeline.runner import PipelineRunner
from pipecat.pipeline.task import PipelineParams, PipelineTask
from pipecat.services.openai import OpenAILLMService, OpenAITTSService
from pipecat.services.deepgram.stt import DeepgramSTTService
from pipecat.processors.aggregators.openai_llm_context import OpenAILLMContext
from pipecat.vad.silero import SileroVADAnalyzer

# For local testing, you would use LocalTransport or SmallWebRTCTransport
# instead of DailyTransport

logging.basicConfig(level=logging.INFO)


async def main():
    stt = DeepgramSTTService(
        api_key="your-deepgram-key",
        live_options={"model": "nova-2", "language": "en-US"},
    )

    llm = OpenAILLMService(api_key="your-openai-key", model="gpt-4o-mini")

    tts = OpenAITTSService(
        api_key="your-openai-key",
        model="tts-1",
        voice="nova",
    )

    messages = [
        {"role": "system", "content": "You are a helpful assistant. Keep it brief."}
    ]
    context = OpenAILLMContext(messages)
    context_aggregator = llm.create_context_aggregator(context)

    # Use a local transport for testing
    # transport = LocalTransport(...)  # or SmallWebRTCTransport

    # pipeline = Pipeline([
    #     transport.input(),
    #     stt,
    #     context_aggregator.user(),
    #     llm,
    #     context_aggregator.assistant(),
    #     tts,
    #     transport.output(),
    # ])

    # task = PipelineTask(pipeline, PipelineParams(allow_interruptions=True))
    # runner = PipelineRunner()
    # await runner.run(task)


if __name__ == "__main__":
    asyncio.run(main())
```

---

## 13. Project Structure for a Production Voice Assistant

```
voice-assistant/
|-- bot.py                  # Main pipeline entry point
|-- server.py               # FastAPI server (if using WebSocket transport)
|-- requirements.txt
|-- .env                    # API keys
|
|-- services/
|   |-- __init__.py
|   |-- weather.py          # Weather tool implementation
|   |-- search.py           # Web search tool implementation
|   |-- calendar.py         # Calendar tool implementation
|
|-- prompts/
|   |-- system.txt          # System prompt template
|
|-- config/
|   |-- settings.py         # Configuration management
```

### requirements.txt

```
pipecat-ai[anthropic,deepgram,elevenlabs,daily,silero]
python-dotenv
aiohttp
```

### .env

```
ANTHROPIC_API_KEY=sk-ant-...
DEEPGRAM_API_KEY=...
ELEVENLABS_API_KEY=...
ELEVENLABS_VOICE_ID=...
DAILY_ROOM_URL=https://your-domain.daily.co/room
DAILY_TOKEN=...
```

---

## 14. Key Design Patterns

### 14.1 Pipeline Ordering Matters

The order of processors in the `Pipeline([...])` list defines the data flow. The standard pattern is:
1. Transport input (audio in)
2. STT (audio -> text)
3. Context aggregator user (add to conversation)
4. LLM (generate response)
5. Context aggregator assistant (record response)
6. TTS (text -> audio)
7. Transport output (audio out)

### 14.2 Interruption Handling

When `allow_interruptions=True`, Pipecat uses VAD to detect when the user starts speaking and automatically:
- Stops the current TTS playback
- Cancels any pending LLM generation
- Starts processing the new user input

### 14.3 Frame-Based Architecture

Pipecat uses a **frame-based** architecture where data flows as "frames" through the pipeline. Key frame types include:
- Audio frames
- Text frames
- Transcription frames
- LLM response frames
- Context frames
- Function call frames

### 14.4 Service Composition

Services are composable -- you can swap providers without changing the pipeline structure:

```python
# Easy to swap STT
stt = DeepgramSTTService(...)   # or WhisperSTTService(...)

# Easy to swap LLM
llm = AnthropicLLMService(...)  # or OpenAILLMService(...)

# Easy to swap TTS
tts = ElevenLabsTTSService(...) # or CartesiaTTSService(...)
```

---

## 15. Environment Variables Reference

| Variable | Service | Description |
|----------|---------|-------------|
| `ANTHROPIC_API_KEY` | Anthropic | Claude API key |
| `OPENAI_API_KEY` | OpenAI | GPT/Whisper/TTS API key |
| `DEEPGRAM_API_KEY` | Deepgram | STT/TTS API key |
| `ELEVENLABS_API_KEY` | ElevenLabs | TTS API key |
| `ELEVENLABS_VOICE_ID` | ElevenLabs | Voice ID to use |
| `CARTESIA_API_KEY` | Cartesia | TTS API key |
| `CARTESIA_VOICE_ID` | Cartesia | Voice ID to use |
| `DAILY_ROOM_URL` | Daily | WebRTC room URL |
| `DAILY_TOKEN` | Daily | Participant token |
| `PICOVOICE_ACCESS_KEY` | Picovoice | Wake word detection key |

---

## 16. Tips and Best Practices

1. **Keep responses concise**: Voice assistants should give brief, spoken-friendly responses. Instruct the LLM to avoid markdown, bullet points, and long explanations.

2. **Use interim results**: Enable `interim_results=True` in Deepgram for faster perceived response time.

3. **Endpointing tuning**: Adjust Deepgram's `endpointing` value (in ms) to control how long silence is needed before the system considers speech complete. Lower values = faster response but more false triggers.

4. **Choose low-latency TTS**: Cartesia and ElevenLabs turbo models offer the lowest latency. OpenAI TTS is higher quality but slower.

5. **VAD threshold**: Silero VAD works well out of the box, but you can tune sensitivity if needed.

6. **Function call error handling**: Always handle cases where function calls fail gracefully -- the user is listening and expects a response.

7. **Context window management**: For long conversations, implement context truncation or summarization to stay within the LLM's context window.

8. **Testing locally**: Use `LocalTransport` or `SmallWebRTCTransport` for development before deploying with `DailyTransport`.

---

## Sources

- [Pipecat GitHub Repository](https://github.com/pipecat-ai/pipecat)
- [Pipecat Documentation](https://docs.pipecat.ai)
- [Pipecat on PyPI](https://pypi.org/project/pipecat-ai/)
- [Daily.co Blog](https://daily.co/blog)
- [Deepgram Documentation](https://developers.deepgram.com)
- [Anthropic API Documentation](https://docs.anthropic.com)
- [ElevenLabs API Documentation](https://elevenlabs.io/docs)
- [Cartesia Documentation](https://cartesia.ai/docs)
