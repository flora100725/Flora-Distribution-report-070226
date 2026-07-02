醫療器材來源流向智慧監管系統 (Medical Device Distribution Analytics & Regulatory System, MD-DARS)進階多智慧體 AI 稽核平台技術規格說明書 (Technical Specification Document)本技術規格說明書旨在為「衛生福利部食品藥物管理署 (TFDA) 監管稽核官員」打造一款專屬的進階大數據與主動式智慧體稽核平台。系統完全基於台灣《醫療器材來源流向資料建立及管理辦法》之法規業務邏輯設計，用於自動化貫通進口採購（Purchase）、物流分銷（Distribution）與機構實體位置（Geolocation）之異質數據鏈路，提供秒級斷鏈推理、合規漏洞警告及一鍵緊急回收追溯之核心治理能力。第一章、系統架構與架構設計 (System Architecture & Framework)MD-DARS 採用先進的解耦式多智慧體協作架構 (Decoupled Multi-Agent Architecture)，底層運算資源全面採用 Gemini 2.5 Flash 作為預設之核心大型語言模型（LLM）。1.1 預設 LLM 路由策略與配置原則系統將 Gemini 2.5 Flash 作為預設模型，主因其具備極低的推理延遲（Latency）與高度優化的上下文處理效率，極度適合高頻率的資料庫結構映射、SQL 語法即時生成（Text-to-SQL）與大規模序列文本的清洗校驗。為確保系統之高度靈活性，平台內建「動態模型路由閘道器 (Dynamic Model Routing Gateway)」，提供圖形化介面（UI）供稽核官員手動切換核心大模型，並支援即時修改 System Prompt 與 Few-Shot 範例。切換矩陣與適用場景如下表：表一：核心 LLM 路由與場景配置矩陣模型名稱角色定位適用場景最大上下文 (Context Window)官員可自訂度Gemini 2.5 Flash(預設模型)全量數據 ETL 轉換器即時 SQL 生成代理結構化 CSV 清洗、欄位自動模糊對齊、動態報表生成1,048,576 Tokens系統 Prompt、Few-Shot 樣版、高風險異常閾值設定Gemini 2.5 Pro跨法人法律漏洞深度推理器綜合合規報告終審撰寫官幽靈序號成因分析、跨年度申報趨勢預測、正式行政處分書公文撰寫2,097,152 Tokens法律條文權重設定、公文語氣語調微調（嚴厲/宣導/警告）其他第三方模型(如 Qwen / ChatGPT / Grok)外部異質 API 多模態串接器外部網頁食藥署查驗登記公告抓取、跨國 UDI 條碼特徵比對依原生模型而定外部 API 欄位映射與自訂 Prompt 邏輯1.2 多智慧體 (Multi-Agent) 協作拓撲系統拒絕採用傳統的單一線性 Prompt 流程，而是透過 LangGraph / AutoGen 框架建構了四個具備獨立狀態機 (State Machine) 的專屬 Agent。各 Agent 透過「中央訊息匯流排 (Central Message Bus)」共享目前稽核的 Context。                       ┌─────────────────────────┐
                       │   官員使用者介面 (UI)   │
                       └────────────┬────────────┘
                                    │ 1. 提交 CSV 數據 / 自然語言提問
                                    ▼
                       ┌─────────────────────────┐
                       │ 模型路由閘道器 (預設 Flash)│
                       └────────────┬────────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│ 1. ETL 清洗 Agent │ ───►  │ 2. 合規稽核 Agent │ ───►  │ 3. 空間計量 Agent │
│ (Data Cleaning)  │       │ (Compliance Core)│       │ (Spatial Analytics)
└──────────────────┘       └──────────────────┘       └──────────────────┘
         │                          │                          │
         └──────────────────────────┼──────────────────────────┘
                                    │ 2. 彙整各智慧體中間推理鏈 (CoT)
                                    ▼
                       ┌─────────────────────────┐
                       │ 4. 報告總成 Agent (Pro) │
                       └────────────┬────────────┘
                                    │ 3. 生成 4000字 結構化綜合報告
                                    ▼
                       ┌─────────────────────────┐
                       │   法規遵循與查核完成     │
                       └─────────────────────────┘
