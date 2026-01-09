# LFM2 Agent - Browser-Based Agentic System

🌊 **Reference implementation of a browser-based agentic system using sub-2B parameter models.** Implements Liquid AI's tool calling format with LFM2-1.2B-Tool and semantic retrieval via EmbeddingGemma-300M embeddings. Designed to explore the lower bounds of model size for reliable agent behavior on edge devices.

![LFM2 Agent](https://img.shields.io/badge/LFM2-Agent-00d4ff?style=for-the-badge)
![WebGPU](https://img.shields.io/badge/WebGPU-Enabled-green?style=for-the-badge)
![100% Local](https://img.shields.io/badge/100%25-Local-purple?style=for-the-badge)
![Sub-2B Parameters](https://img.shields.io/badge/Sub--2B-Parameters-orange?style=for-the-badge)

## ✨ Features

- **🚀 100% Local Inference**: No backend server required, everything runs in your browser
- **⚡ WebGPU Acceleration**: Leverages your device's GPU for fast inference
- **🔧 Tool Calling**: Native function calling support with Liquid AI's format
- **🔍 RAG with EmbeddingGemma**: Semantic search over personal knowledge base
- **💾 Context Management**: Automatic conversation summarization and compaction
- **📱 Responsive Design**: Works on both mobile and desktop devices

## 🧠 Context Engineering Principles

This implementation applies key [context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) principles for building capable agents:

### Why Context Engineering Matters

LLMs have a finite **attention budget** that gets depleted as token count increases. Research shows that as context window size grows, the model's ability to accurately recall information decreases—a phenomenon known as **context rot**. This project implements strategies to maximize signal while minimizing noise in the context window.

### Implemented Techniques

| Technique | Implementation | Purpose |
|-----------|----------------|---------|
| **Compaction** | `summarizeConversation()` | Summarizes conversation history when nearing token limits, preserving critical context while discarding redundant information |
| **Structured Note-Taking** | `conversationSummary` | Maintains persistent memory across context resets |
| **Just-in-Time Retrieval** | RAG with EmbeddingGemma | Dynamically loads relevant context at runtime rather than pre-loading everything |
| **Token-Efficient Tools** | Minimal tool output | Tools return concise, high-signal responses |

### The Guiding Principle

> *"Find the smallest possible set of high-signal tokens that maximize the likelihood of the desired outcome."*

## 📝 Prompt Engineering Best Practices

This project follows [prompt engineering best practices](https://claude.com/blog/best-practices-for-prompt-engineering) in its system prompts:

### Core Techniques Applied

1. **Be Explicit and Clear**
   - Direct action verbs: "You MUST call...", "DO NOT guess..."
   - Specific quality expectations for tool usage

2. **Provide Context and Motivation**
   - Explains WHY the model should use tools: "You do NOT have access to real-time information"
   - Clear reasoning behind constraints

3. **Be Specific**
   - Exact output format: `<|tool_call_start|>[function_name(param="value")]<|tool_call_end|>`
   - Explicit constraints and requirements

4. **Use Examples**
   - Few-shot examples for tool calling format
   - Demonstrates expected behavior patterns

5. **Permission to Express Uncertainty**
   - Tools return "No info found" rather than hallucinating
   - Model guided to use tools instead of guessing

### Advanced Techniques

- **Prompt Chaining**: Multi-turn inference for tool execution → response generation
- **Structured Output Control**: Enforced tool call format with start/end tokens
- **Minimal Viable Prompts**: System prompt calibrated to the "Goldilocks zone"—specific enough to guide behavior, flexible enough to handle edge cases

## 🛠️ Available Tools

| Tool | Description |
|------|-------------|
| `search_profile` | Searches personal knowledge base for experience, skills, and projects |
| `calculator` | Performs mathematical calculations and percentages |
| `get_current_date` | Returns current date and time |
| `get_weather` | Retrieves weather conditions for a city (simulated) |

### Tool Design Principles

Following the principle of **minimal viable tool sets**:
- Each tool has a single, clear purpose
- No overlapping functionality between tools
- Input parameters are descriptive and unambiguous
- Tools return token-efficient responses

## 🧠 Supported Models

| Model | Parameters | Use Case |
|-------|------------|----------|
| **LFM2 1.2B Tool** | 1.2B | Optimized for function calling (recommended) |
| **LFM2 1.2B General** | 1.2B | General purpose conversations |
| **LFM2 700M** | 700M | Lightweight option for memory-constrained devices |

## 📋 Requirements

- Modern browser with support for:
  - **WebGPU** (Chrome 113+, Edge 113+, or Firefox Nightly)
  - ES Modules
  - Web Workers
- Minimum 4GB available RAM
- WebGPU-compatible GPU for optimal performance

> 💡 **Note**: If WebGPU is unavailable, the system falls back to WebAssembly (WASM).

## 🚀 Usage

### Option 1: Direct File
Simply open `lfm2-agent-v2-clean.html` in your browser.

### Option 2: Local Server
```bash
# Python
python -m http.server 8080

# Node.js
npx serve .

# PHP
php -S localhost:8080
```

Then navigate to `http://localhost:8080/lfm2-agent-v2-clean.html`

## 📖 Example Prompts

```
What is my experience in ML?
→ Uses search_profile to search knowledge base

What date and time is it?
→ Uses get_current_date to get current timestamp

Calculate 18% of 4500
→ Uses calculator to perform the calculation

What's the weather in Madrid?
→ Uses get_weather to query weather data
```

## ⚙️ Customization

### Personal Knowledge Base

Edit the `PERSONAL_KNOWLEDGE_BASE` constant in the HTML file:

```javascript
const PERSONAL_KNOWLEDGE_BASE = [
  { id: "profile", content: "Your name and role..." },
  { id: "experience", content: "Your professional experience..." },
  { id: "skills", content: "Your technical skills..." },
  { id: "projects", content: "Your projects..." },
  { id: "education", content: "Your education..." },
  { id: "interests", content: "Your interests..." }
];
```

### Adding New Tools

1. Define the tool schema in the `TOOLS` array
2. Implement the executor in `toolExecutors`
3. The model will automatically use the new tool

**Best Practices for New Tools:**
- Write clear, unambiguous descriptions
- Avoid functionality overlap with existing tools
- Return minimal, high-signal outputs

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Web Browser                         │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │   LFM2      │  │ Embedding   │  │    Tool         │ │
│  │   Model     │  │   Gemma     │  │   Executors     │ │
│  │  (Q4/ONNX)  │  │  (300M)     │  │                 │ │
│  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘ │
│         │                │                   │          │
│  ┌──────┴────────────────┴───────────────────┴────────┐ │
│  │              Transformers.js Runtime               │ │
│  └────────────────────────┬───────────────────────────┘ │
│                           │                             │
│  ┌────────────────────────┴───────────────────────────┐ │
│  │           WebGPU / WASM Backend                    │ │
│  └────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Context Flow

```
User Query → RAG Retrieval → System Prompt + Context → LLM Inference
                                       ↓
                              Tool Call Detection
                                       ↓
                              Tool Execution → Result
                                       ↓
                              Second Inference → Response
                                       ↓
                              Context Compaction (if needed)
```

## 🔧 Technologies

- **[Transformers.js](https://huggingface.co/docs/transformers.js)** - Hugging Face's ML library for the browser
- **[LFM2 (Liquid Foundation Model 2)](https://www.liquid.ai/)** - Liquid AI's language model
- **[EmbeddingGemma](https://huggingface.co/onnx-community/embeddinggemma-300m-ONNX)** - Embedding model for RAG
- **WebGPU** - GPU acceleration API for the web
- **ONNX Runtime** - Optimized model inference runtime

## 📚 References

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Anthropic
- [Best Practices for Prompt Engineering](https://claude.com/blog/best-practices-for-prompt-engineering) - Anthropic
- [Building Effective AI Agents](https://www.anthropic.com/research/building-effective-agents) - Anthropic
- [Writing Tools for AI Agents](https://www.anthropic.com/engineering/writing-tools-for-agents) - Anthropic

## 📝 License

This project is available for personal and educational use.

## 🤝 Contributing

Contributions are welcome! If you have ideas to improve the agent:

1. Fork the project
2. Create a feature branch (`git checkout -b feature/new-tool`)
3. Commit your changes (`git commit -m 'Add new tool'`)
4. Push to the branch (`git push origin feature/new-tool`)
5. Open a Pull Request

---

**Built with 💜 using Liquid AI + Hugging Face + WebGPU**
