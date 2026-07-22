# 🧠 LLM Fine-Tuning Deep Dive: Hugging Face vs Unsloth
### 📋 Session Notes — Generative AI / Agentic AI Course

**🎙️ Speaker:** Sunny Savita (Generative AI Engineer, PwC)
**⏱️ Duration:** ~4 hours (incl. dedicated doubt session) | **🎯 Session Type:** Live Class + Q&A

---

## 🧭 Where This Class Fits

This is a continuation of the **fine-tuning module**. The previous class covered fine-tuning fundamentals; this session pivots to the **frameworks** used to actually implement it.

```mermaid
flowchart LR
    A["✅ Previous Class<br/>What is fine-tuning?<br/>Which stage can we fine-tune from?"] --> B["🔧 Today's Class<br/>Hugging Face vs Unsloth<br/>frameworks + practical"]
    B --> C["🚀 Next Classes<br/>Actual fine-tuning<br/>via HF, then via Unsloth"]

    style B fill:#6366f1,color:#fff,stroke:#4338ca,stroke-width:2px
    style A fill:#dbeafe,color:#000,stroke:#3b82f6
    style C fill:#d1fae5,color:#000,stroke:#10b981
```

> 💡 **Recap check:** Fine-tuning = retraining a model's weights (in the decoder's self-attention & feed-forward layers) on custom data. It can be performed starting from *any* of the 3 LLM training stages — Pre-training, SFT, or Preference Tuning.

---

## 🏗️ The 3 Stages of LLM Training (Quick Recap)

```mermaid
flowchart LR
    P1["1️⃣ Pre-training<br/>Raw/base model"] --> P2["2️⃣ SFT<br/>(Instruction Fine-Tuning)"]
    P2 --> P3["3️⃣ Preference Tuning<br/>RLHF • DPO • PPO • ORPO • GRPO"]

    style P1 fill:#fef3c7,color:#000,stroke:#f59e0b
    style P2 fill:#fca5a5,color:#000,stroke:#ef4444
    style P3 fill:#a5b4fc,color:#000,stroke:#6366f1
```

✅ Fine-tuning can start from **any** stage — pre-trained/raw or already instruct-tuned.
✅ SFT methods split into **Old-school (full fine-tuning)** and **PEFT** (LoRA, DoRA, QLoRA — parameter-efficient).
✅ Preference alignment methods: RLHF, DPO (main focus), plus PPO/GRPO/ORPO (glimpsed).

---

## 🧰 Two Core Frameworks for Fine-Tuning

| | 🤗 Hugging Face | ⚡ Unsloth |
|---|---|---|
| Nature | The **foundation** of the fine-tuning ecosystem | An **ultra-optimized performance layer** built on top of Hugging Face |
| Speed | Slower, standard baseline | **2–3x faster training** |
| VRAM usage | High (e.g., 24GB to load a Llama-class model) | Very low (same task in ~7–8GB) |
| Long-context training | Runs into out-of-memory errors sooner | Can train 20K–30K+ tokens even on modest GPUs |
| Cost | More expensive (time = compute cost) | Cheaper, works even on **free GPUs** (e.g., T4) |
| Accuracy | Baseline | **No meaningful accuracy loss** — no approximation tricks used |
| Maturity | Older, foundational | Newer, but already an industry favorite for real-time fine-tuning |

> 🗣️ *"Whenever we are talking about the Hugging Face, so there is some use issue — the training is going to be slow, it's going to consume huge VRAM, and it is expensive."* — Sunny, explaining why Unsloth exists

### 📊 Concrete Comparison (as claimed by Unsloth)

```mermaid
flowchart TD
    A["🔢 Same fine-tuning task"] --> B["🤗 Hugging Face<br/>~10 hrs • ~40GB RAM"]
    A --> C["⚡ Unsloth<br/>~5 hrs • 12-16GB RAM"]

    D["🖥️ 80GB GPU on Hugging Face"] --> E["Trains only ~28K tokens"]
    F["🖥️ 24GB GPU on Unsloth"] --> G["Trains the same ~28K tokens<br/>(30–40 PDF pages worth)"]

    style B fill:#fca5a5,color:#000,stroke:#ef4444
    style C fill:#86efac,color:#000,stroke:#10b981
    style E fill:#fca5a5,color:#000,stroke:#ef4444
    style G fill:#86efac,color:#000,stroke:#10b981
```

---

## 🔍 Why Is Unsloth So Much Faster? (Under the Hood)

