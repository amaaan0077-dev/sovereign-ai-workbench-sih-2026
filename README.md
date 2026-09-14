# Sovereign On-Premise Agentic AI Workbench

**SIH 2026 | Open-Weight Models | On-Premise AI | Clearance-Aware RAG | Agentic Workflows**

> **Sovereign On-Premise Agentic AI Workbench using Open-Weight Multimodal LLMs for Confidential Industrial Work**

A self-hosted AI workbench for organisations that need modern AI assistance without moving operational or business data to public AI services.

The project is designed around a simple operational requirement:

**Industrial data stays inside the organisation's controlled environment.**

The workbench combines local language models, company knowledge retrieval, clearance-aware access control, sandboxed engineering calculations, audit logging and document generation behind a single operator console.

---

## 1. Why This Project Exists

Refineries, public-sector enterprises, defence-linked manufacturing units and government organisations produce large volumes of routine knowledge work that is not necessarily difficult, but is often too sensitive to send to a public AI service.

Typical examples include:

- Engineering calculations
- Equipment and inspection report analysis
- Approval notes
- Maintenance reports
- Internal correspondence
- SOP and manual lookup
- Production and operational information
- Vendor and procurement information
- Financial and business documents
- Piping & Instrumentation Diagrams (P&IDs)
- Engineering drawings and scanned documents
- Internal software and automation code
- Board and management presentations

In many organisations, this creates a difficult trade-off:

**Use cloud AI for productivity, or keep sensitive work completely inside the organisation.**

The Sovereign AI Workbench takes the second approach.

Instead of sending company information to an external AI provider, the AI stack is intended to run on infrastructure controlled by the organisation. The application can use smaller local models on ordinary development hardware and larger open-weight models when deployed on a suitable GPU server.

---

# 2. Project Objective

The objective is to build a practical, self-hosted alternative to cloud AI assistants for confidential industrial knowledge work.

The workbench is designed to provide:

1. **Local AI inference**
2. **Dynamic task-based model routing**
3. **Two security-aware knowledge lanes**
4. **Clearance-filtered local document retrieval**
5. **Agent-style multi-step task execution**
6. **Isolated engineering calculations**
7. **Word and Excel deliverable generation**
8. **Audit and provenance information**
9. **Runtime network telemetry**
10. **A control-room style operator interface**

The important distinction is that privacy is treated as an architectural property rather than only a prompt instruction.

---

# 3. High-Level Architecture

```text
                         ┌──────────────────────────────┐
                         │       OPERATOR / USER        │
                         │  Engineer / Officer / Admin  │
                         └──────────────┬───────────────┘
                                        │
                                        ▼
                  ┌──────────────────────────────────────────┐
                  │          INDUSTRIAL OPERATOR UI          │
                  │          React + Vite Dashboard          │
                  └───────────────────┬──────────────────────┘
                                      │ REST API
                                      ▼
                  ┌──────────────────────────────────────────┐
                  │             FASTAPI BACKEND              │
                  │                                          │
                  │  Query → Security → Routing → Agent      │
                  └───────────────┬───────────────┬──────────┘
                                  │               │
                   ┌──────────────┘               └──────────────┐
                   ▼                                             ▼
        ┌─────────────────────┐                       ┌─────────────────────┐
        │   QUERY ROUTER      │                       │   SECURITY LAYER    │
        │                     │                       │                     │
        │ Lane A / Lane B     │                       │ Internal L1         │
        │ Task classification │                       │ Confidential L2     │
        │ Model selection     │                       │ Secret L3           │
        └──────────┬──────────┘                       └──────────┬──────────┘
                   │                                             │
          ┌────────┴─────────┐                                   │
          ▼                  ▼                                   ▼
   ┌──────────────┐   ┌───────────────┐              ┌──────────────────────┐
   │ Local LLM    │   │ Local RAG     │              │ Clearance-Aware      │
   │ Provider     │   │ Vector Store  │◄─────────────┤ Pre-Retrieval Filter │
   │ Ollama/vLLM  │   │ Embeddings    │              └──────────────────────┘
   └──────┬───────┘   └───────────────┘
          │
          ▼
   ┌───────────────────────────────────────────────────────────────┐
   │                         AGENT WORKFLOW                         │
   │                                                               │
   │  Route → Ground → Research/BYPASS → Calculate → Generate     │
   │  → Verify → Audit                                             │
   └───────────────┬──────────────────────────┬────────────────────┘
                   │                          │
          ┌────────▼────────┐        ┌────────▼─────────┐
          │ Code Sandbox    │        │ Deliverable      │
          │                 │        │ Generators       │
          │ Restricted      │        │                  │
          │ Python          │        │ .docx / .xlsx    │
          └─────────────────┘        └──────────────────┘
                   │                          │
                   └────────────┬─────────────┘
                                ▼
                     ┌─────────────────────────┐
                     │ Audit + Network Monitor │
                     │ SQLite + Socket Telemetry│
                     └─────────────────────────┘
```

