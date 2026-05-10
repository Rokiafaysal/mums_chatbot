# 🍼 Mumz Chatbot — Arabic Baby Care Assistant

<a href="https://colab.research.google.com/github/Rokiafaysal/mums_chatbot/blob/main/mumz_chatbot_graduation_projectipynb.ipynb" target="_parent">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

**Mumz** is an Arabic-language AI chatbot designed to assist mothers with questions about infant health, nutrition, sleep, milestones, and early childhood care — for children from birth up to 3 years old. Built as a graduation project, it combines Retrieval-Augmented Generation (RAG), a multilingual LLM, and a rule-based safety layer to deliver trustworthy, age-aware answers in Egyptian Arabic.

---

## ✨ Features

- 🌍 **Full Arabic Support** — Understands Egyptian colloquial Arabic (عامية مصرية), including spelling variations and synonyms
- 🧠 **RAG-Powered Answers** — Retrieves relevant knowledge from a curated vector database before generating a response
- 👶 **Age-Aware Responses** — Extracts the child's age from natural language and tailors answers accordingly
- 🚨 **Safety Layer** — Detects emergency situations, refuses to prescribe medications, and always recommends a doctor when appropriate
- 🔁 **Session Memory** — Tracks child info (age, feeding type) and conversation history within a session; supports multiple children
- 📊 **Milestone Tracking** — Provides normal developmental ranges and red flags for walking, talking, crawling, and teething
- 🧩 **Smart Suggestions** — Generates contextual follow-up questions after each response
- 🖥️ **Gradio UI** — Clean, streaming chat interface runnable in Google Colab

---

## 🏗️ Architecture

```
User Message (Arabic)
        │
        ▼
┌───────────────────┐
│  Safety & Intent  │  ← Greeting / Danger / Medical / General
│  Classification   │
└────────┬──────────┘
         │
    ┌────┴─────┐
    │          │
 Rule-based  RAG Retrieval
 Response    (Qdrant + LlamaIndex)
    │          │
    └────┬─────┘
         │
         ▼
┌────────────────────┐
│  LLM Generation    │  ← Cohere command-r (streaming)
│  (Aya-Expanse 8B   │     or Aya-Expanse 8B (4-bit)
│   as fallback)     │
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│  Post-Processing   │  ← Hallucination filter, deduplication,
│  & Sanitization    │     medical dosage stripping
└────────┬───────────┘
         │
         ▼
    Final Response + Smart Suggestions
```

---

## 🧰 Tech Stack

| Component | Technology |
|---|---|
| **LLM** | `CohereForAI/aya-expanse-8b` (4-bit quantized via bitsandbytes) |
| **Streaming LLM** | Cohere `command-r-08-2024` API |
| **Embeddings** | `intfloat/multilingual-e5-large` (HuggingFace) |
| **Vector Store** | Qdrant (local) |
| **RAG Framework** | LlamaIndex + LangChain |
| **UI** | Gradio |
| **Runtime** | Google Colab (GPU) |
| **Language** | Python 3 |

---

## 📁 Project Structure

```
mums_chatbot/
├── mumz_chatbot_graduation_projectipynb.ipynb   # Main notebook (all code)
└── README.md
```

The entire project lives in a single Jupyter notebook, organized into the following sections:

| Section | Description |
|---|---|
| `Installation` | `pip` installs for all dependencies |
| `Imports` | All library imports |
| `Setup & Config` | Google Drive mount, Qdrant client init, HuggingFace login |
| `Constants & Rules` | Greetings, disclaimers, food rules, medical rules, milk quantities |
| `Memory` | `BabyAssistantMemory` class — session state, child info, history |
| `Helper Functions` | Age extraction, Arabic normalization, query rewriting, context cleaning |
| `LLM & RAG` | LLM call wrappers (streaming + non-streaming), retriever setup, `retrieve_context()`, `generate()` |
| `Safety & Intent` | `classify_intent()`, `safety_check()`, `apply_medical_rules()`, `check_milestone()`, out-of-scope detection |
| `Post Processing` | Hallucination pattern detection, response sanitization, semantic relevance scoring |
| `Core Chat` | `ask()` — main orchestration function |
| `UI & Gradio` | `respond()` + Gradio chat interface with streaming |
| `Test` | RAG unit tests, memory tests, eval suite with semantic scoring, results dashboard |

