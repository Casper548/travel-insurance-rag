# AI Travel Insurance RAG - 智能旅平險條款分析與文件檢索系統

[![Python 3.12.9](https://img.shields.io/badge/python-3.12.9-blue.svg)](https://www.python.org/downloads/)
[![LangChain](https://img.shields.io/badge/Framework-LangChain-red.svg)](https://github.com/langchain-ai/langchain)
[![VectorDB](https://img.shields.io/badge/VectorDB-Chroma-orange.svg)](https://github.com/chroma-core/chroma)

本專案是一個基於 **RAG (Retrieval-Augmented Generation, 檢索增強生成)** 架構的智能保險諮詢系統 。

專門針對台灣複雜的「海外旅行平安保險」與「旅行不便險」條款進行語意解析，打造一個專屬的 AI 保險專員，協助使用者或親朋好友在投保前徹底解答理賠疑慮 。

傳統大語言模型（LLM）因存在知識過時與幻覺（Hallucination）等致命缺點，無法直接滿足嚴謹的企業與金融需求 。

本專案建立「正確性優先」與「講對話」的核心思維 ，透過深度預處理將私有化的 PDF 保單條款進行向量化 ，利用向量資料庫高效抓取最相關的文本切片（約 500 字）投遞給 AI 讀取 ，在大幅節省 Token 成本並提升回應速度的同時 ，強迫 AI 具備 100% 的答案可溯源性（Citations），能精確標註條款出處 。

---
## 環境要求

作業系統：Windows 10 / 11

開發 IDE：Visual Studio Code (VSCode)

AI 工具 ：ChatGPT (OpenAI API) 和 Gemini

AI 模型 :BAAI/bge-m3、Llama-3.1-8B-Instruct

執行環境：Python 3.12.9 (使用 Jupyter Notebook / Anaconda)

RAG 核心框架：LangChain (包含 PyPDFLoader, RecursiveCharacterTextSplitter)

向量資料庫：Chroma (本地端向量資料庫)

資料來源：私有化 PDF 保單條款文件 

版本控制：Git + GitHub

---

## 安裝方式

### 1. 建立虛擬環境

本專案建議使用 Anaconda 建立獨立的 Python 3.12.9 執行環境：

```bash
conda create -n rag python=3.12.9 -y
conda activate rag
```

### 2. 安裝 PyTorch (支援 CUDA GPU 加速)

為確保本地端語意向量模型（Embedding Model）能發揮完整硬體效能，請先安裝支援 CUDA 的 PyTorch 核心：

```Bash
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia -y
```

### 3. 安裝項目依賴套件

透過專案內附的極簡版 `requirements.txt` 快速安裝所有 RAG 系統必要套件：

```bash
pip install -r requirements.txt
```
---

## 核心功能與技術亮點

**混合結構文本處理**：保險合約充滿了表格（理賠金額）、清單（除外責任）與長篇法律術語 ，本系統透過優化的 PDF 解析（Parsing）與分塊（Chunking）技術，完美消化混合結構文本 。

**高精度語意歧義分辨**：整合頂尖中文 Embedding（語意向量）模型，能精確捕捉並分辨保單術語中細微的語意歧義  。

**複合邏輯推理能力**：面對旅平險常見的「多層條件」邏輯，系統能提供準確的上下文供模型進行複雜推理。

**嚴格的防幻覺機制 (Anti-Hallucination)**：設計嚴謹的 System Prompt  。
若檢索內容未提及相關規定，系統會引導模型主動拒答（如回答：「抱歉，根據目前條款未能找到相關理賠規定」），杜絕商業保險中最忌諱的 AI 幻覺 。

---

## 系統架構與全棧 AI 鏈結

本專案完整掌握全棧 AI 鏈結，展現高水準的 **AI Orchestration (AI 編排)** 能力 ：

1. **資料清洗與收集 (Data Clean & Parsing)**：使用 `PyPDFLoader` 批次解析專案目錄下的保單條款 PDF ，並在走訪過程中將檔案名稱動態寫入 `metadata` 確保可溯源性 。

2. **客製化文本切分 (Text Splitting)**：採用 `RecursiveCharacterTextSplitter` 限制 `chunk_size=500`、`chunk_overlap=100` 。特別針對台灣保單排版，自訂 `separators=["\n第", "\n一、", "\n\n", "\n", " "]` 作為切分依據，避免法律條文脈絡中途斷裂。

3. **向量資料庫儲存 (Vector DB)**：利用 `Chroma` 向量資料庫建立本地端索引與管理檢索演算法 。

4. **語意檢索與生成 (Retrieval & Generation)**：透過語意相似度檢索動態抓取最相關的 3 個條款區塊，投遞給開源對話推理引擎進行嚴謹的條款推理，並在句尾標註資料出處 。

---

## 專案目錄結構

```text
.
├── RAG正式版.ipynb         
├── README.md                 
└── requirements.txt 
```
---




