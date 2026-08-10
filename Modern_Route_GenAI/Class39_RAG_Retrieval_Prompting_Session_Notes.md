# 📚 RAG Retrieval Techniques — Recap, Prompting & Multi-modal RAG Kickoff
### 📋 Weekend Live Session Notes

**🎙️ Speaker:** Sunny (Mentor)
**⏱️ Duration:** ~3 hours 15 mins (incl. 20-min dinner break + doubt session)
**🎯 Session Type:** Retrieval Recap + Prompting Deep-Dive + Live Q&A
**📅 Session Date:** 8th August (covering catch-up from 29th July & 5th August Wednesday classes)

---

## 🧭 Session Roadmap

```mermaid
flowchart LR
    A["🔁 Quick Recap<br/>Retrieval Techniques"] --> B["💬 Prompting<br/>Fundamentals & Practicals"]
    B --> C["🗂️ Prompt<br/>Management"]
    C --> D["🏗️ Multi-modal RAG<br/>Project Setup (Preview)"]

    style A fill:#dbeafe,stroke:#3b82f6,color:#000
    style B fill:#fef3c7,stroke:#f59e0b,color:#000
    style C fill:#ede9fe,stroke:#8b5cf6,color:#000
    style D fill:#d1fae5,stroke:#10b981,color:#000
```

> 💡 **Context:** No weekend class was held the previous Sat/Sun — two Wednesday classes (29th July & 5th Aug) covered retrieval instead. This session gives the quick recap before moving to prompting and the Multi-modal RAG project (using the previously parsed PDF).

---

## 🗺️ The Complete Retrieval Pipeline (Line Diagram)

Everything starts with the **user query** — this is the mental model to hold onto for the entire retrieval chapter.

```mermaid
flowchart TD
    Q["❓ User Query"] --> QT["🔄 Query Transformation<br/>Rewrite • Expand • Multi-Query • HyDE • Decompose"]
    QT --> R{"🧭 Routing<br/>Where does the data live?"}
    R -->|Vector DB| VDB["🧩 Vector Database<br/>(Semantic Retrieval)"]
    R -->|Other Sources| OS["🗄️ SQL DB • Knowledge Graph<br/>Web Search • APIs"]
    VDB --> RET["📥 Retrieval Techniques<br/>Sparse • Dense • Hybrid • Parent Doc<br/>Sentence Window • Multi-Hop • Metadata Filter"]
    RET --> F["⚖️ Result Fusion<br/>Weighted Fusion • Reciprocal Rank Fusion"]
    F --> RR["🏆 Re-ranking<br/>(Cross-Encoder)"]
    RR --> CC["✂️ Contextual Compression<br/>(Trim irrelevant text)"]
    CC --> LLM["🤖 LLM"]

    style Q fill:#dbeafe,stroke:#3b82f6,color:#000
    style QT fill:#fde68a,stroke:#f59e0b,color:#000
    style VDB fill:#c4b5fd,stroke:#8b5cf6,color:#000
    style OS fill:#fca5a5,stroke:#ef4444,color:#000
    style RET fill:#a5b4fc,stroke:#6366f1,color:#000
    style F fill:#93c5fd,stroke:#3b82f6,color:#000
    style RR fill:#6ee7b7,stroke:#10b981,color:#000
    style CC fill:#fbcfe8,stroke:#ec4899,color:#000
    style LLM fill:#6366f1,color:#fff,stroke:#4338ca,stroke-width:2px
```

> ⚠️ **Important caveat (repeated throughout the session):** All of these techniques are **experimental**. Sunny was explicit — *"I cannot give you the guarantee that this technique will work very well... you will have to try it out."* When to apply which retriever is decided through **evaluation**, not a fixed rulebook.

> 🌐 **Real-world note:** Retrieval techniques discussed here assume the **vector database** as the source. In real production systems, data can also live in SQL DBs, knowledge graphs, or behind APIs — routing logic to decide *where* to fetch from is covered later under **Agentic RAG**.

