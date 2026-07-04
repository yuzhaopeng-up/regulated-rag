# Regulated RAG

> **零依赖监管行业 RAG 工具包 — BM25 + TF-IDF + RRF 融合检索，无需向量数据库，无需 Embedding 模型，pip-free**
>
> 金融产品问答 · 监管政策检索 · 合规咨询 · 即开即用知识库

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen.svg)]()
[![Hit%401](https://img.shields.io/badge/Hit%401-100%25-success.svg)]()
[![Latency](https://img.shields.io/badge/Avg_Latency-<3ms-blue.svg)]()

**中文** | [English](./README.md)

---

## 为什么做这个项目

每个 RAG 教程都让你装 LangChain、启动向量数据库、下载 Embedding 模型、配置分块策略……两小时过去了，你还没问出第一个问题。

**Regulated RAG** 反其道而行：**纯 BM25 + TF-IDF + Reciprocal Rank Fusion，只用 Python 标准库，零配置**。开箱即用，内置金融产品和监管政策知识库。

| 特性 | LangChain RAG / LlamaIndex | Regulated RAG |
|------|---------------------------|---------------|
| 安装时间 | 30-120 分钟 | **30 秒** |
| 外部依赖 | 向量数据库 + Embedding 模型 | **无** |
| 配置项 | chunk_size、overlap、top-k、模型选择 | **零配置** |
| 中文支持 | 需要特定 Embedding 模型 | **内置 Jieba 分词** |
| 内置知识库 | 无 | **11 篇文档，覆盖 2 个领域** |
| 基准透明度 | 不等 | **Hit@1=100%，<3ms** |
| 监管行业聚焦 | 无 | **银保监 + 金融产品手册** |

---

## 快速演示

### 金融产品问答 — 随问随答

```bash
cd product-manual-rag
python scripts/rag_cli.py ask "大额存单的起存金额是多少？"
```
```
## 答案

大额存单的个人起存金额为20万元，机构起存金额为1000万元。

**来源：**
- [大额存单] 第3条: "个人投资者认购大额存单起点金额不低于20万元..."

**置信度：** 0.95 | **检索耗时：** 1.2ms
```

### 监管政策检索 — 精准命中法规

```bash
cd regulatory-policy-rag
python scripts/rag_cli.py ask "商业银行金融资产风险分为哪几类？"
```
```
## 答案

根据《商业银行金融资产风险分类办法》，金融资产分为五类：正常、关注、次级、可疑、损失，其中后三类合称不良资产。

**来源：**
- [商业银行金融资产风险分类办法] 第四条: "商业银行将金融资产...

**置信度：** 0.92 | **检索耗时：** 2.1ms
```

---

## 双引擎架构

### 1. 金融产品 RAG

查询金融产品手册（理财、信用卡、贷款、信托等），自动引用来源。

| 特性 | 详情 |
|------|------|
| 检索方式 | BM25 + TF-IDF 双路检索，RRF 融合 |
| 分词器 | Jieba（中文优先） |
| Hit@1 | **100%** |
| 平均延迟 | **< 3ms** |
| 输出 | 答案 + 来源文档 + 条款号 + 置信度 |
| 内置知识库 | 6 篇产品手册（基金、存单、信托、信用卡、小微贷、理财） |

### 2. 监管政策 RAG

查询银保监/央行/证监会的金融法规，附带合规建议。

| 特性 | 详情 |
|------|------|
| 检索方式 | BM25 + TF-IDF 双路检索，RRF 融合 |
| 分词器 | Jieba（中文优先） |
| 自动分类 | 识别适用监管机构（CBIRC/PBOC/CSRC） |
| 输出 | 答案 + 政策来源 + 合规建议 + 适用范围 |
| 内置知识库 | 5 篇监管文件，覆盖 3 个监管机构 |

---

## 系统架构

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
│            │  企微 / 飞书 / 命令行   │               │
│            └─────────────────────────┘               │
└──────────────────────────────────────────────────────┘
```

---

## 快速开始

```bash
git clone https://github.com/yuzhaopeng-up/regulated-rag.git
cd regulated-rag

# 无需 pip install！纯 Python 标准库。
# （可选）安装 jieba 用于中文分词：
pip install jieba
```

### 命令行使用

```bash
# 金融产品问答
cd product-manual-rag
python scripts/rag_cli.py ask "基金的定投策略是什么？"
python scripts/rag_cli.py ask "家族信托的最低门槛是多少？"

# 监管政策问答
cd regulatory-policy-rag
python scripts/rag_cli.py ask "理财产品的销售管理有哪些规定？"
python scripts/rag_cli.py ask "反洗钱客户身份识别的要求是什么？"

# 运行基准测试
python -m pytest tests/
```

### Python API

```python
from product_manual_rag import ProductManualRAG

# 初始化（自动加载内置知识库）
rag = ProductManualRAG()

# 提问
result = rag.query("大额存单的起存金额是多少？")
print(f"答案: {result.answer}")
print(f"来源: {result.source_document} - 第{result.section}条")
print(f"置信度: {result.confidence:.2f}")
print(f"检索耗时: {result.latency_ms:.1f}ms")
```

---

## 构建自己的知识库

Regulated RAG 不仅适用于银行，**适用于任何受监管行业**：

```python
from rag_engine import RAGEngine

# 创建新的 RAG 实例
engine = RAGEngine(knowledge_base_dir="my_kb/")

# 添加文档（Markdown 或纯文本）
engine.index_documents(["policy_2026.md", "regulation_v3.md"])

# 检索
results = engine.search("报告要求是什么？", top_k=3)
```

### 适用行业

| 行业 | 典型场景 |
|------|---------|
| **银行** | 产品手册问答、合规检查 |
| **保险** | 条款检索、理赔指引查询 |
| **医疗** | 临床指南查找、药物相互作用 |
| **法律** | 案例法规检索、合同条款搜索 |
| **政务** | 政策解读、市民 FAQ |
| **电信** | 服务协议问答、资费对比 |

---

## 基准测试结果

| 指标 | 金融产品 RAG | 监管政策 RAG |
|------|-------------|-------------|
| Hit@1 | 100% | 100% |
| Hit@3 | 100% | 100% |
| 平均检索延迟 | 1.8ms | 2.3ms |
| 知识库规模 | 6 篇文档 | 5 篇文档，3 个监管机构 |
| 零样本准确率 | 95%+ | 92%+ |

自行验证：
```bash
cd product-manual-rag && python -m pytest tests/ -v
cd regulatory-policy-rag && python -m pytest tests/ -v
```

---

## 与向量 RAG 的对比

| 维度 | 向量 RAG (LangChain/LlamaIndex) | Regulated RAG |
|------|-------------------------------|---------------|
| **安装** | 安装向量数据库 + 下载 Embedding 模型 | 克隆即用 |
| **依赖** | chromadb/faiss/qdrant + sentence-transformers | Python 标准库 + jieba |
| **所需配置** | chunk_size、chunk_overlap、embedding_model、top_k、score_threshold | 无 |
| **中文支持** | 需要多语言 Embedding 模型 | 原生 Jieba 分词 |
| **GPU 需求** | 推荐使用 | **永不** |
| **确定性** | 否（Embedding 方差） | **是**（BM25+TF-IDF 确定性检索） |
| **可复现** | 取决于模型版本 | **逐位可复现** |
| **最佳场景** | 大语料（1万+文档）、语义相似 | **监管领域、精准合规匹配** |

**为什么 BM25+TF-IDF 在监管领域更优：** 合规查询要求精准术语匹配（"反洗钱"必须命中"反洗钱"，而非模糊的语义近似结果）。确定性检索意味着审计友好、结果可复现——当监管机构问"你如何找到这个答案？"时，这是关键。

---

## 相关项目

| 仓库 | 描述 |
|------|------|
| [financial-ai-skills](https://github.com/yuzhaopeng-up/financial-ai-skills) | 104 个金融 AI 技能（规则引擎） |
| [soe-compliant-office](https://github.com/yuzhaopeng-up/soe-compliant-office) | 20 个央国企合规办公技能 |
| [skill-framework](https://github.com/yuzhaopeng-up/skill-framework) | L0-L4 技能治理框架 |
| [fintech-h5-demos](https://github.com/yuzhaopeng-up/fintech-h5-demos) | 12 个零依赖金融 H5 演示 |
| **regulated-rag**（本仓库） | 监管行业零依赖 RAG 工具包 |

## 贡献指南

欢迎 PR！请确保：
1. 代码和知识库中不含公司内部信息
2. 新增知识库使用通用文档名（如 `BankX-Product.md`）
3. 提交前运行 `python -m pytest tests/`
4. 仅使用 BM25+TF-IDF——不引入向量数据库或 Embedding 依赖

## 许可证

[MIT License](LICENSE) — 自由使用、修改和分发，需保留署名。
