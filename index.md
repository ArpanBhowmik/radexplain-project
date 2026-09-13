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

  <div class="link-buttons">
    <a href="https://github.com/ArpanBhowmik/RadExplain" class="link-btn" target="_blank">
      <svg viewBox="0 0 16 16" fill="currentColor"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg>
      GitHub Repository
    </a>
  </div>
</header>

<div class="abstract">
  <div class="abstract-title">Abstract</div>
  <p>RadExplain is a multi-agent AI system for clinical radiation oncology decision support. It combines a LangGraph-orchestrated agent pipeline with Retrieval-Augmented Generation (RAG) over QUANTEC literature to provide evidence-based dose safety assessments, treatment plan summaries, and clinical guideline interpretation.</p>
  <p>The system employs deterministic mathematical tools for QUANTEC safety checks (eliminating LLM numerical hallucinations), a two-stage RAG pipeline with cross-encoder reranking and organ-aware filtering (MRR: 0.94, Hit@1: 91.3%), and a multi-agent architecture that detects clinical edge cases missed by standard LLMs — including calculation volume warnings and fractionation context confusion. Evaluated on a 41-question adversarial clinical dataset using an LLM-as-a-Judge framework, RadExplain consistently outperforms the baseline LLM across all six evaluation categories.</p>
</div>

<div class="toc">
  <div class="toc-title">Contents</div>
  <ol>
    <li><a href="#architecture">Architecture</a></li>
    <li><a href="#example-queries">Example Queries</a></li>
    <li><a href="#evaluation-results">Evaluation Results</a></li>
    <li><a href="#key-design-decisions">Key Design Decisions</a></li>
    <li><a href="#tech-stack">Tech Stack</a></li>
    <li><a href="#limitations--future-work">Limitations & Future Work</a></li>
  </ol>
</div>

<h2 id="architecture">Architecture</h2>

<figure>
  <img src="assets/Architecture.png" alt="RadExplain Architecture Diagram">
  <figcaption><strong>Figure 1.</strong> RadExplain system architecture. The Supervisor agent orchestrates Data, Math, Knowledge, and Summary agents via a LangGraph StateGraph. Deterministic QUANTEC safety checks and two-stage RAG retrieval operate as specialized tool nodes.</figcaption>
</figure>

<h2 id="example-queries">Example Queries</h2>

<figure>
  <img src="assets/Clinical.gif" alt="RadExplain Clinical Assistant">
  <figcaption><strong>Figure 2.</strong> Clinical AI Assistant dashboard demonstrating real-time multi-agent query processing.</figcaption>
</figure>

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

<h2 id="evaluation-results">Evaluation Results</h2>

<p>The multi-agent system is benchmarked against a baseline LLM (no RAG, no specialized tools) on a 41-question clinical dataset. The results are scored by an LLM-as-a-Judge (GPT-OSS 120B). The baseline represents the lower bound of LLM-only performance without augmentation. We chose this comparison to quantify the specific contribution of each agentic component (deterministic tooling, RAG retrieval, multi-step orchestration) rather than to claim superiority over state-of-the-art clinical AI systems. We acknowledge the limitations of LLM-based evaluation and plan to supplement with expert human evaluation in future work.</p>

### 1. RadExplain vs Baseline by Category

<figure>
  <img src="assets/headline_grouped_bar.png" alt="Category Performance">
  <figcaption><strong>Figure 3.</strong> Mean LLM-as-Judge scores (0–100) for RadExplain vs. baseline LLM across six adversarial evaluation categories.</figcaption>
</figure>

To rigorously benchmark the system, we constructed a **41-question adversarial clinical dataset** explicitly designed to trigger known generative AI failure modes in healthcare. The questions are divided into six critical evaluation categories:

