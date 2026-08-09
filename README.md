# OrbitDesk Support Agent

An AI-powered customer support agent built using LangGraph, RAG, Hugging Face Transformers, and semantic search. The system classifies customer queries, retrieves relevant information from the OrbitDesk knowledge base, generates grounded responses, verifies them, and safely handles clarification and escalation cases.

## Overview

The OrbitDesk Support Agent is designed to provide reliable and safe customer support for the OrbitDesk workspace platform.

The agent follows a structured workflow:

User Query
    ↓
Triage
    ↓
Retrieval
    ↓
Generation
    ↓
Verification
    ↓
Revision (if required)
    ↓
Final Response

The system uses retrieved evidence from the knowledge base and resolved support cases while applying verification rules to reduce unsupported or hallucinated responses.

## Key Features

- Query classification and routing
- Retrieval-Augmented Generation (RAG)
- Semantic similarity search
- Local LLM-based response generation
- Knowledge-base grounding
- Resolved support case integration
- Response verification
- Automatic response revision
- Safe failure handling
- Human escalation
- Structured JSON output
- LangGraph-based workflow orchestration

## Query Classification

The agent classifies incoming queries into the following categories:

### Answerable

Used when the request is related to OrbitDesk and can be answered using the available knowledge base.

### Requires Clarification

Used when the user has not provided enough information to properly troubleshoot the issue.

### Requires Escalation

Used when the issue requires human intervention, account-specific investigation, or an action that is not documented in the knowledge base.

### Out of Scope

Used when the request is unrelated to OrbitDesk support.

### Safe Failure

Used when the system cannot safely provide a grounded answer.

## Retrieval-Augmented Generation

The system uses a RAG pipeline to retrieve relevant information before generating an answer.

The retrieval process is:

Knowledge Base
    ↓
Document Loading
    ↓
Text Chunking
    ↓
Embedding Generation
    ↓
Vector Representation
    ↓
User Query Embedding
    ↓
Similarity Search
    ↓
Relevant Context
    ↓
LLM Response Generation

The embedding model used is:

`all-MiniLM-L6-v2`

## Language Model

The project uses the following local Hugging Face instruction-following model:

`Qwen/Qwen2.5-1.5B-Instruct`

The model is used for tasks such as:

- Query classification
- Response generation
- Response verification
- Response revision

## Knowledge Base

The OrbitDesk knowledge base contains:

- `01_product_overview.md`
- `02_roles_and_permissions.md`
- `03_workspace_settings_and_timezones.md`
- `04_scheduled_exports.md`
- `05_api_credentials.md`
- `06_connections_and_refreshes.md`
- `07_delivery_destinations.md`
- `08_escalation_and_diagnostics.md`
- `09_audit_logs.md`
- `10_security_and_safe_responses.md`

These documents provide information about OrbitDesk product functionality, roles and permissions, workspace settings, scheduled exports, API credentials, connections, delivery destinations, diagnostics, audit logs, escalation, and security.

## LangGraph Workflow

The complete agent workflow is implemented using LangGraph.

```text
START
  │
  ▼
Triage
  │
  ├── requires_clarification ──┐
  ├── requires_escalation ─────┤
  ├── out_of_scope ────────────┤
  │                             ▼
  │                       Final Response
  │
  └── answerable
          │
          ▼
      Retrieval
          │
          ▼
      Generation
          │
          ▼
      Verification
          │
       ┌──┴──┐
       │     │
      PASS  FAIL
       │     │
       │     ▼
       │  Revision
       │     │
       │     └──────► Verification
       │
       ▼
 Final Response
       │
       ▼
      END
```

## Safety and Grounding

A major objective of this project is to prevent unsupported responses.

The agent should only make claims that are supported by:

1. User-provided information
2. Retrieved knowledge-base information
3. Validated workflow instructions
4. Appropriate historical evidence

Resolved cases are treated as historical evidence and are not automatically treated as facts about the current user.

For example, if an old support case contains a particular symptom, the agent must not automatically assume that the current user's account has the same symptom. It should use conditional and evidence-based language when appropriate.

## Response Verification

Every generated response goes through a verification stage.

The verifier checks whether the response:

- Is supported by retrieved evidence
- Correctly uses information provided by the user
- Avoids unsupported claims
- Avoids invented UI elements
- Avoids unsupported troubleshooting steps
- Does not incorrectly treat historical cases as current facts
- Follows OrbitDesk support guidelines

If the response fails verification, it is sent to the revision stage.

## Automatic Revision

When a response does not pass verification, the system automatically attempts to revise it.

The revised response is then verified again.

```text
Generation
    ↓
Verification
    ↓
Failure
    ↓
Revision
    ↓
Verification
    ↓
Pass
    ↓
Final Response
```

