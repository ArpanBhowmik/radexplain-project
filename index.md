# RadExplain

**RadExplain** is a multi-agent AI system for clinical radiation oncology decision support. It combines a LangGraph-orchestrated agent pipeline with Retrieval-Augmented Generation (RAG) over QUANTEC literature to provide evidence-based dose safety assessments, treatment plan summaries, and clinical guideline interpretation.

## Architecture

![RadExplain Architecture Diagram](Architecture.png)

## Example Queries

**Clinical AI Assistant Dashboard**
![RadExplain Clinical Assistant](assets/Clinical.gif)

### 1. Treatment Plan Summary

**Query:** `Summarise the treatment plan of pt_1`

**RadExplain Response:**

> **Plan Overview:** Curative intent plan delivering high-dose radiation to three PTVs. Parotid glands and mandible exceed QUANTEC recommendations.
>
> | Structure | Dose | QUANTEC Limit | Status |
> |---|---|---|---|
> | Brainstem | 40.4 Gy | ≤ 54 Gy | ✅ Safe |
> | Spinal Cord | 32.0 Gy | ≤ 50 Gy | ✅ Safe |
> | Left Parotid | 61.7 Gy mean | ≤ 20 Gy | ❌ Violated |
> | Right Parotid | 56.3 Gy mean | ≤ 20 Gy | ❌ Violated |
> | Mandible | 73.8 Gy | ≤ 70 Gy | ❌ Violated |
>
> ⚠️ **SpinalCord:** Only 62.5% of volume inside calculation grid — reported Dmax may underestimate true maximum dose. Verification recommended.

---

### 2. OAR Violation Detection

**Query:** `Which OARs are violated in patient with id pt_1's plan?`

**Agent Trace (Simplified):**
```text
1. Supervisor → Data Agent: retrieve all OAR doses for pt_1
2. Data Agent → Database: query all metrics
3. Supervisor → Math Agent: check all OARs against QUANTEC
4. Math Agent (deterministic limits):
   ✅ Brainstem 4040.9 cGy — passes 5400 cGy limit
   ✅ SpinalCord 3202.6 cGy — passes 5000 cGy limit  
   ❌ Mandible 7376.1 cGy — FAILS 7000 cGy limit by 376.1 cGy
   ❌ Parotids — neither QUANTEC rule met
5. Supervisor → Summary Agent: generate clinical report
```

**Result:** Mandible and both parotid glands violated. Brainstem and spinal cord within tolerance.

---

### 3. Reliability Warning (Agent vs Baseline)

**Query:** `Is pt_10 brainstem dose safe?`

**RadExplain Response:**

> The reported brainstem maximum dose of 3.26 Gy (325.9 cGy) is far below the QUANTEC limit of 54 Gy — safety margin of 50.74 Gy.
>
> ⚠️ **Reliability Warning:** Only 17.89% of the brainstem volume falls within the calculation grid. The reported Dmax reflects only a small peripheral fragment of the brainstem. The dose to approximately 82% of the structure is unknown. Clinical verification is strongly recommended before concluding the brainstem is safe.

**Standard LLM Baseline Response:**
> The brainstem dose of 3.26 Gy is well within the safe limit of 54 Gy. No concerns identified.

*Notice how the standard baseline missed the calculation volume warning — a clinically significant safety hazard.*

---

### 4. Clinical Knowledge & Guideline Retrieval

**Query:** `What complications are associated with high mandible dose?`

**Agent Trace (Simplified):**
```text
1. Supervisor → Knowledge Agent: retrieve clinical literature regarding mandible dose complications
2. Knowledge Agent → RAG Pipeline:
   a. Embed query using BGE-base-en-v1.5
   b. Semantic search in ChromaDB across QUANTEC PDFs
   c. Cross-encoder reranking of top-10 chunks using BGE-reranker-base
3. RAG Pipeline → Knowledge Agent: Returns top-3 highly relevant clinical text chunks
4. Supervisor → Summary Agent: generate clinical response based on retrieved evidence
```

**RadExplain Response:**

> According to the QUANTEC guidelines, the primary complication associated with high radiation dose to the mandible is Osteoradionecrosis (ORN). The risk of ORN increases significantly when the maximum dose exceeds 70 Gy. Preventive measures such as pre-radiotherapy dental evaluation and maintaining excellent oral hygiene are strongly recommended to mitigate this risk.

## Evaluation Results

The multi-agent system is benchmarked against a baseline LLM (no RAG, no specialized tools) on a 41-question clinical dataset. The results are scored by an LLM-as-a-Judge (GPT-OSS 120B).

**Interactive Evaluation Dashboard**
![Evaluation Dashboard](assets/Evaluation.gif)

### 1. RadExplain vs Baseline by Category
![Category Performance](assets/headline_grouped_bar.png)

### 2. Overall Performance Comparison
![Overall Performance](assets/overall_radar.png)

### 3. Question-Level Heatmap
![Question Heatmap](assets/heatmap.png)

### 4. RAG Retrieval Performance
![RAG Metrics](assets/rag_metrics.png)

## Key Design Decisions

| Decision | Rationale |
|---|---|
| **LangGraph StateGraph** | Enables a cyclic Planner → Agent → Planner loop, allowing multi-step reasoning across agents |
| **Deterministic Math Tools** | QUANTEC safety checks use hardcoded Python functions (not LLM generation) to eliminate numerical hallucinations |
| **BGE Cross-Encoder Reranking** | Two-stage retrieval (embedding similarity → cross-encoder reranking) maximizes precision of clinical evidence |
| **Organ-Aware Retrieval** | RAG queries are filtered by detected organ to prevent cross-organ contamination in retrieved chunks |
| **LLM-as-Judge Evaluation** | GPT-OSS 120B scores responses on a 0-100 rubric with structured pass/fail classification |
| **Separate Baseline Runner** | Enables direct A/B comparison: same questions evaluated by raw LLM vs. full agentic pipeline |

## Tech Stack

| Component | Technology |
|---|---|
| **Backend** | FastAPI · Uvicorn |
| **Agent Orchestration** | LangGraph (LangChain) |
| **LLM** | LLaMA 3.3 70B via Groq |
| **Judge LLM** | GPT-OSS 120B via Groq |
| **Embeddings** | BGE-base-en-v1.5 (HuggingFace) |
| **Reranker** | BGE-reranker-base (Cross-Encoder) |
| **Vector Store** | ChromaDB |
| **Frontend** | Vanilla HTML/CSS/JS |
| **Evaluation UI** | Gradio |
| **Visualization** | Matplotlib · Seaborn |