<table class="eval-table">
  <thead>
    <tr>
      <th>Category</th>
      <th>What It Tests</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Single OAR Safety</td>
      <td>Accurate extraction of organ dosimetrics and correct application of deterministic QUANTEC limits. The baseline LLM often hallucinates the raw numbers.</td>
    </tr>
    <tr>
      <td>Multi-OAR Violation Scans</td>
      <td>Evaluation of an entire patient DVH. The baseline frequently hallucinates "threshold-free comparisons." RadExplain's Math Agent systematically checks every organ.</td>
    </tr>
    <tr>
      <td>Plan Summaries</td>
      <td>Synthesis of raw data and safety violations into a professional, cohesive clinical chart note.</td>
    </tr>
    <tr>
      <td>Clinical Context</td>
      <td>Pure medical knowledge queries (e.g., serial vs. parallel organ architecture). Tests RAG retrieval precision independently of patient data.</td>
    </tr>
    <tr>
      <td>Failure Mode Probes</td>
      <td>Deliberate trick questions: <em>Fractionation Context Confusion</em> and <em>Calculation Volume Omission</em>. The baseline LLM scored significantly lower on these probes, whereas RadExplain consistently detected and flagged the embedded traps.</td>
    </tr>
    <tr>
      <td>Edge Cases</td>
      <td>Graceful failure on non-existent patient IDs (e.g., <code>pt_99</code>), ensuring the system refuses rather than hallucinating fake clinical records.</td>
    </tr>
  </tbody>
</table>

### 2. Overall Performance Comparison

<figure>
  <img src="assets/overall_radar.png" alt="Overall Performance">
  <figcaption><strong>Figure 4.</strong> Radar plot comparing RadExplain and baseline LLM across seven evaluation dimensions scored by the LLM-as-a-Judge.</figcaption>
</figure>

To evaluate the system objectively, we utilize an **LLM-as-a-Judge framework** (powered by a 120-Billion parameter model). The judge evaluates both the baseline LLM and RadExplain against a verified Ground Truth using a strict **100-point clinical rubric**.

<details>
  <summary>View the 7-Dimension Scoring Rubric & Critical Failure Deductions</summary>
  <div class="details-content">
    <p><strong>The 7-Dimension Scoring Rubric:</strong></p>
    <ul>
      <li><strong>Numerical Accuracy (20 pts):</strong> Strict verification of dose conversions (cGy to Gy) and safety margin arithmetic.</li>
      <li><strong>Clinical Classification (20 pts):</strong> Binary check if the model correctly passed or failed the organ based on QUANTEC limits.</li>
      <li><strong>Reliability Warning (15 pts):</strong> Ensures the AI flags physical data warnings (e.g., if the calculation volume for a Serial OAR is dangerously low at &lt;80%).</li>
      <li><strong>Faithfulness (15 pts):</strong> Strict penalization for any fabricated statistics or thresholds not grounded in the retrieved literature.</li>
      <li><strong>Metric Matching (10 pts):</strong> Ensures the correct metric is applied to the correct organ architecture (e.g., Mean Dose for parallel parotids, Max Dose for serial brainstem).</li>
      <li><strong>Appropriate Hedging (10 pts) &amp; Concision (10 pts):</strong> Evaluates clinical tone and refusal to definitively approve doses when critical context is missing (e.g., SBRT vs Conventional fractionation).</li>
    </ul>
    <p><strong>Critical Failure Deductions:</strong></p>
    <p>Standard AI benchmarks often forgive "close" answers. In clinical radiotherapy, a close answer is a clinically significant error. The Judge actively deducts points for <em>Critical Failures</em>:</p>
    <ul>
      <li><strong>-3 pts</strong> for any mathematical error in dose comparison.</li>
      <li><strong>-3 pts</strong> for failing to warn the physician about missing grid calculation volume on a serial organ.</li>
      <li><strong>-2 pts</strong> for applying the wrong evaluation metric without flagging it.</li>
    </ul>
    <p>RadExplain's multi-agent architecture effectively mitigates these critical failures, whereas the baseline LLM frequently triggers them due to mathematical hallucinations and metric confusion.</p>
  </div>
</details>

### 3. RAG Retrieval Performance

To ensure the Knowledge Agent grounds its responses in accurate clinical literature without hallucination, the Retrieval-Augmented Generation (RAG) pipeline was evaluated against a dataset of **150 clinical queries** derived from QUANTEC guidelines.

