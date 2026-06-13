醫材智慧流向與配送分析工作室 (WOW Medical Distribution Studio)
系統架構與完整技術規格說明書 (System Architecture & Technical Specification)
本文件為「醫材智慧流向與配送分析工作室 (WOW Medical Distribution Studio)」之完整技術規格書。本系統旨在建立一個兼具深度資料工程（標準化、多維度篩選、雙資料比較、導出處理）與 Agentic AI 智慧（8大 AI 魔法、4大雲端大模型路由、YAML & Markdown 設定庫管理）的企業級醫療器材追溯分析平台。系統採用 Hugging Face Spaces + Streamlit (Python) + Plotly 作為核心底座，結合先進的前端視覺與 Pantone 設計美學，提供極致的資料檢索與商業決策探索體驗。
1. 系統架構與資料流路徑 (System Architecture & Data Pipelines)
本系統的核心設計主打「資料標準化—視覺化觸發—Agentic 串接—多模式導出」的無縫管線（Data Pipeline）。整體技術架構由以下四大關鍵層級構成：
code
Code
+-----------------------------------------------------------------------------------+
|                            WOW UI 觸控與美學展示層                                 |
|      (亮暗色、20種畫家風格、中英雙語、Glassmorphism 玻璃外觀、實時 Live Log、指標晶片)       |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                            資料匯入與標準化處理引擎 (ETL)                            |
| (Paste/Upload Parser -> Canonical Converter -> Heuristic Map -> Standardization Report)|
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+------------------+---------------------+-------------------+---------------------+
|   動態篩選器矩陣  | 11大 WOW 核心視覺化 |    雙資料集對比   |     Agentic AI 核心 |
|  (6+ 複合維度欄位 | (Sankey, Network,   | (A/B Sandbox,     |  (YAML Config,     |
|   含 SN/批號過濾) | Heatmap, Trend ...) |  5大對比圖, 下鑽) |  8大 AI Magic Workspace) |
+------------------+---------------------+-------------------+---------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                            底層大模型路由與導出層                                |
|  (Gemini 2.5/2.0/1.5, OpenAI GPT-4o, Anthropic Claude, Grok / Export XML,JSON,CSV) |
+-----------------------------------------------------------------------------------+
1.1 系統部署與託管環境 (Deployment & Runtime Environment)
宿主平台：Hugging Face Spaces。
執行架構：單一 app.py 入口，配合輕量級內部模組，基於 Streamlit 框架，實現超低延遲的響應式單頁應用（SPA）。
主要依賴：
核心運行：streamlit
資料運算：pandas, numpy, openpyxl
動態視覺：plotly, networkx
模型通訊：google-genai (v2.4.0+), openai, anthropic
配置管理：pyyaml
2. 標準化資料模型與轉換引擎 (Canonical Data Model & Standardization Engine)
在醫療器材監管體系中，不同角色所申報的配送資料格式差異巨大。例如，「A. 供應商配送資料集」與「B. 顧客採購資料集」內部的欄位命名不一致、編碼格式多變。系統採用「規範化模型 (Canonical Schema)」作為流體分析的基礎。
2.1 規範化欄位定義 (Canonical Schema Specification)
系統內部統一存儲、計算與關聯以下 11 個核心 Canonical 欄位：
Canonical 欄位名 (JSON Key)	資料類型	說明	對應來源 A (供應商) 範例值	對應來源 B (買方申報) 範例值
supplier_id	string	供應商/宣告業者代碼	B00160	C05104 (流向供應商)
deliver_date	date	交貨及申報日期	2026/05/15	20260520 (收貨日期)
customer_id	string	客戶/收貨業者代碼	C05282	A00575 (申報業者/買方)
license_no	string	醫療器材許可證字號	衛部醫器輸字第033951號	衛部醫器輸字第032820號
category	string	醫療器材次類別/品類	E.3610植入式心律器...	M.3600人工水晶體
udi_di	string	UDI_DI 識別碼	00802526576324	05050474610569
device_name	string	產品中文/英文名稱	“波士頓科技”英吉尼...	“眼力健”添視明增視型...
lot_no	string	產品生產批號	926617	2187072532
serial_no	string	關鍵序號 (SN)	L110 (型號與序號之複合)	ICB00I0105 (型號對應)
model	string	產品型號	L110 / L331	ICB00I0105 / DET225
quantity	int	本次交易/配送數量	1	1
2.2 啟發式欄位映射矩陣 (Heuristic Field Mapping Rules)
系統在接受使用者貼上（Paste）或上傳（Upload）的原始 CSV/TXT/JSON 時，觸發第一階段啟發式同義詞映射（Heuristic Synonym Grouping）。具體同義詞定義如下：
supplier_id 同義詞：供應商, 供應商代號, 申報業者, 出貨業者, Supplier ID, Supplier, vendor
deliver_date 同義詞：配送日期, 出貨日期, 收貨日期, 交貨日期, 申報日期, Delivery Date, Date, deliver_date, 收貨日期
customer_id 同義詞：客戶, 客戶狀態, 收貨業者, 受配業者, 買方, Client, Customer ID, Customer, destination
license_no 同義詞：許可證號, 許可證字號, 證號, License No, License, license_no
category 同義詞：分類, 次類別, 醫療器材次類別, Category, Sub-Category, device_class
udi_di 同義詞：UDI, UDI-DI, UDI_DI, 識別碼, udi_di
device_name 同義詞：中文品名, 裝置品名, 品名, 設備名稱, Device Name, Name, device_name
lot_no 同義詞：批號, 產品批號, Batch No, Lot No, Lot, lot_no
serial_no 同義詞：序號, 產品序號, Serial No, SN, serial_number
model 同義詞：型號, 產品型號, Model, Model No, model
quantity 同義詞：數量, 件數, 額度, Quantity, Qty, count
2.3 ETL 轉換管道與清洗機制 (Cleaning Pipes & Safe Enforcer)
為了防止髒資料（Malformed data）造成後續繪圖和 Pandas 運算崩潰，系統設有強健的安全清洗管道：
日期標準化（Date Parsing Security）：
自動偵測格式。若為 8 碼純數字串（例：20260514），先將其強制轉換為 YYYY-MM-DD。
若含有斜線或橫槓（例：2026/05/15 或 2026-05-15），採用 pd.to_datetime(..., errors="coerce") 解析。
解析失敗之無效日期，預設回落至當前系統時間（例：2026-06-13），並將原始字串附加於 Live Log。
數量安全型別轉換（Quantity Cast）：
對包含千分位逗號（例：1,250）或包含中文單位（例：12組、5個）的數量欄位，採用正則表達式提取數字：r'(\d+)'。
若轉換仍失敗，底層預設強制回落為 1，絕不拋出未捕獲異常。
無用字符去除（Sanitizer）：
特殊全形引號及不規則控制符（如來源資料中的 “波士頓科技”）將統一轉換為英制半形雙引號（"波士頓科技"）避免 JSON 解析崩潰。
所有字串類型的欄位在標準化過程中強制執行 .strip()。
不完整行過濾（Null Row Sieve）：
若輸入資料中某整行均為 NaN，或必填欄位（例如沒有許可證、也沒有產品名稱且數量為 0）缺失達到 80%，該行將移入「異常資料緩衝區」，不進入主資料集，並輸出於標準化報告。
3. Dataset Studio：資料集匯入、標準化與 A/B 比較
3.1 匯入機制設計 (Dataset Ingestion Block)
在 UI 側邊欄中，使用者可切換三種匯入路徑（預設資料集、貼上文字、上傳檔案）：
預設讀取器：系統啟動時，若檢測到目錄下存在 defaultdataset.json，將自動解鎖。其格式支援 { "version": "1.0", "records": [...] } 的 JSON Array。
上傳解析器：支援 .csv (以 Pandas 引擎解析，檢測編碼如 UTF-8, Big5, GB18030)、.txt (以 Tab/逗號逗點分割解析) 與 .json 文件。
動態標準化報告：完成轉換後，UI 會渲染一個 Standardization Report Box (標準化成效檢驗盒)，列出 canonical 對映的匹配列、被丟棄的髒資料列數、缺失欄位及欄位同義詞判定結果。
多筆預覽：允許使用者以 Slider 或 Selectbox 自訂 5, 10, 20, 50, 100 筆資料的動態預覽（預設 20 筆），提供 Raw 原生資料與 Standardized 規範資料的雙表對比分頁。
3.2 頂部複合過濾矩陣 (Filtering Matrix Config)
為支援龐大供應網實體篩選，系統頂部設有可折疊或動態響應的全過濾配置器。所有篩選均採 DataFrame 串聯（AND）過濾：
供應商 ID (Supplier ID)：基於 supplier_id 的動態 Multiselect（多選下拉選單）。
許可證字號 (License Number)：基於 license_no 的模糊比對與多選。
醫療器材分類 (Classification)：對 category 進行精確過濾。
產品型號 (Product Model Number)：對 model 的匹配過濾。
配送日期區間 (Distribution Date Range)：支援 Streamlit 內建雙日曆日期選擇器（Date input），預設自適應當前資料集中的 min(date) 至 max(date)。
序號/批號 (SN / Lot Number)：文本模糊輸入框，支援輸入連續 UDI 或產品 SN。
3.3 A/B 資料集對比沙盒 (Comparative Analysis Sandbox)
使用者可同時載入兩個獨立的配送資料集：Dataset A (如供應商配送鏈) 與 Dataset B (受配端採購紀錄)。雙資料集架構如下：
code
Code
[ Dataset A Ingestion ]  -----> [ Filter Cluster A ] -----> Standardized Dataset A'
                                                                  |
                                                                  v
                                                        [ Comparative Engine ] <--- (KPI Alignment / Overlap Stats)
                                                                  ^
                                                                  |