---

# 4. Core Design Principles

## 4.1 Data Sovereignty

The application is designed for deployment on an organisation-controlled workstation or GPU server.

The default development configuration uses a local Ollama endpoint:

```text
http://localhost:11434
```

The application does not require a cloud LLM API to generate its normal local responses.

---

## 4.2 Zero-Egress by Default

The project disables common telemetry settings at process startup:

```text
HF_HUB_DISABLE_TELEMETRY=1
ANONYMIZED_TELEMETRY=False
LANGCHAIN_TRACING_V2=false
DO_NOT_TRACK=1
TOKENIZERS_PARALLELISM=false
```

A runtime network monitor is also exposed through the backend.

The UI can display the current network state, including external packet information.

The project therefore does not rely only on a statement such as "the application is private"; it provides runtime evidence that can be inspected during a demonstration.

### Important deployment note

The application includes an optional public-web research component for environments where current public information is required. It is **disabled by default**:

```text
WEB_RESEARCH_ENABLED=false
```

For a strict air-gapped deployment, this setting should remain disabled and the host/network should be isolated according to the organisation's security policy.

The network monitor is an application-level verification mechanism; production deployments should additionally use firewall rules, VLAN/network isolation and infrastructure-level egress controls.

---

# 5. Dual-Lane Knowledge Architecture

One of the main design decisions is separating general knowledge from company knowledge.

## Lane A — General Knowledge

Used for questions that do not contain obvious company/private indicators.

Example:

```text
Explain the working principle of a centrifugal pump using Bernoulli's theorem.
```

The intended path is:

```text
User Query
    ↓
Lane A
    ↓
Local Model
    ↓
Response
```

The private document store is bypassed.

This prevents unnecessary company-document retrieval for normal knowledge questions.

---

## Lane B — Company / Confidential Work

Used when the request contains indicators of internal or private organisational information.

Example:

```text
Summarise the internal inspection report for Unit-4.
```

The intended path becomes:

```text
User Query
    ↓
Lane B
    ↓
Resolve User Clearance
    ↓
Apply Clearance Filter
    ↓
Search Only Authorised Documents
    ↓
Local Grounded Context
    ↓
Local Model
    ↓
Cited Response
```

The system does not depend on the language model remembering not to reveal a document.

The access decision is made in the retrieval layer.

---

# 6. Clearance-Aware RAG

The current implementation defines three clearance levels:

| Level | Clearance | Example |
|---:|---|---|
| 1 | INTERNAL | Floor engineers / internal operators |
| 2 | CONFIDENTIAL | Reliability officers / plant managers |
| 3 | SECRET | Senior executives / restricted operations |

Documents are tagged with a clearance tier.

During vector search, a document is excluded before it can become a returned match if:

```text
document clearance > user clearance
```

Conceptually:

```python
if document.clearance_tier > user_clearance:
    continue
```

Only authorised chunks proceed to semantic similarity matching and result ranking.

This is an important security difference from simply adding a sentence such as:

> "Do not reveal confidential information."

A prompt instruction can be ignored by a model.

A retrieval-layer access rule is enforced before the restricted document becomes part of the retrieved context.

---

# 7. Agentic Execution

The workbench is not intended to be only a chat interface.

The backend contains a multi-stage agent workflow that can combine several local capabilities for one request.

A typical execution can include:

```text
1. Resolve operator
2. Classify security lane
3. Classify task
4. Select local model
5. Retrieve authorised company context
6. Decide whether public research is required
7. Run engineering verification if required
8. Generate deliverables if requested
9. Produce final response
10. Record audit information
11. Inspect network state
```