```mermaid
flowchart TD
    A["🎮 GPU Hardware"] --> B["🔧 CUDA Kernel<br/>(NVIDIA's low-level GPU programming layer)"]
    B --> C["🚀 Triton Kernel<br/>Python-like GPU language,<br/>optimization layer on top of CUDA"]
    C --> D["⚡ Unsloth Optimizations"]
    D --> E["Fuse Attention<br/>(updated attention mechanism)"]
    D --> F["Smart checkpointing"]
    D --> G["Manual backpropagation"]
    D --> H["Automatic sequence packing"]

    style A fill:#e0e7ff,color:#000,stroke:#6366f1
    style B fill:#c4b5fd,color:#000,stroke:#8b5cf6
    style C fill:#a5b4fc,color:#000,stroke:#6366f1
    style D fill:#6366f1,color:#fff,stroke:#4338ca,stroke-width:2px
```

**Key talking points (interview-ready):**
- Unsloth builds **custom Triton kernels** on top of the standard CUDA kernel for faster tensor processing.
- It introduces an updated attention mechanism called **Fuse Attention**.
- It uses **flash attention**, manual backpropagation, smart checkpointing, and auto-sequence packing.
- Crucially, it does **not** rely on approximation/precision-cutting tricks — so accuracy loss is negligible.
- Backend stack: PyTorch → CUDA → Triton kernel → GPU. Internally still uses `transformers`, `peft`, and `trl`, just heavily optimized.

---

## 🧱 The Hugging Face Ecosystem

```mermaid
flowchart TD
    HF["🤗 Hugging Face<br/>(AI company — similar business model to LangChain)"] --> Hub["📦 Hugging Face Hub<br/>(like GitHub — upload/download models)"]
    HF --> DS["📊 Dataset Hub<br/>(community + custom datasets)"]
    HF --> Space["🖥️ Hugging Face Space<br/>(host your app — limited scalability)"]
    HF --> Ent["💰 Hugging Face Enterprise<br/>(paid — this is how HF earns money)"]

    style HF fill:#6366f1,color:#fff,stroke:#4338ca,stroke-width:2px
    style Ent fill:#fbbf24,color:#000,stroke:#f59e0b
```

### 🆚 Native vs. Outsider Libraries

Not everything under the Hugging Face umbrella was *built* by Hugging Face — but it's all part of the ecosystem.

| ✅ Native (built by HF) | 🔗 Outsider (integrated, built elsewhere) |
|---|---|
| `transformers`, `datasets`, `tokenizers` | `bits-and-bytes` (quantization) |
| `peft`, `accelerate`, `trl` | `sentence-transformers` |
| `diffusers`, `evaluate`, `optimum`, `safetensors` | `timm`, `gradio`, `distil-label` |

> 💡 Similarly, in the RAG/Agentic space, **LangChain** is a separate open-source company (LangGraph, LangSmith, LangFuse) — it monetizes via enterprise support on LangSmith/LangFuse, just like HF monetizes via HF Enterprise. **Unsloth, Llama Factory, and Axolotl** are all libraries built *on top of* the Hugging Face fine-tuning foundation, with their own optimizations layered in.

---

## 🗺️ The Full Fine-Tuning Pipeline (Hugging Face Workflow)

```mermaid
flowchart TD
    S1["📊 1. Dataset<br/>HF Dataset Hub or custom data"] --> S2["🧹 2. Preprocessing & Cleaning"]
    S2 --> S3["🔤 3. Tokenization"]
    S3 --> S4["🤖 4. Load Transformer-based Model"]
    S4 --> S5["🎯 5. PEFT (LoRA adapters)<br/>full fine-tuning usually not feasible"]
    S5 --> S6["🗜️ 6. Quantization (bits-and-bytes)<br/>→ QLoRA / QDoRA"]
    S6 --> S7["🖧 7. Distributed Training (optional)<br/>via Accelerate for multi-GPU"]
    S7 --> S8["⚖️ 8. Preference Tuning (TRL)<br/>RLHF, DPO, PPO, GRPO"]
    S8 --> S9["📏 9. Evaluation<br/>accuracy, BLEU, ROUGE, perplexity, benchmarks"]
    S9 --> S10["💾 10. Save Checkpoint<br/>safetensors / GGUF / GGML"]
    S10 --> S11["☁️ 11. Push to Hugging Face Hub"]
    S11 --> S12["🚀 12. Inference & Deployment<br/>incl. HF Space hosting"]
    S12 --> S13["📈 13. Monitoring & Iteration<br/>latency, cost, token usage,<br/>hyperparameter tuning, further RLHF/DPO"]

    style S1 fill:#fef3c7,color:#000,stroke:#f59e0b
    style S5 fill:#fca5a5,color:#000,stroke:#ef4444
    style S6 fill:#fca5a5,color:#000,stroke:#ef4444
    style S8 fill:#a5b4fc,color:#000,stroke:#6366f1
    style S13 fill:#6ee7b7,color:#000,stroke:#10b981
```

