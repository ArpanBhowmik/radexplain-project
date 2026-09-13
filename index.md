---
layout: default
---
<header class="hero">
  <h1>RadExplain</h1>
  <p class="subtitle">A Multi-Agent AI System for Clinical Radiation Oncology Decision Support</p>
  
  <div class="authors">
    <div class="author">
      <span class="author-name">Arpan Bhowmik</span>
      <span class="author-role" style="text-align: center;">Lead Software Engineer<br>Samsung R&D Institute Bangladesh</span>
    </div>
    <div class="author">
      <span class="author-name">Sarna Rani Bhaduri</span>
      <span class="author-role" style="text-align: center;">M.Sc. in Physics<br>University of Rajshahi</span>
    </div>
  </div>
</header>

<p><strong>RadExplain</strong> is a multi-agent AI system for clinical radiation oncology decision support. It combines a LangGraph-orchestrated agent pipeline with Retrieval-Augmented Generation (RAG) over QUANTEC literature to provide evidence-based dose safety assessments, treatment plan summaries, and clinical guideline interpretation.</p>

<h2>Architecture</h2>
<img src="assets/Architecture.png" alt="RadExplain Architecture Diagram">

<h2>Example Queries</h2>
<p><strong>Clinical AI Assistant Dashboard</strong></p>
<img src="assets/Clinical.gif" alt="RadExplain Clinical Assistant">

<h3>1. Treatment Plan Summary</h3>
<div class="chat-container">
  <div class="query-box">
    <code>Summarise the treatment plan of pt_1</code>
  </div>
  
  <div class="response-box">
    <p><strong>Plan Overview:</strong> Curative intent plan delivering high-dose radiation to three PTVs. Parotid glands and mandible exceed QUANTEC recommendations.</p>
    
    <table>
      <thead>
        <tr>
          <th>Structure</th>
          <th>Dose</th>
          <th>QUANTEC Limit</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Brainstem</td>
          <td>40.4 Gy</td>
          <td>&le; 54 Gy</td>
          <td><span class="badge-safe">✅ Safe</span></td>
        </tr>
        <tr>
          <td>Spinal Cord</td>
          <td>32.0 Gy</td>
          <td>&le; 50 Gy</td>
          <td><span class="badge-safe">✅ Safe</span></td>
        </tr>
        <tr>
          <td>Left Parotid</td>
          <td>61.7 Gy mean</td>
          <td>&le; 20 Gy</td>
          <td><span class="badge-violated">❌ Violated</span></td>
        </tr>
        <tr>
          <td>Right Parotid</td>
          <td>56.3 Gy mean</td>
          <td>&le; 20 Gy</td>
          <td><span class="badge-violated">❌ Violated</span></td>
        </tr>
        <tr>
          <td>Mandible</td>
          <td>73.8 Gy</td>
          <td>&le; 70 Gy</td>
          <td><span class="badge-violated">❌ Violated</span></td>
        </tr>
      </tbody>
    </table>
    
    <div class="alert-warning">
      <strong>⚠️ SpinalCord:</strong> Only 62.5% of volume inside calculation grid — reported Dmax may underestimate true maximum dose. Verification recommended.
    </div>
  </div>
</div>

<h3>2. OAR Violation Detection</h3>
<div class="chat-container">
  <div class="query-box">
    <code>Which OARs are violated in patient with id pt_1's plan?</code>
  </div>

  <div class="agent-trace">
    <div class="agent-trace-header">Agent Trace (Simplified)</div>
    1. Supervisor → Data Agent: retrieve all OAR doses for pt_1<br>
    2. Data Agent → Database: query all metrics<br>
    3. Supervisor → Math Agent: check all OARs against QUANTEC<br>
    4. Math Agent (deterministic limits):<br>
       &nbsp;&nbsp;&nbsp;✅ Brainstem 4040.9 cGy — passes 5400 cGy limit<br>
       &nbsp;&nbsp;&nbsp;✅ SpinalCord 3202.6 cGy — passes 5000 cGy limit<br>  
       &nbsp;&nbsp;&nbsp;❌ Mandible 7376.1 cGy — FAILS 7000 cGy limit by 376.1 cGy<br>
       &nbsp;&nbsp;&nbsp;❌ Parotids — neither QUANTEC rule met<br>
    5. Supervisor → Summary Agent: generate clinical report
  </div>

  <div class="response-box">
    <p>Mandible and both parotid glands violated. Brainstem and spinal cord within tolerance.</p>
  </div>
</div>

