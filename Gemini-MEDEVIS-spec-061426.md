系統架構與完整技術規格說明書 (System Architecture & Technical Specification)
專案名稱：醫材智慧流向與配送分析工作室 (WOW Medical Distribution Studio)
版本：v2.5 Stable | 類別：企業級醫療數據整合與 AI 探勘平台
本技術規格書（Technical Specification）旨在為「WOW Medical Distribution Studio」之開發、部署、運營與升級提供詳盡、精確且符合「Professional Polish（專業拋光）」設計美學的架構藍圖。本系統建立在以資料工程為骨幹、大語言模型為分析大腦、Python (Streamlit/Plotly/NetworkX/Matplotlib) 為視覺呈現載體的高效能全棧數據架構上，協助衛生主管機關、醫療院所及供應商針對 WHO Medevis 與 TUDID 兩套完全不同源的醫療器材資料集進行標準化、極致比對、智慧探索與合規性稽核。
1. 系統架構與資料流路徑 (System Architecture & Data Pipelines)
本系統的核心設計主打「資料標準化 — 互動式篩選 — A/B 雙邊沙盒比對 — Agentic 動態串接 — 自動導出」的無縫數據管線（Data Pipeline）。整體技術架構由以下五大關鍵層級構成，各模組間透過嚴格定義的介面（API / Session State）進行訊息傳遞，藉此達到低耦合、高内聚與極致流暢之操作體驗。
1.1 系統架構分層圖 (Layered Architecture Blueprint)
code
Code
+-----------------------------------------------------------------------------------+
|                            WOW UI 觸控與美學展示層                                 |
|      (亮暗色、10種 Panton 風格、中英雙語、Glassmorphism 玻璃外觀、實時 Live Log、指標晶片)     |
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
1.2 執行架構與託管環境 (Runtime & Execution Environment)
託管管道：Hugging Face Spaces。
執行核心：單一 app.py 入口，配合輕量級內部靜態類與自定義函數，基於 Streamlit 框架運作。由於 AI Studio Preview 針對多端口有限制，本系統在 Port 3000 上以單頁應用（SPA）的高效能模式執行，完全避開分散式微服務所帶來的異步通訊延遲。
包依賴（Dependencies）：
運行時：streamlit、pandas、numpy、openpyxl
繪圖與拓撲：plotly、networkx、matplotlib
大模型接入：google-genai (v2.4.0+)、openai、anthropic
配置讀取：pyyaml
1.3 資料流路徑說明 (Step-by-Step Data Pipeline)
數據攝入（Data Ingestion）：使用者透過前端側邊欄貼上原始字串或上傳 .csv、.txt、.json。
啟發式對齊（Heuristic Alignment）：ETL 引擎自動掃描首行（Header），映射至 Canonical Unified 格式。
清洗與型態轉換（Pipes & Filters）：對非標準日期（20260515 -> 2026-05-15）、數量（"5組" -> 5）、字串（去除全形引號 “）進行正則與補缺值處理。
對比沙盒緩衝（A/B Sandbox Buffer）：將處理好的 DataFrame 快取至 st.session_state 的 dataset_a 與 dataset_b。
聯合矩陣篩選（Matrix Sieve）：套用供應商、許可證、品類、型號、日期區間、批號 6+ 複合過濾器，取得過濾後的子集 
 與 
。
圖表序列生成（Visualization Builder）：將 
 與 
 投入 Plotly 與 NetworkX 進行動態繪圖。
