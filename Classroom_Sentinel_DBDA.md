# The Syllabus Sentinel (Google Classroom Edition)

**Project Goal:** To automate curriculum compliance tracking for the PG-DBDA program by treating **Google Classroom** as the "Single Source of Truth." This tool empowers the Class Representative (CR) to maintain course integrity without manual data entry.

---

## 1. Core Philosophy: "Integrity in Execution"

The PG-DBDA syllabus is vast (900 hours, 10 modules). It is easy for critical but niche topics (e.g., *HBase Security* or *Cloud Pricing models* ) to be overlooked during the rush of the 24-week program.

The **Syllabus Sentinel** acts as an autonomous auditor. It connects directly to the digital classroom, parses the daily materials uploaded by faculty, and performs a "Gap Analysis" against the official C-DAC syllabus PDF.

---

## 2. System Architecture

The system follows a standard **ELT (Extract, Load, Transform)** pattern, orchestrated by **Apache Airflow**.

### A. The Ingestion Layer (The Crawler)

* **Source:** Google Classroom REST API.
* **Targets:**
* `courses.courseWork` (Assignments/Labs): Tracks practical progress (e.g., "Submit Spark Lab").
* `courses.courseWorkMaterials` (Slides/Notes): Tracks theoretical progress.


* **Frequency:** Daily (via Airflow DAG).

### B. The Processing Layer (The Parser)

* **Text Extraction:**
* Downloads attachments (PDFs, PPTs, Docx) to a temporary landing zone.
* Uses **OCR/Text Extraction** (e.g., `unstructured` or `PyPDF2`) to convert slide decks into raw text.


* **Metadata Tagging:**
* Extracts timestamps to correlate content with the timeline (e.g., "Week 12 materials").



### C. The Agentic Layer (The Sentinel)

This is the core intelligence. It uses an LLM (or semantic search) to compare *what was taught* vs. *what was promised*.

* **Input:** Extracted text from `Day45_Spark.pdf`.
* **Reference:** The Official PG-DBDA Syllabus (parsed into a Vector Store).
* **Logic:**
1. **Identification:** Recognizes the topic is "Apache Spark".
2. 
**Depth Check:** Scans for specific required sub-topics like *RDD Persistence* , *Shared Variables* , or *Spark Streaming*.


3. **Verdict:**
* ✅ **Covered:** All key terms found.
* ⚠️ **Gap Detected:** "Spark RDDs covered, but 'Integration with Kafka'  is missing."







---

## 3. Implementation Logic

### Step 1: Syllabus Indexing (One-time Setup)

The Agent first "learns" the curriculum by ingesting the `PG-DBDA August 2025 Syllabus.pdf`. It breaks it down into granular checkpoints:

* 
**Module 6 (Big Data):** 150 Hours.


* 
*Checkpoint:* "Design of HDFS" 


* 
*Checkpoint:* "Map Reduce Failures" 


* 
*Checkpoint:* "Apache Airflow" 





### Step 2: The Airflow DAG (Daily Run)

A Python DAG runs every evening at 18:00.

```python
# Pseudo-code for the Syllabus Sentinel DAG

@task
def fetch_classroom_materials():
    """Hits Google Classroom API to get today's slides/labs"""
    # Returns: ["intro_to_spark.pdf", "lab_manual_05.docx"]
    pass

@task
def extract_context(files):
    """Converts PDF/Docx to string content"""
    # Returns: "Today we discussed RDDs, Transformations, and Actions..."
    pass

@task
def gap_analysis_agent(daily_context):
    """
    Agent Prompt:
    'You are a rigorous Academic Auditor. 
    Compare the following class notes: {daily_context}
    Against the syllabus requirements for 'Module 6'.
    
    Did the faculty cover 'Shuffle and Sort' and 'Partitioner' logic?
    Output a completion score (0-100%).'
    """
    pass

@task
def update_dashboard(gaps):
    """Updates the CR's Streamlit Dashboard"""
    # If gap found -> Flag "Needs Review"
    pass

```

---

## 4. Value for the Technical Class Representative (CR)

1. **Automated Audit Trail:**
* If students complain that "Practical Machine Learning" was rushed, the CR has data: "On Nov 12th, only 14 slides were uploaded for 'Gradient Boosting', covering 5% of the required topics."




2. **Lab vs. Theory Distinction:**
* The system distinguishes between a **Lecture** (Theory) and a **Lab Assignment** (Practice), ensuring the 900-hour split (Theory + Lab + Project)  is respected.




3. **Conflict Resolution:**
* Provides objective data for meetings with the Course Coordinator, removing emotion from the feedback loop.



---

## 5. Recommended Tech Stack

* **Orchestration:** Apache Airflow (running on Docker).
* **API Integration:** `google-api-python-client`.
* **PDF Parsing:** `pypdf` or `langchain` document loaders.
* **Logic:** Python 3.9+ with `pandas` for tracking state.
* **Database:** SQLite or PostgreSQL (to store the "Checklist" state).