[ Dataset B Ingestion ]  -----> [ Filter Cluster B ] -----> Standardized Dataset B'
A/B 對比摘要生成
系統對經過獨立篩選的 A' 和 B' 進行聚合，產生高密度的 A/B 對比資訊，例如：
總體數量對齊度：計算 
。
鏈路完整度檢視：比對兩者出現的供應商代碼與 UDI 交集，生成 UDI 覆蓋率（Overlap Ratio）。
自動對解讀建議：在 Markdown 區塊中以專業、客觀態度陳述此二資料集之數據不一致處（例如：「Dataset A 印記 3 筆 L110，然 Dataset B 僅登記 2 筆，可能存在經銷商入庫延遲或重複申報」）。
5 張高對比對比圖表設計 (5 Comparative Core Visuals)
KPI 雙軌對比橫條圖 (KPI Side-by-Side Bar)：比對兩邊的配送筆數、交易品類總數與唯一 UDI-DI 總量。
每日配送趨勢覆蓋線圖 (Trend Overlay Line Chart)：以高對比透明顏色標記 Dataset A（如冰河藍）與 Dataset B（如珊瑚橙），直觀看出時序差錯。
供應商流向份量對比圖 (Top Suppliers Grouped Bar)：展示同一個 ID 下兩份資料登記發送量的不平衡。
產品型號分佈比較圖 (Top Models Grouped Bar)：橫向比較主要醫材型號數量的記錄落差。
配送鏈網路融合結構圖 (Comparative Network Link Graphic)：
將 Dataset A 與 B 的交易，合併映射至同一個拓撲網路。
節點及邊著色原則：只在 A 出現的連線呈現橙色（A-only），只在 B 出現的連線呈現藍色（B-only），兩者重合的連線呈灰色（Both）。此圖能一眼看出「申報渠道漏失」。
3.4 網絡節點下鑽互動矩陣 (Interactive Node Down-Drill)
使用者在進行比較網路圖、Sankey 或網絡圖探索時，若系統有配置 streamlit-plotly-events，可捕獲點擊事件：
下鑽明細面板 (Granular Bottom Panel)：點擊具體供應商節點（例 B00079）或許可證節點，下方對話盒立即篩選展示該節點相關的 Top 10 接續鏈結（例如哪些型號去往了哪些客戶）。
無點擊依賴回落機制 (Fallback Mechanism)：若執行環境無配置點擊監聽，系統將在側邊欄直接啟用一組動態點擊選單，使用者可手動點選「聚焦特定供應商」，以達到相同的高密度下鑽效果。
4. WOW 級互動儀表板與 11 大核心視覺化
本分析工作室配備 11 種極致視覺化圖表。這些圖表全部使用 Python plotly.graph_objects 或 plotly.express 動態構建。
4.1 核心 6 視覺化說明（原始需求）
① 配送渠道動態流向圖 (Sankey Flow Chart)
維度層級：Supplier ID (供應商) 
 License No (許可證) 
 Model (產品型號) 
 Customer ID (受配機構)。