---

## 🧩 Query Transformation Techniques

### 1️⃣ HyDE (Hypothetical Document Embedding)

```mermaid
flowchart LR
    UQ["User Query"] --> LLM1["LLM generates<br/>hypothetical answer"]
    LLM1 --> EMB["Convert to<br/>embedding"]
    EMB --> VDB["Vector DB search<br/>using that embedding"]
    VDB --> RES["Retrieve real,<br/>similar document"]

    style UQ fill:#dbeafe,stroke:#3b82f6,color:#000
    style LLM1 fill:#fde68a,stroke:#f59e0b,color:#000
    style EMB fill:#fca5a5,stroke:#ef4444,color:#000
    style VDB fill:#c4b5fd,stroke:#8b5cf6,color:#000
    style RES fill:#6ee7b7,stroke:#10b981,color:#000
```

- **Idea:** Instead of embedding the raw query, generate a hypothetical answer first — its embedding tends to be **semantically closer** to real documents than the bare question.
- **Example:** *"How does LLM2 improve safety?"* → LLM drafts a plausible answer → that answer is embedded → vector DB retrieves the real matching document.
- 🎯 **Goal:** Improve semantic matching.
- ⚠️ Doesn't work well if the LLM has zero domain knowledge (e.g., private company docs) — in that case, guide it with explicit context/examples inside the prompt itself.

### 2️⃣ Multi-Query Retriever

- Generates **multiple rephrasings** of the same user query → runs retrieval for each → merges/deduplicates results → passes final set to the LLM.
- 🎯 **Goal:** Improve recall (also called **query expansion**).

### 🆚 HyDE vs Multi-Query

| | 🧪 HyDE | 🔀 Multi-Query |
|---|---|---|
| Generates | One hypothetical **answer** | Multiple **query versions** |
| Search basis | Answer embedding | Several query embeddings |
| Main goal | Improve semantic matching | Improve recall |
| Category | Query Transformation | Query Transformation |

---

## 📥 Core Retrieval Mechanisms (Actual retrieval inside the Vector DB)

### 🌳 Parent Document Retriever vs 🪟 Sentence Window Retriever

```mermaid
flowchart TD
    subgraph Parent["🌳 Parent Document Retriever"]
        P1["Search: small child chunk"] --> P2["Return: large parent section"]
    end
    subgraph Sentence["🪟 Sentence Window Retriever"]
        S1["Search: single sentence"] --> S2["Return: sentence + nearby sentences"]
    end

    style P1 fill:#dbeafe,stroke:#3b82f6,color:#000
    style P2 fill:#6ee7b7,stroke:#10b981,color:#000
    style S1 fill:#fde68a,stroke:#f59e0b,color:#000
    style S2 fill:#fca5a5,stroke:#ef4444,color:#000
```

| | 🌳 Parent Document | 🪟 Sentence Window |
|---|---|---|
| Searches on | Small chunk | Single sentence |
| Returns | Larger parent section/chunk | Sentence + surrounding sentences |
| Why | Small chunks = accurate retrieval; large sections = better context | One sentence may lack context for the LLM alone |
| Example | 2,000-token policy doc → best child chunk found → full parent section returned | Doc split into sentences → best sentence found → returns window (e.g. sentence 8–10) |
| ⚠️ Known limitation | Can hit **LLM context window limits** if parent sections are large | Window size is a tunable hyperparameter |

### 🔁 Multi-Hop Retriever