Agentic 大腦分析（LLM Orchestration）：過濾子集以極簡 JSON-Summary 與 Schema 配置傳遞給大模型，經由 Unified Client 產出多模態報告。
數據同步導出（Export Engine）：資料分析結果經由 XML / JSON / CSV 編碼器，允許使用者直接下載實體檔案。
2. 標準化資料模型與轉換引擎 (Canonical Data Model & Standardization Engine)
在跨國及本土醫材追溯體系中，雙邊資料集（WHO Medevis 與 TUDID）的欄位定義與表達方式有著巨大落差。例如，台灣的 TUDID（Taiwan Unique Device Identification）著重「許可證字號、基本 DI、型號與中文品名」，而 WHO Medevis 主要關注「Nomenclature Code (EMDN/GMDN)、Healthcare Settings、Reusable 屬性、Sex 限制、Delivery Platforms 等層級分類」。為在不丟失任何屬性的前提下進行雙邊交易或庫存比對，系統底層特別設計了一款「規範化資料模型 (Canonical Schema)」。
2.1 規範化欄位定義 (Canonical Schema Specification)
系統內部統一存儲、計算與關聯以下 11 個核心 Canonical 欄位。非核心或異構欄位（如 WHO Medevis 的 Sex 或 TUDID 的 UDI發碼機構）將全數包裝於擴充 JSON 字典欄位 meta_attributes 中，極大限度地保持資料彈性、避免硬性欄位丟失。
Canonical 欄位名 (JSON Key)	資料類型	系統定義與約束條件	對應來源 A (WHO Medevis) 範例對照	對應來源 B (TUDID) 範例對照
supplier_id	string	供應商 / 申報業者代碼（不可為空，預設 "UNKNOWN"）	N/A (從 WHO 組織對應獲取)	衛部醫器輸字第034848號之持有人/愛爾康
deliver_date	date/str	交易/配送/申報日期（統一為 YYYY-MM-DD 格式）	MeDevIS version / 2025-02-01	解析自動對應或當前處理時間 2026-06-13
customer_id	string	客戶/受配醫療機構/終端通路代碼	N/A (Healthcare settings 對照預設)	收貨業者/特約院所代碼或名稱
license_no	string	醫療器材許可證字號 / 官方註冊證號	Nomenclature code (EMDN / GMDN)	許可證字號（類型）（衛部醫器輸字第034848號）
category	string	醫療器材類別/次類別	Device type / "Surgical instruments"	隱形眼鏡（或從品名規則歸納之隱形眼鏡科）
udi_di	string	UDI 第二層唯一檢索識別碼（GTIN-14 等）	Nomenclature code (GMDN) 條碼	基本DI（00730822294652）
device_name	string	產品中文或英文名稱	Device name ("Adenotome")	產品中文品名（“愛爾康” 水感散光日拋隱形眼鏡）
lot_no	string	產品生產批號（交易比對關鍵）	"L14010101" (從 EMDN code 對映)	PRECSION1 批號（自動從產品描述或隨機截取）
serial_no	string	單一器材產品序號（SN）	64085 (GMDN code)	000000000010165367 (型號/識別)
model	string	產品型號/規格/包裝級別	"ADENOTOMES, REUSABLE"	型號（000000000010165367）
quantity	integer	交易或配送之件數（預設 1）	1 (由 Reusable/單次包裝判定)	1
2.2 啟發式欄位映射矩陣 (Heuristic Field Mapping Rules)
使用者貼上文字或上傳檔案時，系統不會因為欄位名稱不符而發生崩潰，而是會自動調用 啟發式對齊模組，針對以下同義詞字典（Heuristic Synonym Grouping）進行模糊配對與距離運算：
supplier_id 同義詞：供應商, 供應商代號, 申報業者, 出貨業者, 申請商, Supplier ID, Supplier, vendor, manufacturer
deliver_date 同義詞：配送日期, 出貨日期, 收貨日期, 交貨日期, 申報日期, 建檔日期, Delivery Date, Date, deliver_date, MeDevIS version
customer_id 同義詞：客戶, 收貨業者, 受配業者, 買方醫院, 醫療機構, Client, Customer ID, Customer, destination, Healthcare settings
license_no 同義詞：許可證號, 許可證字號, 證號, EMDN Code, GMDN Code, Nomenclature code, License No, License
category 同義詞：分類, 次類別, 醫療器材類別, 品類名稱, Device type, Category, Sub-Category, device_class
udi_di 同義詞：UDI, UDI-DI, UDI_DI, 基本DI, 商品識別碼, GTIN, udi_di
device_name 同義詞：產品品名, 中文品名, 產品名稱, 設備名稱, Device name, Device Name, Name, device_name
lot_no 同義詞：批號, 產品批號, 生產批號, Batch No, Lot No, Lot, lot_no
serial_no 同義詞：序號, 唯一識別碼, 產品序號, Serial No, SN, serial_number
model 同義詞：型號, 產品型號, 規格, Model, Model No, model
quantity 同義詞：數量, 件數, 交易數量, 進貨數量, Quantity, Qty, count, units
2.3 ETL 轉換管道與安全防崩潰機制 (Cleaning Pipes & Safe Enforcer)
ETL (Extract-Transform-Load) 轉換器具有三道過濾關卡，以保障臟數據（Malformed Data）在匯入時就被精準排除或更正：
日期標準化轉換器（Date Parsing Security）：
邏輯：偵測到字串。若為 8 碼純數字（如 20260514），代碼會自動插入橫槓轉換為 2026-05-14；若讀取到 2026/05/14，則使用 pd.to_datetime(..., errors='coerce') 解析。
回落機制 (Fallback)：若解析完後為 NaT（無效日期），一律自動套用當前系統時間 2026-06-13，同時將原始錯誤字串附加至系統實時日誌中。
數量安全型別轉換（Quantity Cast）：
邏輯：遇到含有千分位逗號（如 1,250）或中文單位（如 12組、5個）的數量字串，採用正則表達式 r'(\d+)' 純化數字部分。
回落機制：若轉換後非合規整數，則強制賦予預設值 1，絕不拋出未捕獲異常中斷程式。
無用控制字符與特殊引號清洗（Sanitizer）：
邏輯：原始數據中常伴有不規範的全形引號（如 “愛爾康”）或 Windows 特有換行字符（\r\n），一律統一轉換為英制半形雙引號（"愛爾康"）及標準 \n。同時對於所有欄位皆行使 .strip() 處理，防止前後空格阻礙 Groupby 的精確聚合。
不完整行過濾器（Null Row Sieve）：
邏輯：若一列數據中，必填的 license_no 或 device_name 缺失，且填寫率低於 30%，此行被判定為空垃圾行，將其轉移存入 st.session_state.unresolved_rows（髒資料緩衝區），並在「ETL 欄位對應與標準化報告」中清晰提示，不進入核心數據流。
3. Dataset Studio：資料集匯入、標準化與雙資料 A/B 比較沙盒
本平台不僅能夠匯入單一資料集，更能建立兩個獨立資料集的並行處理流水線：Dataset A（代表 WHO Medevis 國際規範、或供應鏈最上游出貨紀錄）與 Dataset B（代表 TUDID 本地申報、或醫療院所實際點收紀錄）。
code
Code
[ Dataset A Ingestion ]  -----> [ Filter Cluster A ] -----> Standardized Dataset A'
                                                                  |
                                                                  v
                                                        [ Comparative Engine ] <--- (KPI Alignment / Overlap Stats)
                                                                  ^
                                                                  |
[ Dataset B Ingestion ]  -----> [ Filter Cluster B ] -----> Standardized Dataset B'
3.1 匯入機制與預覽面板 (Ingestion & Multi-Preview Section)
貼上與上傳雙軌支援：支援使用者將 CSV、TXT 或 JSON 字串直接貼入大文本框，或是將實體檔案拉入 Drag&Drop 區域。
自適應編碼檢測：上傳之檔案底層會自動嘗試 utf-8、big5（繁體中文 Windows 預設）、gb18030（簡體中文）、utf-16 編碼，避免中文字體亂碼。
多筆預覽控制器：介面提供 Selectbox 讓使用者自訂預覽筆數（5, 10, 20, 50, 100 筆，預設 20）。預覽採用 Tab 分頁：
Tab 1：Standardized Normalized Data（已標準化規範數據表）：展示 11 大核心欄位，並加上標色區分。
Tab 2：Raw Source Data（原始導入數據表）：直接展示未修改的原始資訊，方便對照。
3.2 頂部複合過濾矩陣 (Collapsible Dynamic Filtering Matrix)
系統頂端配備一個可展開的複合篩選抽屜，內部為聯級式（Cascading）過濾系統，包含：
供應商 ID（Supplier ID）：動態多選選單（Multiselect），只列出當前資料集存在的供應商。
許可證字號（License Number）：支援模糊字串比對與精確勾選。
醫材品類分層（Classification）：依據 category 自動分級，支援多選。
產品型號篩選（Model Range）：快速進行型號對比。
日期配送區間（Delivery Date Picker）：雙日曆（Date Input）選擇，預設自適應資料集的最早日期與最晚日期。
序號/批號過濾（SN/Lot Target）：輸入框比對，支援英文與數字部分模糊。
3.3 A/B 雙邊資料精準比對沙盒 (Comparative Analysis Sandbox)
當使用者在 A 與 B 兩個槽中各載入一份資料集後，沙盒對比引擎會自動激活：
對齊度與交集計算 (Traceability Sync Rate)：計算 A 與 B 的 udi_di 或 license_no 交集：
不一致性多維度抓取：對比引擎能自动提示「Dataset A 有 3 筆出貨紀錄，但在 Dataset B 中無對應收貨，可能存在經銷商漏報」或「兩造 UDI_DI 規格描述不符」等邏輯錯誤。
5 張高對比雙資料集對比圖表：
KPI 雙軌對比橫條圖 (KPI Side-by-Side Bar)：橫向並列對比 A 與 B 的總出貨筆數、品類數、唯一許可證數、累計件數。
時序日配送趨勢覆蓋線圖 (Trend Overlay Line Chart)：以高對比透明顏色標記 Dataset A（如冰河藍）與 Dataset B（如珊瑚橙），直觀看出時序差錯。
供應商申報分量對比圖 (Top Suppliers Grouped Bar)：橫向比較主要供應商 ID 下兩份資料登記發送量的不平衡。
產品型號分佈比較圖 (Top Models Grouped Bar)：對比同型號在 A 側與 B 側的數量落差。
配送鏈網路融合結構圖 (Comparative Network Link Graphic)：將 Dataset A 與 B 的交易，合併映射至同一個拓撲網路。
節點及邊著色原則：只在 A 出現的連線呈現橙色（A-only），只在 B 出現的連線呈現藍色（B-only），兩者重合的連線呈灰色（Both）。此圖能一眼看出「申報渠道漏失」。
3.4 網絡節點下鑽互動矩陣 (Interactive Node Down-Drill)
使用者在進行比較網絡圖、Sankey 或網絡圖探索時，若系統有配置 streamlit-plotly-events，可捕獲點擊事件：
點擊下鑽 (Click to Zoom)：當使用者點擊圖中特定「關係節點」（如某一家經銷商代碼），系統下方的「下鑽明細面板 (Granular Bottom Panel)」會立刻自動過濾出該經銷商的所有子向明細。
無依賴回落選單 (Fallback Mechanism)：若執行環境沒有安裝點擊交互元件（Iframe 隔離），系統會提供一個精緻的 Dropdown：「手動探索聚焦節點（Manual Focus Node）」，確保 100% 能達到同等的高密度過鑽與稽核體驗。
4. WOW 級動態視覺化：11 大核心圖表
這是一套基於 Python 作為底座、整合 Plotly 及 NetworkX 繪製的高視覺衝擊力、無卡頓數據圖表陣列。
4.1 核心 6 視覺化說明（原始需求）
① 配送渠道動態流向圖 (Sankey Flow Chart)
層級邏輯：供應商 (Supplier) 
 許可證 (License) 
 品項(Model) 
 收貨終端 (Customer)。