連線寬度：對應彙總的配送數量（Quantity）。
說明：將醫材自最上游申報商經法規授權，過渡到最終醫療院所的物流骨架完整繪出。
② 分層配送網絡圖 (Layered Network Graph)
結構定義：以 networkx 計算位置拓撲。按縱向分層（第一層 Supplier，第二層 License，第三層 Model，第四層 Customer），節點之間以細線相連。
降低負載最佳化：若有上千條記錄，拓撲圖將自動縮減（Top 50 Flow Nodes），避免瀏覽器渲染大量結點崩潰。
③ 配送時序與週期波動圖 (Time-Series Trends)
維度：橫軸為 deliver_date（按日/週累計），縱軸為 quantity。
交互：使用者可切換為「日波動」、「週波動」或「月份趨勢」，支援滑動區間條。
④ 供應商佔比 Pareto 柱狀圖 (Top Suppliers Pareto)
設計：左軸顯示單一供應商的累加配送數量（柱狀），右軸繪製累計份額百分比折線（80-20 定律臨界線）。用於稽核「哪些是市場高集中度供應商」。
⑤ 階層分佈太陽圖 (Sunburst Multi-level Chart)
層次：supplier_id (內環) 
 license_no (中環) 
 category / model (外環)。
亮點：支持點擊內圈局部聚焦，內置靈活扇形面積折疊。
⑥ 供應商與型號交叉熱力密度圖 (Supplier-Model Volume Heatmap)
維度：Y 軸為主要 supplier_id，X 軸為產品 model，色塊填充數值為 log(quantity + 1)，以避免異常大單扭曲其餘格子的色彩對比。
4.2 額外 5 個 WOW 時尚視覺化（新增）
⑦ 客戶分佈 Pareto 累進圖 (Top Customers Pareto)
維度：對 customer_id 進行排序彙總，直觀檢視排名前 20% 的大客戶是否佔據了 80% 的醫材配送總量，透視配送終端集中度風險。
⑧ 雙維流向密度矩阵圖 (Supplier x Customer Flow Matrix)
維度：Y 軸代表頂級供應商們，X 軸代表核心客戶（Hospitals）。方塊代表兩者間交易流通的總數量，能一眼識別「專屬經銷商-院所迴路」。
⑨ 營運日曆與節奏矩陣 (Week-Weekday Rhythm Heatmap)
維度：Y 軸為一年中第幾週 (Week of Year)，X 軸為星期一至星期日 (Day of Week)。
作用：展示特定醫材配送是否呈現極強的時序規律性（例如每週一集中配送入庫）。
⑩ 醫材法規品相多維樹狀圖 (Treemap: Supplier -> License -> Model)
外顯架構：以矩形嵌套網狀比例區塊，用層次展示不同供應商旗下，各類法規許可證的醫材體積配比。
⑪ 包裝數量波動極端值分析圖 (Boxplot Distributions)
維度：Y 軸為配送量，X 軸為 supplier_id 或 category。
說明：透過箱形圖標示出交易外圍異常值，快速排查填單錯誤或超額非典型調劑行為。
4.3 技術 Plotly 與 Pandas 序列化防崩潰保護 (Technical Guardrails)
在 Streamlit 與 Plotly 整合中，有兩個惡名昭彰的報錯可能導致系統崩潰，本規格書予以妥善解決：
避免 Plotly Scatter/Network marker.color 型別錯誤
成因：在 plotly 散點圖（go.Scatter）中，若將非數值的字串分類（如 “Supplier”, “Customer”）強行賦值給 marker.color，Plotly 檢驗器會崩潰，報 Invalid value ... in marker.color 錯誤。
保護措施：
系統拒絕使用非合法色碼的純文字作為顏色向量。
預先定義「節點類型配色映射表」（例如：{"supplier": "#0F4C81", "license": "#5B84B1", "model": "#95B4CC", "customer": "#FC766A"}）。
根據節點類別（NodeType）屬性套用對應的 Web 色碼與透明度，徹底防範硬崩潰。
Pandas 日期時間 JSON 序列化失敗保護 (Timestamp NaT Protection)
成因：當對 DataFrame 進行 to_dict(orient='records') 導出或進行 API 交互時，若資料型態中參雜 Pandas 的 Timestamp 物件或缺失值（NaT、NaN），直接調用系統 json.dumps() 將引發 Object of type Timestamp is not JSON serializable 的異常。
保護措施：
程式自定義 SafeJSONEncoder，繼承 json.JSONEncoder。
若檢測到 pandas.Timestamp，強制執行 .isoformat() 轉為 ISO 字串；
若遇到 pandas.NaT，強制序列化為 None (JSON 中呈現 null)。
導出前調用安全打包函式：
code
Python
def safe_json_serialize(df_to_export):
    # 優先使用內建 df.to_json ，並指定 ISO 日期格式
    return df_to_export.to_json(orient="records", date_format="iso")
5. 多通道 AI Agent 引擎與代理工作空間 (Agent Studio)
大語言模型是配送工作室的核心大腦。系統不僅提供靜態分析，還建立一個可編修、可配置、可管道串接的 Agent Studio。
5.1 YAML & SKILL.md 大綱管理 (Workflow Database Studio)
系統依賴本地設定庫，使用者可自定義、貼上或下載兩個關鍵配置：
agents.yaml：定義可選的 Agent 實體、設定檔架構如下：
code
Yaml
agents:
  - id: "trace_anomaly"
    name: "配送異常溯源代理 (Trace Anomaly Agent)"
    provider: "gemini"
    model: "gemini-3.1-flash-lite"
    temperature: 0.1
    max_tokens: 4000
    system_prompt: "你是一個資深的醫療器材流通稽核專家。請仔細分析以下配送鏈路..."
    user_prompt: "分析此二資料集之數據不一致處，找出 UDI 與 SN 無配對的幽靈進貨，直接列出疑似異常交易明細與原因。"
  - id: "flow_predictor"
    name: "營運流向預測代理 (Flow Prediction Agent)"
    provider: "openai"
    model: "gpt-4o"
    temperature: 0.3
    max_tokens: 4500
    system_prompt: "你是一個專業的供應鏈物流預測官..."
    user_prompt: "基於歷史配送時序數據，預測接下來下半年的配額瓶頸點..."