## Structured Output

The agent produces structured output using a JSON schema.

Example:

```json
{
  "classification": "answerable",
  "answer": "Support response...",
  "sources": [],
  "confidence": 0.9,
  "requires_human": false,
  "reason": "..."
}
```

Possible classifications include:

- `answerable`
- `requires_clarification`
- `requires_escalation`
- `out_of_scope`
- `safe_failure`

The final output is validated using JSON Schema.

## Testing

The project includes sample questions for testing different routing scenarios.

The routing tests verify that the system can distinguish between different types of support requests.

Example expected routing:

```text
Q1 → answerable
Q2 → answerable
Q3 → requires_clarification
Q4 → requires_escalation
Q5 → out_of_scope
```

A graph-routing test is also included to verify that queries are sent to the correct workflow paths.

## Monitoring

The notebook records information related to the local model and execution environment, including:

- Generation model
- Model revision
- Embedding model
- Embedding revision
- Device
- CPU information
- GPU information when available
- Generation latency

## Technologies Used

- Python
- LangGraph
- LangChain
- Hugging Face Transformers
- Sentence Transformers
- PyTorch
- JSON Schema
- Jupyter Notebook
- Google Colab
- Qwen 2.5
- all-MiniLM-L6-v2

## Project Structure

```text
.
├── 01_product_overview.md
├── 02_roles_and_permissions.md
├── 03_workspace_settings_and_timezones.md
├── 04_scheduled_exports.md
├── 05_api_credentials.md
├── 06_connections_and_refreshes.md
├── 07_delivery_destinations.md
├── 08_escalation_and_diagnostics.md
├── 09_audit_logs.md
├── 10_security_and_safe_responses.md
│
├── Assignment.ipynb
├── assignment_graph.png
├── output_schema.json
├── resolved_cases.json
├── sample_questions.json
└── README.md
```

## Installation

Install the required dependencies:

```bash
pip install -U langgraph langchain transformers accelerate sentence-transformers bitsandbytes jsonschema
```

## Running the Project

The complete implementation is available in:

`Assignment.ipynb`

The notebook can be opened using Google Colab or Jupyter Notebook.

Make sure the knowledge-base Markdown files and JSON files are available in the expected working directory before running the notebook.

## Example Workflow

Example user query:

```text
Can a read-only user create API credentials?
```

The agent processes the query as follows:

```text
User Query
    ↓
Triage
    ↓
Determine Query Type
    ↓
Retrieve Relevant Knowledge
    ↓
Generate Answer
    ↓
Verify Answer
    ↓
Return Structured Response
```

The response is generated using the retrieved OrbitDesk documentation rather than relying only on the language model's internal knowledge.

## Project Objectives

The main objectives of this project are:

- Build an AI-powered customer support agent
- Implement an agentic workflow using LangGraph
- Implement Retrieval-Augmented Generation
- Use semantic search for knowledge retrieval
- Generate grounded responses using a local LLM
- Implement query classification
- Implement response verification
- Automatically revise failed responses
- Handle unsupported queries safely
- Support human escalation
- Produce structured and validated outputs
- Reduce hallucination and unsupported claims

## Safe Response Philosophy

The agent follows an evidence-first approach.

```text
Do not guess
     ↓
Retrieve evidence
     ↓
Generate grounded response
     ↓
Verify response
     ↓
Revise if necessary
     ↓
Return safe answer
```

When sufficient evidence is not available, the system prefers clarification, escalation, or safe failure instead of inventing an answer.

## Files Included

### Assignment.ipynb
Main implementation notebook containing the agent workflow.

### assignment_graph.png
Visual representation of the LangGraph workflow.

### output_schema.json
Schema used to validate the final structured output.

### resolved_cases.json
Collection of resolved historical support cases used as additional evidence.

### sample_questions.json
Sample queries used for testing the agent.

### Knowledge Base Markdown Files
The Markdown files contain OrbitDesk product and support information used by the retrieval system.

## Author

**Aditya Garg**

AI / ML | Generative AI | Agentic AI | Data Science

## Project Summary

OrbitDesk Support Agent demonstrates a complete AI support-agent pipeline combining:

```text
LangGraph
   +
RAG
   +
Semantic Search
   +
Local LLM
   +
Response Verification
   +
Automatic Revision
   +
Safe Failure
   +
Human Escalation
```

The project focuses on building a reliable, grounded, and safe AI customer-support system rather than simply generating responses from an LLM.

## AI Assistance Disclosure

AI coding assistants were used during the development of this project for guidance, debugging, documentation, and code improvement. The submitted implementation was reviewed and understood by the author.