---

## 🚀 Getting Started

### Prerequisites

- A Google Colab account (free tier works; GPU recommended for Aya-Expanse)
- A [Cohere API key](https://cohere.com/)
- A [HuggingFace API key](https://huggingface.co/settings/tokens)
- A pre-built Qdrant vector database (`rag_data_backup.zip`) stored in Google Drive

### Setup

1. **Open the notebook in Colab** using the badge at the top of this README.

2. **Add secrets** in Colab (click the 🔑 key icon on the left sidebar):
   - `HUGGINGFACE_API_KEY` — your HuggingFace token
   - `COHERE_API_KEY` — your Cohere API key

3. **Upload your RAG data** to Google Drive at the path:
   ```
   MyDrive/rag_data_backup.zip
   ```
   The zip should contain:
   - `my_qdrant_data/` — Qdrant collection directory
   - `docstore.json` — LlamaIndex document store

4. **Run all cells** in order. The notebook will:
   - Install dependencies
   - Mount Google Drive and extract the RAG data
   - Load the embedding model and LLM
   - Launch the Gradio chat interface

---

## 💬 How It Works

### Intent Classification

Every user message is first classified into one of four categories:

- **GREETING** — Social messages (`أهلا`, `شكرا`, `تمام`) → Fixed warm response
- **DANGER** — Emergency keywords (`تشنج`, `اختناق`, `نزيف شديد`) → Immediate emergency instruction
- **MEDICAL** — Dosage/prescription requests (`جرعة`, `باراسيتامول`) → Politely refused, doctor recommended
- **GENERAL** — Health, nutrition, sleep, development questions → Full RAG + LLM pipeline

### Age Extraction

The chatbot parses the child's age from free Arabic text, handling:
- Numerals and Arabic-Indic digits (`٤ شهور`, `4 months`)
- Words like `سنتين`, `شهرين`, `سنة ونص`
- Pronouns and context from conversation history

### RAG Pipeline

1. The query is rewritten using conversation history and synonym expansion
2. Top-3 relevant chunks are retrieved from Qdrant (`similarity_top_k=3`, threshold `0.65`)
3. Chunks are cleaned, trimmed to 1200 chars, and injected into the LLM prompt
4. The LLM (Cohere streaming or Aya-Expanse) generates a response grounded only in the retrieved context

### Safety & Post-Processing

- Regex-based hallucination patterns catch fabricated doctor names, fake formulas, forbidden food combinations, and incorrect dosages
- Dosage numbers (`\d+ مل`, `\d+ mg`) are stripped from all outputs
- Semantic relevance score (cosine similarity) validates that the response actually addresses the question

---

## 🧪 Testing

The notebook includes a built-in evaluation suite (`#test` section) covering:

- **Happy path** — Normal nutrition and care questions
- **Safety** — Danger detection, medication refusal
- **Boundary** — Edge cases (very young ages, unclear questions)
- **Out-of-scope** — Adult recipes, non-baby topics
- **Greetings** — Social and thanks messages

Results are visualized in a 4-panel Matplotlib dashboard showing pass/fail by category, response times, and semantic relevance scores.

---

## ⚠️ Disclaimer

Mumz is an informational assistant only. It does not provide medical diagnoses or prescribe medications. Always consult a qualified pediatrician for medical decisions regarding your child.

---

## 👩‍💻 Author

**MUMZ Team** — Graduation Project

---

## 📄 License

This project is for educational purposes. Please check with the author before reuse or redistribution.
