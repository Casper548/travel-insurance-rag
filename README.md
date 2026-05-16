# AI Travel Insurance RAG - 智能旅平險條款分析與文件檢索系統

[![Python 3.10](https://img.shields.io/badge/python-3.10-blue.svg)](https://www.python.org/downloads/)
[![LangChain](https://img.shields.io/badge/Framework-LangChain-red.svg)](https://github.com/langchain-ai/langchain)
[![VectorDB](https://img.shields.io/badge/VectorDB-Chroma-orange.svg)](https://github.com/chroma-core/chroma)

[cite_start]本專案是一個基於 **RAG (Retrieval-Augmented Generation, 檢索增慢生成)** 架構的智能保險諮詢系統 [cite: 14][cite_start]。專門針對台灣複雜的「海外旅行平安保險」與「旅行不便險」條款進行語意解析，打造一個專屬的 AI 保險專員，協助使用者或親朋好友在投保前徹底解答理賠疑慮 [cite: 5, 6]。

[cite_start]傳統大語言模型（LLM）因存在知識過時與幻覺（Hallucination）等致命缺點，無法直接滿足嚴謹的企業與金融需求 [cite: 13][cite_start]。本專案建立「正確性優先」與「講對話」的核心思維 [cite: 23][cite_start]，透過深度預處理將私有化的 PDF 保單條款進行向量化 [cite: 15][cite_start]，利用向量資料庫高效抓取最相關的文本切片（約 500 字）投遞給 AI 讀取 [cite: 18][cite_start]，在大幅節省 Token 成本並提升回應速度的同時 [cite: 19][cite_start]，強迫 AI 具備 100% 的答案可溯源性（Citations），能精確標註條款出處 [cite: 17]。

---

## 核心功能與技術亮點

* [cite_start]**混合結構文本處理**：保險合約充滿了表格（理賠金額）、清單（除外責任）與長篇法律術語 [cite: 8][cite_start]。系統透過優化的 PDF 解析（Parsing）與分塊（Chunking）技術，完美消化混合結構文本 [cite: 8]。
* [cite_start]**高精度語意歧義分辨**：整合頂尖中文 Embedding（語意向量）模型，能精確捕捉並分辨保單術語中細微的語意歧義（例如：「出發飛機延後」與「回程飛機延後」在合約定義中非常嚴謹的理賠差異） [cite: 9]。
* [cite_start]**複合邏輯推理能力**：面對旅平險常見的「多層條件」邏輯（例如：「若在特定國家且住院超過 3 天，則加成 5%」） [cite: 10]，系統能提供準確的上下文供模型進行複雜推理。
* [cite_start]**嚴格的防幻覺機制 (Anti-Hallucination)**：設計嚴謹的 System Prompt 與極低溫度控制 [cite: 13][cite_start]。若檢索內容未提及相關規定，系統會引導模型主動拒答（如回答：「抱歉，根據目前條款未能找到相關理賠規定」），杜絕商業保險中最忌諱的 AI 幻覺 [cite: 13]。

---

## 系統架構與全棧 AI 鏈結

[cite_start]本專案完整掌握全棧 AI 鏈結，展現高水準的 **AI Orchestration (AI 編排)** 能力 [cite: 22]：

1. [cite_start]**資料清洗與收集 (Data Clean & Parsing)**：使用 `PyPDFLoader` 批次解析專案目錄下的保單條款 PDF [cite: 22][cite_start]，並在走訪過程中將檔案名稱動態寫入 `metadata` 確保可溯源性 [cite: 17]。
2. [cite_start]**客製化文本切分 (Text Splitting)**：採用 `RecursiveCharacterTextSplitter` 限制 `chunk_size=500`、`chunk_overlap=100` [cite: 19]。特別針對台灣保單排版，自訂 `separators=["\n第", "\n一、", "\n\n", "\n", " "]` 作為切分依據，避免法律條文脈絡中途斷裂。
3. [cite_start]**向量資料庫儲存 (Vector DB)**：利用 `Chroma` 向量資料庫建立本地端索引與管理檢索演算法 [cite: 18, 22]。
4. [cite_start]**語意檢索與生成 (Retrieval & Generation)**：透過語意相似度檢索動態抓取最相關的 3 個條款區塊 [cite: 18][cite_start]，投遞給開源對話推理引擎進行嚴謹的條款推理，並在句尾標註資料出處 [cite: 17]。

---

## 專案目錄結構

```text
.
├── RAG正式版.ipynb         # 系統核心主程式 (Jupyter Notebook)
├── README.md               # 本專案說明文件
├── .gitignore              # Git 屏蔽配置檔（已隔離 .env 密鑰與暫存檔）
├── requirements.txt        # 核心環境相依套件清單（一行一個極簡格式）
├── 01.臺灣產物旅遊綜合保險_保單條款.pdf
├── 個人海外旅行不便保險旅程更改費用傳染病及檢疫給付附加條款.PDF
├── 個人海外旅行不便保險旅程取消費用傳染病及檢疫給付附加條款.PDF
├── 旅行平安保險單示範條款1090708.pdf
├── 海外突發疾病醫療健康保險附約示範條款.pdf
├── Insurance2026.pdf
└── overseas_travel.pdf
