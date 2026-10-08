# Agentic RAG Chatbot

A multi-utility conversational assistant built with **LangGraph**, **Google Gemini** and **Streamlit**. Upload a PDF and ask questions about it, or let the agent decide to search the web, check a stock price or do arithmetic. Each chat is a separate thread with persistent history.

---

## Demo

![Agentic RAG Chatbot Demo UI](<img width="960" height="540" alt="Screenshot 2026-10-09 004638" src="https://github.com/user-attachments/assets/3b7f3b9b-b916-4b8a-bddf-a8c33114f536" />
)

*Chat interface with PDF upload, conversation threads and live tool-usage status.*

---

## Features

- **Agentic tool use**: a LangGraph state machine lets the model choose whether to answer directly or call a tool, then loop back with the result.
- **PDF question answering (RAG)**: uploaded PDFs are chunked, embedded and indexed in FAISS, scoped to the current chat thread.
- **Web search**: live results via DuckDuckGo.
- **Stock prices**: latest quotes through the Alpha Vantage API.
- **Calculator**: add, subtract, multiply and divide, with division-by-zero handling.
- **Persistent conversations**: thread state is checkpointed to SQLite (`chatbot.db`).
- **Streaming UI**: token-by-token responses with a live status indicator while tools run.

---

## Architecture

```
            ┌──────────────┐
  START ──▶ │  chat_node   │ ◀───────────┐
            │ (Gemini LLM) │             │
            └──────┬───────┘             │
                   │ tools_condition     │
          tool call│          no tool    │
                   ▼            │        │
            ┌──────────────┐    ▼        │
            │    tools     │───────▶ END │
            │  (ToolNode)  │─────────────┘
            └──────────────┘
```

**Tools available to the agent**

| Tool | Purpose |
|------|---------|
| `rag_tool` | Retrieves the top 4 relevant chunks from the PDF uploaded to the current thread |
| `DuckDuckGoSearchRun` | Web search |
| `get_stock_price` | Latest stock quote via Alpha Vantage |
| `calculator` | Basic arithmetic (`add`, `sub`, `mul`, `div`) |

**RAG pipeline**

1. PDF is loaded with `PyPDFLoader`.
2. Text is split with `RecursiveCharacterTextSplitter` (chunk size 1000, overlap 200).
3. Chunks are embedded with `gemini-embedding-001` and stored in a FAISS index.
4. A similarity retriever (`k=4`) is registered per chat thread.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| LLM | Google Gemini (`ChatGoogleGenerativeAI`) |
| Embeddings | `gemini-embedding-001` |
| Orchestration | LangGraph, LangChain |
| Vector store | FAISS (CPU) |
| Persistence | SQLite via `langgraph-checkpoint-sqlite` |
| Frontend | Streamlit |
| Search | DuckDuckGo |
| Language | Python 3.13 |

---

## Project Structure

```
Agentic_RAG_chatbot/
├── assets/
│   └── demo-ui.png             # Demo screenshot used in this README
├── langraph_rag_backend.py     # LangGraph agent, tools, RAG ingestion, checkpointing
├── streamlit_rag_frontend.py   # Streamlit chat UI
├── requirements.txt            # Python dependencies
├── chatbot.db                  # SQLite conversation store (auto-created)
└── .env                        # API keys (not committed)
```

---

## Getting Started

### Prerequisites

- Python 3.10 or newer (developed on 3.13)
- A [Google AI Studio](https://aistudio.google.com/) API key
- An [Alpha Vantage](https://www.alphavantage.co/) API key (only needed for the stock tool)

### Installation

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd Agentic_RAG_chatbot

# 2. Create and activate a virtual environment
python -m venv myenv
# Windows
myenv\Scripts\activate
# macOS / Linux
source myenv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt
pip install langchain-community langchain-google-genai faiss-cpu pypdf ddgs requests
```

### Configuration

Create a `.env` file in the project root:

```env
GOOGLE_API_KEY=your_google_api_key
ALPHAVANTAGE_URL=https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=AAPL&apikey=your_alphavantage_key
```

> `ALPHAVANTAGE_URL` is used as-is by the stock tool, so the symbol is fixed by this URL. See *Known Limitations*.

### Run

```bash
streamlit run streamlit_rag_frontend.py
```

The app opens at `http://localhost:8501`.

---

## Usage

1. Click **New Chat** in the sidebar to start a thread.
2. Upload a PDF from the sidebar and wait for the "PDF indexed" confirmation.
3. Ask questions about the document, or ask anything that needs a tool:
   - *"Summarize chapter 2 of the uploaded document."*
   - *"What is the latest news about LangGraph?"*
   - *"What is 1,284 divided by 12?"*
   - *"What is the current price of AAPL?"*
4. A status box shows which tool the agent is using while it works.

---

## Known Limitations

- **PDF indexes are in memory.** FAISS retrievers live in process memory, so after a restart the chat history remains but PDFs must be re-uploaded.
- **Stock tool ignores the requested symbol.** `get_stock_price` calls the URL in `ALPHAVANTAGE_URL` directly, so it returns whichever symbol that URL contains.
- **Past-conversation switching is not wired up.** Past chats are listed in the sidebar, but the handler that loads a selected thread is currently commented out in the frontend. Chat titles are also kept only for the current browser session.
- **`requirements.txt` is incomplete.** It does not list `langchain-community`, `langchain-google-genai`, `faiss-cpu`, `pypdf`, `ddgs` or `requests`, which the code imports (hence the extra install command above).

---

## Roadmap

- Persist FAISS indexes to disk so documents survive restarts
- Accept the stock symbol as a tool argument
- Enable loading past conversations from the sidebar
- Support multiple documents per thread
- Add source citations (page numbers) to RAG answers

---

## License

Distributed under the MIT License. Add a `LICENSE` file to the repository, or replace this section with your preferred license.

---

## Author

**Vishal Prajapati**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/vishal-prajapati93/)