> ⚠️ Note: guardrails and RAG are **not** part of this training-level pipeline — those apply during inference, if the model hallucinates or drifts off-flow.

---

## 💻 Where to Actually Run Training (GPU Options)

Local GPUs (gaming/commercial cards) are **not built for LLM training** — even a good local card (e.g., RTX 5070 Ti) can only do basic training. For this module, all practicals move to **cloud notebooks**.

```mermaid
flowchart LR
    A["🎓 Learning / Small Training"] --> B["🟢 Google Colab<br/>(recommended — free T4 GPU)"]
    A --> C["🟢 Kaggle Notebooks<br/>(also free GPU)"]
    D["🏭 Production-Grade Training"] --> E["🟡 RunPod"]
    D --> F["🟡 Paperspace Gradient"]
    D --> G["🟡 Vast.ai"]
    D --> H["🟡 Lightning.ai"]
    D --> I["🟡 Cloud GPUs (AWS/GCP/Azure)"]

    style B fill:#86efac,color:#000,stroke:#10b981
    style C fill:#86efac,color:#000,stroke:#10b981
```

### Google Colab Free Tier Resources (as demoed in class)
| Resource | Amount |
|---|---|
| GPU | T4 (free tier) |
| GPU RAM (VRAM) | ~15 GB |
| System RAM | ~12.7 GB |
| Disk | ~112 GB (≈53 GB free) |

**Paid tiers:** Colab Pro (~$11/mo, 100 compute units, faster GPU) → Colab Pro+ (~$58/mo) → Colab Enterprise (org-level).

---

## 🛠️ Practical Walkthrough — Hugging Face Hub API

Demoed live in a fresh Colab notebook (`Hugging Face Explore Live Class.ipynb`).

### 🔑 Step 1: Secrets Setup
- Generate a **Read Token** and a **Write Token** from Hugging Face → Profile → Access Tokens.
- Store them in Colab's **Secrets** (key 🔑 icon).
- ⚠️ **Important gotcha raised repeatedly in doubts:** the environment variable Hugging Face auto-detects is named exactly **`HF_TOKEN`** — not "HF Read"/"HF Write" (those custom names were just for the class's own understanding and won't be picked up automatically unless passed manually into the token parameter).

| Token Type | Use Case |
|---|---|
| 📖 Read Token | Downloading/reading models, datasets from HF Hub |
| ✍️ Write Token | Uploading your own model/dataset to HF Hub |

### 🧪 Step 2: `HfApi` Object — What You Can Do

```mermaid
flowchart LR
    API["🤗 HfApi() object"] --> M1["model_info(repo_id)<br/>→ tags, commits, downloads, likes"]
    API --> M2["list_models(search='llama', limit=50)<br/>→ browse/search model repos"]
    API --> M3["list_repo_files(repo_id)<br/>→ see all files in a repo"]
    API --> M4["dataset_info(repo_id)<br/>→ metadata for any dataset"]

    style API fill:#6366f1,color:#fff,stroke:#4338ca,stroke-width:2px
```

### 💬 Step 3: Calling a Model for Inference

```python
from huggingface_hub import InferenceClient

client = InferenceClient(...)
response = client.chat.completions.create(...)  # prompt goes here
```

- This is a direct **API call** to Hugging Face — the model is not downloaded/hosted locally.
- The **same** call can be made via **LangChain's Hugging Face wrapper** (`ChatHuggingFace` + `HuggingFaceEndpoint`), which also supports prompt templates, structured output, and `RunnablePassthrough` — this becomes essential later in the RAG chapter.

> 🎯 Takeaway demoed: any free Hugging Face model can be pulled into a RAG or agentic pipeline via LangChain's wrapper.

---

## 📚 What's Deferred to Next Class