```mermaid
flowchart LR
    Q["User Query:<br/>Who founded the company<br/>that developed Llama 2?"] --> H1["Hop 1:<br/>Which company developed Llama 2?"]
    H1 --> A1["Answer: Meta"]
    A1 --> H2["Hop 2:<br/>Who founded Meta?"]
    H2 --> A2["Answer: Mark Zuckerberg<br/>& co-founders"]
    A2 --> FA["✅ Final Combined Answer"]

    style Q fill:#dbeafe,stroke:#3b82f6,color:#000
    style H1 fill:#fde68a,stroke:#f59e0b,color:#000
    style A1 fill:#c4b5fd,stroke:#8b5cf6,color:#000
    style H2 fill:#fde68a,stroke:#f59e0b,color:#000
    style A2 fill:#c4b5fd,stroke:#8b5cf6,color:#000
    style FA fill:#6ee7b7,stroke:#10b981,color:#000
```

- Used when **one retrieval isn't enough** — the system retrieves a piece of info, generates the next query from that answer, retrieves again, and repeats until it has enough evidence.
- 🆚 **Multi-Query vs Multi-Hop:** Multi-Query fires several queries **in parallel** and merges results; Multi-Hop is a **sequential chain** where each answer feeds the next query.

---

## ⚖️ Hybrid Retrieval, Fusion & Re-ranking

```mermaid
flowchart TD
    BM["🔤 Sparse (BM25) Retriever"] --> COMBINE{"Combine Results"}
    DE["🧠 Dense/Semantic Retriever"] --> COMBINE
    COMBINE -->|No ranking logic| SM["⚠️ Simple Merge<br/>(not production-grade)"]
    COMBINE -->|Score-based| WF["📊 Weighted Fusion"]
    COMBINE -->|Rank-based| RRF["🏅 Reciprocal Rank Fusion (RRF)"]
    WF --> FR["🎯 Final Ranked Output"]
    RRF --> FR
    FR --> RRK["🥇 Re-ranking (Cross-Encoder)<br/>Top 50 → Top 5"]

    style BM fill:#fca5a5,stroke:#ef4444,color:#000
    style DE fill:#93c5fd,stroke:#3b82f6,color:#000
    style SM fill:#fee2e2,stroke:#ef4444,color:#000
    style WF fill:#fde68a,stroke:#f59e0b,color:#000
    style RRF fill:#c4b5fd,stroke:#8b5cf6,color:#000
    style FR fill:#6ee7b7,stroke:#10b981,color:#000
    style RRK fill:#6366f1,color:#fff,stroke:#4338ca,stroke-width:2px
```

### 📊 Weighted Fusion
- Combines **scores** from multiple retrievers using assigned weights (hyperparameter — often 50/50, or tuned by evaluation).
- **Formula:** `Final Score = (weight₁ × score₁) + (weight₂ × score₂)`
- Example: BM25 score `0.08` (weight 0.4) + Vector score `0.9` (weight 0.6) → Final ≈ `0.86`
- Scores must be **normalized** if retrievers use different scales.

### 🏅 Reciprocal Rank Fusion (RRF)
- Combines **rank positions**, not raw scores.
- **Formula:** `RRF Score = 1 / (k + rank of document)`
- A document ranked near the top across **multiple lists** gets a higher combined score. `k` is a small constant to avoid divide-by-zero/infinite scores.

### 🔀 MMR (Maximum Marginal Relevance)
- An **advanced search technique** (variant of cosine similarity/dot product) that balances **relevance vs diversity**, avoiding duplicate/repetitive documents.

**Formula:**
`MMR = λ × Sim(query, candidate) − (1−λ) × Max[Sim(candidate, already_selected)]`

- `λ` (0 to 1) controls the trade-off: higher λ → more relevance-focused; lower λ → more diversity-focused.
- **Analogy given in class:** Recommending YouTube videos — without MMR you get "Python Tutorial 1, 2, 3, 4"; with MMR you get "Python Tutorial, Python Project, Python Interview Qs, Python Best Practices" — still relevant, but diversified.

### ✂️ Contextual Compression
- **Two-stage wrapper** around any retriever:
  1. Retrieve relevant documents (broadly).
  2. Compress/trim them against the query — keep only the useful text, discard the rest — **before** sending to the LLM.
