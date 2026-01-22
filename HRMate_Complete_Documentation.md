# HRMate - Complete Project Documentation

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [What Does HRMate Do?](#2-what-does-hrmate-do)
3. [Technologies Used](#3-technologies-used)
4. [Project Structure](#4-project-structure)
5. [Detailed File Breakdown](#5-detailed-file-breakdown)
6. [Workflow Pipeline](#6-workflow-pipeline)
7. [How RAG (Retrieval-Augmented Generation) Works](#7-how-rag-works)
8. [Configuration & Environment Variables](#8-configuration--environment-variables)
9. [How to Run the Project](#9-how-to-run-the-project)
10. [Technical Deep Dive](#10-technical-deep-dive)

---

## 1. Project Overview

**HRMate** is an intelligent, automated HR (Human Resources) assistant that answers employee questions via email. Think of it as a smart email bot that knows all your company's HR policies and can respond to employee queries automatically.

### In Simple Words:
- Employees send emails asking questions like "How many sick days do I get?" or "What is the leave policy?"
- HRMate reads these emails automatically
- It looks up the answer in the company's HR policy document
- It writes a friendly, professional reply and sends it back to the employee

### Key Features:
- **Automatic email monitoring** - Checks for new emails every 2 seconds
- **Smart document search** - Uses AI to find relevant policy information
- **Natural language responses** - Replies like a real HR person would
- **Context-aware answers** - Only answers questions based on actual company policies

---

## 2. What Does HRMate Do?

### The Problem It Solves
HR departments often receive repetitive questions from employees:
- "How many vacation days do I get?"
- "What is the overtime policy?"
- "How do I apply for sick leave?"

Answering these questions manually takes time. HRMate automates this process.

### The Solution
HRMate uses a technology called **RAG (Retrieval-Augmented Generation)** which combines:
1. **Document Search** - Finding relevant information from HR policies
2. **AI Generation** - Creating human-like responses using the found information

---

## 3. Technologies Used

### Programming Language
| Technology | Version | Purpose |
|------------|---------|---------|
| Python | 3.x | Main programming language |

### Core Libraries & Their Purposes

#### For Email Handling
| Library | What It Does |
|---------|--------------|
| `imaplib` | Connects to email server to READ incoming emails |
| `smtplib` | Connects to email server to SEND replies |
| `email` | Parses email content (subject, body, sender address) |
| `email.mime` | Creates properly formatted email replies |

#### For AI & Machine Learning
| Library | What It Does |
|---------|--------------|
| `openai` | Connects to OpenAI's API for text generation and embeddings |
| `pinecone` | Connects to Pinecone vector database for similarity search |

#### For Configuration
| Library | What It Does |
|---------|--------------|
| `python-dotenv` | Loads sensitive information (API keys, passwords) from .env file |

### AI Models Used

| Model Name | Provider | Dimension | Purpose |
|------------|----------|-----------|---------|
| `text-embedding-3-large` | OpenAI | 3072 | Converts text into numbers (vectors) for similarity search |
| `gpt-4.1` | OpenAI | - | Generates human-like responses |

### External Services

| Service | Purpose |
|---------|---------|
| **Pinecone** | Cloud-based vector database to store and search document chunks |
| **OpenAI API** | AI service for embeddings and text generation |
| **Email Server (IMAP/SMTP)** | Your email service (e.g., Gmail, Outlook) |

---

## 4. Project Structure

```
HRMate/
├── main.py                 # Main entry point - Email polling bot
├── llm_runner.py           # Bridge between email handler and RAG system
├── rag_runner.py           # Script to set up the knowledge base
├── requirements.txt        # List of required Python packages
├── README.md               # Brief project description
├── .gitignore              # Files to exclude from Git
├── .env                    # Environment variables (API keys, passwords) - NOT in Git
│
└── rag/                    # RAG (Retrieval-Augmented Generation) module
    ├── chunker.py          # Splits documents into smaller pieces
    ├── embbeding.py        # Converts text to vectors using OpenAI
    ├── llm.py              # Generates responses using GPT-4.1
    ├── vectorstore.py      # Manages Pinecone vector database operations
    │
    └── doc/                # Document storage
        ├── policy.txt      # The HR policy document (knowledge base)
        └── system_prompt.md # Instructions for the AI on how to respond
```

---

## 5. Detailed File Breakdown

### 5.1 `main.py` - The Email Bot (Entry Point)

**Location:** Root folder  
**Lines of Code:** 124  
**Purpose:** This is the main file that runs the email polling bot.

#### What It Does Step by Step:

1. **Loads environment variables** - Reads email credentials and server settings from `.env` file
2. **Connects to email server** - Uses IMAP protocol to access the inbox
3. **Polls for new emails** - Checks every 2 seconds for unread emails
4. **Extracts email content** - Gets sender address, subject, and body
5. **Sends query to RAG system** - Passes the email content for processing
6. **Sends reply** - Emails back the AI-generated response

#### Key Constants:
```python
SMTP_PORT = 587              # Standard port for secure email sending
POLL_INTERVAL_SECONDS = 2    # How often to check for new emails
```

#### Key Functions:

| Function | Parameters | Returns | Purpose |
|----------|------------|---------|---------|
| `extract_body(msg)` | Email message object | String (email body) | Extracts text from email; prefers plain text, falls back to HTML |
| `check_and_reply_emails()` | None | None | Main loop that checks inbox and processes each new email |

#### How `extract_body()` Works:
1. Checks if email has multiple parts (multipart)
2. Skips attachments
3. Looks for plain text content first
4. If no plain text, looks for HTML content
5. Decodes and returns the text

---

### 5.2 `llm_runner.py` - The Bridge

**Location:** Root folder  
**Lines of Code:** 28  
**Purpose:** Connects the email handler to the RAG system. Acts as a bridge.

#### What It Does:

1. **Takes the query** - Receives the email content as a "query"
2. **Creates embedding** - Converts query text to vector numbers
3. **Searches vector database** - Finds similar document chunks in Pinecone
4. **Loads system prompt** - Reads instructions for the AI
5. **Builds final prompt** - Combines user query + retrieved documents + instructions
6. **Gets AI response** - Sends to GPT-4.1 and returns the answer

#### Key Function:

```python
def get_query_response(query):
    # Step 1: Convert query to vector
    vector = get_embeddings(query)
    
    # Step 2: Find similar documents
    text = query_vector(vector)
    
    # Step 3: Load AI instructions
    system_prompt = ... # from file
    
    # Step 4: Build final prompt
    Updated_query = f"<user_query>{query}</user_query>\n\n<policy_document>{text}</policy_document>"
    
    # Step 5: Get AI response
    response = get_response(Updated_query, system_prompt)
    return response
```

---

### 5.3 `rag_runner.py` - The Setup Script

**Location:** Root folder  
**Lines of Code:** 19  
**Purpose:** Sets up the knowledge base by processing the HR policy document.

#### When to Run:
- Run this **once** when setting up the project
- Run again if you update the policy document

#### What It Does:

1. **Reads policy document** - Opens `rag/doc/policy.txt`
2. **Splits into chunks** - Divides into 1000-character pieces with 200-character overlap
3. **Creates embeddings** - Converts each chunk to a vector
4. **Uploads to Pinecone** - Stores vectors with their original text

#### Chunking Parameters:
| Parameter | Value | Meaning |
|-----------|-------|---------|
| `chunk_size` | 1000 characters | Maximum size of each piece |
| `chunk_overlap` | 200 characters | Overlap between pieces to maintain context |

---

### 5.4 `rag/chunker.py` - The Text Splitter

**Location:** `rag/` folder  
**Lines of Code:** 19  
**Purpose:** Splits large documents into smaller, manageable pieces.

#### Why Chunking is Needed:
- AI models have token limits (can't process very long texts at once)
- Smaller chunks = more precise search results
- Overlap ensures context isn't lost at boundaries

#### Functions:

| Function | Parameters | Returns | Purpose |
|----------|------------|---------|---------|
| `open_file(filepath)` | File path | String | Reads entire file content |
| `chunk_file(filepath, chunk_size, chunk_overlap)` | Path, size, overlap | List of strings | Splits file into overlapping chunks |

#### How Chunking Works (Example):
If you have text: "ABCDEFGHIJ" with chunk_size=5 and overlap=2:
- Chunk 1: "ABCDE" (positions 0-4)
- Chunk 2: "DEFGH" (positions 3-7, overlaps with "DE")
- Chunk 3: "GHIJ" (positions 6-9, overlaps with "GH")

---

### 5.5 `rag/embbeding.py` - The Vectorizer

**Location:** `rag/` folder  
**Lines of Code:** 21  
**Purpose:** Converts text into numerical vectors using OpenAI's embedding model.

#### What is an Embedding?
An **embedding** is a list of numbers that represents the meaning of text. Similar texts have similar numbers.

Example (simplified):
- "vacation policy" → [0.23, 0.45, 0.12, ...]
- "leave rules" → [0.22, 0.44, 0.13, ...]  (similar numbers because similar meaning)
- "salary increase" → [0.89, 0.11, 0.67, ...]  (different numbers, different topic)

#### Functions:

| Function | Parameters | Returns | Purpose |
|----------|------------|---------|---------|
| `get_openai_client()` | None | OpenAI client | Creates connection to OpenAI API |
| `get_embeddings(text)` | String | List of 3072 floats | Converts text to vector |

#### Technical Details:
- **Model:** `text-embedding-3-large`
- **Output Dimensions:** 3072 numbers per text
- **Usage:** Both for indexing documents AND for searching

---

### 5.6 `rag/llm.py` - The Response Generator

**Location:** `rag/` folder  
**Lines of Code:** 20  
**Purpose:** Generates natural language responses using OpenAI's GPT-4.1 model.

#### Functions:

| Function | Parameters | Returns | Purpose |
|----------|------------|---------|---------|
| `get_openai_client()` | None | OpenAI client | Creates connection to OpenAI API |
| `get_response(query, system_prompt)` | Query string, Instructions | String response | Generates AI answer |

#### How It Calls the API:
```python
response = client.responses.create(
    model="gpt-4.1",           # The AI model to use
    input=query,               # The user's question + context
    instructions=system_prompt # How the AI should behave
)
return response.output_text
```

---

### 5.7 `rag/vectorstore.py` - The Database Manager

**Location:** `rag/` folder  
**Lines of Code:** 53  
**Purpose:** Manages all interactions with the Pinecone vector database.

#### What is Pinecone?
Pinecone is a cloud database specifically designed to store and search vectors (lists of numbers). It can quickly find similar vectors.

#### Functions:

| Function | Parameters | Returns | Purpose |
|----------|------------|---------|---------|
| `get_pinecone_client()` | None | Pinecone client | Creates connection to Pinecone |
| `upsert_chunk(id, values, text)` | ID, vector, original text | True | Stores a chunk in database |
| `query_vector(vector, top_k)` | Search vector, result count | String | Finds similar documents |

#### upsert_chunk Explained:
"Upsert" = Update OR Insert
- If the ID exists → Updates the data
- If the ID doesn't exist → Inserts new data

```python
index.upsert(
    vectors=[{
        "id": id,           # Unique identifier like "chunk_0"
        "values": values,   # The 3072-number vector
        "metadata": {"text": text}  # Original text for retrieval
    }]
)
```

#### query_vector Explained:
```python
response = index.query(
    vector=vector,          # What we're searching for
    top_k=10,              # Return 10 most similar results
    include_metadata=True   # Include the original text
)
```

---

### 5.8 `rag/doc/system_prompt.md` - AI Instructions

**Location:** `rag/doc/` folder  
**Lines of Code:** 82  
**Purpose:** Contains instructions that tell the AI how to behave.

#### Key Instructions:
1. **Identity:** "You are a friendly, helpful, and professional HR Assistant"
2. **Strict Adherence:** Only answer based on provided policy documents
3. **Tone:** Be courteous, approachable, and professional
4. **Format:** Output should be a complete email draft
5. **Handling Unknowns:** If answer isn't in policies, say so politely
6. **Persona:** Act as a human HR assistant, not as AI

#### Prompt Structure:
The system prompt includes few-shot examples showing:
- How to answer questions that ARE in the policy
- How to handle questions that are NOT in the policy

---

### 5.9 `rag/doc/policy.txt` - The Knowledge Base

**Location:** `rag/doc/` folder  
**Lines of Code:** 673  
**Size:** ~56KB  
**Purpose:** The company's Employee Handbook that the AI uses to answer questions.

#### Contents Include:
| Section | Topics Covered |
|---------|---------------|
| Section 1 | Introduction, Mission, Values, At-Will Employment |
| Section 2 | Equal Opportunity, Anti-Harassment, Accommodations |
| Section 3 | Employee Classifications, Work Schedules, Payroll |
| Section 4 | Code of Conduct, Attendance, Dress Code, Confidentiality |
| Section 5 | Health & Safety, Drug-Free Workplace, Violence Prevention |
| Section 6 | Compensation, Performance Reviews, Disciplinary Actions |
| Section 7 | Benefits (Health, Dental, Vision, 401k, Life Insurance) |
| Section 8 | Time Off (Holidays, Vacation, Sick Leave, FMLA, Parental Leave) |
| Section 9 | Technology, Remote Work Policy, Data Security |
| Section 10 | Travel & Expense Reimbursement |
| Section 11 | Separation from Employment |
| Section 12 | Acknowledgment Form |

---

### 5.10 `requirements.txt` - Python Dependencies

**Location:** Root folder  
**Purpose:** Lists all Python packages the project needs.

#### Key Dependencies:
| Package | Version | Purpose |
|---------|---------|---------|
| `openai` | 1.107.3 | OpenAI API client |
| `pinecone` | 7.3.0 | Pinecone vector database client |
| `python-dotenv` | 1.1.1 | Load environment variables from .env file |
| `httpx` | 0.28.1 | HTTP client (used by openai) |
| `pydantic` | 2.11.9 | Data validation (used by openai) |
| `grpcio` | 1.75.0 | gRPC client (used by pinecone) |

---

## 6. Workflow Pipeline

### Phase 1: Initial Setup (Run Once)

```
┌─────────────────┐
│  policy.txt     │  (HR Policy Document)
└────────┬────────┘
         │ Read file
         ▼
┌─────────────────┐
│  chunker.py     │  Split into 1000-char chunks
└────────┬────────┘
         │ 200-char overlap
         ▼
┌─────────────────┐
│  embbeding.py   │  Convert to vectors (3072 dimensions)
└────────┬────────┘
         │ OpenAI API
         ▼
┌─────────────────┐
│  vectorstore.py │  Store in Pinecone
└─────────────────┘
```

### Phase 2: Runtime (Continuous Loop)

```
┌─────────────────┐
│  Incoming Email │
└────────┬────────┘
         │ Check every 2 seconds
         ▼
┌─────────────────┐
│    main.py      │  Extract sender, subject, body
└────────┬────────┘
         │ Pass query
         ▼
┌─────────────────┐
│  llm_runner.py  │  Bridge to RAG system
└────────┬────────┘
         │
         ├──────────────────┐
         │                  ▼
         │         ┌─────────────────┐
         │         │  embbeding.py   │  Convert query to vector
         │         └────────┬────────┘
         │                  │
         │                  ▼
         │         ┌─────────────────┐
         │         │  vectorstore.py │  Search for similar chunks
         │         └────────┬────────┘
         │                  │ top 10 results
         ▼                  ▼
┌─────────────────────────────────────┐
│           Build Prompt              │
│  User Query + Retrieved Docs +      │
│  System Instructions                │
└────────────────┬────────────────────┘
                 │
                 ▼
        ┌─────────────────┐
        │     llm.py      │  Generate response with GPT-4.1
        └────────┬────────┘
                 │
                 ▼
        ┌─────────────────┐
        │    main.py      │  Send email reply
        └─────────────────┘
```

---

## 7. How RAG Works

### What is RAG (Retrieval-Augmented Generation)?

RAG is a technique that makes AI answers more accurate and relevant by:
1. **Retrieving** relevant information from a knowledge base
2. **Augmenting** the AI prompt with this information
3. **Generating** a response based on the retrieved context

### Why Use RAG?

| Without RAG | With RAG |
|-------------|----------|
| AI makes up answers ("hallucination") | AI answers based on actual documents |
| Generic, possibly incorrect responses | Specific, accurate responses |
| No source verification | Answers traceable to policy documents |

### The RAG Process in HRMate:

#### Step 1: Document Preparation (One-time)
```
HR Policy Document (56KB)
        ↓
Split into ~70+ chunks (1000 chars each)
        ↓
Each chunk converted to 3072-dimension vector
        ↓
Vectors stored in Pinecone with original text
```

#### Step 2: Query Processing (Every email)
```
Employee Question: "How many sick days do I get?"
        ↓
Convert to vector using same embedding model
        ↓
Search Pinecone for similar vectors
        ↓
Return top 10 most relevant policy sections
        ↓
Build prompt: Question + Relevant Sections + Instructions
        ↓
GPT-4.1 generates answer based on real policy
        ↓
Send as email reply
```

### Vector Similarity Explained

Think of vectors as coordinates in space:
- Each word/phrase has a "location" in 3072-dimensional space
- Similar meanings = Close locations
- Different meanings = Far locations

When searching:
1. Convert query to a point in this space
2. Find the closest stored points
3. Those are your most relevant documents

---

## 8. Configuration & Environment Variables

### Required Environment Variables (in `.env` file)

| Variable | Description | Example |
|----------|-------------|---------|
| `EMAIL_USER` | Your email address | `hrbot@company.com` |
| `EMAIL_PASS` | Email password or app password | `your-app-password` |
| `IMAP_SERVER` | IMAP server address | `imap.gmail.com` |
| `SMTP_SERVER` | SMTP server address | `smtp.gmail.com` |
| `OPENAI_API_KEY` | Your OpenAI API key | `sk-...` |
| `PINECONE_API_KEY` | Your Pinecone API key | `pcsk_...` |
| `PINECONE_INDEX_HOST` | Your Pinecone index URL | `https://xxx.pinecone.io` |

### Example `.env` File:
```env
EMAIL_USER=hrbot@yourcompany.com
EMAIL_PASS=your-app-specific-password
IMAP_SERVER=imap.gmail.com
SMTP_SERVER=smtp.gmail.com
OPENAI_API_KEY=sk-your-openai-api-key-here
PINECONE_API_KEY=pcsk_your-pinecone-key-here
PINECONE_INDEX_HOST=https://your-index-name.pinecone.io
```

> **Note:** The `.env` file is in `.gitignore` and should NEVER be committed to version control.

---

## 9. How to Run the Project

### Prerequisites:
1. Python 3.x installed
2. OpenAI API account with API key
3. Pinecone account with an index created
4. Email account with IMAP/SMTP access

### Step 1: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 2: Configure Environment
Create a `.env` file with all required variables (see Section 8).

### Step 3: Set Up Knowledge Base (Run Once)
```bash
python rag_runner.py
```
This will:
- Read the policy document
- Chunk it into pieces
- Create embeddings
- Upload to Pinecone

### Step 4: Start the Email Bot
```bash
python main.py
```
This will:
- Start polling your inbox every 2 seconds
- Process new emails automatically
- Send AI-generated replies

### Stopping the Bot:
Press `Ctrl+C` in the terminal.

---

## 10. Technical Deep Dive

### Email Processing Flow

```python
# Simplified flow in main.py

while True:  # Infinite loop
    # 1. Connect to email server
    mail = imaplib.IMAP4_SSL(IMAP_SERVER)
    mail.login(EMAIL_USER, EMAIL_PASS)
    mail.select("inbox")
    
    # 2. Search for unread emails
    status, messages = mail.search(None, "UNSEEN")
    
    # 3. Process each email
    for email_id in email_ids:
        # Get email content
        msg = fetch_and_parse(email_id)
        sender = msg["from"]
        subject = msg["subject"]
        body = extract_body(msg)
        
        # 4. Get AI response
        query = f"Subject: {subject}, body:{body}"
        response = get_query_response(query)
        
        # 5. Send reply
        send_email(sender, f"Re: {subject}", response)
    
    # 6. Wait before next check
    time.sleep(2)
```

### Embedding Pipeline

```python
# How text becomes searchable vectors

text = "What is the sick leave policy?"
         ↓
# API Call to OpenAI
response = openai_client.embeddings.create(
    input=text,
    model="text-embedding-3-large"
)
         ↓
# Result: List of 3072 numbers
vector = [0.023, -0.891, 0.456, ..., 0.123]  # 3072 values
```

### Prompt Construction

```python
# Final prompt sent to GPT-4.1

Updated_query = """
<user_query>
Subject: Sick Leave Question, body: How many sick days do I get per year?
</user_query>

<policy_document>
[Retrieved chunk 1]: Section 8.3 Sick Leave - Full-time employees accrue sick leave at a rate of...
[Retrieved chunk 2]: ...for absences of three or more consecutive days, a doctor's note may be required...
[... more relevant chunks ...]
</policy_document>
"""

# Combined with system_prompt which instructs:
# - Be friendly and professional
# - Only answer from provided documents
# - Format as complete email draft
```

### Error Handling

The main loop includes try-catch for error handling:
```python
try:
    check_and_reply_emails()
except Exception as e:
    print(f"An error occurred: {e}")
```

This ensures the bot continues running even if individual emails fail to process.

---

## Summary

**HRMate** is a complete AI-powered HR assistant that:

1. **Monitors** an email inbox automatically
2. **Understands** employee questions using AI embeddings
3. **Searches** relevant policy documents using vector similarity
4. **Generates** professional, accurate email responses
5. **Responds** automatically to employee queries

The key technologies working together:
- **Python** for the application logic
- **OpenAI** for understanding text and generating responses
- **Pinecone** for fast document search
- **IMAP/SMTP** for email communication

This creates a seamless, automated HR query resolution system that scales to handle any volume of employee questions.

---

*Documentation generated on: December 30, 2025*  
*Project: HRMate v1.0*