The workflow returns an execution trace that can be displayed by the frontend.

This makes the system easier to demonstrate and easier to inspect than a black-box "prompt in / answer out" application.

---

# 8. Dynamic Model Routing

The backend maintains a model registry rather than hard-coding one model into every workflow component.

Current registry entries include:

| Model Role | Model | Purpose | Current Status |
|---|---|---|---|
| General | Qwen2.5 General | General knowledge and reasoning | Available |
| Coding | Qwen2.5-Coder | Coding and technical calculations | Available |
| Vision | Qwen2.5-VL | Image/document understanding | Registry entry / future deployment |
| Advanced reasoning | Qwen3 14B | Larger general reasoning model | Registry entry / future deployment |

The router first identifies the task and then selects the appropriate model role.

Examples:

```text
General question
        ↓
Qwen2.5 General

Coding / engineering calculation
        ↓
Qwen2.5-Coder

Image / scanned-document task
        ↓
Vision model route
```

The actual local model used by the current development machine is controlled through configuration.

For example:

```text
LLM_MODEL=qwen2.5:3b
```

A larger local model can be selected later without redesigning the complete application.

For a stronger deployment machine, the configuration can be changed to a model such as:

```text
LLM_MODEL=qwen3:14b
```

provided that the required model is installed and the available GPU/CPU resources are sufficient.

---

# 9. Task Classification

The router distinguishes between different kinds of requests.

Current task types include:

- `GENERAL_KNOWLEDGE`
- `CODE_SANDBOX`
- `VISION_DOCUMENT`
- `DELIVERABLE_DRAFTING`
- `INCIDENT_ANALYSIS`

The router also contains safeguards against overly broad keyword matching.

For example, the system avoids treating:

```text
photosynthesis
```

as an image request merely because it contains the characters:

```text
photo
```

Complete-word/phrase matching is used for these classifications.

The router also distinguishes between:

```text
"What is Python exception handling?"
```

and:

```text
"Run this Python code."
```

The first is a knowledge question.

The second is an execution request.

---

# 10. Local RAG and Document Grounding

The project contains a local vector-store implementation.

Current components:

```text
backend/app/rag/
├── embeddings.py
├── vector_store.py
└── sample_data.py
```

The vector store contains document chunks with metadata such as:

- Document ID
- Chunk ID
- Title
- Department
- Page
- Clearance tier
- Content
- Semantic embedding

The search process:

```text
User Query
    ↓
Create Query Embedding
    ↓
Check Document Clearance
    ↓
Exclude Unauthorised Chunks
    ↓
Calculate Semantic Similarity
    ↓
Apply Similarity Threshold
    ↓
Sort Matches
    ↓
Return Top-K Authorised Chunks
```

Returned chunks include provenance information such as:

```text
[DOCUMENT_ID, p.PAGE]
```

This provenance is carried into the response and can also be included in generated deliverables.

---

# 11. Engineering Calculation Sandbox

Engineering calculations should not depend entirely on language-model arithmetic.

The project therefore contains a restricted Python execution layer.

Location:

```text
backend/app/agent/tools/sandbox_code.py
```

The sandbox includes several controls:

- Separate subprocess execution
- Execution timeout
- Maximum source size
- Maximum output size
- Restricted imports
- AST validation
- Network-related names blocked
- File access blocked
- OS/process execution blocked
- Dynamic execution functions blocked
- Temporary isolated working directory
- Minimal execution environment

Allowed standard modules currently include:

```text
math
statistics
decimal
```

Examples of blocked capabilities include:

```text
socket
requests
subprocess
os
sys
open()
eval()
exec()
compile()
```

### Security boundary

This sandbox is a restricted Python execution layer, not a claim of complete operating-system isolation.

For production deployment, it should be combined with stronger infrastructure isolation such as containers, virtual machines, Windows Job Objects/AppContainer controls or equivalent enterprise sandboxing.

---

# 12. Engineering Verification Example

The current demonstration workflow contains an equipment safety calculation involving:

- Design pressure
- Crack depth
- Wall thickness
- MAWP derating
- Vibration measurements
- Alert thresholds

A representative calculation used by the demonstration is:

```text
MAWP_safe =
    P_design ×
    (1 - (crack_depth / wall_thickness × 1.5))
```

With the demonstration values:

```text
Design pressure = 18.5 bar
Crack depth     = 4.2 mm
Wall thickness  = 25.0 mm
```

the workflow verifies the calculation through the sandbox before incorporating the result into the generated deliverable.

The system also records the sandbox exit status and network status in the execution trace.

> The numerical example is demonstration data. Real plant engineering decisions must use validated equipment data, applicable engineering codes and approval by qualified personnel.

---

# 13. Deliverable Generation

The workbench is designed to turn an analysis into an actual work product rather than stopping at a chat response.

## Word Approval Notes

The project generates native Microsoft Word `.docx` files.

The current generator includes sections such as:

- Classification
- Reference number
- Date/time
- Originating officer
- Department
- Operational context
- Technical findings
- Engineering calculations
- Grounding/provenance citations
- Executive recommendation
- Approval/signature area

Implementation:

```text
backend/app/agent/tools/doc_gen.py
```

---

## Excel Analysis Reports

The project also contains an Excel report generator:

```text
backend/app/agent/tools/xlsx_gen.py
```

The workbook can contain:

- Summary
- Request metadata
- Clearance level
- System information
- Network status
- Executive summary
- Recommendation
- Technical findings
- Engineering calculations
- Citations/provenance

The output is a real `.xlsx` file rather than a formatted text response.

---

# 14. Audit Trail

The backend records execution information in a local SQLite audit database.

Location:

```text
backend/storage/audit/audit_trail.db
```

The audit layer is intended to provide traceability for:

- User identity
- Clearance tier
- Query
- Routing decision
- Selected model
- Tools used
- Retrieval activity
- Sandbox execution
- Deliverable generation
- Network state
- Execution details

The API exposes recent audit records through:

```text
GET /api/audit/logs
```

This is useful for demonstrations because a judge can see not only the final answer but also the path taken to produce it.

---

# 15. Network Monitoring

The network monitor is implemented in:

```text
backend/app/services/network_monitor.py
```

The backend exposes:

```text
GET /api/network/status
```

The frontend polls this endpoint and displays the current network/security state.

The purpose is to make the sovereignty claim observable.

During a demonstration, the operator can show:

```text
AIR-GAP INTEGRITY
VERIFIED ZERO-EGRESS

EXTERNAL PACKETS
0
```

The exact interpretation should always be understood as application/runtime telemetry, supplemented by operating-system and network controls in a production deployment.

---

# 16. Public Web Research: Optional and Disabled by Default

The codebase contains a public web research component:

```text
backend/app/agent/tools/web_research.py
```

It is controlled by:

```text
WEB_RESEARCH_ENABLED=false
```

The privacy filter checks a query before a web request can be made.

The filter looks for indicators such as:

- Confidential
- Secret
- Internal
- Classified
- Proprietary
- Inspection report
- Company report
- Internal document
- API keys/tokens
- Passwords
- Private file paths
- Other sensitive patterns

If sensitive indicators are found, public web research is blocked.

### Strict air-gapped mode

For the SIH sovereign deployment, keep:

```text
WEB_RESEARCH_ENABLED=false
```

This means the normal application path remains local.

The optional web component exists so that the architecture can support controlled public-information retrieval in a future deployment where the organisation explicitly permits it.

---

# 17. Multimodal Direction

The architecture includes a dedicated vision/document task category and a model registry entry for a vision-capable model.

The router can classify requests involving:

- Images
- Photos
- Scanned documents
- Drawings
- Diagrams
- Blueprints
- Charts
- Handwritten material

as:

```text
VISION_DOCUMENT
```

The current development setup keeps the vision model as a separate registry entry rather than requiring the small development model to handle every modality.

This allows a deployment to attach an appropriate local vision model when suitable GPU hardware and document-processing components are available.

The intended production workflow is:

```text
Scanned PDF / Image / Drawing
          ↓
Local OCR / Vision Model
          ↓
Extracted Information
          ↓
Clearance-Aware Grounding
          ↓
Agent Workflow
          ↓
Calculation / Analysis
          ↓
Word / Excel / Other Deliverable
```

---

# 18. Frontend

The frontend is built with:

- React
- Vite
- Lucide React

The interface is intentionally designed more like an industrial operator console than a generic AI chat page.

The current UI includes:

### Top Rail

Displays:

- Air-gap status
- Network telemetry
- Active operator
- Clearance selection

### Grounding Panel

Displays:

- Indexed documents
- Document clearance
- Local model registry
- Grounding information

### Workspace Panel

Provides:

- Operator session
- Chat/query input
- Evaluation presets
- Agent response feed
- Routing information
- Execution latency
- Grounded citations

### Audit / Provenance Panel

Displays:

- Agent execution trace
- Generated deliverables
- Network information
- Citations
- Verification status

### System Footer

Displays system/runtime information and network state.

The visual language uses an industrial control-console approach with a dark chassis, compact telemetry, monospace system information and clear security states.

---

# 19. Repository Structure

```text
sovereign-ai-workbench/
│
├── backend/
│   │
│   ├── app/
│   │   ├── core/
│   │   │   ├── config.py
│   │   │   ├── security.py
│   │   │   └── audit_logger.py
│   │   │
│   │   ├── models/
│   │   │   └── router.py
│   │   │
│   │   ├── rag/
│   │   │   ├── embeddings.py
│   │   │   ├── vector_store.py
│   │   │   └── sample_data.py
│   │   │
│   │   ├── agent/
│   │   │   ├── workflow.py
│   │   │   └── tools/
│   │   │       ├── doc_search.py
│   │   │       ├── sandbox_code.py
│   │   │       ├── doc_gen.py
│   │   │       ├── xlsx_gen.py
│   │   │       └── web_research.py
│   │   │
│   │   ├── services/
│   │   │   ├── llm_provider.py
│   │   │   ├── network_monitor.py
│   │   │   └── privacy_filter.py
│   │   │
│   │   └── api/
│   │       ├── schemas.py
│   │       └── routes.py
│   │
│   ├── storage/
│   │   ├── audit/
│   │   │   └── audit_trail.db
│   │   └── deliverables/
│   │
│   ├── requirements.txt
│   ├── run.py
│   └── test_demo.py
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── TopRail.jsx
│   │   │   ├── GroundingPanel.jsx
│   │   │   ├── WorkspacePanel.jsx
│   │   │   ├── PipelineView.jsx
│   │   │   ├── AuditProvenancePanel.jsx
│   │   │   └── SystemFooter.jsx
│   │   │
│   │   ├── services/
│   │   │   └── api.js
│   │   │
│   │   ├── App.jsx
│   │   ├── App.css
│   │   ├── index.css
│   │   └── main.jsx
│   │
│   ├── package.json
│   ├── vite.config.js
│   └── index.html
│
└── README.md
```

---

# 20. Technology Stack

## Backend

```text
Python
FastAPI
Pydantic
Uvicorn
NumPy
python-docx
openpyxl
SQLite
Requests
psutil
```

## Frontend

```text
React
Vite
Lucide React
JavaScript / JSX
CSS
```

## Local AI

```text
Ollama
Open-weight LLMs
Qwen model family
Optional vLLM-compatible local endpoint
```

## Local Knowledge

```text
Dense embeddings
In-process vector store
Clearance-aware retrieval
Document provenance
```

## Security / Runtime

```text
Process-level telemetry controls
Network monitoring
RBAC-style clearance tiers
Restricted Python sandbox
Local audit trail
Privacy filter
```

---

# 21. Configuration

The primary configuration is located at:

```text
backend/app/core/config.py
```

Important variables include:

```text
LLM_MODEL
OLLAMA_BASE_URL
OLLAMA_CHAT_MODEL
VLLM_BASE_URL
WEB_RESEARCH_ENABLED
BRAVE_SEARCH_API_KEY
WEB_SEARCH_URL
WEB_MAX_RESULTS
WEB_SEARCH_TIMEOUT
WEB_MIXED_QUERY_POLICY
```

For a strict local deployment:

```text
LLM_MODEL=qwen2.5:3b
WEB_RESEARCH_ENABLED=false
```

For a larger local model deployment, change `LLM_MODEL` to the locally installed model.

The application code is designed so that changing the local model does not require rewriting the agent architecture.

---

# 22. Installation

## Prerequisites

Recommended development environment:

- Windows 10/11 or Linux
- Python 3.10+
- Node.js
- npm
- Ollama
- A local model supported by the deployment machine

For larger models, use hardware with sufficient RAM/VRAM.

---

## Backend Setup

Open PowerShell:

```powershell
cd D:\sovereign-ai-workbench\backend
```

Create a virtual environment:

```powershell
python -m venv .venv
```

Activate it:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```powershell
pip install -r requirements.txt
```

---

## Local Model

Start Ollama if it is not already running.

Check installed models:

```powershell
ollama list
```

For the development configuration, install the configured model if necessary:

```powershell
ollama pull qwen2.5:3b
```

The actual model name can be changed through `LLM_MODEL`.

---

## Start Backend

From:

```text
D:\sovereign-ai-workbench\backend
```

run:

```powershell
python run.py
```

The backend is expected to run on:

```text
http://127.0.0.1:8000
```

FastAPI documentation:

```text
http://127.0.0.1:8000/docs
```

Health endpoint:

```text
http://127.0.0.1:8000/health
```

---

# 23. Start Frontend

Open a second PowerShell window:

```powershell
cd D:\sovereign-ai-workbench\frontend
```

Install packages:

```powershell
npm install
```

Start the development server:

```powershell
npm run dev
```

The Vite development server normally starts at:

```text
http://localhost:5173
```

---

# 24. Backend API

Current API routes include:

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/chat` | Main agentic query endpoint |
| POST | `/api/sandbox/execute` | Execute restricted Python |
| GET | `/api/network/status` | Network/egress telemetry |
| GET | `/api/models` | Local model registry |
| GET | `/api/personas` | Operator personas and clearance |
| GET | `/api/documents` | Indexed document metadata |
| GET | `/api/audit/logs` | Recent audit records |
| GET | `/api/deliverables/{filename}` | Retrieve generated deliverable |

The root endpoints are also available:

```text
GET /
GET /health
```

---

# 25. Demo Personas

The current demonstration includes three operator personas.

| Operator | Role | Clearance |
|---|---|---|
| Rahul Verma | Plant Process Engineer | INTERNAL / L1 |
| Priya Nair | Lead Reliability Inspector | CONFIDENTIAL / L2 |
| Dr. V. K. Sharma | Chief General Manager (Operations) | SECRET / L3 |

These are demonstration identities used to show the clearance model.

In a production deployment, they would be replaced by the organisation's authentication and identity system.

---

# 26. Demonstration Scenarios

The frontend contains evaluation presets for the main capabilities.

## Scenario 1 — General Knowledge

Example:

```text
Explain the working principle of centrifugal pumps using Bernoulli's theorem.
```

Expected behaviour:

```text
Lane A
↓
General local model
↓
No company citations
```

This demonstrates that normal knowledge requests do not automatically search the private knowledge base.

---

## Scenario 2 — Clearance Test

Example:

```text
What are the strategic underground crude reserve cavern quotas at Visakhapatnam?
```

Run the query as:

```text
Rahul Verma
INTERNAL / L1
```

The restricted document should not be returned.

Run the same query as:

```text
Dr. V. K. Sharma
SECRET / L3
```

The authorised user can retrieve the relevant restricted document when it matches the query.

This demonstrates that retrieval access changes with clearance.

---

## Scenario 3 — Agentic Engineering Approval Note

Example:

```text
Draft official approval note for Pump P-102 vibration analysis
and calculate MAWP derating for Column CD-01 crack W-14.
```

The workflow can combine:

```text
Query routing
      ↓
Local document grounding
      ↓
Engineering calculation
      ↓
Sandbox verification
      ↓
Approval note generation
      ↓
