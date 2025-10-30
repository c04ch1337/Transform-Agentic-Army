Source: Deepseek

Of course! Reverse-engineering Relevance AI's tech stack is a fascinating project. They are at the intersection of several cutting-edge technologies. Based on their public information, product behavior, job postings, and the nature of their service, we can make highly educated guesses about their architecture.

Here is a detailed breakdown and reverse-engineering plan for Relevance AI.

Executive Summary Relevance AI is a "Agent-as-a-Service" platform. Their core product is a no-code/low-code builder for creating multi-agent workflows that leverage various AI models (like GPT-4, Claude, etc.) and tools (like web search, code execution, data extraction).

Their stack is a modern, cloud-native JAMstack application for the frontend, with a robust, event-driven microservices backend built on Python (likely FastAPI) and Node.js, all hosted on a major cloud provider (AWS/GCP). The "secret sauce" lies in their agent orchestration engine and their proprietary low-code workflow definition language.

1. The Hypothesized Tech Stack Frontend & Client-Side Core Framework: React (highly likely, given modern web trends and their UI components).

Language: TypeScript (essential for a complex, data-heavy application to ensure type safety).

Styling & UI Library: They could use Tailwind CSS for utility-first styling or a custom component library. Their clean, modern interface suggests a well-designed system.

State Management: Zustand or Redux Toolkit for managing complex application state (like workflow steps, agent configurations, and run histories).

Frontend Framework: Possibly Next.js or Remix for server-side rendering (SSR) to improve initial load times and SEO for their marketing pages.

Graphs & Diagrams: A key part of their UI is the visual workflow builder. This likely uses a library like React Flow or X6 to render the node-based agent graphs.

Backend & API Layer Core Language: Python. This is almost a certainty. The entire AI/ML ecosystem (LangChain, LlamaIndex, model SDKs) is Python-native. Their public Python SDK confirms this.

Web Framework: FastAPI (ideal for async operations and auto-generated OpenAPI docs) or Django/Django REST Framework (for a more "batteries-included" approach). FastAPI is a strong candidate for its performance and modern async capabilities.

Secondary Language: Node.js (with TypeScript). Likely used for specific services like real-time features (WebSockets), background jobs, or the BFF (Backend For Frontend) layer.

API Design: RESTful API for core CRUD operations (managing projects, agents, datasets). They also heavily use Server-Sent Events (SSE) or WebSockets to stream the progress and outputs of running AI agents back to the UI in real-time.

AI & Machine Learning Core This is their most critical and proprietary layer.

Orchestration Framework: They are likely built on top of, or inspired by, frameworks like LangChain and LlamaIndex. However, they have almost certainly created a significant abstraction layer to provide their unique "workflow" and "agent" semantics.

Agent Engine: A custom Python service that parses a workflow definition (a JSON/YAML-like structure), manages the state of execution, routes data between agents, and handles conditional logic.

Model Abstraction: A core service that provides a unified interface to various LLMs (OpenAI GPT-4, Anthropic Claude, Cohere, open-source models via Replicate or Together AI). This handles API keys, rate limiting, retries, and cost tracking.

Tooling System: A framework that allows agents to use "tools." These are likely defined as Python functions with a standard signature. Examples include:

web_search(query)

execute_python_code(code_string)

extract_data_from_url(url, schema)

query_vector_database(collection_id, query)

Data & Infrastructure Cloud Provider: AWS (most common) or Google Cloud Platform (GCP).

Database:

PostgreSQL: The primary transactional database. Stores user data, project metadata, agent definitions, workflow configurations, and execution histories.

Redis: Used for caching, session storage, and as a message broker (with Celery or similar) for background task queues.

Vector Database: Essential for their "Indexing" and "Search" features. Strong candidates are Pinecone, Weaviate, or Qdrant. This stores embeddings for users' documents to enable semantic search within agents.

Job Queue: Celery (Python) or Bull (Node.js) for managing long-running agent workflows. When you click "Run," a job is enqueued and processed by a worker fleet.

Object Storage: AWS S3 or GCP Cloud Storage for storing user-uploaded files (PDFs, DOCs, etc.) and generated outputs.

Containerization: Docker. The entire application is likely running as a set of containerized microservices.

Orchestration: Kubernetes (k8s) on AWS (EKS) or GCP (GKE) to manage, scale, and deploy their diverse services (API, Agent Runner, Model Gateway, etc.).

2. Reverse-Engineering Methodology & Tools Here's a step-by-step plan to validate the hypotheses above and dig deeper.

Step 1: Passive Reconnaissance View Source & Network Tab: The most direct method.

