Title: How TieredRAG Separates Relevant Chunks Without Using the LLM
Date: 2026-08-07
Category: GenAI
Tags: GenAI, LLM, RAG, retrieval-augmented-generation, cost-optimization, BM25, embeddings, context-scoring
Slug: how-tieredrag-separates-relevant-chunks-without-llm

A common misunderstanding about TieredRAG: "Set A is an executive summary, so the LLM must read all 100 pages first — how do we save cost?" This is wrong. Set A is NOT an executive summary. Set A is the 20 pages most relevant to the user's specific question, selected by cheap methods that cost almost nothing. The LLM never touches the separation step.

## The Correct Mental Model

**Set A** = The 20 pages most relevant to the user's specific question.

**Set B** = The 80 pages least relevant to the user's specific question.

We don't use the LLM to create Set A. We use cheap, fast methods that cost almost nothing.

## How We Separate A and B (Without the LLM)

Two cheap scoring methods run in milliseconds and cost fractions of a penny:

### Method 1: Keyword Matching (BM25) — Costs $0.00

```python
# User asks: "What is the payment due date?"
# We check every page: how many times do these words appear?

Page 2:  "Section 2: Payment Terms. The payment is due..."
         → Contains "payment", "due", "date" → Score: 0.95

Page 45: "The weather in 1998 was..."
         → Contains none of these words → Score: 0.02

Page 67: "Payment disputes shall be resolved..."
         → Contains "payment" → Score: 0.60
```

This is just counting words. No LLM. No AI. Just math.

### Method 2: Embedding Similarity — Costs ~$0.001

```python
# Convert the question into a number vector (384 numbers)
# Convert each page into a number vector (384 numbers)
# Check which page vectors are closest to the question vector

Question: "What is the payment due date?"
→ Vector: [0.12, -0.05, 0.88, ...]

Page 2:  "Section 2: Payment Terms..."
→ Vector: [0.10, -0.03, 0.85, ...] → Very close! → Score: 0.91

Page 45: "The weather in 1998..."
→ Vector: [-0.70, 0.40, 0.10, ...] → Very far! → Score: 0.08
```

This uses a tiny free model (BGE-small) that runs on your CPU in milliseconds. No LLM API call.

## The Full Process

```
┌─────────────────────────────────────────────────────────────┐
│  USER QUESTION: "What is the payment due date?"             │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: CHEAP SCORING (No LLM, costs $0.001)              │
│                                                             │
│  Method 1: Count keywords (BM25)                           │
│  Method 2: Compare vectors (Embeddings)                    │
│                                                             │
│  Page 2  → Score: 0.95  ┐                                  │
│  Page 67 → Score: 0.60  │  SET A (Top 20 pages)            │
│  Page 12 → Score: 0.45  │  → Most relevant to THIS question│
│  ...                    │                                  │
│  Page 45 → Score: 0.02  ┘                                  │
│  Page 89 → Score: 0.01     SET B (Bottom 80 pages)         │
│                            → Least relevant to THIS question│
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 2: SEND ONLY SET A TO LLM (Costs $0.05)              │
│                                                             │
│  LLM reads only 20 pages                                   │
│  LLM answers: "Payment is due within 30 days."             │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 3: CONFIDENCE CHECK (Costs $0.01)                    │
│                                                             │
│  "Is this answer fully supported?" → YES                   │
│                                                             │
│  ✅ OUTPUT ANSWER. SET B NEVER TOUCHED.                    │
│  Total cost: $0.06 (instead of $1.00)                      │
└─────────────────────────────────────────────────────────────┘
```

## Concrete Example with Real Numbers

**Document:** 100-page contract (200 chunks)

**Question:** *"What is the payment due date?"*

| Chunk | Content | BM25 Score | Embedding Score | Final Score | Set |
|-------|---------|------------|-----------------|-------------|-----|
| Chunk 3 | "Section 2: Payment Terms. The payment is due within 30 days..." | 0.95 | 0.92 | **0.94** | **A** |
| Chunk 7 | "Payment disputes shall be resolved through arbitration..." | 0.70 | 0.75 | **0.73** | **A** |
| Chunk 12 | "The invoice shall be issued on the first of each month..." | 0.50 | 0.55 | **0.53** | **A** |
| Chunk 45 | "The weather in 1998 was unusually warm..." | 0.00 | 0.05 | **0.03** | **B** |
| Chunk 89 | "The history of cloud computing dates back to the 1960s..." | 0.00 | 0.02 | **0.01** | **B** |

**Set A = Chunks 3, 7, 12 (and 37 more top-scored chunks)**

**Set B = Chunks 45, 89 (and 157 more low-scored chunks)**

The LLM never sees Set B unless the confidence check fails.

## Why This Is Cheap

| Step | Method | Cost | Time |
|------|--------|------|------|
| Scoring all 200 chunks | BM25 + Embeddings | **$0.001** | **0.5 seconds** |
| Generating answer from Set A | GPT-4o-mini | **$0.05** | **1 second** |
| Confidence check | GPT-4o-mini | **$0.01** | **0.5 seconds** |
| **Total for easy question** | | **$0.06** | **2 seconds** |

**Compare to NaiveRAG:** Reading all 100 pages with GPT-4o = **$1.00** and **5 seconds**

## Summary: The Key Insight

| Wrong Understanding | Correct Reality |
|---------------------|-----------------|
| Set A = Executive summary created by LLM | Set A = Top-scored chunks selected by cheap keyword/vector matching |
| LLM must read all 100 pages first | LLM only reads Set A (20 pages). Scoring is done by cheap methods. |
| No cost savings | **94% cost savings** on easy questions |

The LLM is only used for generating the answer. The separation of A and B is done by cheap, fast methods that cost almost nothing.

## The One-Sentence Takeaway

> We don't use the LLM to separate A and B. We use keyword counting and vector math — which costs $0.001 and takes half a second. The LLM only reads the small Set A.

The separation happens **before** the LLM ever sees the document.
