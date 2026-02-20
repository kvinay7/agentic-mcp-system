# Agentic MCP System

A production-oriented agentic AI system that combines LLM orchestration (LangGraph) with deterministic enterprise tools exposed via an MCP (Model Context Protocol) layer.

This system enforces strict architectural boundaries between:

- Stateless HTTP gateway
- LLM orchestration layer
- Tool invocation boundary (MCP)
- Enterprise systems (databases & external APIs)

---

# High-Level Architecture

```text
Client
↓
Flask API (Stateless HTTP Host)
↓
LangGraph Orchestration Layer
↓
Master Agent (LLM Supervisor)
↓
Utility Agents (Fraud / Policy / Eligibility)
↓
MCP Client
↓
MCP Tool Servers
↓
Enterprise Systems (DB / APIs / Logs)
```

---

# System Goals

- Safely orchestrate multi-agent AI workflows
- Prevent LLM from directly accessing enterprise systems
- Enforce deterministic tool execution
- Provide reliable HTTP-compliant responses
- Protect downstream systems from overload
- Maintain observability across agent executions

---

# Core Components

## 1. API Layer — `app.py`

Acts as a stateless HTTP gateway.

### Responsibilities

- Accept client requests (`POST /api/agent/run`)
- Enforce `Authorization: Bearer <token>`
- Validate request payload
- Generate unique `requestId` for tracing
- Handle downstream failures
- Return structured JSON responses
- Structured logging

---

## 2. Orchestration Layer — `graph.py`

Defines agent execution flow using LangGraph.

### Responsibilities

- Manage multi-agent execution pipeline
- Control tool invocation sequence
- Prevent uncontrolled LLM behavior
- Ensure deterministic workflow transitions

Separates: HTTP concerns from AI reasoning logic 

---

## 3. Agents — `agents.py`

### Master Agent (LLM Supervisor)

- Interprets user request
- Decides which utility agent to invoke
- Determines whether tool invocation is required

### Utility Agents

- Fraud Agent
- Policy Agent
- Eligibility Agent

### LLM Risk Mitigation

- Agents cannot directly access databases
- All tool execution passes through MCP boundary

---

## 4. MCP Integration Layer — `mcp_client.py`

Acts as a secure HTTP client for tool invocation.

### Responsibilities

- Send structured tool requests to MCP server
- Apply timeout configuration
- Handle connection errors
- Map failures to appropriate HTTP responses

### Failure Mapping

| Scenario              | Upstream Response |
|----------------------|------------------|
| MCP unreachable       | 502 Bad Gateway |
| MCP timeout           | 504 Gateway Timeout |
| Tool validation error | 400 Bad Request |
| Tool internal error   | 500 Internal Server Error |

---

## 5. MCP Tool Server — `mcp_server.py`

Exposes deterministic tool endpoints.

Example: ```POST /tools/run```


### Responsibilities

- Validate tool invocation request
- Execute deterministic business logic
- Return structured, predictable output
- Prevent arbitrary LLM execution paths

---

## 6. Deterministic Tools — `tools.py`

Examples:

- Fraud Risk Engine
- Eligibility Rules Engine
- Policy RAG Server (Vector DB integration)

### Characteristics

- No randomness
- No direct LLM dependency
- Fully testable
- Safe execution