Audit trace
```

The result is a native:

```text
.docx
```

file.

---

## Scenario 4 — Engineering Analysis Spreadsheet

Example:

```text
Calculate safe operating pressure and prepare an analysis
report table for Column CD-01 crack W-14 derating.
```

The calculation path can use the restricted sandbox and generate:

```text
.xlsx
```

analysis output.

---

# 27. Automated Verification Suite

The backend includes:

```text
backend/test_demo.py
```

Run it from the backend directory:

```powershell
python test_demo.py
```

The current verification suite covers five core areas:

### Test 1 — Zero-Egress Status

Checks the network monitor and expects:

```text
External packets = 0
```

### Test 2 — Lane A

Checks that a general physics query is classified as:

```text
LANE_A_GENERAL
```

and returns no company-document citations.

### Test 3 — Clearance Gate

Checks that an INTERNAL user cannot receive the restricted demonstration document.

### Test 3B — Authorised Secret User

Checks that the SECRET persona can execute the same request without being blocked by the clearance tier.

### Test 4 — Sandbox

Runs a controlled vibration calculation and verifies the expected result.

### Test 5 — End-to-End Deliverable

Runs the approval-note workflow and checks that a real `.docx` file is created on disk.

A successful run ends with:

```text
ALL 5 CORE HACKATHON OBJECTIVES VALIDATED SUCCESSFULLY!
```

---

# 28. What Makes the Project Different

The project is not positioned as another generic chatbot.

Its focus is the combination of several requirements that normally exist as separate systems:

### 1. AI + Data Sovereignty

The model is designed to run inside the organisation instead of requiring confidential prompts to be sent to a hosted AI service.

### 2. Security-Aware Retrieval

The RAG layer understands document clearance rather than treating all company documents as one searchable pool.

### 3. Task-Based Model Routing

Different work types can be assigned to different local model capabilities.

### 4. Agentic Tool Use

The system can move from answering to performing local operations such as retrieval, calculation and document generation.

### 5. Verifiable Calculation

Engineering calculations are executed through a restricted computational layer rather than relying solely on generated arithmetic.

### 6. Real Deliverables

The output can be a Word approval note or Excel analysis workbook, not just a chat message.

### 7. Observable Security

Network status, routing, provenance and audit information are exposed to the operator.

The goal is therefore not simply:

```text
"Chat with a local LLM."
```

It is:

```text
"Operate an AI workbench for sensitive industrial knowledge work
inside the organisation's security boundary."
```

---

# 29. Expected Industrial Applications

The architecture can be adapted for:

## Refineries

- Equipment inspection review
- Maintenance analysis
- Pump and compressor reports
- Pressure/derating calculations
- SOP lookup
- Approval-note preparation

## PSUs

- Internal reports
- Procurement documentation
- Technical reviews
- Management notes
- Operational analysis
- Document-heavy workflows

## Defence-Linked Manufacturing

- Controlled engineering documentation
- Maintenance records
- Internal technical reports
- Restricted knowledge retrieval
- On-premise AI-assisted analysis

## Government Organisations

- Internal correspondence
- File and note preparation
- Policy/document search
- Report drafting
- Controlled knowledge assistance

The exact deployment, model and security controls should be adapted to the organisation's classification and compliance requirements.

---

# 30. Deployment Model

A practical deployment can look like:

```text
┌─────────────────────────────────────────────────────────────┐
│                 ORGANISATION NETWORK                        │
│                                                             │
│   ┌───────────────┐       ┌────────────────────────────┐    │
│   │ Operator PC   │──────▶│ Sovereign AI Server        │    │
│   │ Browser       │       │                            │    │
│   └───────────────┘       │ FastAPI                    │    │
│                           │ Local LLM                  │    │
│                           │ Local RAG                  │    │
│                           │ Sandbox                    │    │
│                           │ Audit DB                   │    │
│                           │ Deliverables               │    │
│                           └─────────────┬──────────────┘    │
│                                         │                   │
│                              Organisation-controlled        │
│                              storage / documents             │
│                                                             │
│                    ─── EGRESS CONTROL ───                  │
│                    External network blocked                 │
└─────────────────────────────────────────────────────────────┘
```

The same application can run on:

- A development workstation
- An engineering workstation with a local GPU
- An on-premise GPU server
- A restricted internal server cluster

The model size can be adjusted to available hardware.

---

# 31. Development Hardware Strategy

The project deliberately does not require a 100B+ parameter model to demonstrate its architecture.

A smaller model can be used during development and hackathon demonstration.

For example:

```text
Development
    ↓
Qwen2.5 3B
```

A more capable deployment can use:

```text
Production / stronger workstation
    ↓
