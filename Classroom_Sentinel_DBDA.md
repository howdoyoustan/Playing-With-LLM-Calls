# The Syllabus Sentinel (Google Classroom Edition)

**Project Goal:** To automate curriculum compliance tracking for the PG-DBDA program by treating **Google Classroom** as the "Single Source of Truth." This tool empowers the Class Representative (CR) to maintain course integrity without manual data entry.

---

## 1. Core Philosophy: "Integrity in Execution"

The PG-DBDA syllabus is vast (900 hours, 10 modules). It is easy for critical but niche topics (e.g., *HBase Security* or *Cloud Pricing models* ) to be overlooked during the rush of the 24-week program.

The **Syllabus Sentinel** acts as an autonomous auditor. It connects directly to the digital classroom, parses the daily materials uploaded by faculty, and performs a "Gap Analysis" against the official C-DAC syllabus PDF.

---

## 2. System Architecture

Here is the comprehensive architectural design for the **Syllabus Sentinel** implemented using **LangGraph**. This design leverages LangGraph's stateful orchestration to handle the cyclical nature of checking, validating, and reporting on course progress.

---

# The Syllabus Sentinel (LangGraph Architecture)

**Project Goal:** To build an autonomous "Academic Auditor" agent that ensures the **PG-DBDA** curriculum is delivered as promised. It validates daily **Google Classroom** uploads against the official **C-DAC Syllabus**, distinguishing between "Theory" and "Lab" hours.

**Core Tech:** LangGraph (State Orchestration), Google Classroom API (Source), Vector Database (Syllabus Knowledge), and LLM (Evaluator).

---

## 1. System High-Level Design

The system is modeled as a **StateGraph**. Unlike a linear pipeline, this graph maintains a "memory" of the current audit session, allowing the agent to dynamically decide if it needs to fetch more context or flag a partial coverage issue.

### The Graph State (`AuditState`)

This dictionary is passed between all nodes, maintaining the context of the current run.

```python
class AuditState(TypedDict):
    current_date: str
    materials_found: List[Dict]      # Metadata from Google Classroom
    extracted_content: str           # Raw text from PDFs/Slides
    identified_module: str           # e.g., "Module 6: Big Data"
    syllabus_requirements: List[str] # Retrieved from RAG (e.g., "HDFS Architecture")
    coverage_gaps: List[str]         # Output of the Gap Analysis
    report_status: str               # "draft", "sent", "failed"

```

---

## 2. The Node Architecture

The workflow consists of **5 primary nodes** and **2 conditional edges**.

### Node 1: `fetch_classroom_materials` (The Crawler)

* **Function:** Connects to the **Google Classroom REST API**.
* **Action:**
* Queries `courses.courseWork` (Assignments/Labs) and `courses.courseWorkMaterials` (Theory).


* Filters for items posted within the last 24 hours.


* **State Update:** Populates `materials_found` with file IDs and titles.

### Node 2: `classify_and_extract` (The Parser)

* **Function:** Downloads files and extracts text.
* **Action:**
* Uses `unstructured` or `pypdf` to parse content.
* **Intelligent Routing (Agent Decision):** Determines if the content is **Lab** (code snippets, manuals) or **Theory** (slides, conceptual diagrams). This is crucial because the syllabus allocates specific hours to each (e.g., 150 hours for Big Data).




* **State Update:** Updates `extracted_content`.

### Node 3: `retrieve_syllabus_context` (The RAG Node)

* **Function:** Fetches the "Ground Truth" from the vector database.
* **Action:**
* Performs a semantic search on the `extracted_content`.
* 
*Example:* If the slide mentions "RDDs", it retrieves the specific line items from **Module 6**: "Resilient Distributed Datasets (RDDs), External Datasets, RDD Operations".




* **State Update:** Populates `syllabus_requirements`.

### Node 4: `evaluate_coverage` (The Auditor Agent)

