# Regulated RAG

> **Zero-dependency RAG toolkit for regulated industries — BM25 + TF-IDF + RRF, no vector DB, no Embedding model, pip-install-free**
>
> Product Manual Q&A · Regulatory Policy Search · Compliance Advisory · Instant Knowledge Base

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen.svg)]()
[![Hit%401](https://img.shields.io/badge/Hit%401-100%25-success.svg)]()
[![Latency](https://img.shields.io/badge/Avg_Latency-<3ms-blue.svg)]()

[中文文档](./README_CN.md) | **English**

---

## Why This Exists

Every RAG tutorial tells you to install LangChain, spin up a vector database, download an Embedding model, configure chunking strategies... and 2 hours later you still haven't asked a single question.

**Regulated RAG** takes the opposite approach: **pure BM25 + TF-IDF with Reciprocal Rank Fusion**, Python standard library only, zero configuration. It works out of the box with built-in knowledge bases for financial products and regulatory policies.

| Feature | LangChain RAG / LlamaIndex | Regulated RAG |
|---------|---------------------------|---------------|
| Setup time | 30-120 min | **30 seconds** |
| External dependencies | Vector DB + Embedding model | **None** |
| Configuration | Chunk size, overlap, top-k, model selection | **Zero config** |
| Chinese support | Requires specific Embedding model | **Built-in, Jieba tokenizer** |
| Included knowledge bases | None | **11 documents across 2 domains** |
| Benchmark transparency | Varies | **Hit@1=100%, <3ms** |
| Regulated industry focus | None | **Banking regulators + product manuals** |

---

## Quick Demo

### Product Manual Q&A — Ask About Any Financial Product

```bash
cd product-manual-rag
python scripts/rag_cli.py ask "大额存单的起存金额是多少？"
```
```
## Answer

大额存单的个人起存金额为20万元，机构起存金额为1000万元。

**Sources:**
- [大额存单] 第3条: "个人投资者认购大额存单起点金额不低于20万元..."

**Confidence:** 0.95 | **Retrieval time:** 1.2ms
```

### Regulatory Policy Search — Query Banking Regulations

```bash
cd regulatory-policy-rag
python scripts/rag_cli.py ask "商业银行金融资产风险分为哪几类？"
```
```
## Answer

根据《商业银行金融资产风险分类办法》，金融资产分为五类：正常、关注、次级、可疑、损失，其中后三类合称不良资产。

**Sources:**
- [商业银行金融资产风险分类办法] 第四条: "商业银行将金融资产...

**Confidence:** 0.92 | **Retrieval time:** 2.1ms
```

---

## Two Engines

### 1. Product Manual RAG

Query product manuals (wealth management, credit cards, loans, trusts, etc.) with source citation.

| Feature | Detail |
|---------|--------|
| Retrieval | BM25 + TF-IDF dual retrieval with RRF fusion |
| Tokenizer | Jieba (Chinese-first) |
| Hit@1 | **100%** on test set |
| Avg latency | **< 3ms** |
| Output | Answer + source document + section + confidence score |
| Included KB | 6 product manuals (fund, deposit, trust, credit card, SME loan, wealth management) |

### 2. Regulatory Policy RAG

Query banking and financial regulations from CBIRC, PBOC, and CSRC with compliance advisory.

| Feature | Detail |
|---------|--------|
| Retrieval | BM25 + TF-IDF dual retrieval with RRF fusion |
| Tokenizer | Jieba (Chinese-first) |
| Auto-classification | Detects applicable regulator (CBIRC/PBOC/CSRC) |
| Output | Answer + policy source + compliance advisory + applicability |
| Included KB | 5 regulatory documents across 3 regulators |

---

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                  Regulated RAG                        │
├──────────────────────┬───────────────────────────────┤
│  Product Manual RAG  │  Regulatory Policy RAG        │
│  (6 documents)       │  (5 documents, 3 regulators)  │
├──────────────────────┴───────────────────────────────┤
│              Shared RAG Engine                        │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐ │
│  │  BM25       │  │  TF-IDF     │  │  RRF Fusion  │ │
│  │  Retriever  │  │  Retriever  │  │  + Re-rank   │ │
│  └─────────────┘  └─────────────┘  └──────────────┘ │
│         │                │                │          │
│         └────────────────┼────────────────┘          │
│                          ▼                           │
│              Jieba Chinese Tokenizer                  │
│                          │                           │
│                          ▼                           │
│            ┌─────────────────────────┐               │
│            │  WeCom / Feishu / CLI   │               │
│            └─────────────────────────┘               │
└──────────────────────────────────────────────────────┘
```

---

## Quick Start

```bash
git clone https://github.com/yuzhaopeng-up/regulated-rag.git
cd regulated-rag

# No pip install needed! Pure Python standard library.
# (Optional) Install jieba for Chinese tokenization:
pip install jieba
```

### CLI Usage

```bash
# Product manual Q&A
cd product-manual-rag
python scripts/rag_cli.py ask "基金的定投策略是什么？"
python scripts/rag_cli.py ask "家族信托的最低门槛是多少？"

# Regulatory policy Q&A
cd regulatory-policy-rag
python scripts/rag_cli.py ask "理财产品的销售管理有哪些规定？"
python scripts/rag_cli.py ask "反洗钱客户身份识别的要求是什么？"

# Run benchmark tests
python -m pytest tests/
```

### Python API

```python
from product_manual_rag import ProductManualRAG

# Initialize with built-in knowledge base
rag = ProductManualRAG()

# Ask a question
result = rag.query("大额存单的起存金额是多少？")
print(f"Answer: {result.answer}")
print(f"Source: {result.source_document} - Section {result.section}")
print(f"Confidence: {result.confidence:.2f}")
print(f"Retrieval time: {result.latency_ms:.1f}ms")
```

---

## Build Your Own Knowledge Base

Regulated RAG is designed for **any regulated industry**, not just banking:

```python
from rag_engine import RAGEngine

# Create a new RAG instance
engine = RAGEngine(knowledge_base_dir="my_kb/")

# Add documents (Markdown or plain text)
engine.index_documents(["policy_2026.md", "regulation_v3.md"])

# Query
results = engine.search("What are the reporting requirements?", top_k=3)
```

### Applicable Industries

| Industry | Example Use Case |
|----------|-----------------|
| **Banking** | Product manual Q&A, regulatory compliance |
| **Insurance** | Policy document search, claims guideline |
| **Healthcare** | Clinical guideline lookup, drug interaction |
| **Legal** | Case law retrieval, contract clause search |
| **Government** | Policy interpretation, citizen FAQ |
| **Telecom** | Service agreement Q&A, tariff comparison |

---

## Benchmark Results

| Metric | Product Manual RAG | Regulatory Policy RAG |
|--------|-------------------|-----------------------|
| Hit@1 | 100% | 100% |
| Hit@3 | 100% | 100% |
| Avg retrieval latency | 1.8ms | 2.3ms |
| Knowledge base size | 6 documents | 5 documents, 3 regulators |
| Zero-shot accuracy | 95%+ | 92%+ |

Run the benchmarks yourself:
```bash
cd product-manual-rag && python -m pytest tests/ -v
cd regulatory-policy-rag && python -m pytest tests/ -v
```

---

## Comparison with Vector-based RAG

| Dimension | Vector RAG (LangChain/LlamaIndex) | Regulated RAG |
|-----------|-----------------------------------|---------------|
| **Setup** | Install vector DB + download Embedding model | Clone and run |
| **Dependencies** | chromadb/faiss/qdrant + sentence-transformers | Python stdlib + jieba |
| **Config required** | chunk_size, chunk_overlap, embedding_model, top_k, score_threshold | None |
| **Chinese support** | Needs multilingual Embedding model | Native Jieba tokenization |
| **GPU required** | Recommended for Embedding | **Never** |
| **Deterministic** | No (Embedding variance) | **Yes** (BM25+TF-IDF are deterministic) |
| **Reproducible** | Depends on model version | **Bit-for-bit reproducible** |
| **Best for** | Large corpora (10K+ docs), semantic similarity | **Regulated domains, exact-match compliance** |

**Why BM25+TF-IDF wins in regulated domains:** Compliance queries demand precise term matching ("反洗钱" must hit "反洗钱", not a vague semantic neighbor). Deterministic retrieval means audit-friendly, reproducible results — critical when regulators ask "how did you find this answer?"

---

## Ecosystem

| Repo | Description |
|------|------------|
| [financial-ai-skills](https://github.com/yuzhaopeng-up/financial-ai-skills) | 104 financial AI skills (rule engines) |
| [soe-compliant-office](https://github.com/yuzhaopeng-up/soe-compliant-office) | 20 SOE-compliant office skills |
| [skill-framework](https://github.com/yuzhaopeng-up/skill-framework) | L0-L4 skill governance framework |
| [fintech-h5-demos](https://github.com/yuzhaopeng-up/fintech-h5-demos) | 57 zero-dependency H5 demos |
| **regulated-rag** (this repo) | Zero-dependency RAG for regulated industries |

## Contributing

PRs welcome! Please ensure:
1. No company-internal information in code or knowledge bases
2. New knowledge bases use generic document names (e.g., `BankX-Product.md`)
3. Run `python -m pytest tests/` before submitting
4. BM25+TF-IDF only — no vector DB or Embedding dependencies

## License

[MIT License](LICENSE) — Free to use, modify, and distribute with attribution.