寬度加權：依據配送數量（Quantity）動態調整流道寬度。
互動性：指針懸停時，顯示該物流節點的流量流速和流失率。
② 分層配送網路圖 (Layered Network Graph)
算法原理：使用 NetworkX 計算節點的 multipartite_layout（多層結構布局），節點沿 Y 軸垂直整齊對齊。
層級排布：首層為 Suppliers（供應節點），第二層為 Licenses（許可證法規節點），第三層為 Models（器材型號節點），末層為 Customers（醫院接收節點）。
高載過濾：若節點超過 150 個，系統會自動在後台過濾保留「Top 50 流量實體」，避免前端 SVG 節點過載導致瀏覽器卡頓崩潰。
③ 配送時序與週期波動圖 (Time-Series Trends overlay)
橫軸與縱軸：橫軸為 deliver_date，縱軸為 quantity。
週期轉換器：支援使用者利用 Toggle 按鈕切換「日（Daily）」、「週（Weekly）」或「月（Monthly）」累進。
對比覆蓋：Dataset A 與 B 的折線圖覆蓋渲染，並透過雙滑桿區間控制時間，便於比對過往異常配送峰值。
④ 供應商佔比 Pareto 柱狀圖 (Top Suppliers Pareto)
雙軸 Y 軸設計：左軸為單一供應商的累積配送數量（長條圖），右軸為該供應商佔據總市場的累計百分比折線（Pareto Curve）。
標記線：在 80% 的位置設有一條醒目的螢光色虛線（80-20 Rule Line），明確標示哪些供應商掌控了市場主要流通。
⑤ 階層分佈太陽圖 (Sunburst Multi-level Chart)
嵌套網狀結構：中心圈為 supplier_id，第二圈為其所屬 license_no，最外圈為個別的 model 或配送的 category。
響應能力：點選內環節點（如某張特許供應商字號），整個太陽圖會順滑旋轉放大，以該節點重新渲染中心點，顯示極深層子級分佈。
⑥ 供應商與型號交叉熱力密度圖 (Supplier-Model Volume Heatmap)
坐標定義：Y 軸為排名前 10 的 supplier_id，X 軸為排名前 20 的醫材 model。
色帶保護：填充值經過 Logarithmic scaling（對數縮放：
）處理。此機制可以防止某家醫院或經銷商出現單個百萬級大訂單時，導致其餘正常百/十件等級交易顏色完全變為死白色、喪失對比度之問題。
4.2 額外增加的 5 個 WOW 時尚視覺化
為了能更有深度地解剖配送數據，系統額外搭載了以下五個企業級視覺化圖表：
code
Code
+-------------------------------------------------------------------------------------------------------------------+
|                                                 11 大核心視覺化矩陣                                                 |
+--------------------------------------------------------+----------------------------------------------------------+
|                 6 大原始核心圖表（基本要求）              |                    5 大全新擴充圖表（新增）                 |
+--------------------------------------------------------+----------------------------------------------------------+
| ① Sankey Flow Chart (渠道動態流向圖)                    | ⑦ Top Customers Pareto (受配終端 Pareto 分佈圖)          |
| ② Layered Network Graph (網絡關係拓撲圖)                | ⑧ Supplier x Customer Flow Matrix (流向密度矩陣網格)      |
| ③ Time-Series Trends Overlay (配送時序與波動圖)          | ⑨ Week-Weekday Rhythm Heatmap (營運日曆與配送節奏冷熱圖)  |
| ④ Top Suppliers Pareto (原始 Pareto 累積分析圖)        | ⑩ Treemap: Supplier-License-Model (包裝品相多維樹狀圖)    |
| ⑤ Sunburst Chart (多級徑向大圈圖)                       | ⑪ Boxplay Distributions (交易包裝數量極端值分析箱形圖)    |
| ⑥ Supplier-Model Heatmap (供應商與型號交叉熱力圖)        |                                                          |
+--------------------------------------------------------+----------------------------------------------------------+
⑦ 客戶分佈 Pareto 累進圖 (Top Customers Pareto)
維度：對 customer_id 進行排序彙總，直觀檢視排名前 20% 的大客戶是否佔據了 80% 的醫材配送總量，透視配送終端集中度風險。
⑧ 雙維流向密度矩陣圖 (Supplier x Customer Flow Matrix)
維度：Y 軸代表頂級供應商們，X 軸代表核心客戶（Hospitals）。方塊代表兩者間交易流通的總數量，能一眼識別「專屬經銷商-院所迴路」。
⑨ 營運日曆與節奏矩陣 (Week-Weekday Rhythm Heatmap)
** Y 軸與 X 軸**：Y 軸為一年中第幾週 (Week of Year)，X 軸為星期一至星期日 (Day of Week)。
作用：展示特定醫材配送是否呈現極強的時序規律性（例如每週一集中配送入庫）。
⑩ 醫材法規品相多維樹狀圖 (Treemap: Supplier -> License -> Model)
外顯架構：以矩形嵌套網狀比例區塊，用層次展示不同供應商旗下，各類法規許可證的醫材體積配比。
⑪ 包裝數量波動極端值分析圖 (Boxplot Distributions)
維度：Y 軸為配送量，X 軸為 supplier_id 或 category。
說明：透過箱形圖標示出交易外圍異常值，快速排查填單錯誤或超額非典型調劑行為。
4.3 核心視覺化防崩潰保護機制 (Critical Plotly/Pandas Guardrails)
在 Python Plotly/Pandas 與 Streamlit 結合的架構下，有兩個常見的技術報錯可能導致整個介面硬崩潰（Blank UI Screen）。本規格書予以透明解決：
避免 Plotly Scatter/Network marker.color 型別錯誤
成因：在 plotly 散點圖中，若將非數值的字串分類（如 "Supplier", "Customer"）強行賦值給 marker.color，Plotly 檢驗器會崩潰，報 Invalid value ... in marker.color 錯誤。
保護措施：系統拒絕使用非合法色碼的純文字作為顏色向量。預先定義「節點類型配色映射表」（例如：{"supplier": "#0F4C81", "license": "#5B84B1", "model": "#95B4CC", "customer": "#FC766A"}）。根據節點類別（NodeType）屬性套用對應的 Web 色碼與透明度，徹底防範硬崩潰。
Pandas 日期時間 JSON 序列化失敗保護 (Timestamp NaT Protection)
成因：當對 DataFrame 進行 to_dict(orient='records') 導出或進行 API 交互時，若資料型態中參雜 Pandas 的 Timestamp 物件或缺失值（NaT、NaN），直接調用系統 json.dumps() 將引發 Object of type Timestamp is not JSON serializable 的異常。
保護措施：程式自定義 SafeJSONEncoder，繼承 json.JSONEncoder。若檢測到 pandas.Timestamp，強制執行 .isoformat() 轉為 ISO 字串；若遇到 pandas.NaT，強制序列化為 None (JSON 中呈現 null)。導出前調用安全打包函式：
code
Python
def safe_json_serialize(df_to_export):
    # 優先使用內建 df.to_json ，並指定 ISO 日期格式
    return df_to_export.to_json(orient="records", date_format="iso")