SKILL.md：存儲全域 System Prompt，為 AI 引入專業醫材（GUDID, UDI 等）的法規理論常識底座。每次呼叫任何特定 Agent 時，系統自動執行：
####Heuristic 容錯標準化規則（Heuristic YAML Standardizer）
若使用者在上傳或貼上 agents.yaml 時缺少 format、max_tokens 或 provider 鍵值，系統在底層運行啟發式屬性對齊：
無 provider 
 預設賦予 "gemini"。
無 max_tokens 
 預設賦予 12000（最大支援限制）。
無 temperature 
 預設給予穩定度較高的 0.2。
對於完全混亂的 YAML，如果系統配有可用的大模型 Api Key，則會啟用 LLM 輔助校準（LLM Auto-Standardizer）：調用極小溫度的 Flash 模型，將亂序 YAML 編排為符合系統期待的規範 Schema。
5.2 多通路模型路由與最大 Tokens 控制 (Model Routers)
本系統全面解除單一模型依賴，建立包含四大服務源的動態 Client 路由器：
Google Gemini：對接 @google/genai (v2.4.0+)。推薦：gemini-3.1-flash-lite, gemini-2.0-flash, gemini-1.5-pro。
OpenAI：推薦使用 gpt-4o, gpt-4o-mini。
Anthropic：支援 claude-3-5-sonnet, claude-3-haiku。
xAI Grok：支援 grok-2（透過 API key 及對應 URL 頭）。
統一 API Key 設定規則
系統使用靈活的密鑰管理器：
優先提取環境變數：若偵測到宿主環境設置有 GEMINI_API_KEY, OPENAI_API_KEY 等變量，將直接使用，並在 UI 上以藍色晶片（● Key Loaded from Env）示意。此時隱去輸入框，防止前端操作不慎外洩。
前端臨時存儲：若環境變數無，提供可折疊的 API Key Input 表單（Password 混淆模式），僅保存在當前 Session State，禁止將其序列化至硬碟。
最大輸出代幣控制 (Max Tokens Controller)
設置動態 Slider，允許使用者在 256 至 12000 之間調節模型的 max_output_tokens 限制。預設極限提高至 12,000 tokens，以利於生成完整、詳盡的總體鏈路分析，絕不因提早中斷而遺漏關鍵統計。
5.3 5 大原始 AI 魔法 + 3 大全新 AI 核心空間 (8 AI Workspaces)
本平台最核心的高附加價值模組。八大 AI 魔法空間規格如下：
code
Code
+------------------------------------------------------------------------------------------+
|                                    8大 AI 魔法核心空間                                    |
+------------------------------------+-----------------------------------------------------+
|         5大原裝 AI 魔法功能          |                  3大全新 WOW 魔法功能                |
+------------------------------------+-----------------------------------------------------+
| ① 異常追溯 (Trace Anomaly)          | ⑥ 醫療器材許可證一致性稽核 (License Align Audit)       |
| ② 物流流量預報 (Flow Predictor)     | ⑦ UDI與SN標籤可追溯性診斷 (UDI-SN Trace Diagnostics) |
| ③ 集中度集中風險識別 (Supply Cluster)| ⑧ 多合一配送流向合規預警器 (Cross-Chain Compliancer) |
| ④ 許可證合規自動審核 (DocuAudit)    |                                                     |
| ⑤ 全域配送鏈分析報告 (Summary MD)    |                                                     |
+------------------------------------+-----------------------------------------------------+
① 異常配對追溯魔法 (Trace Anomaly Space)
原理：分析 Dataset A 與 B 的交集與補集。
核心價值：發現單向登記、發貨多於收貨、幽靈單號。大模型扮演「連環稽核員」，向有嫌疑的代理商提出「紅色指引」。
② 渠道物流流量預報 (Flow Predictor Space)
原理：向 LLM 餵入以日、為單位的時間序列極簡統計序列。
核心價值：由 AI 結合產品型號與醫療常識（例如夏季心臟器材或特定手術網片庫存週轉規律），預報配送缺口。
③ 高集中度風險集群 (Supply Cluster Space)
原理：分析供應商和客戶的二維分佈（HHI，赫芬達爾指數原理）。
核心價值：識別系統中 80% 流量在少數實體的壟斷結構，產出「去中心化配送備份方案」。
④ 產品法規合規與重分類審查 (License Audit Space)
原理：輸入許可證字號與對應器材中文品名。
核心價值：檢查類別名稱（如 E.3610）是否與許可證的正式結構（衛部醫器輸/衛署醫器製）相符，並對照法規庫警告品項，防範過期使用。
⑤ 綜合配送鏈分析大報告 (Summary Markdown Space)
原理：將整套標準化後的數值矩陣拼為描述性字串（Summary Data JSON）傳遞。
核心價值：產出一份排版講究、使用 Mermaid 示意圖的 Markdown 格式高階決報，內含診斷發現、隱患列表與改善建議。
⑥ 【全新 WOW 額外 1】醫療器材許可證一致性稽核 (License Align Audit)
原理解析：比對主資料集裡同一許可證字號，其 UDI_DI 與產品型號、中文品名在跨地區配送時是否出現不同步的「品名漂移」（同證不同名、同名不同號（UDI））。
檢驗產出：發現潛在的仿冒醫材竄貨，或代理商用同一許可證套利不同規格。
⑦ 【全新 WOW 額外 2】UDI與SN標籤可追溯性診斷 (UDI-SN Trace Diagnostics)
原理解析：對醫療器材最核心的身分印記（UDI_DI 條碼）進行幾何合規校正。分析來源資料中的 UDI 編碼位數（如 GTIN 14 碼合規檢驗）、SN 符號規則。
檢驗產出：羅列出所有寫法不規範、亂打空格或可能導致條碼掃描失敗的序號，建立「一物一碼」品質雷達圖。
⑧ 【全新 WOW 額外 3】多合一配送流向合規預警器 (Cross-Chain Compliancer)
原理解析：自動評估出貨日期 (deliver_date) 與產品有效期限 (expiration_date) 的安全距離。
檢驗產出：列印所有「臨期進貨/即將過期」警告（例：申報日期為 2026/05/15，但器材在 2026/08/06 即過期，餘期不足百天），警告醫療機構防止植入體內發生悲劇醫療糾紛。
5.4 序列化 Agent 管道 (Iterative Pipelines)
本系統最特殊的 AI 執行機制是：支援將前一個 Agent 的輸出（經過使用者二次編修）直接作為下一個 Agent 的輸入背景！
一條典型的高效工作流管道配置如下：
code
Code
[ 步驟 1 ] Raw CSV 標準化
              |
              v