larger local Qwen model
```

The application separates the model registry and routing logic from the rest of the agent workflow so the model layer can evolve as better open-weight models become available.

---

# 32. Security Considerations

This project demonstrates architectural controls, but it is not presented as a complete production security certification.

A real enterprise deployment should additionally include:

- Enterprise authentication
- Active Directory / LDAP / SSO integration
- Hardware-backed security where required
- OS-level sandboxing
- Container or VM isolation
- Host firewall rules
- Network segmentation
- Centralised logging
- Encryption at rest
- Encryption in transit inside the organisation
- Key management
- Secure model/package transfer procedures
- Document lifecycle management
- Backup and recovery controls
- Vulnerability scanning
- Patch management
- Security monitoring
- Formal classification handling procedures

The SIH implementation focuses on proving the core architecture and workflow on a local deployment.

---

# 33. Current Implementation vs Future Expansion

The project is intentionally structured so that additional capabilities can be added without replacing the entire backend.

### Current implementation

- Local FastAPI backend
- Local LLM integration through Ollama
- Model registry and task routing
- Lane A / Lane B classification
- Clearance-aware local vector retrieval
- Local sample industrial documents
- Restricted Python sandbox
- Word approval-note generation
- Excel analysis generation
- SQLite audit trail
- Runtime network telemetry
- Privacy filtering
- React/Vite industrial console
- Automated demonstration tests

### Expansion path

- Enterprise authentication
- Production vector database
- Large-scale document ingestion
- OCR pipeline
- Production vision model
- Full scanned-PDF processing
- P&ID/drawing understanding
- More open-weight model backends
- Container/VM execution isolation
- Enterprise storage connectors
- Production policy engine
- More document formats
- PPTX generation
- Human approval workflows
- Fine-grained document-level permissions

---

# 34. Project Status

**SIH Hackathon Prototype / Proof of Concept**

The current implementation demonstrates the core architecture on local infrastructure.

The project is intended to prove that confidential industrial AI workflows can be assembled around open-weight models without making a cloud AI service the centre of the system.

The next step toward production is not simply "use a bigger model".

It is strengthening the surrounding platform:

```text
Identity
+
Security
+
Document ingestion
+
RAG
+
Model serving
+
Agent tools
+
Sandboxing
+
Audit
+
Enterprise deployment
```

---

# 35. Key Files

For anyone reviewing the implementation, these files contain the main system logic:

```text
backend/app/models/router.py
```

Task classification, lane classification and model selection.

```text
backend/app/rag/vector_store.py
```

Clearance-aware vector retrieval.

```text
backend/app/services/llm_provider.py
```

Local LLM provider integration.

```text
backend/app/services/network_monitor.py
```

Runtime network telemetry.

```text
backend/app/services/privacy_filter.py
```

Sensitive-query detection for optional public research.

```text
backend/app/agent/workflow.py
```

Main agent execution pipeline.

```text
backend/app/agent/tools/sandbox_code.py
```

Restricted calculation/code execution.

```text
backend/app/agent/tools/doc_gen.py
```

Word approval-note generation.

```text
backend/app/agent/tools/xlsx_gen.py
```

Excel analysis generation.

```text
backend/app/core/security.py
```

Clearance levels and demonstration personas.

```text
backend/app/core/audit_logger.py
```

Local audit trail.

```text
backend/test_demo.py
```

End-to-end hackathon verification.

---

# 36. Quick Start

```powershell
# Terminal 1
cd D:\sovereign-ai-workbench\backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python run.py
```

```powershell
# Terminal 2
cd D:\sovereign-ai-workbench\frontend
npm install
npm run dev
```

Then open:

```text
http://localhost:5173
```

Backend API documentation:

```text
http://127.0.0.1:8000/docs
```

Run verification:

```powershell
cd D:\sovereign-ai-workbench\backend
python test_demo.py
```

---

# 37. Closing

The central idea behind the Sovereign AI Workbench is straightforward:

**Sensitive industrial AI should be able to operate inside the organisation's own security boundary.**

A useful enterprise AI system needs more than a language model.

It needs:

```text
Local Models
    +
Controlled Knowledge
    +
Access Control
    +
Tool Execution
    +
Verification
    +
Auditability
    +
Real Deliverables
```

This project brings those pieces together into one on-premise workbench designed around the operational realities of confidential industrial environments.

**Sovereign by architecture.  
Useful by workflow.  
Inspectable by design.**