Sunny walked through (but didn't fully execute) a second, more detailed notebook covering:

- Different Hugging Face login methods
- Loading datasets for unsupervised pre-training, SFT, and RLHF; streaming large datasets
- Building & uploading a **custom dataset** in HF format
- Tokenization deep-dive: BPE, SentencePiece, custom tokenizer training
- Loading pre-trained tokenizers per model family (Llama, Mistral, Qwen, etc.)
- Loading models via `AutoModel` / `AutoModelForCausalLM`
- Downloading model files locally via `snapshot_download`
- Evaluation metrics (deferred to a dedicated evaluation chapter)

---

## ❓ Live Doubt Session Highlights

| Question | Answer |
|---|---|
| Where is "data" stored inside a 30GB model file? | Nothing — the model file size = the compiled weights/biases (learnable parameters), not raw training data. The model was *trained on* data, but doesn't store it. |
| How does the model "know" facts like who the PM is? | It was taught during training to predict the next token; it generates responses based on patterns learned, not stored memory. For facts beyond training data or that have changed, an app-level layer (RAG/agent) fetches fresh context from the web and feeds it into the prompt — the model itself has no built-in memory update. |
| Can we fine-tune closed models like Gemini/GPT/Claude? | Yes, but only via their provided fine-tuning APIs/classes — you can't download the underlying weights or control infrastructure. |
| Real-time industry choice: Hugging Face or Unsloth? | **Unsloth** — faster, industry-grade, strong community support, works across most model families. Hugging Face is taught first mainly to build foundational understanding. |
| RAG/Agents vs. Fine-tuning — which dominates in industry? | RAG and agentic pipelines are used far more often. Fine-tuning/retraining is not required for most use cases. |
| Are all Hugging Face models live/hosted 24×7 via API? | Yes — Hugging Face hosts community-uploaded models on their servers (like OpenAI hosts GPT), mostly free, though some advanced/paid API tiers may exist. |
| Do I need PyTorch/TensorFlow to do this course? | Not required for RAG, agentic apps, or fine-tuning (ready-made frameworks handle that) — only relevant if coding a neural network/LLM from scratch. |
| How to check if a fine-tuned/community model is trustworthy? | Check Hugging Face's Open Source Leaderboard / Arena Leaderboard for benchmark rankings. |
| Career advice: how to build a portfolio-worthy project? | Don't just build notebook demos — design a full pipeline: business requirement → high-level architecture → development → deployment. Combine AI logic with real software engineering (APIs, frontend, deployment). Also build a learning habit: daily exploration + regular content creation (blog/newsletter/video) deepens retention. |
| At which step does Colab actually consume GPU? | Only during model **training** (fine-tuning) — API calls, listing models, or fetching metadata don't touch the GPU. |

---

## 🗓️ Course Roadmap Shared in Class

```mermaid
flowchart LR
    A["🔧 Fine-Tuning<br/>~3-4 more classes"] --> B["🔍 RAG + Agents +<br/>Guardrails + Evaluation + MCP<br/>~2-3 months"]
    B --> C["🏗️ 3 End-to-End Projects<br/>~2 months"]
    C --> D["✅ ~90-95% Curriculum Complete<br/>+ Cloud/.NET topics"]

    style A fill:#fca5a5,color:#000,stroke:#ef4444
    style B fill:#a5b4fc,color:#000,stroke:#6366f1
    style C fill:#6ee7b7,color:#000,stroke:#10b981
    style D fill:#6366f1,color:#fff
```

> 📌 Prompt engineering is **not** a standalone chapter — it's folded into the RAG chapter.

---

## ✅ Action Items for Learners

- [ ] 🔑 Create Hugging Face Read Token and Write Token; store both under **`HF_TOKEN`** in Colab Secrets (not custom names) to avoid auth errors
- [ ] 💻 Set up a Google Colab account and connect to the free T4 GPU (Runtime → Change runtime type)
- [ ] 📓 Recreate the practical: install `huggingface_hub`, explore `model_info`, `list_models`, `list_repo_files`, `dataset_info`
- [ ] 🔗 Try calling a free Hugging Face model both via `InferenceClient` and via the LangChain `ChatHuggingFace` wrapper
- [ ] 📖 Review GitHub-hosted class notes for the full theoretical breakdown (Hugging Face vs. Unsloth)
- [ ] 🏆 Bookmark the Hugging Face Open Source Leaderboard for model trustworthiness checks
- [ ] ✍️ Start a personal habit of content creation (blog/notes/video) alongside the course to reinforce learning

---

*📝 Notes compiled from the full class transcript — LLM Fine-Tuning: Hugging Face vs. Unsloth, including the live doubt session.*
