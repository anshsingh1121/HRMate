# HRMate Project Documentation

## 1. Project Overview
HRMate is an automated HR assistant designed to handle email inquiries by leveraging Retrieval-Augmented Generation (RAG). It monitors an email inbox, processes incoming queries using an LLM (Large Language Model) augmented with internal policy documents, and sends automated, context-aware replies.

## 2. Technologies Used

### Core Technologies
*   **Programming Language:** Python
*   **LLM Provider:** OpenAI
*   **Vector Database:** Pinecone

### Key Libraries & Dependencies
*   **`openai`**: For interacting with OpenAI's language and embedding models.
*   **`pinecone-client`**: For vector storage and retrieval operations.
*   **`imaplib` & `smtplib`**: Standard Python libraries for email reception (IMAP) and transmission (SMTP).
*   **`email`**: For parsing email message formats.
*   **`python-dotenv`**: For managing environment variables (API keys, credentials).
*   **`requests` / `httpx`**: For HTTP requests (underlying dependencies).

### Models
*   **Embedding Model:** `text-embedding-3-large` (OpenAI) - Used to convert text chunks into 3072-dimensional vectors.
*   **LLM:** `gpt-4.1` (OpenAI) - Used for generating natural language responses based on retrieved context.

## 3. Workflow Pipeline

The project operates in two main phases: **Data Ingestion** (Setup) and **Query Processing** (Runtime).

### Phase 1: Data Ingestion (RAG Setup)
*File: `rag_runner.py`*

1.  **Load Document**: The system reads the HR policy document from `rag/doc/policy.txt`.
2.  **Chunking**: The text is split into smaller segments using a sliding window approach.
    *   **Chunk Size**: 1000 characters
    *   **Overlap**: 200 characters
3.  **Embedding Generation**: Each text chunk is passed to OpenAI's `text-embedding-3-large` model to generate a vector representation.
4.  **Upsert to Vector Store**: The generated vectors, along with their corresponding text metadata, are uploaded (upserted) to the Pinecone vector database.

### Phase 2: Query Processing (Runtime)
*File: `main.py` & `llm_runner.py`*

1.  **Email Polling**:
    *   The system continuously polls the configured email inbox (every 2 seconds) using `imaplib`.
    *   It filters for `UNSEEN` (unread) messages.
2.  **Email Parsing**:
    *   When a new email is found, the system extracts the sender's address, subject, and body (preferring plain text, falling back to HTML).
3.  **Retrieval (RAG)**:
    *   The email content (Subject + Body) is treated as the user query.
    *   The query is embedded using the same model (`text-embedding-3-large`).
    *   The system queries Pinecone for the top 10 most similar chunks (`top_k=10`).
4.  **Prompt Engineering**:
    *   A system prompt is loaded from `rag/doc/system_prompt.md`.
    *   A final prompt is constructed combining:
        *   `<user_query>`: The email content.
        *   `<policy_document>`: The aggregated text from the retrieved chunks.
        *   System Instructions.
5.  **Response Generation**:
    *   The constructed prompt is sent to the OpenAI LLM (`gpt-4.1`).
    *   The model generates a context-aware response.
6.  **Reply Dispatch**:
    *   The generated response is formatted into an email reply.
    *   The reply is sent back to the sender using `smtplib`.

## 4. File Structure Description

*   **`main.py`**: The main entry point. Handles email polling, parsing, and sending replies. Orchestrates the flow.
*   **`llm_runner.py`**: Bridge between the main application and the RAG module. Handles the logic of getting a response for a specific query.
*   **`rag_runner.py`**: Script for initializing the knowledge base. chunks documents and uploads them to Pinecone.
*   **`rag/`**: Package containing RAG implementation details.
    *   **`chunker.py`**: Logic for splitting text files into chunks.
    *   **`embbeding.py`**: Wrapper for OpenAI's embedding API.
    *   **`llm.py`**: Wrapper for OpenAI's completion API.
    *   **`vectorstore.py`**: Wrapper for Pinecone interactions (upsert, query).
    *   **`doc/`**: Directory for storing source documents (`policy.txt`) and prompts (`system_prompt.md`).
*   **`requirements.txt`**: List of Python project dependencies.