5. 多通道 AI Agent 引擎與代理工作空間 (Agent Studio)
大語言模型是配送工作室的核心大腦。本系統不流於一般的問答機器，而是建構出具有持續狀態保存（Iterative State）、可修改上下文（Editable Markdown）、具有 YAML 載入自定義實體（Custom Agents） 的 Agent 工作空間。
5.1 YAML & SKILL.md 大綱管理 (Workflow Configuration Studio)
系統並非將 Prompt 死寫在程式碼中，而是引進了本地數據配置器，使用者可隨時調整二個關鍵檔案：
agents.yaml (自訂代理集)：
code
Yaml
agents:
  - id: "trace_anomaly"
    name: "配送異常追溯代理"
    provider: "gemini"
    model: "gemini-3.1-flash-lite"
    temperature: 0.15
    max_tokens: 4000
    system_prompt: "你是一位資深的醫材流通合規審判長，專長為 GUDID 條碼檢測..."
    user_prompt: "對齊 Dataset A 與 B，挑出不平等的交易行，回報涉嫌竄貨、或幽靈配送的明細。"
  - id: "flow_predictor"
    name: "物流流量預警代理"
    provider: "google"
    model: "gemini-2.0-flash"
    temperature: 0.4
    max_tokens: 6000
    system_prompt: "你是一個專業健康醫療物流調度長..."
    user_prompt: "分析本配送時序，列出未來可能缺貨的許可證型號..."
SKILL.md (全域學科知識庫)：
此 Markdown 內容為全域 LLM 隱性注入中、英雙語之國際醫療器材法規制度（例如 FDA UDI 機制、歐盟 EMDN 代碼解碼、台灣 GUDID 系統等）。每當呼叫任何模型，系統都會在 System Prompt 頂部載入此知識，防止大模型幻覺（Hallucination）。
Heuristic 容錯標準化規則（Heuristic YAML Standardizer）
若使用者在上傳或貼上 agents.yaml 時缺少 format、max_tokens 或 provider 鍵值，系統在底層運行啟發式屬性對齊：
無 provider 
 預設賦予 "gemini"。
無 max_tokens 
 預設賦予 12000（最大支援限制）。
無 temperature 
 預設給予穩定度較高的 0.2。