各智慧體職責分工如下：ETL 數據工程智慧體 (Data Ingestion Agent)：負責解析使用者上傳之進出貨原始 CSV / Excel，自動識別包含日期的變體、UDI 欄位異質性，並透過 Code Interpreter 模組執行資料格式標準化。合規漏洞推理智慧體 (Regulatory Compliance Agent)：內建《來源流向辦法》核心法條知識庫，專職執行「唯一序號 (Serial Number) 外連接碰撞」，揪出有流向無來源、時序倒置、一物多報等重大法規違規。空間幾何計量智慧體 (Spatial Geolocation Agent)：串接 Geolocation API，計算 Distributor 與下游醫療機構間的實體物流軌跡、區域消耗權重與安全庫存熱點。報告生成與公文總成智慧體 (Executive Report Agent)：通常自動升級路由至 Gemini 2.5 Pro，負責收集上述三個 Agent 的中間推理鏈 (Chain-of-Thought, CoT) 數據，編排並撰寫出完全符合 TFDA 格式之綜合合規稽核報告。第二章、數據工程與資料庫架構 (Data Engineering & Database Schema)為確保大數據量下的運算效能，系統採用 PostgreSQL (內建 TimescaleDB 時序擴充與 pgvector 向量擴充) 作為底層核心資料庫。所有上傳的數據均會被實體化存入結構化資料表中，而法規文本則存入向量欄位以供 RAG 檢索。2.1 核心結構化資料表網格 (Database Table Schemas)2.1.1 進貨採購實體表 (md_purchase_records)SQLCREATE TABLE md_purchase_records (
    purchase_id SERIAL PRIMARY KEY,
    seq_num INT NOT NULL,                          -- 原始序號
    declaring_entity VARCHAR(100) NOT NULL,        -- 申報業者代碼
    received_date DATE NOT NULL,                   -- 收貨日期 (標準化 YYYY-MM-DD)
    supplier_code VARCHAR(100) NOT NULL,           -- 供應商代碼
    license_number VARCHAR(100) NOT NULL,          -- 許可證號 (如衛部醫器輸字第030747號)
    chinese_product_name TEXT NOT NULL,           -- 中文品名
    udi_di VARCHAR(50) NOT NULL,                   -- UDI-DI 產品識別碼
    device_subcategory TEXT,                       -- 醫療器材次類別
    lot_number VARCHAR(50),                        -- 產品批號 (Lot)
    serial_number VARCHAR(50) NOT NULL,            -- 產品序號 (SN - 核心鍵值)
    product_model VARCHAR(50),                     -- 產品型號
    quantity INT NOT NULL DEFAULT 1,               -- 數量
    unit_type VARCHAR(20),                         -- 單位 (個/組)
    manufacture_date DATE,                         -- 製造日期
    expiration_date DATE,                          -- 有效期間/保存期限
    return_info INT DEFAULT 0,                     -- 退貨資訊 (0:無, 1:有)
    remaining_quantity INT,                        -- 剩餘數量
    created_at TIMESTAMP NOT NULL,                 -- 系統資料建立日期
    ingested_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP -- 平台載入時間
);
CREATE INDEX idx_purchase_sn ON md_purchase_records(serial_number);
CREATE INDEX idx_purchase_udi ON md_purchase_records(udi_di);
2.1.2 流向交貨實體表 (md_distribution_records)SQLCREATE TABLE md_distribution_records (
    distribution_id SERIAL PRIMARY KEY,
    seq_num INT NOT NULL,
    declaring_entity VARCHAR(100) NOT NULL,        -- 申報經銷商代碼
    delivery_date DATE NOT NULL,                   -- 交貨日期
    supply_target VARCHAR(100) NOT NULL,           -- 供應對象 (下游醫院代碼)
    license_number VARCHAR(100) NOT NULL,
    device_subcategory TEXT,
    udi_di VARCHAR(50) NOT NULL,                   -- 對齊進貨端之 UDID
    chinese_product_name TEXT,
    lot_number VARCHAR(50),
    serial_number VARCHAR(50) NOT NULL,            -- 產品序號 (SN - 核心鍵值)
    product_model VARCHAR(50),
    quantity INT NOT NULL DEFAULT 1,
    unit_type VARCHAR(20),
    manufacture_date DATE,
    expiration_date DATE,
    ingested_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_dist_sn ON md_distribution_records(serial_number);
CREATE INDEX idx_dist_target ON md_distribution_records(supply_target);
2.1.3 地理位置基礎主檔表 (md_geolocation_stations)SQLCREATE TABLE md_geolocation_stations (
    entity_id VARCHAR(100) PRIMARY KEY,            -- 機構代碼 (如 B00047, C05816)
    official_name VARCHAR(200) NOT NULL,           -- 官方登記名稱
    entity_type VARCHAR(50) NOT NULL,              -- 實體類型 (Distributor / Hospital_Group)
    postal_code VARCHAR(10),                       -- 郵遞區號
    street_address TEXT NOT NULL,                  -- 實體完整地址
    latitude NUMERIC(9,6) NOT NULL,                -- 緯度
    longitude NUMERIC(9,6) NOT NULL,               -- 經度
    geom GEOMETRY(Point, 4326)                     -- PostGIS 地理幾何空間欄位
);
CREATE INDEX idx_geo_spatial ON md_geolocation_stations USING GIST(geom);
第三章、智慧體功能核心與三大 Wow AI 亮點功能 (Wow AI Features)為了大幅超越傳統 BI 圖表系統，本平台特化研發了三大具備顛覆性（Wow Effect）的監管 AI 功能，旨在直接賦予稽核官員預知風險、反向穿透與自動化執法公文產出之能力。3.1 Wow AI 亮點一：全島 UDI 溯源鏈「時空斷鏈反向推理引擎」(Spatio-Temporal Broken-Chain Deduction)核心痛點：傳統系統在面對經銷商漏報、多重報、錯填序號時，只能回傳「找不到資料」，需要人工翻閱無數 Excel 交叉比對。Wow 解決方案：本系統開發了「時空斷鏈反向推理引擎」。當合規 Agent 發現某一筆流向紀錄（例如：臺灣百特於 3月31日 交貨給奇美醫院之序號 RNE644378S）在進貨表中完全缺乏來源時，Agent 會自動啟動演算法與大模型推理：地理鄰近度比對：自動掃描半徑 50 公里內所有擁有相同產品型號（W2SR01）之醫院與經銷商庫存。編輯距離字元修正：利用 Levenshtein Distance 演算法，檢查是否有鄰近機構誤將 RNE644378S 打成 RNE6443785 或 RNE644378B。模型語意推理：調用預設之 Gemini 2.5 Flash，結合歷史多重代理商採購路徑知識（如美敦力與百特存在複配關係），動態模擬出該序號最可能之真實進口源頭（機率權重百分比），並在前端拓撲圖上以虛線標示出「幽靈流向之潛在實體軌跡」。3.2 Wow AI 亮點二：次世代多模態「一鍵全台緊急回收與病患照會智慧體」(One-Click Smart Recall & Patient Notification Planner)核心痛點：當 TFDA 收到國際 FDA 警訊（如美敦力 Azure 心臟節律器某特定批號存在電池液外漏風險）時，官員需耗費數週發文、催收各醫院回報，嚴重耽誤病患救治時機。Wow 解決方案：平台介面設有「緊急回收指揮艙 (Emergency Recall Cockpit)」。官員僅需輸入一通自然語言指令或直接上傳國外 FDA 之英文 PDF 公告。多模態解析：Gemini 模型自動讀取英文 PDF，主動擷取受影響之產品型號、產品批號或序號區間。數據庫秒級碰撞：自動執行跨表關聯，精確撈取目前該批號在全台各醫療機構之進貨賸餘數量與地理經緯度。智慧化病患查核計劃生成：Agent 會在一秒內自動為官員產出一份「全台緊急回收部署計劃書」，詳細列出：哪幾家醫院（如台中榮總）目前收領最多、哪些序號即將面臨過期、並自動生成完全客製化、符合台灣醫療公文格式的「各院所限期回收公文」與「病患緊急照會簡訊草稿」。3.3 Wow AI 亮點三：生成式 AI「行政裁罰與合規建議書自動生成智慧體」(Generative Regulatory Enforcement & Citation Composer)核心痛點：官員在抓到業者違反《來源流向辦法》（如成大醫院超期 47 天才申報）後，需花費大量時間翻閱法規、撰寫嚴謹的行政處分書，耗時耗力。Wow 解決方案：當合規稽核 Agent 判定某機構違規屬實後，系統會自動切換路由至 Gemini 2.5 Pro，並啟動此執法公文生成器。法規精確 RAG 檢索：自動調閱《醫療器材管理法》第七十條等相關裁罰條款與金額範圍。公文自動化編寫：將稽核發現之具體違規事實（包含洗淨後之原始數據序號、超期天數、違反法條款次）完全填入。一鍵匯出：自動生成一份具備極高法律嚴謹度之行政裁罰書與企業合規改善要求建議書草稿（Word/PDF 格式），官員僅需覆核、簽核即可一鍵發文，執法效率提升 $90\%$ 以上。第四章、使用者介面與交互設計 (UI/UX Design Specification)MD-DARS 拋棄了傳統死板的 BI 系統，採用混合式 UI 架構。介面分為三大核心區塊：對話式 AI 指揮艙 (Chat-Centric Command Window)、動態時空地圖看版 (Spatio-Temporal Dashboard)、以及 Agent 思考鏈透明化面版 (CoT Trace Board)。4.1 對話式 AI 指揮艙與 Prompt 管理系統頂部為全寬之自然語言輸入框。官員可直接輸入：「幫我查一下，今年三月到四月間，全台有哪些醫院申報心臟節律器進貨嚴重超期？請列出前三名並幫我準備給他們的催辦公文草稿。」介面右側提供「Prompt 進階管理面板」，允許資深官員自由切換底座模型（如將預設的 Gemini 2.5 Flash 切換為 Gemini 2.5 Pro），並可展開檢視與修改系統 Prompt（System Role Text），例如微調 Agent 對於「異常申報行為」之統計學閾值定義（例如定義 $Z-Score > 2.5$ 爲異常）。4.2 Agent 思考鏈透明化面板 (Chain-of-Thought Visualizer)為徹底消除大模型的「黑盒子」疑慮，當官員提問後，介面左側會以手風琴（Accordion）動態下、拉式組件展示 Agent 的思考鏈。官員可以清晰看到：[ETL Agent]：正在將上傳之 "2026/05/06" 轉換為標準 SQL Date "2026-05-06"... [成功][Compliance Agent]：正在執行 SQL 語法：SELECT * FROM md_purchase_records WHERE ...[Compliance Agent]：發現序號 RNE644378S 存在分裂漏洞。正在呼叫時空斷鏈反向推理引擎...[Report Agent]：正在呼叫 Gemini 2.5 Pro 編排最終 Markdown 報告...此設計大幅提升了行政執法之可信賴度（Trustworthiness）。第五章、安全性、資安合規與多租戶架構 (Security & Compliance)由於醫療器材來源流向數據涉及廠商之進口商業機密，甚至可能進一步串接病患之實體隱私，本平台在資安設計上採取最高規格之防禦部署。5.1 數據去識別化（Anonymization Pipeline）任何上傳至平台的 CSV 檔案，若經系統偵測包含敏感個資（如醫師姓名、手術病患身分證字號、病歷號碼），將在 Memory（記憶體空間） 階段即刻觸發 PII（個人識別資訊）遮罩引擎，以 SHA-256 加密演算法將其轉換為虛擬雜湊碼，絕不將原始敏感個資寫入實體資料庫盤區。5.2 多租戶與嚴格角色權限控制 (RBAC)系統原生支援多租戶隔離（Multi-Tenancy）。權限分級如下表：表二：使用者權限與功能矩陣角色安全級別適用對象資料庫讀取範圍是否可調用三大 Wow 功能是否能更換模型與修改 PromptLevel 3 (最高權限)TFDA 總署高級稽核官員全台所有進口商、經銷商、醫療機構之全量數據是（全功能解鎖）是（可調整核心 System Prompt 與路由）Level 2 (分區權限)地方衛生局查核人員僅限該管轄區域（如台中市）內醫療機構之數據僅限緊急回收與病患照會功能否（僅能使用預設 Prompt）Level 1 (外部租戶)申報醫療器材廠商 / 醫院僅能讀取與寫入該法人自身之申報紀錄（完全隔離）否（僅具備基本進銷存校驗功能）否第六章、優化與系統擴展延伸提問 (Follow-up Questions)為協助系統架構師、資訊處主管及 TFDA 執法專家進一步收斂細部規格，本平台特別梳理出以下 20 個全面性的技術與法規追蹤提問：Gemini 2.5 Flash 的動態 Token 管理：面對單次上傳超過 50 萬筆的巨量流向 CSV，若直接將原始文本丟入大模型會面臨 Token 溢出與成本高昂問題。我們是否應強制規定：預設狀態下，ETL Agent 必須先使用 Python pandas 執行局部的區塊分割（Chunking）與結構化寫入 PostgreSQL，而非讓大模型直接吞入整張表？System Prompt 的版本控制：當官員在圖形化介面修改了 Agent 的核心 System Prompt（如變更合規判定的嚴格度）後，系統是否需要內建 Git 等級的版本控制（Version Control）與回滾（Rollback）機制，以防止 Prompt 遭到意外損壞？Text-to-SQL 的安全沙箱 (SQL Injection 防護)：當官員輸入自然語言，Gemini 自動生成並執行 PostgreSQL 語法時，如何確保模型生成的 SQL 不會包含惡意的刪除指令（如 DROP TABLE）？我們是否應強制對 Agent 之 SQL 執行帳號採取僅限唯讀（Read-Only）之權限防禦？Wow 功能一的模糊比對閾值設定：時空斷鏈反向推理引擎在執行序號字元修正時，編輯距離（Levenshtein Distance）的容許最大閾值應設為多少（如最多容許錯漏 2 個字元）？若設得太寬，是否會導致大量不相關序號產生錯誤關聯？Wow 功能二的多模態 PDF 解析邊界：若國外 FDA 發布之緊急回收公告為「掃描版（Scan Only）之圖片型 PDF」而非電子字型 PDF，預設之 Gemini 2.5 Flash 多模態視覺解析能力在處理複雜表格時之精確度極限為何？是否應強制引入地端 OCR 作為輔助線路？Wow 功能三之行政處分書法律效力審查：Gemini 2.5 Pro 生成的行政裁罰公文草稿，如何與現行的「公文電子交換系統 (DI-EM)」進行標準 API 串接？是否需要設計專門的「人機協同 (Human-in-the-Loop)」查核關卡，強制要求法務官員電子簽章後方可發文？PostgreSQL 與 PostGIS 的效能瓶頸：在執行全台醫材物流熱點空間計量時，若結合TimescaleDB 進行跨年度時序分析，底層 GIST 幾何索引（Spatial Index）應如何優化以維持秒級響應？跨法人一物多報之法律認定原則：當發生美敦力與百特同時宣告同一序號之實體去向分裂時，在法律認定上，平台應預設「兩者皆為違規」並同時發出照會，還是優先判定原廠總代理（美敦力）之數據為真？離線地端部署（Air-Gapped Deployment）可能性：考量到國安與特種醫療機密，若 TFDA 要求本系統必須完全安裝於「不連外網的實體隔離機房」，我們應如何解決大模型（Gemini 系列之 API 依賴）的替代方案？是否需要改採國產之中文化地端開源模型（如 TAIDE）？單位轉換與醫療器材 BOM 主檔：進貨端常出現一「組」包含多個獨立序號之組件（如導線、脈搏產生器、電極扣）。系統是否需要增設一張 md_device_bom 主檔表，供 Agent 在計算剩餘數量時自動展開組件核銷？時序倒置（時間悖論）之容錯區間：若經銷商出貨日比醫院收到貨日提早了 1 至 2 天，這在物流實務上可能僅屬「兩者結帳時差或在途物資（Goods in Transit）」。系統應設定多少天數之內之時序錯位可被判定為正常物流誤差而非違規？多租戶隔離之安全性稽核（SaaS Security）：若未來開放全台各家經銷商登入平台檢視自身申報合規度，PostgreSQL 的 Row-Level Security (RLS, 行級安全原則) 應如何撰寫，以確保絕對不會發生跨廠商之商業數據外洩？高風險效期預警（Shelf-Life Buffer）之動態調整：針對心臟節律器與一般骨科人工關節，其法定包裝無菌效期與臨床安全緩衝期大不相同。系統是否應支援「依許可證次類別動態調整過期危險期閾值」之功能？異步報告生成機制 (Asynchronous Polling)：Gemini 2.5 Pro 在撰寫包含豐富圖表、高達 4,500 字之綜合合規報告時，生成時間通常長達 30 至 60 秒。前端 UI 應如何配置 Webhook 或 WebSocket 長連接，以優化官員在等待報告生成時之使用者體驗？API 限速與超時處理（Rate Limiting）：在大規模季度稽核期間，多名官員同時發起 Text-to-SQL 與報告生成任務，導致 Gemini API 達到頻率限制（TPM/RPM Limit）時，系統底層之佇列管理（Queue Management）應如何設計？歷史申報快照（Audit Trail Log）：若業者在收到官員之超期警告後，偷偷登入系統「補正並修改」其原始收貨日期，MD-DARS 系統是否應內建不可篡改之「申報軌跡日誌表」，以記錄所有數據之修改歷史？多模態外部 RAG 知識庫之同步頻率：為確保 Agent 引用之法規與裁罰案例絕對正確，底層向量資料庫（pgvector）與全國法規資料庫、衛福部食藥署查驗登記公告之同步頻率應設為每日自動爬取還是每週更新？圖表渲染套件相容性：Markdown 報告中定義之六大核心分析圖表（如桑基圖、地理熱點網絡圖），在前端匯出為 PDF 或 Word 時，系統應採用何種無頭瀏覽器（Headless Browser，如 Puppeteer/WeasyPrint）渲染技術，以確保圖表不發生排版錯位？病患緊急照會簡訊之個資合規（GDPR/個資法）：Wow 功能二在自動生成病患緊急通知簡訊草稿時，如何確保簡訊內容本身不涉及任何敏感病名，以防止病患手機簡訊遭他人窺視而引發醫療糾紛？AI 評估框架建置 (Evaluation Regimen)：在系統上線前，資訊處應如何建置一組包含至少 100 筆人工刻意製造之合規漏洞測試集（包含幽靈序號、時間倒置等），用以評估預設模型在斷鏈推理上之召回率（Recall Rate）與精確度（Precision）？
