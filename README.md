# Retrieval-Augmented Generation (RAG) with MongoDB Vector Search

![Architecture Diagram](RAG%20with%20MongoDB%20Vector%20Search.png)

## Overview
This project demonstrates how to build a robust, scalable Retrieval-Augmented Generation (RAG) backend utilizing **MongoDB Atlas Vector Search**. By embedding text data as high-dimensional vectors and querying them via semantic similarity, we can reliably retrieve relevant context to power Large Language Models (LLMs) and advanced AI applications.

## Why this Architecture?
To optimize inference speed, avoid rate limits, and significantly cut down API costs, this pipeline employs a locally hosted dense embedding model to process unstructured text into 384-dimensional semantic vectors. MongoDB Atlas Vector Search then provides an integrated retrieval fabric without the need for bolting on a standalone vector-only database.

### Core Stack
- **Vector Database**: MongoDB Atlas (Aggregations & `$vectorSearch`)
- **Embedding Model**: `all-MiniLM-L6-v2` (`sentence-transformers` via Hugging Face) producing `384`-dimensional embeddings entirely locally.
- **LLM Generator**: OpenAI (GPT-4o) via the official `openai` SDK to synthesize answers from retrieved context.
- **Data Drivers**: `pymongo` and `langchain` text splitters for handling data digestion, chunking, and routing.

## Features
- **Automated Vector Indexing**: The pipeline leverages `SearchIndexModel` to programmatically build a MongoDB Atlas Vector Search index directly from Python code—handling index dimensions, cosine similarity definitions, and sync polling automatically.
- **Local Embedding Computations**: Fully self-contained local embedding pipeline requiring zero external third-party embedding API keys.
- **Semantic Vector Search Pipeline**: Employs MongoDB's semantic `$vectorSearch` pipeline stage to find nearest document neighbors, utilizing `numCandidates` scaling for performance tuning. 

## Installation & Setup

### Prerequisites
- Python 3.12+
- A running MongoDB Atlas Cluster (the `M0` Free Tier works beautifully)
- An active OpenAI API Key (for the final generation step)

### Setup Instructions
1. Clone the repository and navigate into it:
   ```bash
   git clone https://github.com/HiteshM2107/RAG-MongoDB.git
   cd RAG-MongoDB
   ```

2. Initialize a virtual environment and install dependencies:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

3. Environment Variables:
   Create a `.env` file in the root of the directory and define your OpenAI API key so it can be loaded natively:
   ```env
   OPENAI_API_KEY="sk-proj-your-key-here"
   ```

4. Set up your MongoDB Connection:
   Open `rag.ipynb` and locate the `MongoClient` definition. Replace the connection string with your secure MongoDB Atlas URI.

## Usage

Execution code and experiments are entirely driven inside the **`rag.ipynb`** Jupyter Notebook. 

Workflow sequence:
1. **Load Data**: The notebook breaks a target corpus down into ingestible chunks via `langchain-text-splitters`.
2. **Generate Embeddings**: Pushes data chunks through `SentenceTransformer('all-MiniLM-L6-v2')` to extract dense vectors.
3. **Ingest to MongoDB**: Connects to the Database (`RAGApplication`) and stages vectors into the Database Collection (`documents`).
4. **Create Search Index**: Creates the Vector Index on your Atlas cluster specifically tuned for `384` dimensions.
5. **Run a Query**: Use the helper function `get_query_results("Search Query")` to test the semantic retrieval outputs!
6. **Generation**: The notebook uses `python-dotenv` to load your API key, then leverages `gpt-4o` combined with your retrieved context documents to generate a fully informed, semantic response.

## Troubleshooting

### Vector Dimension Mismatches
If you plan to switch embedding models resulting in an `OperationFailure` (e.g., `vector field is indexed with X dimensions but queried with Y`), remember that:
- Your MongoDB Search Index `numDimensions` must exactly match your embedding model's dimensionality. 
- Example: The native `all-MiniLM-L6-v2` utilizes `384` dimensions, whereas scaling to OpenAI `text-embedding-3-small` leverages `1536` dimensions.
- **Fix**: When changing models, explicitly alter the `numDimensions` property and build a new search index via a unique `index_name` to force MongoDB to resync the new dimensions.
