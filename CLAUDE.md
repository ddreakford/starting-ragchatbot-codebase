# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Add new dependency
uv add package_name

# Add development dependency  
uv add --dev package_name
```

### Environment Setup
Create `.env` file in root directory:
```bash
ANTHROPIC_API_KEY=your_key_here
```

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) chatbot** that answers questions about course materials using semantic search and Claude AI.

### Core Architecture Pattern

The system follows a **tool-based RAG pattern** where Claude AI decides when to search the knowledge base:

1. **Frontend** (`frontend/`) - Vanilla HTML/JS interface
2. **FastAPI Backend** (`backend/app.py`) - REST API server  
3. **RAG System** (`backend/rag_system.py`) - Main orchestrator
4. **AI Generator** (`backend/ai_generator.py`) - Claude API integration with tool execution
5. **Vector Store** (`backend/vector_store.py`) - ChromaDB wrapper for semantic search
6. **Search Tools** (`backend/search_tools.py`) - Claude-callable search functions

### Key Components

**RAG System (`rag_system.py`)**
- Main orchestrator that coordinates all components
- Manages document ingestion from `docs/` folder on startup
- Handles query processing flow: Query → AI → Tools → Response

**AI Generator (`ai_generator.py`)**
- Manages Claude API interactions with tool execution
- System prompt configures Claude to use search tools selectively
- Handles tool execution workflow: Initial response → Tool calls → Final response

**Tool System (`search_tools.py`)**
- Implements Anthropic's tool interface pattern
- `CourseSearchTool` provides semantic search with course/lesson filtering
- `ToolManager` registers and executes tools during AI generation

**Vector Store (`vector_store.py`)**
- ChromaDB wrapper for document storage and retrieval
- Separate collections for course metadata and content chunks
- Semantic similarity search with filtering capabilities

**Document Processing (`document_processor.py`)**
- Parses course documents into structured format (Course → Lessons → Chunks)
- Chunking strategy: 800 characters with 100 character overlap
- Extracts course metadata and lesson structure

### Data Models (`models.py`)

```python
Course: title, course_link, instructor, lessons[]
Lesson: lesson_number, title, lesson_link  
CourseChunk: content, course_title, lesson_number, chunk_index
```

### Configuration (`config.py`)

All settings centralized in dataclass:
- API keys and model selection
- Chunking parameters (CHUNK_SIZE: 800, CHUNK_OVERLAP: 100)
- Search limits (MAX_RESULTS: 5, MAX_HISTORY: 2)
- Database paths

### Query Processing Flow

1. Frontend sends POST to `/api/query` with query and session_id
2. FastAPI validates request and calls `rag_system.query()`
3. RAG system formats prompt and calls AI generator with tools
4. Claude decides if search is needed and calls `search_course_content` tool
5. Tool executes semantic search against ChromaDB
6. Claude generates final response using search results
7. Response returned with sources for frontend display

### Session Management (`session_manager.py`)

- Maintains conversation context across queries
- Stores last 2 exchanges per session (configurable via MAX_HISTORY)
- Session IDs generated automatically if not provided

## Important Implementation Details

### Tool Execution Pattern
The AI Generator uses a two-phase approach:
1. Initial Claude call with tools available
2. If tools used, execute them and make follow-up call with results

### Search Strategy
- Course name matching uses semantic similarity on course titles
- Content search performs similarity search on document chunks
- Results filtered by course and/or lesson if specified

### Document Loading
- Automatic document loading from `../docs/` on FastAPI startup
- Supports PDF, DOCX, and TXT files
- Incremental loading (skips existing courses by title)

### Frontend Architecture
- Single-page application with vanilla JavaScript
- Markdown rendering for AI responses using marked.js
- Source citations displayed in collapsible sections
- Real-time loading states and error handling