* **Function:** The "Brain" of the operation.
* **Prompt Logic:**
> "You are the C-DAC Course Auditor.
> **Requirement:** {syllabus_requirements}
> **Evidence:** {extracted_content}
> Compare them strictly. Did the evidence cover 'Job Optimization' and 'Shuffle Operations'?
> If partially covered, list the missing sub-topics."


* **State Update:** specific items are added to `coverage_gaps`.

### Node 5: `generate_report` (The Notifier)

* **Function:** formats the output for the Class Representative.
* **Action:** Creates a Markdown summary or a structured JSON record for the dashboard.

---

## 3. The Control Flow (Edges)

### Conditional Edge A: `has_new_materials?`

* **Logic:** After `fetch_classroom_materials`, checks if the list is empty.
* **If Empty:**  `END` (No cost incurred).
* **If Present:**  `classify_and_extract`.



### Conditional Edge B: `requires_deep_scan?`

* **Logic:** After `evaluate_coverage`, the Agent self-reflects.
* **Scenario:** The slides were too vague (e.g., just images).
* **Action:** The Agent can route *back* to `fetch_classroom_materials` to check for "Class Comments" or "attached links" for more context (Looping capability).
* **Else:**  `generate_report`.



---

## 4. Implementation Details (Python & LangGraph)

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List

# 1. Define State
class AuditState(TypedDict):
    materials: List[str]
    syllabus_context: str
    audit_report: str

# 2. Define Nodes
def fetch_materials(state):
    # Google Classroom API logic here
    # Check courseWorkMaterials vs courseWork for Theory vs Lab separation
    return {"materials": ["slide_deck_v1.pdf"]}

def retrieve_syllabus(state):
    # Query Pinecone/Chroma for "Module 6: Big Data" requirements
    # [cite_start]e.g., "HDFS Java API, Map Reduce Anatomy, Failures" [cite: 96]
    return {"syllabus_context": "Required: HDFS Architecture, NameNode, DataNode..."}

def audit_agent(state):
    # LLM comparison logic
    return {"audit_report": "Gap Detected: 'Failures' in MapReduce not covered."}

# 3. Build Graph
workflow = StateGraph(AuditState)

# Add Nodes
workflow.add_node("crawler", fetch_materials)
workflow.add_node("rag_retriever", retrieve_syllabus)
workflow.add_node("auditor", audit_agent)

# Add Edges
workflow.set_entry_point("crawler")
workflow.add_edge("crawler", "rag_retriever")
workflow.add_edge("rag_retriever", "auditor")
workflow.add_edge("auditor", END)

# Compile
app = workflow.compile()

```

---

## 5. Technology Stack

| Component | Tool | Purpose |
| --- | --- | --- |
| **Orchestrator** | **LangGraph** | Managing the cyclic workflow and state. |
| **LLM** | **GPT-4o** or **Claude 3.5 Sonnet** | High-reasoning capability for "Gap Analysis". |
| **Vector Store** | **ChromaDB** (Local) or **Pinecone** | Storing the chunked PDF Syllabus (e.g., Module definitions). |
| **Ingestion** | **Google Classroom API** | Fetching daily `courseWork` and `materials`. |
| **Parsing** | **Unstructured.io** | Extracting text from PDF slides and Lab Manuals. |
| **Deployment** | **Docker** | Containerizing the Python environment. |

---

## 6. Specific Syllabus Mapping Strategy

To ensure high accuracy, the Vector Store will be indexed by **Module ID**:

* 
**Index Key `MOD_01` (Linux):** Covers "Installation, Shells, Commands".


* 
**Index Key `MOD_06` (Big Data):** Covers "Hadoop, HDFS, Map Reduce, Spark".


* 
**Index Key `MOD_08` (ML):** Covers "Supervised/Unsupervised Learning, Kmeans, KNN".



This allows the Agent to fetch the *exact* checklist for the day's topic.