对于完全混亂的 YAML，如果系統配有可用的大模型 Api Key，則會啟用 LLM 輔助校準（LLM Auto-Standardizer）：調用極小溫度的 Flash 模型，將亂序 YAML 編排為符合系統期待的規範 Schema。
5.2 多通路模型路由與最大 Output Tokens 限制 (Model Routers)
平台拒絕被任何單一雲端供應商綁定，在 Agent Studio 內建統一路由客戶端（Unified Router Client），支援：
Google Gemini 通道：連接 @google/genai (v2.4.0+)。推薦：gemini-3.1-flash-lite（低延遲、高回覆性）、gemini-2.0-flash、gemini-1.5-pro。
OpenAI 通道：支援 gpt-4o 與 gpt-4o-mini。
Anthropic 通道：支援 claude-3-5-sonnet (思維能力極佳) 與 claude-3-haiku。
xAI Grok 通道：支援 grok-2 以提供特異流向分析。
統一 API Key 安全配置：
預設優先級：系統啟動時，若檢測到雲端宿主端設定有 GEMINI_API_KEY 或 OPENAI_API_KEY，前端將主動載入並在 API 配置面版呈顯綠色 LED 徽章（● Key Loaded from Env - Protected），同時隱藏金鑰輸入框，杜絕二次演示或投影錄製時洩漏之風險。
臨時存儲：若環境中無金鑰，提供可折疊的 Password Input 表單，儲存於 st.session_state 中，重刷網頁即丟失，防範敏感流出。
最大 Output Tokens 深度調節：提供 Slider 控制。鑒於醫療分析需產出冗長、專業的稽核，平台特別將 max_output_tokens 滑動上限提高至實質的 12,000 tokens，保證大模型產出文獻級完整稽核報告，避免中途強行中斷。
5.3 5 大原始 AI 魔法 + 3 大全新 AI 核心空間 (8 AI Workspaces)
為釋放 Agent 的究極性能，平台在側邊欄與右側工作區設置了 8 個「一鍵式 AI 魔法空間」：
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
① 配送渠道異常追溯魔法 (Trace Anomaly Space)
原理：分析 Dataset A 與 B 的交集與補集。
價值：發現單向登記、發貨多於收貨、幽靈單號。大模型扮演「連環稽核員」，向有嫌疑的代理商提出「紅色指引」。
② 渠道物流流量預報魔法 (Flow Predictor Space)
原理：向 LLM 餵入以日、為單位的時間序列極簡統計序列。
價值：由 AI 結合產品型號與醫療常識（例如夏季心臟器材或特定手術網片庫存週轉規律），預報配送缺口。
③ 集中度供需配額風險魔法 (Supply Cluster Space)
原理：分析供應商和客戶的二維分佈（HHI，赫芬達爾指數原理）。
價值：識別系統中 80% 流量在少數實體的壟斷結構，產出「去中心化配送備份方案」。
④ 產品許可證合規與重分類審查 (License Audit Space)
原理：輸入許可證字號與對應器材中文品名。
價值：檢查類別名稱（如 E.3610）是否與許可證的正式結構（衛部醫器輸/衛署醫器製）相符，並對照法規庫警告品項，防範過期使用。
⑤ 綜合配送鏈分析大報告魔法 (Summary Markdown Space)
原理：將整套標準化後的數值矩陣拼為描述性字串（Summary Data JSON）傳遞。
價值：產出一份排版講究、使用 Mermaid 示意圖的 Markdown 格式高階決報，內含診斷發現、隱患列表與改善建議。
⑥ 【全新-WOW 額外 1】醫療器材品相一致性跨區稽核魔法 (License Align Audit)
工作原理：自動比對同一個 license_no 下的行資料，分析在大陸、台灣等不同受配地區時，是否出現「品名漂移」（同一個牌照編碼，在 A 地寫作 A 品名，B 地寫作 B 品名，存在盜牌或拼裝風險）。
特異產出：一致性分析矩陣，明確揪出冒牌醫材竄貨。
⑦ 【全新-WOW 額外 2】一物一碼 UDI 與 SN 標籤診斷魔法 (UDI-SN Trace Diagnostics)
工作原理：分析 UDI 字串位數（如 GTIN 14 碼合規性檢驗），排除多打空格、底線及亂打符號的情形，並提取批號 lot_no 作為標籤防偽診斷。
特異產出：一物一碼品質雷達圖（Traceability Quality Radar）。
⑧ 【全新-WOW 額外 3】臨期配送與醫療效期安全合規預警魔法 (Cross-Chain Compliancer)
工作原理：分析配送日期 deliver_date，並預估產品有效期限，主動算出配送安全時效（Shelf-life safety index）。
特異產出：列印所有「臨期進貨/即將過期」警告（例：申報日期為 2026/05/15，但器材在 2026/08/06 即過期，餘期不足百天），警告醫療機構防止植入體內發生悲劇醫療糾紛。
5.4 序列化 Agent 管道：雙向編輯工作區 (Iterative Pipelines Workspace)
本系統最特殊的 AI 執行機制是：支援將前一個 Agent 的輸出（經過使用者二次編修）直接作為下一個 Agent 的輸入背景！
一條典型的高效工作流管道如下：
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
6. Python 行動資訊圖 (Infographics) 與視覺引擎
系統內建了 Python matplotlib、seaborn 或 plotly.io 的底層腳本，由系統後台之 Python 直譯器直接調用。系統運行時不向用戶展示源代碼。這 6 幅精緻的 PNG 級靜態或向量科學圖表（可在報告中一鍵下載）是由 Python 底層以高 DPI (Dots Per Inch) 統一導出：
6.1 6 幅 Python 實體資訊圖（Infograph）設計規格
① 醫材合規漏洞與追溯雷達圖 (Traceability Compliance Radar Chart)
繪製原理：使用 matplotlib.pyplot 極坐標軸（Polar Axis）。
指標維度：包含「UDI 填寫率」、「日期一致性」、「許可證有效性」、「數量批配度（Quantity Sync Rate）」以及「經銷商認證數」。五個多維維度繪出完美雷達多邊形，漏洞區域一目瞭然。
② 庫存週轉與過期警示散點分佈圖 (Expiration vs. Lead-Time Scatter Chart)
繪製原理：X 軸為從配送到使用的週轉提前量（天數），Y 軸為醫材總庫存。
警告線：繪製紅黃藍三色漸變散點區塊，標誌出危險臨期品（紅色高光），作為法規警示。
③ 渠道配額不平衡雙向對折柱狀圖 (Inbound-Outbound Butterfly Chart)
繪製原理：採用雙向水平柱狀圖（Butterfly Plot）。
作用：左側展示 Dataset A 登記的批發出庫，右側對稱展示 Dataset B 登記的終端點收，若雙翼不對稱，即代表該醫療器材品牌在二級分銷中存在嚴重虛增。
④ 經銷實體網络複雜度矩陣圖 (Supply-Chain Entropy Heatmap)
繪製原理：針對複雜配送網路計算「香農熵（Shannon Entropy）」，轉為二維矩陣。
作用：顯示特定 UDI 的流動渠道混亂度。高混亂度（深紅色）表明經銷網絡層級繁瑣，有灰色配送與水貨滲入。
⑤ 醫材品類與許可證矩形樹狀圖 (Matplotlib Treemap Block)
繪製原理：使用 squarify 庫在 Matplotlib 套件中進行無重疊分割。
作用：展示骨科、心血管、眼科等品項在整體資料庫中的體積份額，文字自動縮放排版。
⑥ 配送週期頻率時序折線散點綜合圖 (Deliver Frequency Combined Plot)
繪製原理：底部使用 Bar 圖展示配送件數，頂部重疊 Scatter 與 Line 圖。
作用：展示一年四個季度醫材流向時序。
7. Pantone 藝術配色、雙語、與 Interactive Indicator 視覺設計
系統的 UI/UX 設計在視覺上要體現專業級（Professional Polish）的儀表尊貴感，拒絕無序色彩。
7.1 Pantone 巴拿馬經典 10 色系調色盤 (Pantone Palettes Specification)
系統於後台常駐 10 種源自大師級與 PANTONE 年度精選色系 (Painter & Pantone Styles)，每個風格定義了冷、暖、點睛三種 RGB/HEX：
調色盤風格 ID (Style Name)	PANTONE 年度/大師色碼	背景暖色 (Light Back)	主色調 (Primary Acc)	輔助亮色 (Secondary Acc)	系統暗示 (Glow Color)	視覺意象描述
Monet (莫內)	PANTONE Serenity	#F2F4F8	#5B84B1	#FC766A	#87A96B	朦朧藍紫與柔玫瑰粉，古典典雅
Van Gogh (梵谷)	PANTONE Classic Blue	#F7F9FC	#0F4C81	#F5CB5C	#39A2AE	深邃星空藍加金麥浪，對比極為強烈
Classic Classic	PANTONE Navy Blue	#F4F7F9	#0F4C81	#2D3748	#10B981	電商主調，深海藍藍灰，幹練穩健
Sleek Charcoal	PANTONE Grey	#F3F4F6	#374151	#D1D5DB	#EF4444	深炭色與冷灰、適合合規稽核儀表
Warm Peach	PANTONE Peach Fuzz	#FFF8F5	#FE7A5F	#9AC555	#4B5563	暖絨桃，柔和宜人，減低視覺疲勞
Hokusai (葛飾北齋)	PANTONE Great Wave	#F0F3F4	#1D4461	#E1AF65	#82B3AA	神奈川海浪深藍、浪花砂褐，波瀾壯闊
Vibrant Ultraviolet	PANTONE Ultra Violet	#F9F6FC	#5F259F	#FF1493	#64748B	神秘電磁紫，高前瞻感
Green Oasis	PANTONE Greenery	#F4F8F4	#589A4D	#EAA810	#22C55E	草木綠，彰顯醫療健康與配送活力
Solar Light	PANTONE Illuminating	#FFFDF2	#F5E050	#4B5366	#10B981	極致灰與亮麗黃，希望與堅定對決
Living Coral	PANTONE Coral	#FFF3F0	#FF6F61	#12B3B0	#60A5FA	明亮海面珊瑚，活潑且飽滿
「風格拉霸骰子 🎲（Style Jackpot）」：UI 介面一隅提供實體骰子（🎲 Style Jackpot）按鈕。點擊後，系統隨機載入本表 10 款調色盤之一，全局調整所有流動圖、熱力圖的主題與襯色。
7.2 雙語及亮暗雙主題切換引擎 (Dynamic Localization & Themes)
語言快速本地化 (Traditional Chinese / English)：
雙語系統定義了一個全局的翻譯包字典 LOCALIZATION_DICT，其中文介面默認採用標準的**「繁體中文（Traditional Chinese）」**，例如 醫療器材許可證字號（類型）、產品中文品名、基本DI，與台灣官方 GUDID 系統、衛福部食藥署（TFDA）公告字彙全面統一。一鍵點擊 Language 按鈕，介面所有說明文本、表格 Header 與 AI 的底層 Prompt 將在 20 毫秒內原地重載為英文。
亮暗兩合主題切換：
亮色（Light Mode）：背景採用高淨度 #F4F7F9、搭配卡片邊界 #E2E8F0，給人高信任度、臨床乾淨感。
暗色（Dark Mode）：專為醫療監控中心、深夜稽核調配。背景為 #0F172A (Tailwind Slate-900)，文字轉為高對比 #F8FAFC，各圖表色系亮度全域拉升 20% 以維持色差可見度。
7.3 精湛動效與 CSS、Glassmorphism 設定
code
CSS
/* 毛玻璃卡片 Glassmorphism Card Style */
.stMetric {
    background: rgba(255, 255, 255, 0.55);
    backdrop-filter: blur(12px) saturate(180%);
    border: 1px solid rgba(226, 232, 240, 0.4);
    border-radius: 12px;
    box-shadow: 0 4px 16px rgba(15, 76, 129, 0.05);
}