Go to [https://relevance.ai/](https://relevance.ai/) and their web app.

Open Chrome DevTools.

Network Tab: Reload the page and interact with the app (create an agent, run a workflow). Look for:

JavaScript file names: chunk-xxx.js. Sometimes source maps are available, revealing the original source code structure and libraries used.

API Endpoints: Look for URLs like /api/v1/agents, /api/v1/workflows/run. The structure can hint at the backend framework (REST, GraphQL).

Key Insight: Watch for requests with a text/event-stream type. This is the SSE connection streaming agent thoughts and actions in real-time.

Check their SDKs:

Their Python SDK (relevanceai) on PyPI or GitHub is a goldmine. The code will show you how they structure API calls, the data models for agents and workflows, and what underlying services they might be using.

Job Postings:

Search for "Relevance AI careers" or on LinkedIn. Job descriptions for "Full-Stack Engineer," "Backend Engineer," or "ML Engineer" will explicitly list their stack (e.g., "We use React, TypeScript, FastAPI, Kubernetes on GCP...").

Step 2: Active Probing (Be Ethical!) Use the Product: Sign up for a free account. Create a simple workflow (e.g., an agent that uses web search). The behavior gives clues.

How fast does it start? (Indicates job queuing).

Does it stream output token-by-token or step-by-step? (Indicates different levels of orchestration).

Inspect API Responses: Headers can reveal the backend server (server: uvicorn for FastAPI, server: nginx is just the proxy). Response structures can hint at the serialization library.

Analyze their SDK: Install their Python package and inspect it.

bash

Copy

Download pip install relevanceai cd ~/path/to/python/site-packages/relevanceai

# Look at the source code structure, **init**.py files, and main modules.

Step 3: Architectural Deduction Based on the clues, you can piece together the flow of a "Workflow Run":

UI (React App): User drags and drops agents in the visual builder (React Flow). The state is a JSON workflow definition.

API Call (FastAPI/Node.js): On "Run," the frontend sends this JSON to POST /api/v1/workflows/run.

Job Queue (Redis/Celery): The API endpoint validates the payload and places a new job in a queue.

Agent Runner (Python Microservice): A Celery worker picks up the job. This is the core "brain."

It parses the workflow JSON.

It instantiates the first agent, calls the defined LLM with the provided prompt and context.

If the agent uses a tool (e.g., web_search), the runner calls the respective tool function.

It passes the result to the next agent in the workflow, following the graph logic.

Throughout this process, it emits events (e.g., agent_started, tool_used, new_token) back to the backend via a pub/sub system.

Real-Time Updates (SSE/WebSockets): The backend pushes these events to the user's browser, which updates the UI in real-time.

Data Persistence (PostgreSQL): The final result and the entire execution trace are saved to the database for the user to review later.

3. How to Build a Basic Clone (Your Own "Mini Relevance AI") To truly understand it, try building a simplified version.

Tech Stack for the Clone:

Frontend: Next.js (React) with TypeScript and React Flow.

Backend: FastAPI (Python).

AI Core: LangChain for a quick start on agent/tool logic.

Data: PostgreSQL for metadata, Redis for job queue (using Celery).

Real-Time: FastAPI's built-in support for Server-Sent Events (SSE).

Core Components to Build:

Workflow Builder UI: A React app with React Flow that allows dragging "Agent Nodes" and connecting them. It outputs a JSON definition.

Backend API (FastAPI):

POST /workflows: Save a workflow JSON.

POST /workflows/{id}/run: Accepts a workflow ID, validates it, and queues a Celery task. Returns a run ID.

GET /runs/{run_id}/stream: An SSE endpoint to stream events for a specific run.

Agent Runner (Celery Task):

This task takes the workflow JSON.

It uses LangChain to create a custom agent with access to tools (you can start with a simple PythonREPLTool or DuckDuckGoSearchRun).

As the LangChain agent executes, the runner publishes events to a Redis channel specific to the run_id.

Event Streamer: The SSE endpoint (/stream) subscribes to the Redis channel for the run_id and forwards events to the client.

This project would teach you 80% of the core challenges Relevance AI has solved: stateful workflow execution, tool abstraction, model interoperability, and real-time user feedback.

Conclusion Relevance AI's stack is a sophisticated blend of modern web development and AI engineering. The true complexity isn't in any single technology, but in the seamless integration of them all to create a reliable, scalable, and user-friendly platform for orchestrating AI agents.

By following the reverse-engineering plan and attempting a basic clone, you will gain a deep, practical understanding of how such a system is built from the ground up.