- 🆚 Different from re-ranking: **Re-ranking reorders documents; Contextual Compression trims their content.**
- Implementations: `LLMChainExtractor` (uses an LLM to extract relevant text), `EmbeddingFilter` (no LLM needed — filters via embedding similarity), or a cross-encoder re-ranker acting as compressor.
- 💡 Why not just reduce `k`? Because a single document might be 1,000 tokens with only 20 useful ones — reducing `k` still sends the full noisy chunk. Compression solves this directly.

---

## 📌 Retrieval Techniques — Quick Reference

| Technique | Category | One-liner |
|---|---|---|
| Query Rewriting / Expansion | Query Transformation | Reshape the query before hitting the DB |
| HyDE | Query Transformation | Embed a hypothetical answer instead of the raw query |
| Multi-Query Retriever | Query Transformation | Generate & merge results from multiple query versions |
| Sparse Retrieval (BM25) | Retrieval Mechanism | Keyword-based, no dense embeddings |
| Dense Retrieval | Retrieval Mechanism | Embedding-based semantic search |
| Hybrid Retrieval | Retrieval Mechanism | Combination of sparse + dense |
| Parent Document Retriever | Retrieval Mechanism | Search small, return large parent chunk |
| Sentence Window Retriever | Retrieval Mechanism | Search one sentence, return surrounding window |
| Multi-Hop Retriever | Retrieval Mechanism | Sequential, evidence-chained retrieval |
| Metadata Filtering | Retrieval Mechanism | Filter results using metadata |
| Weighted Fusion | Result Fusion | Score-based combination with weights |
| Reciprocal Rank Fusion (RRF) | Result Fusion | Rank-based combination |
| MMR | Search Technique | Relevance + diversity balancing |
| Re-ranking (Cross-Encoder) | Post-processing | Reorders top candidates |
| Contextual Compression | Post-processing | Trims irrelevant text from retrieved chunks |

✅ All of the above (HyDE, Multi-Query, Parent Doc, Hybrid, RRF, Contextual Compression, etc.) are **directly available inside LangChain** — import statements were shared in the reference PDF, with full code in the shared notebook.

---

## 💬 Prompting Fundamentals

### 🧱 Anatomy of a Full-Fledged Prompt

```mermaid
flowchart LR
    I["📝 Instruction<br/>What needs to be done"] --> Q2["❓ Question"]
    Q2 --> C["📚 Context"]
    C --> E["💡 Example"]
    E --> CO["🚧 Constraint"]
    CO --> OF["📤 Output Format"]

    style I fill:#dbeafe,stroke:#3b82f6,color:#000
    style Q2 fill:#fde68a,stroke:#f59e0b,color:#000
    style C fill:#fca5a5,stroke:#ef4444,color:#000
    style E fill:#c4b5fd,stroke:#8b5cf6,color:#000
    style CO fill:#93c5fd,stroke:#3b82f6,color:#000
    style OF fill:#6ee7b7,stroke:#10b981,color:#000
```

> This full structure applies to **industry-standard, production-grade prompts** — not quick one-line test prompts.

### 🎯 Zero-shot vs One-shot vs Few-shot

| Type | Description | Typical Use |
|---|---|---|
| 🚫 Zero-shot | No examples given, just a direct question | Simple, generic tasks |
| 1️⃣ One-shot | One example provided | Light guidance needed |
| 🔢 Few-shot | Multiple examples + context provided | **RAG systems** — context + examples drive the response |

---

## 🛠️ Practical: PromptTemplate vs ChatPromptTemplate

| | `PromptTemplate` (older/deprecated-ish) | `ChatPromptTemplate` (latest) |
|---|---|---|
| Output | Single flat text prompt | Structured chat message (system/human/assistant roles) |
| Best for | Simple prompts | Advanced prompts with role separation |
| Recommendation | Still valid for basic use | ✅ Preferred for production systems |