<h3>3. Reliability Warning (Agent vs Baseline)</h3>
<div class="chat-container">
  <div class="query-box">
    <code>Is pt_10 brainstem dose safe?</code>
  </div>

  <div class="response-box">
    <p>The reported brainstem maximum dose of 3.26 Gy (325.9 cGy) is far below the QUANTEC limit of 54 Gy — safety margin of 50.74 Gy.</p>
    
    <div class="alert-warning">
      <strong>⚠️ Reliability Warning:</strong> Only 17.89% of the brainstem volume falls within the calculation grid. The reported Dmax reflects only a small peripheral fragment of the brainstem. The dose to approximately 82% of the structure is unknown. Clinical verification is strongly recommended before concluding the brainstem is safe.
    </div>
  </div>

  <div class="baseline-box">
    <p>The brainstem dose of 3.26 Gy is well within the safe limit of 54 Gy. No concerns identified.</p>
  </div>
  <p><em>Notice how the standard baseline missed the calculation volume warning — a clinically significant safety hazard.</em></p>
</div>

<h3>4. Clinical Knowledge & Guideline Retrieval</h3>
<div class="chat-container">
  <div class="query-box">
    <code>What complications are associated with high mandible dose?</code>
  </div>

  <div class="agent-trace">
    <div class="agent-trace-header">Agent Trace (Simplified)</div>
    1. Supervisor → Knowledge Agent: retrieve clinical literature regarding mandible dose complications<br>
    2. Knowledge Agent → RAG Pipeline:<br>
       &nbsp;&nbsp;&nbsp;a. Embed query using BGE-base-en-v1.5<br>
       &nbsp;&nbsp;&nbsp;b. Semantic search in ChromaDB across QUANTEC PDFs<br>
       &nbsp;&nbsp;&nbsp;c. Cross-encoder reranking of top-10 chunks using BGE-reranker-base<br>
    3. RAG Pipeline → Knowledge Agent: Returns top-3 highly relevant clinical text chunks<br>
    4. Supervisor → Summary Agent: generate clinical response based on retrieved evidence
  </div>

  <div class="response-box">
    <p>According to the QUANTEC guidelines, the primary complication associated with high radiation dose to the mandible is Osteoradionecrosis (ORN). The risk of ORN increases significantly when the maximum dose exceeds 70 Gy. Preventive measures such as pre-radiotherapy dental evaluation and maintaining excellent oral hygiene are strongly recommended to mitigate this risk.</p>
  </div>
</div>

## Evaluation Results

The multi-agent system is benchmarked against a baseline LLM (no RAG, no specialized tools) on a 41-question clinical dataset. The results are scored by an LLM-as-a-Judge (GPT-OSS 120B). The baseline represents the lower bound of LLM-only performance without augmentation. We chose this comparison to quantify the specific contribution of each agentic component (deterministic tooling, RAG retrieval, multi-step orchestration) rather than to claim superiority over state-of-the-art clinical AI systems. We acknowledge the limitations of LLM-based evaluation and plan to supplement with expert human evaluation in future work.

### 1. RadExplain vs Baseline by Category
![Category Performance](assets/headline_grouped_bar.png)

To rigorously benchmark the system, we constructed a **41-question adversarial clinical dataset** explicitly designed to trigger known generative AI failure modes in healthcare. The questions are divided into six critical evaluation categories:

