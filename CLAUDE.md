# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (preferred)
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install/sync dependencies
uv sync

# Add new dependency
uv add package_name

# Install uv (if needed)
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Environment Setup
Required `.env` file in root directory:
```
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

## Architecture Overview

This is a **tool-based RAG (Retrieval-Augmented Generation) system** with the following key architectural pattern:

### Core Architecture
- **Tool-Based AI**: Claude decides when to search for information using tool calling rather than always retrieving context
- **Semantic Course Resolution**: Vector store performs fuzzy matching on course names before content search
- **Session-Based Conversations**: Maintains conversation history for contextual responses
- **Component Separation**: Clear separation between storage, search, AI generation, and session management

### Request Flow
1. **Frontend** (static HTML/JS) → **FastAPI** (app.py) → **RAG System** (orchestrator)
2. **RAG System** → **AI Generator** → **Claude API** (with tool definitions)
3. **Claude** evaluates query → optionally calls **search_course_content** tool
4. **Tool Manager** → **CourseSearchTool** → **Vector Store** → **ChromaDB**
5. Results flow back through the chain with source tracking

### Key Components

**RAG System** (`backend/rag_system.py`): Main orchestrator that coordinates all components and manages the query workflow.

**AI Generator** (`backend/ai_generator.py`): Handles Claude API interactions with tool support. Contains the system prompt that instructs Claude when to use search tools.

**Tool Architecture** (`backend/search_tools.py`): 
- `Tool` abstract base class for extensible tool system
- `CourseSearchTool` implements semantic search with course name resolution
- `ToolManager` handles tool registration and execution

**Vector Store** (`backend/vector_store.py`): Two-collection ChromaDB setup:
- `course_catalog`: Course metadata for semantic course name matching
- `course_content`: Actual chunked course content with lesson metadata

**Session Manager** (`backend/session_manager.py`): Maintains conversation history with configurable limits.

### Data Models
**Course Processing** (`backend/models.py`): Defines Course, Lesson, and CourseChunk data structures.

**Document Processing** (`backend/document_processor.py`): Handles parsing and chunking of course documents.

### Configuration
All settings centralized in `backend/config.py` using dataclass pattern. Key settings:
- Chunk size/overlap for document processing
- ChromaDB path and embedding model
- Claude model selection and conversation history limits

### Frontend Integration
Static files served by FastAPI with development-friendly no-cache headers. Frontend communicates via REST API (`/api/query`, `/api/courses`) and handles markdown rendering and source display.

## Development Notes

### Adding New Tools
1. Implement `Tool` abstract class in `search_tools.py`
2. Register tool with `ToolManager` in `rag_system.py`
3. Tool definitions automatically available to Claude

### Vector Store Operations
- Course metadata and content are stored in separate ChromaDB collections
- Course name resolution uses semantic search before content search
- Supports filtering by course title and lesson number

### Session Management
- Session IDs generated automatically if not provided
- Conversation history limited by `MAX_HISTORY` setting
- History formatted for Claude's context window

### AI Model Configuration
- Uses Claude Sonnet 4 by default (`claude-sonnet-4-20250514`)
- System prompt in `ai_generator.py` controls tool usage behavior
- Temperature set to 0 for consistent responses