**Runtime variables demoed:** `topic`, `audience`, `word_limit` — passed at runtime and filled into the template before hitting the model (GPT 5.6 used in the demo).

---

## 🧩 Conditional Prompting with Jinja2 Templating

```mermaid
flowchart TD
    T["📄 Jinja Template<br/>{{ variables }} + {% if %} logic"] --> V["🔢 Runtime Variables<br/>topic, audience, include_example, label"]
    V --> F["✅ Final Rendered Prompt"]

    style T fill:#dbeafe,stroke:#3b82f6,color:#000
    style V fill:#fde68a,stroke:#f59e0b,color:#000
    style F fill:#6ee7b7,stroke:#10b981,color:#000
```

- Jinja2 templating lets you write **if/else conditions** and even **for-loops** directly inside a prompt string.
- **Demoed example:** A prompt that changes wording based on `label` (`beginner` → *"use very simple English, avoid complex terminology"*; `advanced` → different phrasing).
- Works with both `PromptTemplate` (→ dynamic single-text prompt) and `ChatPromptTemplate` (→ dynamic **role-based** chat prompt, the more advanced/preferred option for real projects).
- 🎯 **Real-world use case shared:** Sunny confirmed he's used this exact Jinja conditional-prompting pattern in one of his own production projects.

> 💡 **Analogy for the doubt session:** Just like a chatbot conditionally responding *"Sure! Which flight are you looking for?"* — this behavior is defined via **prompt instructions/constraints**, not separate hardcoding, unless you're fine-tuning the model itself.

---

## 🗂️ Prompt Management (Prompt Registry)

```mermaid
flowchart LR
    P["🗃️ Prompt Storage Options"] --> J["📋 JSON File"]
    P --> PY["🐍 Python (.py) File"]
    P --> LS["🔗 LangSmith Prompt Hub"]
    P --> PH["🌐 PromptHub Platform"]
    P --> DB["🗄️ DynamoDB / In-house Store"]

    style P fill:#6366f1,color:#fff,stroke:#4338ca,stroke-width:2px
    style J fill:#dbeafe,stroke:#3b82f6,color:#000
    style PY fill:#fde68a,stroke:#f59e0b,color:#000
    style LS fill:#c4b5fd,stroke:#8b5cf6,color:#000
    style PH fill:#93c5fd,stroke:#3b82f6,color:#000
    style DB fill:#6ee7b7,stroke:#10b981,color:#000
```

- **Why:** Real-time systems can have **tens to hundreds of prompts** — keeping them inline in code doesn't scale. This is **modular coding**: prompts, config, exceptions, and logic all live in separate folders/files (demoed via Sunny's "Document Portal" GitHub project structure).
- **JSON approach demoed:**
  - Simple flat JSON: `{ "rag_prompt": "..." }` → load with a Python dict lookup → `.format()` with runtime values.
  - Nested JSON: includes `version`, `template`, `format`, `message`, `metadata` — a custom `load_prompt()` function walks the hierarchy (`config.message.role.template`) and builds a `PromptTemplate` object, then `.invoke()`s it with runtime keys.
  - ⚠️ Gotcha demoed live: passing the wrong key (e.g., `summarization_prompt` needing `number_of_points` + `text` keys) throws a `KeyError` — keys must match exactly.
- **Enterprise-grade options:** LangSmith Prompt Hub and PromptHub platform — both allow pushing/versioning prompts outside the codebase (to be demoed in an upcoming session).

---

## 🏗️ Multi-modal RAG Project — Setup Preview