/* LED 側邊呼吸提示光 Heartbeat Glow Indicator */
.led-pulse-green {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background-color: #10B981;
    box-shadow: 0 0 10px #10B981, 0 0 20px rgba(16, 185, 129, 0.4);
    animation: pulse 1.8s infinite;
}

@keyframes pulse {
    0% { transform: scale(0.95); opacity: 0.7; }
    50% { transform: scale(1.15); opacity: 1; }
    100% { transform: scale(0.95); opacity: 0.7; }
}
8. 實時日誌 (Live Log)、診斷 LED 指標與數據導出
為杜絕系統在長達數分鐘的 AI 呼叫中呈現卡死狀態，本工作室全面融入了 「即時診斷指示燈矩陣」 與 「即時 Live Log 控制黑匣」：
8.1 系統運行診斷 LED 矩陣（Indicators Matrix）
API ROUTER：
● CLOUD_KEY (翠綠色)：偵測到環境金鑰，通道解鎖。
● SESSION_KEY (天藍色)：用戶臨時金鑰解鎖。
● MOCK_FALLBACK (亮黃色)：無可用 API Key，模型功能回落至本地預編測試資料集。
EXPLOITED DATA：
膠囊晶片直接顯示：Rows: 1,482 (A) | 1,200 (B)（即時計數器）。
SCHEMATIC MAPPING：
● 100% CANONICAL (綠燈)：11 類核心欄位全數在 Header 完美對位。
● PARTIAL_ALIGN (橙燈)：部分對位成功，有 3 欄採用預設未知值填充。
DOWN-DRILL EVENT：
顯著燈泡提示當前是否過濾了特定實體（如 B00079），若無點選則維持淡灰色。
8.2 Live Execution Log 黑匣子（即時運行日誌）
介面右側內嵌等寬字體（JetBrains Mono）的黑底終端機日誌框，精準捕捉以下流水帳：
code
Code
[15:47:37] CORE_INIT: Initialized WOW Medical Studio v2.5 Stable.
[15:47:37] IO_LOAD: Detected defaultdataset.json (A) presence. Parsing 1,482 records...
[15:47:38] ETL_HEURISTIC: Input matched synonyms: (基本DI -> udi_di, 產品描述 -> model).
[15:47:38] DATE_SIEVEY: Sanitized 114 fields from date formatting to YYYY-MM-DD.
[15:47:40] CLIENT_ROUTER: Target set to [gemini-3.1-flash-lite]; Temp: 0.15; MaxTokens: 4000.
[15:47:41] COMPILER_PYTHON: Running Matplotlib subprocess engine for Radar Compliance...
[15:47:42] SYSTEM_HEALTH: Frame performance: Memory at 14%; CPU at 2%. Ready.
8.3 多格式匯出與下載 (Export Hub)
在通過篩選或 AI 的稽核後，用戶可以點擊匯出按鈕：
JSON 導出：產出結構化、已去除 NaT 字符的 Canonical 陣列檔案。
CSV 導出：以標準 UTF-8 Big5 複寫機制下載，保證 Excel 正常開啟不亂碼。
XML 導出 (法規專用)：依照衛福部醫療器材流通標準格式 XML 架構，自動包裹成：
code
Xml
<MedicalDistributionReport version="2.5">
  <ExportTimestamp>2026-06-13T02:24:37</ExportTimestamp>
  <Record>
    <LicenseNo>衛部醫器輸字第034848號</LicenseNo>
    <DeviceName>“愛爾康” 水感散光日拋隱形眼鏡</DeviceName>
    <UDIDI>00730822294652</UDIDI>
    <Model>000000000010165367</Model>
    <Status>Verified</Status>
  </Record>
</MedicalDistributionReport>
9. 系統防禦與高容錯保障 (Fault-Tolerant Gateways)
醫療配送數據格式往往極其混亂。為應對現實環境中可能發生的各類邊界狀況與錯誤，系統在各核心處理模組設有完備的防禦機制：
9.1 除零與空陣列防禦保護 (Arithmetic Safe Enforcer)
在使用多維度複合篩選器時，極易因為篩選條件過於嚴苛，導致篩選後的過濾子集 
 或 
 的行數歸零（Row Counts = 0）。
