# LLM Interaction with RAG and Vector Search

A Retrieval-Augmented Generation (RAG) system that demonstrates how to combine LLMs with vector search for accurate, context-aware question answering. This project showcases multiple vector storage backends and search strategies for FAQ data ingestion and retrieval.

## Overview

This project implements a complete RAG pipeline that:
- **Ingests FAQ data** from external sources (DataTalks.Club courses)
- **Indexes documents** using multiple vector search backends
- **Retrieves relevant context** for queries using semantic search
- **Generates answers** using LLMs with retrieved context

The system is designed to answer domain-specific questions by providing the LLM with relevant context, enabling more accurate and reliable responses.

## Features

- 🔍 **Multiple Vector Search Backends**
  - In-memory MinSearch indexing
  - SQLite with persistent storage
  - PostgreSQL with pgvector extension for production use

- 📚 **Flexible FAQ Data Ingestion**
  - Fetches course FAQ data from external APIs
  - Supports filtering by course
  - Extensible document structure

- 🤖 **LLM Integration**
  - OpenAI API integration for generating answers
  - Configurable prompts and instructions
  - Customizable model selection

- 🎯 **RAG Pipeline**
  - Relevance-based search with configurable boosting
  - Context-aware prompt construction
  - Fallback responses for unknown questions

## Project Structure

```
.
├── ingest.py                      # Data ingestion module
├── rag_helper.py                  # RAG pipeline implementation
├── main.py                        # Main application entry point
├── pyproject.toml                 # Project dependencies and metadata
├── vector_search.ipynb            # Basic vector search exploration
├── vector_search_persistent.ipynb # SQLite-based persistent search
├── vector_search_pgvector.ipynb   # PostgreSQL pgvector backend
└── README.md                      # This file
```

## Installation

### Prerequisites
- Python 3.12+
- OpenAI API key
- (Optional) PostgreSQL with pgvector extension for production deployment

### Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd llm-interactionwithragwithvectorsearch
   ```

2. **Create and activate a virtual environment**
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -e .
   ```

4. **Configure environment variables**
   Create a `.env` file with your OpenAI API key:
   ```
   OPENAI_API_KEY=your_api_key_here
   ```

## Architecture

### Core Components

#### `ingest.py`
Handles FAQ data ingestion:
- Fetches FAQ data from DataTalks.Club API
- Loads and structures documents for indexing
- Supports course-level filtering

```python
def load_faq_data():
    """Fetch FAQ data from external source"""
    # Downloads course FAQ data

def build_index(documents):
    """Create searchable index from documents"""
    # Builds MinSearch index with configured fields
```

#### `rag_helper.py`
Core RAG implementation with `RAGBase` class:
- **search()**: Retrieves relevant documents based on query
- **build_context()**: Formats retrieved documents as context
- **build_prompt()**: Constructs LLM prompt with context
- **llm()**: Calls LLM API for answer generation
- **rag()**: Orchestrates the complete RAG pipeline

```python
# Usage example
rag = RAGBase(index=search_index, llm_client=client)
answer = rag.rag("What is the course about?")
```

### Search Strategies

**Default Search Configuration:**
- Boosts "question" field (3.0x weight) for higher relevance
- Reduces "section" field weight (0.5x)
- Filters by course to scope results
- Returns top 5 most relevant documents

## Usage

### Basic Example

```python
from ingest import load_faq_data, build_index
from rag_helper import RAGBase
from openai import OpenAI

# Load data
documents = load_faq_data()

# Build search index
index = build_index(documents)

# Initialize LLM client
client = OpenAI()

# Create RAG pipeline
rag = RAGBase(
    index=index,
    llm_client=client,
    course='llm-zoomcamp',
    model='gpt-4-mini'
)

# Ask a question
answer = rag.rag("How do I submit my homework?")
print(answer)
```

### Using Different Vector Search Backends

This project includes three notebooks demonstrating different approaches:

1. **`vector_search.ipynb`** - Basic in-memory search
   - Quick exploration and prototyping
   - No persistence between sessions
   - Ideal for development

2. **`vector_search_persistent.ipynb`** - SQLite backend
   - Persistent storage using SQLite
   - Maintains index between sessions
   - Good for single-machine deployments

3. **`vector_search_pgvector.ipynb`** - PostgreSQL backend
   - Enterprise-grade vector database
   - Scalable for production use
   - Advanced vector operations support

## Configuration

### RAGBase Parameters

```python
RAGBase(
    index,                    # Search index object
    llm_client,              # OpenAI client
    instructions=...,        # System instructions for LLM
    prompt_template=...,     # Template for LLM prompts
    course='llm-zoomcamp',   # Course to filter by
    model='gpt-4-mini'       # LLM model to use
)
```

### Search Customization

Modify search behavior in `RAGBase.search()`:
- `num_results`: Number of documents to retrieve (default: 5)
- `boost_dict`: Field weights for relevance scoring
- `filter_dict`: Filters to apply (e.g., by course)

## Requirements

```
jupyter>=1.1.1
minsearch>=0.1.0
openai>=2.41.0
psycopg[binary]>=3.3.4          # PostgreSQL support
python-dotenv>=1.2.2
requests>=2.34.2
sentence-transformers>=5.5.1
```

## Development

### Running Tests & Notebooks

Use Jupyter to explore the different implementations:

```bash
jupyter notebook vector_search.ipynb
jupyter notebook vector_search_persistent.ipynb
jupyter notebook vector_search_pgvector.ipynb
```

### Project Dependencies

This project uses:
- **minsearch**: Fast, lightweight search indexing
- **sentence-transformers**: Semantic embeddings for documents and queries
- **psycopg**: PostgreSQL adapter with pgvector support
- **openai**: Official OpenAI Python client
- **python-dotenv**: Environment variable management

## Extending the Project

### Add New Data Sources

Modify `ingest.py` to fetch from different sources:

```python
def load_custom_data():
    """Load FAQ data from your source"""
    documents = []
    # Your data loading logic here
    return documents
```

### Custom RAG Implementations

Extend `RAGBase` for specialized behavior:

```python
class CustomRAG(RAGBase):
    def search(self, query, num_results=5):
        # Custom search logic
        pass
    
    def build_context(self, search_results):
        # Custom context formatting
        pass
```

### Switch Vector Backends

Each notebook demonstrates how to use different backends:
- Replace `Index` from `minsearch` with your backend's API
- Adapt the search/storage methods accordingly
- Update configuration as needed

## Troubleshooting

### API Key Issues
- Ensure `.env` file exists with valid `OPENAI_API_KEY`
- Check OpenAI account has available API credits

### Index Building Issues
- Verify network access to fetch FAQ data
- Check document structure matches index field requirements

### Vector Search Performance
- For large datasets, use PostgreSQL pgvector backend
- Consider caching frequently asked questions
- Tune `boost_dict` weights for better relevance

## License

[Add your license information here]

## Contributing

Contributions welcome! Please feel free to submit issues and pull requests.