```mermaid
flowchart TD
    S1["📁 Create Project Folder<br/>mkdir Multi-modal-RAG-Project"] --> S2["🐍 Create Virtual Env<br/>uv venv --python 3.12"]
    S2 --> S3["📄 Add requirements.txt<br/>+ .gitignore + .env"]
    S3 --> S4["🔧 git init"]
    S4 --> S5["📦 __init__.py inside src/"]
    S5 --> S6["🔀 3 Pipelines to Build:<br/>Data Parsing • Retrieval • Generation"]
    S6 --> S7["🎨 Optional: Streamlit Front-end<br/>+ API layer + Logger + Custom Exceptions"]

    style S1 fill:#fef3c7,stroke:#f59e0b,color:#000
    style S2 fill:#fde68a,stroke:#f59e0b,color:#000
    style S3 fill:#fca5a5,stroke:#ef4444,color:#000
    style S4 fill:#93c5fd,stroke:#3b82f6,color:#000
    style S5 fill:#a5b4fc,stroke:#6366f1,color:#000
    style S6 fill:#c4b5fd,stroke:#8b5cf6,color:#000
    style S7 fill:#6ee7b7,stroke:#10b981,color:#000
```

> 📌 The Multi-modal RAG project will be built entirely inside **VS Code**, using the PDF from earlier classes as source data. Remaining practical retrieval demos (not covered live today) will be added to the notebook before the project kickoff.

---

## ❓ Live Q&A Highlights

| Question | Answer |
|---|---|
| **(Shiv)** How is the `-0.14` / final MMR score value calculated from `0.5 × 0.40`? | It's the **similarity score** fed into the MMR formula — final value is the computed MMR score, not the raw similarity itself. |
| **(Shiv)** How does a chatbot know to ask "which flight?" when a user says "book a flight"? | Defined via **explicit prompt instructions/constraints** (e.g., "if no destination is given, ask the user for it") — not separate hardcoding unless fine-tuning. |
| **(Shiv)** How do chatbots generate rich/dynamic UI (HTML) responses? | The LLM can generate output in HTML (or any format); your backend logic converts/render that output for the browser — it's custom engineering on top of the LLM response. |
| **(Bala)** How do production systems keep RAG pipelines (query transform + re-ranking etc.) low-latency? | Depends on the full system: **server/RAM specs, smaller embedding dimensions, smaller models, caching (e.g., Redis)**, and minimizing `k`/context size. Ingestion and retrieval should be **decoupled pipelines**. |
| **(Bala)** How does HyDE work if the LLM doesn't know private/company documents? | The hypothetical answer is generated based on **explicit context given in the prompt itself** — the LLM doesn't need prior knowledge if you feed it enough grounding context (prompts can run 500–1000 lines if needed). |
| **(Bala)** Isn't Parent Document Retrieval limited by the LLM's context window? | **Yes, confirmed as a real limitation** — mitigated by keeping parent chunk sizes reasonably small. |
| **(Sunil)** How does the system reliably extract prompt parameters (like `topic`) from unstructured user input? | This is a form of **query decomposition** — done using LLM capability (or regex/custom logic for simple, well-defined cases) to parse structured variables out of free-text input before filling the prompt template. |

---

## ✅ Action Items for Learners

- [ ] 📖 Revise the shared **retriever PDF** (`retrieverupdated.pdf`) — updated notes pushed to GitHub
- [ ] 💻 Go through the previous class's IPYNB notebook if HyDE/Multi-Query/Hybrid retrieval practicals were missed
- [ ] 🧪 Complete the **homework** on Fusion / MMR / Contextual Compression techniques (if not already done)
- [ ] 📁 Set up the **Multi-modal RAG project skeleton** locally (venv, requirements.txt, .gitignore, .env, git init) ahead of the next session
- [ ] 🐍 Get comfortable with Python imports/modular structure (`__init__.py`, prompt registry pattern) before the project walkthrough
- [ ] 🎯 Await Sunny's **mega assignment** (to be released after Multi-modal RAG is completed)
- [ ] 📅 Watch for confirmation on the **15th August class status** (via committee chat/email)

---

*📝 Notes compiled from the full session transcript — Retrieval Recap, Prompting & Multi-modal RAG Setup.*