[ 步驟 2 ] 啟動 [ anomaly_trace ] 
          -> 生成異常 Markdown（例如發現兩筆 UDI 不吻合）
              |
              v
[ 使用者干預 ] ( 在 UI 上的 Text Area 裡直接修改修正異常的原因，或補上稽核備註 )
              |
              v
[ 步驟 3 ] 啟動 [ flow_predictor ] 或 [ compliancer ] 
          -> 自動讀取「修正後的異常摘要」與「量化資料」，產出符合實務現況的流向安全性警告！
為此，在 Streamlit 介面中，每次 Agent 生成 Markdown 後，我們在下方渲染一個 Active Edit Text Area (動態編輯工作區)：
實時加載：初次生成時填入原始 LLM 響應。
動態修改：使用者可在此區塊裡增刪資訊。
管道開關：下一個 AI Agent 介面提供一個 Checkbox：[x] 載入上一個代理的編輯成果作為分析基底。
6. Pantone 美學配色、雙語、與 Interactive Indicator 視覺設計
系統的 UI/UX 設計在視覺上要體現專業級（Professional Polish）的儀表尊貴感，拒絕拼湊。
6.1 Pantone 巴拿馬經典 10 色系調色盤 (Pantone Palettes Specification)
系統於後台常駐 10 種源自大師級與 PANTONE 年度精選色系 (Painter & Pantone Styles)，每個風格定義了冷、暖、點睛三種 RGB/HEX：
調色盤風格 ID (Style Name)	PANTONE 年度/大師色碼	背景暖色 (Light Back)	主色調 (Primary Acc)	輔助亮色 (Secondary Acc)	系統暗示 (Glow Color)	視覺意象描述
Monet (莫內)	PANTONE 15-3919 Serenity	#F2F4F8	#5B84B1	#FC766A	#87A96B	朦朧藍紫與柔玫瑰粉，古典典雅
Van Gogh (梵谷)	PANTONE 19-4052 Classic Blue	#F7F9FC	#0F4C81	#F5CB5C	#39A2AE	深邃星空藍加金麥浪，對比極為強烈
Classic Classic (精緻經典)	PANTONE 19-4052 Blue	#F4F7F9	#0F4C81	#2D3748	#10B981	電商主調，深海藍藍灰，幹練穩健
Sleek Charcoal (精練工業)	PANTONE 19-3901 Grey	#F3F4F6	#374151	#D1D5DB	#EF4444	深炭色與冷灰、適合合規稽核儀表
Warm Peach (溫暖桃李)	PANTONE 13-1023 Peach Fuzz	#FFF8F5	#FE7A5F	#9AC555	#4B5563	暖絨桃，柔和宜人，減低視覺疲勞
Hokusai (葛飾北齋)	PANTONE 19-3932 Great Wave	#F0F3F4	#1D4461	#E1AF65	#82B3AA	神奈川海浪深藍、浪花砂褐，波瀾壯闊
Vibrant Ultraviolet	PANTONE 18-3838 Ultra Violet	#F9F6FC	#5F259F	#FF1493	#64748B	神秘電磁紫，高前瞻感
Green Oasis (綠洲生命)	PANTONE 15-0343 Greenery	#F4F8F4	#589A4D	#EAA810	#22C55E	草木綠，彰顯醫療健康與配送活力
Solar Light (日光閃耀)	PANTONE 13-0647 Illuminating	#FFFDF2	#F5E050	#4B5366	#10B981	極致灰與亮麗黃，希望與堅定對決
Living Coral (活力珊瑚)	PANTONE 16-1546 Coral	#FFF3F0	#FF6F61	#12B3B0	#60A5FA	明亮海面珊瑚，活潑且飽滿
「彩蛋累積 Jackpot (幸運抽籤)」：UI 介面一隅提供實體骰子（🎲 Style Jackpot）按鈕。點擊後，系統隨機載入本表 10 款調色盤之一，全局調整所有流動圖、熱力圖的主題與襯色。
6.2 雙語及亮暗雙主題切換引擎 (Dynamic Localization & Themes)
語言快速本地化 (Traditional Chinese / English)
本系統所有 UI 組件、過濾器、AI 指令與說明支持全面雙語（1-Click Switcher），定義一個全局字典：
code
Python
LOCALIZATION_DICT = {
    "title": {"en": "WOW Medical Distribution Studio", "zh": "WOW 醫材配送與流向分析工作室"},
    "ingest_dataset": {"en": "Dataset Ingest", "zh": "資料集資料載入"},
    "standard_report": {"en": "Standardization Report", "zh": "ETL 欄位對應與標準化報告"},
    "preview": {"en": "Dataset Preview", "zh": "資料预览與下載"},
    "kpi_rows": {"en": "Analyzed Records", "zh": "當前篩選數據列"},
    "kpi_sync": {"en": "Traceability Sync", "zh": "追溯對齊率"}
}
所有文本調用直接通過 LOCALIZATION_DICT[key][session_state.lang] 渲染。
亮色/暗色主題配置 (Light/Dark Theme Switching)
亮色模式：基於 off-white 乾淨底座，灰色框線，讓龐大的資料表格具備絕佳易讀性。
暗色模式：適用於深夜監管與大屏展示。背景使用 #0F172A (深藍黑)，卡片使用 #1E293B，搭配高亮度霓虹色（Hex Accent），提升視覺對比度，消除沉悶感。
6.3 奢華 UI 動效、CSS 與 Glassmorphism 配置 (UX Enhancements)
為了讓 Streamlit 顯得精緻高端（Professional Polish），我們注入特製的 CSS Custom Theme Styles (自訂 CSS 注入)，實現以下美學特性：
毛玻璃效果 (Glassmorphism Modal Cards)：
code
CSS
div[data-testid="stMetricValue"] {
    background: rgba(255, 255, 255, 0.55);
    backdrop-filter: blur(8px) saturate(180%);
    border: 1px solid rgba(209, 213, 219, 0.3);
    border-radius: 8px;
    padding: 8px 12px;
}
動態呼吸光環晶片 (Heartbeat Sync Glow)：
頂端的「跨渠道對齊率（Sync）」具備微亮漸變光影效果。當追溯率達到 95% 以上，呈現綠色呼吸光；低於 80% 警告線，則呈現柔和霓虹紅（Red Glow）。
漸變側邊欄背景 (Radial Gradient Siderail)：
將側邊欄底圖設定為自左向右微妙的徑向漸變，引導使用者將視覺焦點向中央圖表群聚拢。
6.4 智慧即時日誌與多維運行指標 (Live Execution Log & Widget Check)
即時 Live Log 儀表盤
右側 AI Magics Workspace 中嵌入一個黑底、高亮、等寬字體（JetBrains Mono）的系統事件黑盒子：
實時日誌流 (Chronological Logging Stream)：
以時間戳紀錄 [15:47:37] Ingesting defaultdataset.json...
[15:47:38] Heuristic parsing: matched 8 attributes out of 11.
[15:47:39] COMPILER: Standardized successfully under Monet palette.
[15:47:40] AGENT: anomaly_trace queued for Gemini-3.1-Flash-lite [Temperature: 0.1]...
運行快照組件：可追溯性比對運行中，隨時向日誌投送計數點位。
高維度狀態指標矩陣 (Interactive Indicators)
在標題下方或狀態導航列，系統提供 4 個精緻裝飾性的 LED 狀態膠囊 (Sensor status lights)：
API ROUTER：顯示雲端大模型連線狀態（綠燈: ENV API Key 正常；黃燈: 使用者 Session 金鑰；灰燈: 缺失）。
EXPLOITED DATA：當前加載的標準化配送記錄筆數（例 Rows: 1,482）。
SCHEMATIC MAPPING：表示同義詞匹配是否100%全對應（Auto-Map OK 或 Partial Match）。
DOWN-DRILL EVENT：提示當前是否選中了某個節點進行歷史下鑽審查。
7. 系統部署、安全性、與異常容錯設計
7.1 安全與 API 防禦措施 (Security Gateways)
API 密鑰沙盒：所有從前端（文字輸入框）收集的 API Key，存於 Streamlit Session variables 中。絕不將資料傳送至任何第三方，且本程式不使用本地 JSON 紀錄使用者金鑰。當使用者點擊關閉瀏覽器網頁後，該密鑰即被丟棄。
敏感資訊遮罩：API 密鑰輸入框需支援 type="password"，且在 Live Log 中所有日誌一律將 Key 值替換作 *** 遮盖，防止大螢幕演示或錄製時洩露。
7.2 核心錯誤預警防禦方案 (Fault Tolerant System)
除了 4.3 節詳述的 Plotly 顏色型別與 pandas 時間序列 JSON 化特殊保護外，系統還配備以下安全網：
除零保護與 NaN 遮罩（Arithmetic Safe Enforcer）：
在計算 Pareto 疊加、HHI 集中度或 A/B 覆蓋率可能因為篩選器排空了所有行，而遭遇「除以零（DivisionByZero）」或產生「空陣列。系統在底層強制使用自研安全比例轉換法，當計數為 0 時，直接回傳對應值 0.0。
圖表全局備份（Visual Fallback Box）：
若數據源嚴重混亂（例如某列數值的特定欄位寫作了亂碼字串，導致 Sankey 圖引數傳入失常崩潰），流動圖將自動被替換為簡易 Markdown 階層樹狀表加錯誤提示資訊。
大模型回應截斷與 JSON 重試（LLM Recovery Pipeline）：
呼叫 LLM 輸出的 Anomaly JSON 時，時常遇到大模型將 Markdown 外緣加上 ```json 等邊界造成一般 json.loads 失敗。剖析器會預先利用 Regex 進行邊界切割，倘若依然失敗，自動引導大模型再次執行格式修正，或安全退回到 Markdown 純文本模式，保證 UI 不卡空白。
8. 一步到位：Sample Datasets 解析與導入印證演示
為了確保本工作室能精準載入使用者提供的範例資料，我們在此給出欄位的詳細解析與內部轉換對應。
8.1 供應商配送資料 (A. Supplier Distribution Dataset)
範例原始行
B00079,20260514,C05282,衛部醫器輸字第029100號,E.3610植入式心律器之脈搏產生器,00802526576485,“波士頓科技”艾科雷心臟節律器,161044,,L331,1,組,,,20270325,0,1,2026/05/15,2026/05/15
系統 ETL 解析與 Canonical 對應拆解
B00079 
 supplier_id (第一個欄位，對應申報/出貨業者)。
20260514 
 原始交貨登記時間（轉換為 
 格式）。
C05282 
 customer_id (第三個欄位，受配端醫院代號)。
衛部醫器輸字第029100號 
 license_no (第四個欄位，許可證字號)。
E.3610植入式心律器之脈搏產生器 
 category (第五個欄位，分類次類別)。
00802526576485 
 udi_di (第六個欄位，主要 UDI_DI 碼)。
“波士頓科技”艾科雷心臟節律器 
 device_name (第七個欄位，去掉中文引號存儲為 "波士頓科技"艾科雷心臟節律器)。
161044 
 lot_no (第八個欄位，產品製造批號)。
空值 
 被跳過。
L331 
 model (第十個欄位，產品型號/複合型號)。
1 
 quantity (第十一個欄位，數量，安全轉換為整數 1)。
組 
 unit (第十二個欄位，記錄為輔助統計)。
20270325 
 expiration_date (第十五個欄位，結合【全新 WOW 額外 3】作逾期安全警示分析)。
8.2 客戶採購資料 (B. Customer Purchase Dataset)
範例原始列
A00575,20260520,C05104,衛部醫器輸字第032820號,“眼力健”添視明增視型人工水晶體,05050474610569,M.3600人工水晶體,,2187072532,ICB00I0105,1,個,,,20300804,0,1,2026/05/20
系統 ETL 解析與 Canonical 對應拆解
A00575 
 customer_id (表頭「申報業者」，因為在受配端申報，意即買方 A00575)。
20260520 
 deliver_date (「收貨日期」，轉換為 
 格式)。
C05104 
 supplier_id (供應商代碼)。
衛部醫器輸字第032820號 
 license_no (許可證字號)。
“眼力健”添視明增視型人工水晶體 
 device_name (洗滌中文引號，對應品名)。
05050474610569 
 udi_di (UDI_DI 核心標識碼)。
M.3600人工水晶體 
 category (醫療器材分類次類別)。
空值 
 被跳過。
2187072532 
 lot_no (「產品批號」，轉換為批號存放)。
ICB00I0105 
 model / serial_no (產品序號及型號對應)。
1 
 quantity (數量為 1)。
個 
 unit。
20300804 
 產品效期（用於臨期追溯與效能保證）。
9. 系統交互原型介面包裝與實列設計 (UI Mockup Design)
為了讓使用者更具象理解 Professional Polish 主題之布局，下表描述其單一頁面佈局配置：
code
Code
+---------------------------------------------------------------------------------------------------------+
| [🎨 ICON] WOW MEDICAL DISTRIBUTION STUDIO                     LANG: [EN/繁中] | Pantone: Van Gogh | Sync [●] |
+---------------------------------------------------------------------------------------------------------+
|                                              LED STATUS CHIPS                                           |
| API: [● OpenAI] [● Gemini]  Records: [1,482]  Schema Matching: [● Auto-Map OK]  Down-drill: [Focus: B00079] |
+---------------------------------------------------------------------------------------------------------+
|                                    MAIN WORKSPACE (Three Columns Layout)                                |
|                                                                                                         |
| [LEFT SIDEBAR]                       [CENTER: MAIN DASHBOARD & VISUALIZATION]  [RIGHT: AI MAGIC WORKSPACE]     |
| 1. Dataset Ingest (Tabbed Panel)      +-------------------------------------+  1. Select Active Model   |
|    - Default Loading / Reset          |        11 WOW CORE VISUALIZATIONS   |     [ Gemini-3.1-Flash-lite  v ]|
|    - Paste / Upload Area              |                                     |                           |
|                                       | (Sankey / Network Layer / Treemap)  |  2. Live Event Log Box    |
| 2. Dynamic Filtration Matrix         | (Pareto / Boxplot / Rhythm Heatmap) |     [15:47:37] Data Load OK|
|    - Supplier ID, License No...       |                                     |     [15:47:39] Run Magic...|
|                                       +-------------------------------------+                           |
| 3. File Comparative Sandbox          |     CANONICAL PREVIEW GRID (Top 20) |  3. 8 Active AI Magic   |
|    - [Dataset A] vs [Dataset B]      |                                     |     - [Trace Anomaly]     |
|                                       | Supplier | License | UDI-DI | Qty   |     - [Flow Predictor]    |
| 4. System Action                     |  B00079  |  029100 | 008027 |   1   |     - [License Auditor]   |
|    - [Export CSV/xml/JSON]           |  A00575  |  032820 | 050504 |  12   |                           |
|                                       +-------------------------------------+  4. Active Edit Area       |
|                                                                                |  [The anomaly shows...] |
+---------------------------------------------------------------------------------------------------------+
| SYSTEM READY | THREADS: 12 ACTIVE | COMPILER: ONLINE | SECURE: AES-256                   LATEST SYNC: 15:48  |
+---------------------------------------------------------------------------------------------------------+
此佈局能將極高密度的圖表、資訊點以及 Agent 的互動過程無干擾地呈現在一整個單屏螢幕（1024px ~ 大螢幕自適應）中。
10. 20 個深度架構、功能與商業邏輯 Follow-up 追蹤問題
為了更進一步細化和將此工作室發揮至極致，請評估並選擇答覆以下 20 個專業級追蹤問題，協助我們在下一步為您建立最具革命性的實體產品程式碼：
10.1 ETL 與資料標準化層面 (ETL & Standardization)
時間序列補齊策略：當使用者篩選某個日期區間時，若某幾天中間沒有配送記錄，系統應預設補 NaN（在圖表上呈現斷開）、補 0（拉下波峰），還是直接忽略這些日期不顯示？
型號與序號 (Model vs SN) 提取深度：例如範例 A 中「L110」既是產品型號又是序號的一部分，我們是否需要自定義大模型的正則規則，從中文品名（例 波士頓科技”英吉尼心臟節律器）中，運用特定特徵關鍵字提取出型號？
髒資料判定容限臨界點：如果對上傳的 CSV 進行啟發式分析，欄位識別率（Matched Ratio）低於多少百分比（例：低於 50%），系統應直接阻斷並報錯，還是強行交由「LLM Auto-Standardizer」進行全自動重構？
分包與包裝數量一致性轉換：在醫材範例中出現了「組」與「個」等不同的數量單位，當我們在進行雙資料集 A/B 對比時，若一邊記錄「2組」而另一邊記錄出「24個」，系統是否應引入一個「包裝量比例對照手冊」提供自動乘除對齊？
10.2 11 大 WOW 視覺化細部設計 (Visualizations Details)
Sankey 流向拓撲過濾限制：當輸入的唯一路徑點（Sankey Nodes）超過 200 個時，瀏覽器端的圖表將因線條過密而完全無法閱讀。您是否同意此處自動依據 Quantity 降序排序，僅顯示前 15 大最粗的流通渠道？
分層網路圖（Network Graph）位置模型選擇：在繪製供應鏈拓撲時，更傾向於使用 Spring Layout (彈簧自發射分布，節點隨機分布、視覺炫酷)，還是強制定向的 Layered Bipartite Tree Layout (每一層平行對齊，乾淨俐落)？
Plotly 點擊事件監聽的交互動作：當使用者在網路圖中點擊某一個「受配院所（Customer ID）」時，您希望在下方明細面板僅呈現「流向本院所的所有配送明細」，還是「同步在地圖/圖表中將流向該院所的所有上游供給路徑全部高亮」？
營運日曆熱力圖的維度基準：額外 5 個視覺化中的「Rhythm Heatmap」預設是以「年~第幾週」為 Y 軸。您更希望是以「星期一至日 
 24小時（看配送日內的整點尖峰）」，還是「月份 
 星期」來呈現？
10.3 Agentic AI 與 8 大魔法空間層面 (Agentic AI & 8 Workspaces)
AI Prompts 變更持久化方案：在 Config Studio 中，被使用者動態更改後的 agents.yaml 與 SKILL.md，其變更是打算只記錄在當下的網頁 Session 裡（一刷新就還原），還是自動生成一個「Download Updated Configs ZIP」讓使用者存回本地，還是永久自動覆蓋原檔案？
多通路 AI Agent 呼叫序列執行控制：進行跨 Agent 串聯分析時（例：異常檢測輸出給流量預測），當前導輸出的 Tokens 很大（例：8000字）時，若將其夾帶在下一個 Agent prompt 中，可能導致極高的大模型費用。我們是否需要在中間引入由 Flash 模型先執行的「高密度資訊摘要微縮過濾器（Compressor）」？
多供應商（OpenAI, Gemini, Claude, Grok）切換行為：不同模型擅長的格式不同。當執行「許可證合規一致性審查」時，如果因為 API 故障自動發起跨路由降級備份 (Fallback)（例如 OpenAI 失敗自動無縫啟用 Gemini），Live Log 是否需要進行顯眼的高亮通知警告，及是否需動態調整 Model Schema？
Tokens 上限調節器之安全性監聽：當使用者手動將 max_output_tokens 調整至 12,000 上限時，某些模型的請求延遲將顯著攀升至 30~50 秒。我們是否需要在 UI 下方同步渲染一個由 motion 動畫控制的 3D 智慧沙漏動態進度條（Progress Loader）？
10.4 雙資料集 A/B 比較沙盒 (A/B Sandbox & Down-Drill)
比較資料時的「時間滑動滯後」修正：配送跟採購通常有時間差。供應商大約在 2026/05/15 出貨，受配方在 2026/05/19 才會正式簽收。在進行兩表自動時間交集對位時，系統是否需要對「差值在 N 天之內（例：預設 5 天內）的同型號、同 UDI、同數量配送」予以自動算作成功對齊？
不合法 UDI-DI 編碼標識：在進行「UDI-SN 可追溯性診斷」魔法時，若檢出某個 UDI-DI 在 GS1 官方標準中完全不存在，系統更傾向於直接將其標記為「紅色仿冒品警告 (Counterfeit warning)」，還是僅寫作「條碼編碼長度不合規 (Format error)」？
A/B 網路圖節點點擊細部資訊：使用者點選 A/B Combined Network 節點時，明細列表更傾向於以「極簡卡片 summary」形式並列（Dataset A記錄 3 筆，Dataset B記錄 2 筆），還是直接在下方並排彈出兩個 DataFrame 大表格進行視覺上的「像素級橫向比對」？
10.5 視覺美學、Pantone 調色盤與本地化 (Theme & Aesthetic Settings)
PANTONE 巴拿馬 10 色系中「暗色主題（Dark Theme）之反轉實務」：例如「Warm Peach (13-1023 暖桃)」在亮色模式下非常溫柔乾淨，但在暗色（Dark）模式下，如果是全黑背景加上粉橘色有時會顯得對比度不夠。我們對 10 色系在暗色模式下是否要進行特定柔白邊緣渲染（Glow Outline）？
Traditional Chinese 全自動台灣醫學術語校正：將醫療器材從英文（或簡體中文同義詞）標準化到 Traditional Chinese 時，是否需要對台灣在地的醫材翻譯（如「人工水晶體」、「心臟節律器」、「去顫器」、「骨盆懸吊系統」）進行正規化映射辭典鎖定？
CSS 奢華動畫開關（Performance switch）：為避免極其老舊的辦公室電腦、平板或手機點開 Streamlit 網頁時在渲染 Glassmorphism 和 3D 拓撲時感到卡頓，我們是否應在頂部加入一個「極簡效能模式（⚡ Safe Performance Mode）」的開關，開啟後停用 CSS blur、徑向漸變，並將 Plotly 圖表簡化為 2D 靜態？
10.6 系統導出與合規追溯 (Export Methods)
XML 導出的 Schema 標準：醫材出口與監管通常對 XML 格式有固定格式（如 FDA GUDID 提報格式、或台灣食藥署 UDI 回報規範）。我們的 XML 導出模組是否應提供「自訂 XML 根節點名稱」設置，還是統一遵循通用的 <medical_devices><record>...</record></medical_devices>？
自動 PDF 出海與簽案功能：在全域配送鏈分析報告（Mermaid 結構）生成後，使用者點擊下載時，是否需要我們提供一個額外的「一鍵產生 PDF 官方封存文件」選項，自動包含當前選定的 Pantone 主題配色、圖表截圖和 AI 稽核章戳印記？
📑 結論 (Conclusion)
本技術規格書詳盡地拆解了「WOW 醫材配送與流向分析工作室」的整體架構和核心底層防禦代碼設計。平台結合了精緻美學 UI（20種色調風格、雙主題、動態 LED 狀態列、即時 Live Log）、高強度資料工程（Canonical ETL標準化、11大 Plotly 高級互動圖、A/B 資料庫沙盒下鑽比對）與前沿 Agentic AI 自動配置體系。
請您在深思熟慮後，為我們反饋答覆上述 20 個深度問題。這將極大地幫助我們，在下一步中為您編寫出最完美無瑕、極具震撼力、且符合生產環境強大合規性的 Streamlit & LLM Agent 工作室實體代碼！
