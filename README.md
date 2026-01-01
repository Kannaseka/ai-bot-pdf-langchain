# AI-Powered PDF Chatbot Using LangChain and LangGraph

This repository provides a flexible example of an AI chatbot that processes PDF files, saves their embeddings in a vector database (Supabase), and responds to user inquiries with OpenAI or other LLMs through LangChain and LangGraph.

**Chatbot Interface Preview:**

<img width="1096" alt="Screenshot 2025-02-20 at 05 39 55" src="https://github.com/user-attachments/assets/3a9ddea7-b718-476b-bdae-38839be20c12" />

## Table of Contents

1. [Features](#features)
2. [System Overview](#system-overview)
3. [Requirements](#requirements)
4. [Setup](#setup)
5. [Configuration](#configuration)
   - [Frontend Settings](#frontend-settings)
   - [Backend Settings](#backend-settings)
6. [Development Setup](#development-setup)
   - [Starting the Backend](#starting-the-backend)
   - [Starting the Frontend](#starting-the-frontend)
7. [How to Use](#how-to-use)
   - [PDF Upload and Processing](#pdf-upload-and-processing)
   - [Querying](#querying)
   - [Chat Logs](#chat-logs)
8. [Deployment](#deployment)
9. [Agent Modifications](#agent-modifications)
10. [Common Issues](#common-issues)
11. [Future Enhancements](#future-enhancements)

---

## Features

- **Document Processing Workflow**: Upload PDFs, convert them to Document objects, and embed vectors in a vector database (Supabase example).
- **Query Handling Workflow**: Manage user queries, determine if retrieval is needed, and provide answers with document citations.
- **Live Streaming**: Deliver responses in real-time from server to UI.
- **LangGraph Framework**: Use LangGraph's state machines for processing, visualization, and step-by-step debugging.
- **Next.js Interface**: Web UI for file uploads, live chat, built with React and Tailwind.

---

## System Overview

```ascii
┌─────────────────────┐    1. Upload PDFs    ┌───────────────────────────┐
│Frontend (Next.js)   │ ────────────────────> │Backend (LangGraph)       │
│ - React UI w/ chat  │                      │ - Ingestion Graph         │
│ - Upload .pdf files │ <────────────────────┤   + Vector embedding via  │
└─────────────────────┘    2. Confirmation   │     SupabaseVectorStore   │
(storing embeddings in DB)

┌─────────────────────┐    3. Ask questions  ┌───────────────────────────┐
│Frontend (Next.js)   │ ────────────────────> │Backend (LangGraph)       │
│ - Chat + SSE stream │                      │ - Retrieval Graph         │
│ - Display sources   │ <────────────────────┤   + Chat model (OpenAI)   │
└─────────────────────┘ 4. Streamed answers  └───────────────────────────┘

```
- **Supabase** acts as the vector store for document retrieval.
- **OpenAI** or compatible LLMs handle language tasks.
- **LangGraph** manages workflow steps for processing and responses.
- **Next.js** (React) handles the user interface for uploads and chat.

Components include:
- **Backend**: Node.js/TypeScript with LangGraph workflows:
  - **Ingestion** (`src/ingestion_graph.ts`) - manages document indexing
  - **Retrieval** (`src/retrieval_graph.ts`) - handles Q&A on documents
  - **Configuration** (`src/shared/configuration.ts`) - manages API settings for models and stores
- **Frontend**: Next.js app for PDF uploads and interactive chat.

---

## Requirements

1. **Node.js v18+** (v20 recommended).
2. **Yarn** (npm works, but Yarn is configured).
3. **Supabase account** (for vector storage; see [Supabase Setup](https://supabase.com/docs/guides/getting-started)).
   - Required: `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `documents` table, and `match_documents` function (refer to [LangChain Supabase guide](https://js.langchain.com/docs/integrations/vectorstores/supabase/)).
4. **OpenAI API Key** (or supported LLM provider via LangChain).
5. **LangChain API Key** (optional but useful for tracing; details at [LangSmith](https://docs.smith.langchain.com/administration/how_to_guides/organization_management/create_account_api_key)).

---

## Setup

1. **Download** the repository:

   ```bash
   git clone https://github.com/kannaseka/ai-bot-pdf-langchain.git
   cd ai-pdf-chatbot-langchain
   ```

2. Install packages (from root):

   yarn install

3. Set up environment variables in backend and frontend. Refer to `.env.example` files.

## Configuration

Environment variables handle keys and URLs. Each part has its own `.env.example`. Copy to `.env` and populate.

### Frontend Settings

In `frontend/.env`:

```
NEXT_PUBLIC_LANGGRAPH_API_URL=http://localhost:2024
LANGCHAIN_API_KEY=your-langsmith-api-key-here # Optional
LANGGRAPH_INGESTION_ASSISTANT_ID=ingestion_graph
LANGGRAPH_RETRIEVAL_ASSISTANT_ID=retrieval_graph
LANGCHAIN_TRACING_V2=true # Optional
LANGCHAIN_PROJECT="pdf-chatbot" # Optional
```

### Backend Settings

In `backend/.env`:

```
OPENAI_API_KEY=your-openai-api-key-here
SUPABASE_URL=your-supabase-url-here
SUPABASE_SERVICE_ROLE_KEY=your-supabase-service-role-key-here
LANGCHAIN_TRACING_V2=true # Optional
LANGCHAIN_PROJECT="pdf-chatbot" # Optional
```

**Variable Details:**

- `NEXT_PUBLIC_LANGGRAPH_API_URL`: Backend server URL (default: `http://localhost:2024`).
- `LANGCHAIN_API_KEY`: LangSmith key (optional for tracing).
- `LANGGRAPH_INGESTION_ASSISTANT_ID`: Ingestion assistant ID (default: `ingestion_graph`).
- `LANGGRAPH_RETRIEVAL_ASSISTANT_ID`: Retrieval assistant ID (default: `retrieval_graph`).
- `LANGCHAIN_TRACING_V2`: Enable LangSmith tracing (set to `true`).
- `LANGCHAIN_PROJECT`: LangSmith project name.
- `OPENAI_API_KEY`: OpenAI key.
- `SUPABASE_URL`: Supabase URL.
- `SUPABASE_SERVICE_ROLE_KEY`: Supabase key.

## Development Setup

Uses Turborepo for backend and frontend. Run separately.

### Starting the Backend

1. Go to backend:

   ```bash
   cd backend
   ```

2. Launch LangGraph dev server:

   ```bash
   yarn langgraph:dev
   ```

   Runs on port 2024, opens LangGraph UI. See [LangGraph Studio](https://langchain-ai.github.io/langgraph/concepts/langgraph_studio/).

### Starting the Frontend

1. Go to frontend:

   ```bash
   cd frontend
   ```

2. Run Next.js dev server:

   ```bash
   yarn dev
   ```

   Runs on port 3000. Visit http://localhost:3000.

## How to Use

With both running:

1. Use LangGraph Studio to test workflows.

2. Go to http://localhost:3000 for the chat UI.

3. Upload a small PDF via the bottom button. Triggers ingestion via `app/api/ingest`, storing embeddings in Supabase.

4. Ask questions in chat.

5. Retrieval via `app/api/chat` fetches docs and answers.

### PDF Upload and Processing

Click the paperclip in chat input.

Choose up to 5 PDFs, each <10MB (adjust in `app/api/ingest`).

Backend extracts text and embeds in vector store.

### Querying

Enter question in chat.

Answers stream live. If docs used, "View Sources" links appear.

### Chat Logs

Each session has a unique thread. Messages persist in session state.

Conversations aren't saved across sessions (extend for DB persistence). Documents persist in vector DB.

## Deployment

### Backend Deployment

Deploy to LangGraph Cloud ([guide](https://langchain-ai.github.io/langgraph/cloud/quick_start/?h=studio#deploy-to-langgraph-cloud)) or self-host ([guide](https://langchain-ai.github.io/langgraph/how-tos/deploy-self-hosted/)).

### Frontend Deployment

Deploy to Next.js-compatible hosts (Vercel, Netlify).

Set `NEXT_PUBLIC_LANGGRAPH_API_URL` to backend URL.

## Agent Modifications

### Backend

- Adjust defaults in `src/shared/configuration.ts` (vector store, k-value, filters). Use in graph nodes or pass from frontend.
- Edit prompts in `src/retrieval_graph/prompts.ts`.
- Change retriever in `src/shared/retrieval.ts` by adding a new function and updating `makeRetriever`.

### Frontend

- Update upload limits in `app/api/ingest`.
- Modify configs in `constants/graphConfigs.ts` (model, k, store).

## Common Issues

1. **Env Not Loaded**
   - Copy `.env.example` to `.env` in both folders.
   - Verify variables and restart servers.

2. **Supabase Issues**
   - Set up `documents` table and `match_documents` function. See LangChain docs.

3. **OpenAI Problems**
   - Check `OPENAI_API_KEY` and credits.

4. **LangGraph Errors**
   - Ensure Node >=18 and dependencies installed.

5. **Connection Issues**
   - Confirm `NEXT_PUBLIC_LANGGRAPH_API_URL` points to backend (default: http://localhost:2024).

## Future Enhancements

Contributions welcome via pull requests. Include docs and tests.

For more on LangChain and LangGraph chatbots, see [Learning LangChain (O'Reilly)](https://www.oreilly.com/library/view/learning-langchain/9781098167271/).
