# ChatBot Agent

A production-oriented AI chatbot agent built with **FastAPI**, **Google GenAI SDK (`google-genai`)**, **Pydantic**, and a responsive web UI.

The project demonstrates practical AI engineering concepts including prompt engineering, session memory, Gemini function/tool calling, REST APIs, and real-time response streaming with Server-Sent Events (SSE).

## Features

- **Modular architecture** following SOLID and DRY principles
- **Prompt engineering**
  - Structured XML prompts
  - Few-shot prompting
  - Chain-of-thought prompting
  - Standard baseline prompting
- **Session memory**
  - In-memory conversation management
  - Sliding window limited to `MAX_TURNS = 10`
- **Tool calling**
  - Current date/time lookup
  - Safe mathematical expression evaluation
  - Campus FAQ lookup
- **Dual response modes**
  - Standard JSON responses
  - Real-time token streaming using SSE
- **Interactive web UI**
- **FastAPI Swagger/OpenAPI documentation**
- **Configurable Gemini model and generation parameters**

## Tech Stack

| Category | Technology |
|---|---|
| Backend | FastAPI |
| AI Model Integration | Google GenAI SDK (`google-genai`) |
| Validation & Configuration | Pydantic |
| AI Model | Gemini |
| Frontend | HTML, CSS, JavaScript |
| API | REST |
| Streaming | Server-Sent Events (SSE) |
| Environment Configuration | `.env` |

## Project Structure

```text
chatbot_agent/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── chat.py
│   ├── prompts/
│   │   ├── __init__.py
│   │   ├── system_prompts.py
│   │   └── campus_faqs.py
│   ├── tools/
│   │   ├── __init__.py
│   │   └── campus_tools.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── memory_service.py
│   │   └── agent_service.py
│   └── api/
│       ├── __init__.py
│       └── chat_router.py
├── static/
│   ├── index.html
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── app.js
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```

## Getting Started

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd chatbot_agent
```

### 2. Create a virtual environment

#### Windows

```powershell
python -m venv venv
venv\Scripts\activate
```

#### macOS / Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file from `.env.example`:

#### Windows PowerShell

```powershell
Copy-Item .env.example .env
```

#### macOS / Linux

```bash
cp .env.example .env
```

Add your Google AI Studio Gemini API key:

```env
GEMINI_API_KEY=your_google_ai_studio_api_key_here
GEMINI_MODEL=gemini-3.1-flash-lite

APP_ENV=development
HOST=0.0.0.0
PORT=8000
```

> **Security:** Never commit your `.env` file or API keys to GitHub. Keep secrets in environment variables or a secret-management service.

## Running the Application

Start the FastAPI development server:

```bash
uvicorn app.main:app --reload --port 8000
```

The application will be available at:

- **Web UI:** `http://localhost:8000`
- **Swagger API Docs:** `http://localhost:8000/docs`
- **Health Check:** `http://localhost:8000/health`

## Prompt Engineering

The project demonstrates four prompt styles implemented in `app/prompts/system_prompts.py`.

### Structured XML Prompt

Organizes instructions using sections such as:

```text
<role>
<scope>
<tone>
<instructions>
<rules>
```

### Few-Shot Prompt

Uses multi-turn examples to guide the model toward the desired response format and tone.

### Chain-of-Thought Prompt

Instructs the model to analyze the user's intent and determine whether tools are required before producing an answer.

### Baseline Prompt

Provides a simple role definition that can be used as a baseline for comparison.

## Session Memory

Conversation history is managed by:

```text
app/services/memory_service.py
```

The service maintains sessions using an in-memory structure:

```text
dict[session_id, list[dict]]
```

A sliding window of:

```text
MAX_TURNS = 10
```

limits the amount of conversation history retained for each session.

This keeps the implementation simple and avoids requiring an external database for session memory.

## Tool Calling

The agent can automatically execute Python tools through the Gemini SDK.

### Available Tools

#### Current Date & Time

```text
get_current_datetime()
```

Returns the current system day, date, and time.