崩潰隐患：若直接將空子集傳遞給 Pareto 計算或 HHI 熵計算，會因除以零（DivisionByZero）或調用 .iloc[0] 導致程式強行中斷。
保護措施：所有統計指標、高集中度 HHI 指數的計算函式一律在首行加載空值斷言（Assertion Guard）。若列數為 0，直接回傳常數值 0.0，並在 UI 上顯現「● NO DATA FILTERED」之溫柔提示，防止圖表崩潰。
9.2 圖表全局備份降級方案 (Visual Fallback Box)
崩潰隐患：若使用者貼入的資料中包含了極端不合規的 UDI 條碼或非法的 Unicode 控制字符，plotly 的 Sankey 或是 NetworkX 拓撲圖模組在解析邊與點（Nodes and Edges）的交互矩陣時可能拋出 ValueError: Empty data 異常。
保護措施：所有 11 個 Plotly 繪圖模組皆使用標準 try-except 包裝。一旦捕獲底層渲染錯誤，繪圖函式將立即停止執行 Plotly 繪製，改為在原畫面插補一個特製的「數據結構異常降級面板」。面板會以乾淨的 Markdown 格式，將關係點以直觀的 Markdown 表格顯示，並印出錯誤日誌：[Plotly Fallback Engine] 捕捉到非常規數據點，已安全降級為樹狀結構表呈現，點擊展開詳情。
9.3 LLM Anomaly JSON 容錯還原管線 (LLM Recovery Pipeline)
崩潰隐患：當調用具有高限制、高約束的 AI 魔法產出標準 JSON 或特定 Markdown 格式時，大模型時常在輸出的外圍加上不合規的邊界字符（如 json \n ... \n）或夾雜冗長字串，這會直接導致本地 Python json.loads 拋出 JSONDecodeError 異常。
保護措施：
預先利用正則表達式切除外緣：主動匹配並提取 r"({[\s\S]*})" 或 r"(\[[\s\S]*\])" 區塊，過濾外緣所有 Markdown 裝飾性字眼。
自動退回備份（Self-Correction Pattern）：一旦發現剖析完全破裂，會立即跳過 JSON 格式校準，原封不動地以 Markdown 純文字樣式在 Active Text Area 呈現給使用者編修，絕對不向使用者大白屏報錯。
10. 實例對比對照：Sample Datasets ETL 轉換示範
為了直確判定本平台對於導入數據標準化與轉換的實際成效，以下列出原始 TUDID 數據與 WHO Medevis 數據之解析對照全貌：
10.1 TUDID 數據 ETL 映射實例
原始輸入資料（TUDID Segment）
code
CSV
衛部醫器輸字第034848號,GS1,00730822294652,000000000010165367,“愛爾康” 水感散光日拋隱形眼鏡,PRECSION1 TOR 30P 850 145 -07.00 125 070
系統 ETL 解析與 Canonical 對應拆解
同義詞匹配檢測：
偵測到 6 個欄位。
第一欄 衛部醫器輸字第034848號 符合字串規則「衛[署部]醫器[輸製]字第\d+號」，判斷為核心 license_no。
第三欄 00730822294652 為 GS1 發碼，14 碼數字，判斷為核心 udi_di。
第四欄 000000000010165367 判斷為 serial_no 與產品品相識別符。
第五欄 “愛爾康” 水感散光日拋隱形眼鏡 清除全形引號，存儲為 "愛爾康" 水感散光日拋隱形眼鏡 寫入核心 device_name。
第六欄 PRECSION1 TOR 30P 850... 產品描述，含有型號等複合訊息，提取 PRECSION1 TOR 30P 作為 model。
缺少欄位之填充規則：
supplier_id：自 license_no 全域識別碼「持有人關係資料夾」檢索匹配為 愛爾康，否則為 UNKNOWN_SUPPLIER_034848。
deliver_date：讀取系統當前之本機時區日期 2026-06-13。
quantity：填補預設件數 1。
10.2 WHO Medevis 數據 ETL 映射實例
原始輸入資料（WHO Medevis Segment）
code
CSV
"Adapter, two-way, chest aspiration",2025 v2.1,Reusable,all,Accessories,Treatment / Resuscitation / Palliative care / Surgery,Specialized clinical,"First-level (district) hospital services, Second-level and third-level hospital services and specialized outpatient services",,"Emergency care, General surgery, Inpatient care, Intensive care, Long-term care, Outpatient care, Specialized surgery",A0701,ADAPTERS AND CONNECTORS,64085,"Non-ISO80369-standardized small-bore linear connector..."
系統 ETL 解析與 Canonical 對應拆解
主要欄位映射與型式判別：
第一欄 "Adapter, two-way, chest aspiration" 貼合名稱語境，寫入核心 device_name。
第二欄 2025 v2.1 MeDevIS Version，偵測到年份資訊，轉換為日期型式 2025-02-01 寫入核心 deliver_date，代表該醫材規格之全球法規同步日期。
第十一欄 A0701 為 Nomenclature code (EMDN)，符合層級定義，寫入核心 license_no。
第五欄 Accessories 寫入 category（次類別為 Treatment / Resuscitation）。
第十三欄 64085 EMDN Code，寫入核心 udi_di 作為全球器材條碼識別。
第十四欄 "Non-ISO80369-standardized small-bore linear connector..." 產品細部說明，判斷為核心 model。
缺少欄位之填充與安全機制：
由於原始數據偏向靜態醫材資訊法規結構、不含實體物流配送數據，其 quantity 預設補充為 1。
supplier_id 預設給予 "WHO_MEDEVIS"，customer_id 預設依據 "First-level (district) hospital services" 解析為 "District Hospital Services Level 1"。
此標準化處理，可讓使用者在 A/B 槽中，將 A 槽載入本土醫院點單，B 槽載入 WHO Medevis，藉此直接比對本國採購醫材是否完全符合 WHO 國際通用分類及 EMDN、GMDN 標準。此即為 WOW 級 A/B 沙盒對比與下鑽互動的巨大商用實戰價值。
11. 20 個綜合性 follow-up 討論問題 (20 Comprehensive Socratic Questions)
為了系統的後續詳細設計、性能瓶頸調優與潛在法規細節對位，開發與專案團隊應針對以下 20 個深度架構問題展開追蹤研討：
數據工程與架構設計 (Data Engineering & Scalability)
問題 1（大規模數據瓶頸）：若使用者上傳的 TUDID 資料高達 50 萬筆紀錄，本單頁 SPA 應用在 Streamlit 架構下的 st.session_state 對話記憶體是否會發生 OOM（Out of Memory）溢出錯誤？在不使用本地持久化 SQL 資料庫下，有哪些快取（Caching）或生成器分片（Generators chunking）優化策略？
問題 2（日期演算法覆蓋）：針對台灣 TUDID 資料夾中，可能存在的「民國年」（例如發照日期：1150514 即中華民國 115 年 5 月 14 日）與「西元年」（2026/05/14），ETL 轉換管道如何建立防錯機率，以避免將 1150514 誤算為西元 1150 年之荒謬時間點？
問題 3（UDI 機制對位）：當前 GTIN-14 與 GS1 條碼中，首位補零（如 00730822294652 實際代表 730822294652 產品碼）常導致關聯 A/B 數據時匹配失效。系統應對 udi_di 的匹配引擎進行「零字符對稱化去敏感」還是強行在資料庫端補足 14 碼之對位？
問題 4（XML 格式對接）：台灣 TFDA 的 GUDID 系統及衛福部醫療器材單一識別系統，目前是否有給定官方發布的進出貨與流通 XML Schema (XSD) 模板？本工作室的 XML 導出模組能否做到一鍵契合國家官方網端上傳（Web Service Entry-Points）所要求的安全通道與名空間（Namespace）驗證？
AI 智能與 AI Agent 控制 (AI Agency, Routing & Context Integrity)
問題 5（API 密鑰安全防護）：若 Hugging Face Spaces 採取多帳號共用預覽形式，如何 100% 確保 A 用戶使用 Session 模式在瀏覽器端暫存的 OPENAI_API_KEY，絕對不被 B 用戶在後端運算中跨線、跨請求截獲？
問題 6（Gemini 最低延遲）：在「8 大 AI 魔法空間」中，gemini-3.1-flash-lite 雖然具備極高的響應速率，但在分析極其複雜、存在「多重不一緻性」的雙邊配送鏈時，其邏輯推理深度（Reasoning Depth）是否足夠？本系統是否應採用「分層路由機制（Layered Routing Pattern）」—— 即一般篩選用 Flash-Lite，深度合規漏洞稽核用 gemini-1.5-pro 或 claude-3-5-sonnet？
問題 7（Token 超限處理）：當 A/B 對比後產生了大量的異常對齊報告、涉及上千行文字，拼裝成 JSON 放入 Prompt 後容易超出大模型的 Input Window 最大極限。系統在後台應套用何種「智慧文本壓縮技術（Sieve-Chunking）」或 MapReduce 範式，在不遺失交易特徵下提取精簡資訊？
問題 8（Agent 管道可信度）：在「Iterative Chains（連續步進 Agent 管道）」中，使用者對上一個 Agent 響應內容的「人工修剪與二次修改」如果包含了人為的主觀偏見，如何避免下一個 Pipeline 的 AI 產生思維偏差？是否有必要在 YAML 的 system_prompt 中預設加入「人為偏見干擾保護條款」？
Python 視覺化與互動機制 (Advanced Live Analytics & Infographics)
問題 9（Plotly 網頁防卡顿）：配送網絡拓撲圖（Network Graph）如果含有超過 2,000 個節點，Streamlit 在 IFrame 模式下會發生極致 lag 甚至崩潰。除了限制 Top 50 流外，是否應引入動態自適應算法（如 Node-Clustering 算法），將同一許可證下的多個型號包裝為單一物理大點，在點擊後再展開？
問題 10（Matplotlib 圖像中文字體）：由於 Python 運行於 Hugging Face 的 Linux 容器沙盒環境下，系統常缺省「微软正黑體 (Microsoft JhengHei)」等中文字型，這會導致 Python 原生輸出的 6 幅科學資訊圖（Matplotlib Radars, Bar Plots）出現大量中文字體顯示「□□（豆腐塊）」的問題。系統應如何預載打包 TTF 免費開源中型字體（如思源黑體）並進行字型動態指定定位？
問題 11（下鑽聯動響應）：當前 IFrame 環境隔絕了 Streamlit 原生組件的雙向數據同步。在 streamlit-plotly-events 不穩定的情況下，如何設計一套穩健的局部 state 變更，使得用戶在 Tab 欄選取特定許可證時，熱力圖與 Pareto 圖可以實現接近原生應用的「零感覺閃爍」重載？
問題 12（時序日曆與假日因子修剪）：醫材配送常受限於節日及六日醫院不點收而中斷，週六日之數據量暴跌在時序圖上會形成刺耳的鋸齒。在 11 大核心視覺化中的「營運日曆與節奏矩陣」裡，系統應如何區隔正常無派件日與物流非典型斷鏈日？
Pantone 美學、雙語與美工呈現 (Pantone Polish, Dual-Lang & UI Aesthetics)
問題 13（亮暗色動態色彩轉譯）：當使用者將系統由亮色切換為暗色時，Python 地圖與拓撲圖之背景、節點連線色（通常在 Python 後端預先渲染）並非能隨前端 Tailwind 行內更改而更新。系統如何配置全域色碼監聽器（Global Theme Translator），以保證 Matplotlib 與 Plotly 圖像的主色、輔助色在 30 毫秒內同步更新而不會造成視覺上「黑白不一」的突兀感？
問題 14（Pantone 巴拿馬 10 色兼容）：這 10 套調色盤雖然極具視覺辨識度，但是對於「特定醫材色系警示（例如醫療上通常以紅色標示過期或廢棄）」是否會產生衝突？例如，若調色盤選擇了「葛飾北齋」以藍色和砂褐為主調，而系統需要顯示紅色警示，是否需要定義特定「調色盤抗性警戒色（Static Warning Color）」？
問題 15（UX 毛玻璃可用性）：Streamlit 在暗色模式下，CSS 渲染的 Glassmorphism backdrop-filter 對於老舊顯卡或 Safari 瀏覽器在多圖像運作下常導致頁面嚴重掉幀，是否有動態回落（Bypass CSS filter）之備案？
問題 16（雙語翻譯一致性）：雖然 LOCALIZATION_DICT 解決了前端靜態文字，但是在 Agent 執行的過程中，如何保證 AI 在 Traditional Chinese (繁體中文) 下分析，能精準沿用「許可證字號」等本土正式官僚字眼，而在 En 切換下能自然改用 "GMDN Device Registration Code"？是否需要在 Prompt 首部引入動態 Prompt-Injection 雙語包裹器？
實戰演示與邊界案例測試 (Operational Validations & Extreme Edges)
問題 17（異常臨界值的定義）：在全新的 WOW AI 魔法 8（臨期配送流向預警）中，針對「臨期（Expiration Lead Date）」的警告警戒天數（例如：距離失效日僅剩 180 天或 90 天），應該由系統內部定死為統一值，還是該將其和醫材品類 category 掛鉤（例如隱形眼鏡保質期長短與心臟起搏器存在天壤之別）？
問題 18（UDI 多發碼機構兼容）：TUDID 數據庫首列即列明「UDI發碼機構」（包含 GS1, HIBCC, ICCBBA 等）。HIBCC 與 GS1 在 DI 碼的起頭編碼、檢驗規則有著天壤之別，一物一碼標籤診斷模組（WOW AI 魔法 7）應如何自適應切換其條碼幾何與校驗（Check-digit）校正公式？
問題 19（Live 運行日誌的磁碟防爆）：在多用戶重度點選、重度呼叫 API 的 Spaces 生態下，Live Execution Log 若無限度向記憶體緩衝區追加字串，是否會帶來嚴重的網頁效能耗費？當緩衝區行數大於 1,000 行時，是否該實施「FIFO (First-In, First-Out)」滑動刪除保護？
問題 20（幽靈經銷商稽核）：在 Dataset A「供應商申報配送量」大於 Dataset B「院所採購登記量」時，除了被常規制判定為「未點收延遲」或「代理商竄貨」外，在法律上極高機率涉及「一證多套、贈品黑市販售」等非合規漏洞。面對此類極度敏感的產業不合規指控，Agent 的 Summary Markdown 報告應採取多大的客觀度與法律避險用詞，在既提出警示之時不對用戶商譽造成提前指控之法律風險？
12. 結論與專家設計建議 (Architectural Expert Summary)
本技術規格書所刻畫之 WOW Medical Distribution Studio (v2.5 Stable) 全面擺脫了傳統醫療器材流通展示工具的死板印象，將「巴拿馬藝術美學 Pantone 配色系」與「Agentic AI 連續步進工作空間」做了實質性之技術聯結與理論落地。
通过 11 大核心 Plotly 視覺化矩陣 配合雙資料集 A/B Sandbox (對比沙盒) 的靈巧架構，用戶可以極其敏捷、毫無卡頓、在安全和強悍容錯保護的前提下，完成對大宗醫材數據由巨觀（ Pareto 集中度分析、Sankey 流向）到微觀（單一節點點擊下鑽、條碼 SN 字元幾何校準）的全面探勘與多維稽查。本系統是真正的「Professional Polish」與「強大實戰資料工程」的典範整合，將為跨國醫材數據標準化和供應安全的監管研究，立下無可取代的架構豐碑。
