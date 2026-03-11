
# Agentic RAG using CrewAI

An intelligent **Retrieval-Augmented Generation (RAG)** system built with [CrewAI](https://www.crewai.com/) that searches through user-uploaded PDFs and automatically falls back to web search when the answer isn't found in the documents. Supports multiple LLM backends — cloud-based (OpenAI) or fully local via [Ollama](https://ollama.com/) (DeepSeek-R1, Llama 3.2).


## Features

- **Multi-agent architecture** — A Retriever agent finds relevant information, then a Response Synthesizer agent crafts a coherent answer.
- **Semantic PDF search** — PDFs are chunked semantically using [Chonkie](https://github.com/bhavnicksm/chonkie) and indexed in an in-memory [Qdrant](https://qdrant.tech/) vector store for fast retrieval.
- **Web search fallback** — When the document doesn't contain the answer, the system falls back to web search (SerperDev / FireCrawl).
- **Multiple LLM backends** — Choose between cloud APIs (OpenAI) or local models via Ollama (DeepSeek-R1:7b, Llama 3.2).
- **Streamlit UI** — Simple web interface to upload PDFs and ask questions.
- **Jupyter notebooks** — Step-by-step demos for experimentation and learning.

---

## Architecture

```
User Query (Streamlit UI)
        │
        ▼
┌─────────────────────┐
│   Retriever Agent    │
│  ┌───────────────┐   │
│  │ DocumentSearch │──►  Qdrant (semantic search over PDF)
│  │    Tool        │   │
│  └───────────────┘   │
│         │ (no result)│
│         ▼            │
│  ┌───────────────┐   │
│  │  Web Search   │──►  SerperDev / FireCrawl
│  │    Tool        │   │
│  └───────────────┘   │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────────┐
│ Response Synthesizer    │
│       Agent             │──► Final coherent answer
└─────────────────────────┘
```

### Agents

| Agent | Role | Description |
|-------|------|-------------|
| **Retriever** | Information Retrieval Specialist | Searches the PDF knowledge base first, then falls back to web search if needed. |
| **Response Synthesizer** | Response Crafting Specialist | Takes retrieved information and generates a concise, well-structured answer. |

### Custom Tools

| Tool | Description |
|------|-------------|
| **DocumentSearchTool** | Extracts text from PDFs (via MarkItDown), chunks semantically (Chonkie, 512-token chunks, threshold 0.5), embeds with `minishlab/potion-base-8M`, and stores/queries in Qdrant. |

---

## Project Structure

```
agentic_rag/
├── app.py                  # Streamlit app (cloud LLM + SerperDev search)
├── app_deep_seek.py        # Streamlit app (Ollama DeepSeek-R1:7b)
├── app_llama3.2.py         # Streamlit app (Ollama Llama 3.2)
├── pyproject.toml          # Project metadata & dependencies
├── agentic_rag.ipynb       # Advanced multi-agent RAG notebook (Groq + Tavily)
├── demo_llama3.2.ipynb     # Minimal local RAG demo (Ollama Llama 3.2)
├── knowledge/
│   └── dspy.pdf            # Sample PDF knowledge base
├── assets/                 # Static assets
├── thumbnail/              # YouTube thumbnail
└── src/
    └── agentic_rag/
        ├── __init__.py
        ├── crew.py         # CrewAI crew definition (agents, tasks, tools)
        ├── main.py         # CLI entrypoint (run, train, replay, test)
        ├── config/
        │   ├── agents.yaml # Agent role/goal/backstory definitions
        │   └── tasks.yaml  # Task descriptions & expected outputs
        └── tools/
            ├── __init__.py
            └── custom_tool.py  # DocumentSearchTool implementation
```

---

## App Variants

| Variant | LLM | Web Search | Command |
|---------|-----|------------|---------|
| `app.py` | Cloud (OpenAI) | SerperDev | `streamlit run app.py` |
| `app_deep_seek.py` | Ollama DeepSeek-R1:7b | FireCrawl | `streamlit run app_deep_seek.py` |
| `app_llama3.2.py` | Ollama Llama 3.2 | FireCrawl | `streamlit run app_llama3.2.py` |

---

## Prerequisites

- **Python** 3.10 – 3.13
- **Ollama** (required for local LLM variants) — [Install Ollama](https://ollama.com/download)

---

## Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/patchy631/ai-engineering-hub.git
   cd ai-engineering-hub/agentic_rag
   ```

2. **Install dependencies**

   ```bash
   pip install crewai[tools]>=0.86.0 crewai-tools chonkie[semantic] markitdown qdrant-client fastembed streamlit
   ```

3. **Get API keys**

   | Service | Required For | Link |
   |---------|-------------|------|
   | FireCrawl | Web search (local LLM apps) | [firecrawl.dev](https://www.firecrawl.dev/i/api) |
   | SerperDev | Web search (`app.py`) | [serper.dev](https://serper.dev/) |
   | OpenAI | Cloud LLM (`app.py`) | [platform.openai.com](https://platform.openai.com/) |
   | Groq | Advanced notebook | [console.groq.com](https://console.groq.com/) |
   | Tavily | Advanced notebook | [tavily.com](https://tavily.com/) |

4. **Set environment variables**

   ```bash
   # For app.py (cloud)
   export OPENAI_API_KEY="your-openai-key"
   export SERPER_API_KEY="your-serper-key"

   # For local LLM apps
   export FIRECRAWL_API_KEY="your-firecrawl-key"

   # For advanced notebook
   export GROQ_API_KEY="your-groq-key"
   export TAVILY_API_KEY="your-tavily-key"
   ```

   On Windows (PowerShell):
   ```powershell
   $env:OPENAI_API_KEY = "your-openai-key"
   $env:FIRECRAWL_API_KEY = "your-firecrawl-key"
   ```

5. **Pull Ollama models** (for local variants only)

   ```bash
   ollama pull deepseek-r1:7b    # for app_deep_seek.py
   ollama pull llama3.2           # for app_llama3.2.py
   ```

---

## Usage

### Streamlit App

```bash
# Cloud LLM (OpenAI + SerperDev)
streamlit run app.py

# Local LLM — DeepSeek-R1
streamlit run app_deep_seek.py

# Local LLM — Llama 3.2
streamlit run app_llama3.2.py
```

Upload a PDF via the sidebar, type your question, and the agents will search the document (and optionally the web) to generate an answer.

### CLI (CrewAI)

```bash
# Run the crew
python -m agentic_rag.main

# Or via the installed script
agentic_rag
```

### Notebooks

| Notebook | Description |
|----------|-------------|
| `agentic_rag.ipynb` | Advanced multi-agent pipeline with 5+ agents (Router, Retriever, Grader, Hallucination Grader, Answer Grader) using Groq + Tavily |
| `demo_llama3.2.ipynb` | Minimal 2-agent demo running 100% locally with Ollama Llama 3.2 |

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Agent Orchestration | [CrewAI](https://www.crewai.com/) |
| Vector Database | [Qdrant](https://qdrant.tech/) (in-memory) |
| Embeddings | [FastEmbed](https://github.com/qdrant/fastembed) — `minishlab/potion-base-8M` |
| Text Chunking | [Chonkie](https://github.com/bhavnicksm/chonkie) (semantic, 512 tokens) |
| PDF Extraction | [MarkItDown](https://github.com/microsoft/markitdown) |
| Web UI | [Streamlit](https://streamlit.io/) |
| Local LLMs | [Ollama](https://ollama.com/) |
| LLM Integration | [LangChain](https://www.langchain.com/) |

---