#### Calculator

```text
calculate(expression)
```

Evaluates supported mathematical expressions using a safe AST-based evaluator.

#### Campus FAQ Lookup

```text
lookup_faq(topic)
```

Searches the campus FAQ knowledge base located in:

```text
app/prompts/campus_faqs.py
```

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/chat` | Main chatbot endpoint |
| `POST` | `/api/chat/stream` | Real-time SSE streaming endpoint |
| `GET` | `/api/sessions` | Lists active sessions |
| `GET` | `/api/chat/{session_id}` | Retrieves session history |
| `DELETE` | `/api/chat/{session_id}` | Clears a session |
| `GET` | `/api/prompts` | Lists available prompt styles |
| `GET` | `/health` | Application health check |

### Chat Request

Example:

```json
{
  "session_id": "demo1",
  "message": "What are the hostel fees?",
  "prompt_style": "structured_xml"
}
```

## Testing with cURL

### 1. Scope Enforcement

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"session_id":"demo1","message":"What is the capital of France?","prompt_style":"structured_xml"}'
```

### 2. Session Memory

First request:

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"session_id":"student_session_1","message":"My name is Asha and I study Electrical Engineering.","prompt_style":"structured_xml"}'
```

Second request:

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"session_id":"student_session_1","message":"What is my name and branch?","prompt_style":"structured_xml"}'
```

### 3. Tool Calling

Current date/time:

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"session_id":"demo1","message":"What time is it right now?","prompt_style":"structured_xml"}'
```

Calculator:

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"session_id":"demo1","message":"Calculate 2000 * 0.15 for me.","prompt_style":"structured_xml"}'
```

FAQ lookup:

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"session_id":"demo1","message":"What are the hostel fees?","prompt_style":"structured_xml"}'
```

### 4. Streaming Responses

```bash
curl -N -X POST http://localhost:8000/api/chat/stream \
  -H "Content-Type: application/json" \
  -d '{"session_id":"stream_demo","message":"Tell me 3 interesting facts about DY Patil University campus.","prompt_style":"structured_xml"}'
```

## Model Configuration

Generation behavior can be configured through `.env`.

| Variable | Default | Purpose |
|---|---:|---|
| `MAX_OUTPUT_TOKENS` | `1024` | Maximum generated response length |
| `TEMPERATURE` | `0.7` | Controls randomness/creativity |
| `TOP_P` | Model default | Nucleus sampling |
| `TOP_K` | Model default | Limits candidate tokens |
| `THINKING_LEVEL` | `MINIMAL` | Controls Gemini reasoning depth |

### Factual & Deterministic

```env
TEMPERATURE=0.0
MAX_OUTPUT_TOKENS=512
THINKING_LEVEL=MINIMAL
```

Useful for factual responses, mathematics, and tool calling.

### Creative Assistance

```env
TEMPERATURE=0.9
TOP_P=0.95
TOP_K=40
MAX_OUTPUT_TOKENS=2048
THINKING_LEVEL=MEDIUM
```

Useful for brainstorming and creative writing.

### Deep Problem Solving

```env
TEMPERATURE=0.2
MAX_OUTPUT_TOKENS=2048
THINKING_LEVEL=HIGH
```

Useful for more complex analytical tasks.

## Architecture

The application separates responsibilities across different layers:

```text
Web UI
   │
   ▼
FastAPI Router
   │
   ▼
Agent Service
   │
   ├── Prompt Engineering
   │
   ├── Session Memory
   │
   ├── Gemini Model
   │
   └── Tool Calling
          ├── Date/Time
          ├── Calculator
          └── FAQ Lookup
```

This separation makes the project easier to maintain, test, and extend.

## Future Improvements

Possible extensions include:

- Persistent conversation storage
- Authentication and user management
- Database-backed memory
- RAG-based document retrieval
- Docker containerization
- Deployment to Google Cloud Run
- Secret Manager integration
- CI/CD with GitHub Actions or Cloud Build
- Monitoring and logging
- Automated evaluation and testing