The pipeline utilizes a two-stage retrieval architecture: dense embedding search (`BGE-base-en-v1.5`) followed by cross-encoder reranking (`BGE-reranker-base`) combined with custom Organ-Aware Filtering to prevent cross-organ metric contamination.

**150-Question Benchmark Results:**
* **MRR Score (Mean Reciprocal Rank): `0.9419`** — The correct QUANTEC guideline chunk is typically returned as the top-ranked result.
* **Hit@1 Rate: `91.3%`** — Top-1 Accuracy. In 9 out of 10 queries, the very first retrieved document perfectly contains the required clinical metrics.
* **Hit@3 Rate: `96.7%`** — Top-3 Recall. 
* **Hit@5 Rate: `98.0%`** — Generation Ceiling. This indicates that relevant clinical evidence is reliably surfaced within the top-5 retrieved chunks, providing a strong foundation for downstream reasoning.
* **Context Precision@5: `97.7%` (Organ Cleanliness)** — In medical AI, retrieving literature for the wrong organ is a safety-critical flaw (e.g., pulling a bladder dose limit when asked about the rectum). By actively filtering chunks *before* ranking, the system achieves high precision. This means 97.7% of the clinical literature retrieved is strictly isolated to the specific organ the user asked about, substantially reducing the risk of cross-organ contamination in retrieved evidence.

<figure>
  <img src="assets/rag_metrics.png" alt="RAG Metrics">
  <figcaption><strong>Figure 5.</strong> RAG pipeline retrieval performance on the 150-question QUANTEC benchmark, showing MRR, Hit@k, and Context Precision metrics.</figcaption>
</figure>

<h2 id="key-design-decisions">Key Design Decisions</h2>

| Decision | Rationale |
|---|---|
| **LangGraph StateGraph** | Enables a cyclic Planner → Agent → Planner loop, allowing multi-step reasoning across agents |
| **Deterministic Math Tools** | QUANTEC safety checks use hardcoded Python functions (not LLM generation) to eliminate numerical hallucinations |
| **BGE Cross-Encoder Reranking** | Two-stage retrieval (embedding similarity → cross-encoder reranking) maximizes precision of clinical evidence |
| **Organ-Aware Retrieval** | RAG queries are filtered by detected organ to prevent cross-organ contamination in retrieved chunks |
| **LLM-as-Judge Evaluation** | GPT-OSS 120B scores responses on a 0-100 rubric with structured pass/fail classification |
| **Separate Baseline Runner** | Enables direct A/B comparison: same questions evaluated by raw LLM vs. full agentic pipeline |

<h2 id="tech-stack">Tech Stack</h2>

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

<h2 id="limitations--future-work">Limitations & Future Work</h2>

While RadExplain demonstrates strong automated evaluation metrics, it is designed strictly as a **decision-support research prototype**, not a deployed medical device. To ensure academic rigor and transparency, the following limitations are explicitly noted:

* **Preliminary Automated Benchmarking:** The current evaluation relies on an LLM-as-a-Judge (GPT-OSS 120B) to establish baseline safety metrics. While the judge employs a strict deterministic 100-point clinical rubric, this serves as preliminary validation.
* **Domain Expert Curation:** The evaluation dataset, QUANTEC reference limits, and deterministic mathematical constraints were heavily curated and verified in collaboration with a **Medical Physics domain expert** (M.Sc. Physics), ensuring the foundational dose arithmetic and baseline ground truths are physically and clinically sound.
* **Documented Failure Modes:** Despite the agentic architecture's high performance, we manually identified and documented **14 extreme edge cases** where the baseline LLM's clinical reasoning collapses (e.g., *Fractionation Context Confusion*, *Calculation Volume Omission*). These failure cases illustrate the potential value of multi-agent deterministic tooling over end-to-end LLM generation for safety-critical clinical tasks.
* **Future Work:** Transitioning from automated systems benchmarking to formal, blinded clinical validation with board-certified radiation oncologists.