* **Single OAR Safety:** Tests the system's ability to accurately extract specific organ dosimetrics (e.g., Brainstem Dmax) and correctly apply deterministic QUANTEC limits. The baseline LLM often hallucinates the raw numbers.
* **Multi-OAR Violation Scans:** Prompts the system to evaluate an entire patient DVH blindly. The baseline LLM frequently hallucinates "threshold-free comparisons" (e.g., stating an organ is safe just because its dose is lower than another organ's). RadExplain's Math Agent systematically checks every organ.
* **Plan Summaries:** Tests the Summary Agent's ability to synthesize raw data and safety violations into a professional, cohesive clinical chart note.
* **Clinical Context:** Pure medical knowledge queries (e.g., defining serial vs. parallel architecture). Tests the RAG pipeline's retrieval precision independently of patient data.
* **Failure Mode Probes:** Deliberate trick questions designed to induce hallucinations. We test for *Fractionation Context Confusion* (asking about SBRT when limits are conventional) and *Calculation Volume Omission* (testing if the AI blindly approves a dose when the physical radiation grid is dangerously small). The baseline LLM scored significantly lower on these probes, whereas RadExplain consistently detected and flagged the embedded traps.
* **Edge Cases:** Evaluates graceful failure, ensuring the system refuses to answer when queried with non-existent patient IDs (e.g., `pt_99`) rather than hallucinating fake clinical records.

### 2. Overall Performance Comparison
![Overall Performance](assets/overall_radar.png)

To evaluate the system objectively, we utilize an **LLM-as-a-Judge framework** (powered by a 120-Billion parameter model). The judge evaluates both the baseline LLM and RadExplain against a verified Ground Truth using a strict **100-point clinical rubric**.

**The 7-Dimension Scoring Rubric:**
* **Numerical Accuracy (20 pts):** Strict verification of dose conversions (cGy to Gy) and safety margin arithmetic.
* **Clinical Classification (20 pts):** Binary check if the model correctly passed or failed the organ based on QUANTEC limits.
* **Reliability Warning (15 pts):** Ensures the AI flags physical data warnings (e.g., if the calculation volume for a Serial OAR is dangerously low at <80%).
* **Faithfulness (15 pts):** Strict penalization for any fabricated statistics or thresholds not grounded in the retrieved literature.
* **Metric Matching (10 pts):** Ensures the correct metric is applied to the correct organ architecture (e.g., Mean Dose for parallel parotids, Max Dose for serial brainstem).
* **Appropriate Hedging (10 pts) & Concision (10 pts):** Evaluates clinical tone and refusal to definitively approve doses when critical context is missing (e.g., SBRT vs Conventional fractionation).

**Critical Failures (Deductions):**
Standard AI benchmarks often forgive "close" answers. In clinical radiotherapy, a close answer is a clinically significant error. The Judge actively deducts points for *Critical Failures*:
* **-3 pts** for any mathematical error in dose comparison.
* **-3 pts** for failing to warn the physician about missing grid calculation volume on a serial organ.
* **-2 pts** for applying the wrong evaluation metric without flagging it.

RadExplain's multi-agent architecture effectively mitigates these critical failures, whereas the baseline LLM frequently triggers them due to mathematical hallucinations and metric confusion.

### 3. RAG Retrieval Performance
To ensure the Knowledge Agent grounds its responses in accurate clinical literature without hallucination, the Retrieval-Augmented Generation (RAG) pipeline was evaluated against a dataset of **150 clinical queries** derived from QUANTEC guidelines.

The pipeline utilizes a two-stage retrieval architecture: dense embedding search (`BGE-base-en-v1.5`) followed by cross-encoder reranking (`BGE-reranker-base`) combined with custom Organ-Aware Filtering to prevent cross-organ metric contamination.

**150-Question Benchmark Results:**
* **MRR Score (Mean Reciprocal Rank): `0.9419`** — The correct QUANTEC guideline chunk is typically returned as the top-ranked result.
* **Hit@1 Rate: `91.3%`** — Top-1 Accuracy. In 9 out of 10 queries, the very first retrieved document perfectly contains the required clinical metrics.
* **Hit@3 Rate: `96.7%`** — Top-3 Recall. 
* **Hit@5 Rate: `98.0%`** — Generation Ceiling. This indicates that relevant clinical evidence is reliably surfaced within the top-5 retrieved chunks, providing a strong foundation for downstream reasoning.
* **Context Precision@5: `97.7%` (Organ Cleanliness)** — In medical AI, retrieving literature for the wrong organ is a safety-critical flaw (e.g., pulling a bladder dose limit when asked about the rectum). By actively filtering chunks *before* ranking, the system achieves high precision. This means 97.7% of the clinical literature retrieved is strictly isolated to the specific organ the user asked about, substantially reducing the risk of cross-organ contamination in retrieved evidence.

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

## Limitations & Future Work

While RadExplain demonstrates strong automated evaluation metrics, it is designed strictly as a **decision-support research prototype**, not a deployed medical device. To ensure academic rigor and transparency, the following limitations are explicitly noted:

* **Preliminary Automated Benchmarking:** The current evaluation relies on an LLM-as-a-Judge (GPT-OSS 120B) to establish baseline safety metrics. While the judge employs a strict deterministic 100-point clinical rubric, this serves as preliminary validation.
* **Domain Expert Curation:** The evaluation dataset, QUANTEC reference limits, and deterministic mathematical constraints were heavily curated and verified in collaboration with a **Medical Physics domain expert** (M.Sc. Physics), ensuring the foundational dose arithmetic and baseline ground truths are physically and clinically sound.
* **Documented Failure Modes:** Despite the agentic architecture's high performance, we manually identified and documented **14 extreme edge cases** where the baseline LLM's clinical reasoning collapses (e.g., *Fractionation Context Confusion*, *Calculation Volume Omission*). These failure cases illustrate the potential value of multi-agent deterministic tooling over end-to-end LLM generation for safety-critical clinical tasks.
* **Future Work:** Transitioning from automated systems benchmarking to formal, blinded clinical validation with board-certified radiation oncologists.
