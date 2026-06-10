# Support RAG Agent

An n8n-based Retrieval-Augmented Generation (RAG) chatbot that helps customer support representatives answer technical questions about Agilent LC/HPLC, GC instruments, and consumables — grounded in Agilent's official Community Wiki knowledge base.

## How it works

The project consists of two n8n workflows:

### 1. Support Ingestion (`workflows/Support Ingestion Final(1).json`)

Builds and refreshes the knowledge base:

- Runs manually or on a weekly schedule
- Crawls three Agilent Community Wiki resource pages (LC/HPLC, GC, Columns & Consumables)
- Extracts page title and main content via CSS selectors
- Splits content into ~600-character overlapping chunks with stable IDs
- Embeds chunks with OpenAI embeddings and upserts them into a Pinecone index (`agilent-support-agent`), partitioned into `lc`, `gc`, and `consumables` namespaces

### 2. AI Agent (`workflows/AIAgentFinal(1).json`)

Handles support chat conversations:

- Receives chat messages via an n8n chat trigger
- A "Query Enricher" code node detects Agilent instrument model numbers and product names (e.g. 1260, 7890B, ZORBAX) and appends them to the query to improve retrieval
- A LangChain AI Agent (OpenAI `o4-mini`) routes the enriched query to the right Pinecone namespace:
  - LC/HPLC questions → `lc` namespace
  - GC questions → `gc` namespace
  - Columns/consumables questions → `consumables` namespace
- Maintains short-term conversation memory (last 10 messages per session)
- Always cites the source title and URL in its answer, and points users to Agilent Community Support if nothing relevant is found

## Requirements

- An [n8n](https://n8n.io/) instance (self-hosted or cloud) with the LangChain nodes (`@n8n/n8n-nodes-langchain`) enabled
- An OpenAI API credential
- A Pinecone API credential, with an index named `agilent-support-agent`

## Setup

1. Import both workflow JSON files into your n8n instance (Workflows → Import from File)
2. Reconnect the OpenAI and Pinecone credentials to your own accounts
3. Run **Support Ingestion** once to populate the Pinecone index (it can also run on a weekly schedule)
4. Activate **AI Agent** and use its chat trigger to start asking support questions

## Project structure

```
support-rag-agent/
└── workflows/
    ├── Support Ingestion Final(1).json   # Knowledge base ingestion pipeline
    └── AIAgentFinal(1).json               # Customer support chat agent
```
