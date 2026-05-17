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

## 資料來源 (Data Source)

本專案所使用的資料集源自台灣金融監督管理委員會（金管會）保險局、中華民國人壽保險商業同業公會／產物保險商業同業公會之官方示範條款，以及產物保險公司之保單條款：

* **文件一：個人海外旅行不便保險旅程更改費用傳染病及檢疫給付附加條款**
  * **發布單位**：產物保險公司（依據金管會保險局架構設計之附加條款示範）
  * **檔案格式**：PDF
  * **內容說明**：規範被保險人於海外旅行期間，因傳染病或檢疫要求導致必須更改預定旅程時，交通與住宿費用的理賠範圍與給付限額。

* **文件二：個人海外旅行不便保險旅程取消費用傳染病及檢疫給付附加條款**
  * **發布單位**：金融監督管理委員會（114 年 10 月 29 日金管保產字第 1140431587 號函同意備查）
  * **檔案格式**：PDF
  * **內容說明**：規範被保險人因在國內接受強制檢疫、目的地發生三級國際旅遊疫情建議、或目的地政府關閉邊境等情事取消旅程時，無法取回之預繳團費、交通、住宿及票券之費用理賠。

* **文件三：旅行平安保險單示範條款**
  * **發布單位**：金融監督管理委員會保險局（歷年經財政部、行政院金管會、金管會多次修正，本版適用 108 年 6 月 21 日金管保壽字第 1080132769 號函釋）
  * **檔案格式**：PDF
  * **內容說明**：台灣人身保險市場中「旅行平安保險」的基本示範條款，包含保險契約構成、身故/失能保險金給付及受益人指定等通則。

* **文件四：海外突發疾病醫療健康保險附約示範條款**
  * **發布單位**：金融監督管理委員會保險局 / 財產保險目（民國 111 年 08 月 19 日金管保產字第 11104432901 號函公發布）
  * **檔案格式**：PDF
  * **數據來源**：[金管會主管法規共用系統](https://law.fsc.gov.tw/)
  * **內容說明**：規範在台灣以外地區（海外）發生突發疾病時，需即時在醫院或診所診療之醫療保險金給付、名詞定義與附約終止等條款。

* **文件五：海外旅遊險相關條款 (overseas_travel.pdf)**
  * **發布單位**：國泰產物保險股份有限公司
  * **檔案格式**：PDF
  * **內容說明**：包含國泰產物「新旅行平安保障保險」、「新海外突發疾病醫療健康保險附約」、「享樂遊海外旅行綜合保險」及「傷害保險恐怖主義行為保險限額給付附加條款」等多項市售保單與附加條款組合。

---

### 資料擷取與處理 (Data Extraction)
由於上述原始資料皆為 PDF 格式，本專案透過 Python 相關文字與表格解析工具進行結構化清洗，轉換為機器可讀格式，以利後續條款文本分析或生成式 AI (RAG) 檢索使用。

 **免責聲明**：本專案僅作學術研究與程式開發練習使用。各項保險條款之版權與最終解釋權歸中華民國金融監督管理委員會及各產物保險公司所有，實際理賠請以各保險公司最新公告之官方保單條款為準。

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
## 使用範例

<img width="1092" height="101" alt="Image" src="https://github.com/user-attachments/assets/d96c2ed1-ef19-4cc0-811e-14b6a0fbf2a0" />

<img width="756" height="516" alt="Image" src="https://github.com/user-attachments/assets/f267f034-95de-4bd3-8dbb-c7b443b4bbd4" />






