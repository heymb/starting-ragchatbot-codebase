# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Dependencies
```bash
# Install/sync dependencies
uv sync
```

### Environment Setup
Create `.env` file in root with:
```
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

## Architecture Overview

This is a **RAG (Retrieval-Augmented Generation) chatbot system** built with FastAPI backend and vanilla JavaScript frontend. The system enables semantic search and AI-powered responses about course materials.

### Core Architecture Pattern

The system follows a **tool-calling RAG architecture** where Claude AI decides whether to search course content based on query analysis:

1. **Frontend** → **FastAPI Endpoint** → **RAG Orchestrator** → **AI Generator**
2. **AI Generator** calls **Anthropic Claude** with available tools
3. **Claude decides** whether to use `search_course_content` tool or answer directly
4. **Tool execution** triggers **Vector Store** semantic search via **ChromaDB**
5. **Results flow back** through the chain for final response synthesis

### Key Components

**Backend (`backend/`)**:
- `app.py` - FastAPI server with `/api/query` and `/api/courses` endpoints
- `rag_system.py` - Main orchestrator coordinating all components
- `ai_generator.py` - Anthropic Claude API integration with tool calling
- `search_tools.py` - Tool interface and course search implementation
- `vector_store.py` - ChromaDB wrapper with semantic search and course resolution
- `document_processor.py` - Text chunking and structured document parsing
- `session_manager.py` - Conversation history management
- `config.py` - Centralized configuration with environment variables

**Frontend (`frontend/`)**:
- `index.html` - Chat interface with course stats sidebar
- `script.js` - API communication and UI management
- `style.css` - Styling for chat interface

### Document Processing Pipeline

Documents in `docs/` follow a structured format:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson X: [lesson title]
Lesson Link: [lesson url]
[lesson content...]
```

The system:
1. **Parses metadata** (course title, instructor, lessons)
2. **Chunks content** using sentence-based splitting with configurable overlap
3. **Adds context** to chunks (`"Course X Lesson Y content: ..."`)
4. **Stores in ChromaDB** with metadata for filtering

### Vector Store Design

**Two ChromaDB collections**:
- `course_catalog` - Course metadata for fuzzy name resolution
- `course_content` - Chunked content with course/lesson metadata

**Search capabilities**:
- Semantic similarity search using sentence transformers
- Course name fuzzy matching (`"MCP"` → `"MCP: Build Rich-Context AI Apps with Anthropic"`)
- Lesson number filtering
- Metadata preservation for source attribution

### Tool-Calling Flow

The system uses **Claude's tool calling** rather than direct retrieval:

1. **Query analysis**: Claude determines if search is needed
2. **Tool invocation**: `search_course_content(query, course_name?, lesson_number?)`
3. **Vector search**: ChromaDB semantic search with filters
4. **Result synthesis**: Claude combines search results into natural responses
5. **Source tracking**: UI displays collapsible source attributions

### Session Management

- **Stateless sessions**: Session IDs created client-side or server-side
- **Conversation context**: Limited history (configurable via `MAX_HISTORY`)
- **Memory management**: Automatic truncation of old messages

### Configuration

Key settings in `config.py`:
- `CHUNK_SIZE`: 800 characters (text chunk size)
- `CHUNK_OVERLAP`: 100 characters (overlap between chunks)
- `MAX_RESULTS`: 5 (search results limit)
- `MAX_HISTORY`: 2 (conversation exchanges to remember)
- `EMBEDDING_MODEL`: `all-MiniLM-L6-v2` (sentence transformer model)
- `ANTHROPIC_MODEL`: `claude-sonnet-4-20250514`

### Application Startup

On server start (`app.py:88-99`):
1. Loads documents from `../docs` folder
2. Processes and chunks documents
3. Adds to vector store (skips existing courses)
4. Prints loading statistics

### Development Notes

- **Document format**: Strict parsing expects course metadata in first 3 lines
- **Tool execution**: Single search per query maximum (enforced in AI prompt)
- **Error handling**: Search errors gracefully degrade to empty results
- **CORS**: Configured for development with wildcard origins
- **Static files**: Frontend served from `/` with no-cache headers for development