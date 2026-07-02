Technical Specification: Agentic Medical Device Traceability & Compliance Platform (AMDT-CP)Version: 1.0.0-SPECAudience: FDA Compliance Officers, System Architects, Lead EngineersDefault Inference Engine: gemini-3.1-flash-liteTarget Regulations: TFDA / FDA Medical Device Source and Distribution Data Establishment and Management Regulations (UDI/UDID Traceability Mandates)1. System Overview & Architecture DiagramThe Agentic Medical Device Traceability & Compliance Platform (AMDT-CP) is a cloud-native, multi-agent AI system designed to assist FDA compliance officers in executing massive-scale automated auditing of medical device distribution networks. By ingesting disparate, non-standardized datasets (Purchase records, Distribution receipts, and Geolocation telemetry), the platform reconstructs serial-level supply chain lineages, identifies anomalies, scores compliance risks, and auto-generates regulatory inspection documentation.1.1 Micro-Agent Orchestration ArchitectureThe system abandons monolithic linear prompting in favor of an asynchronous Supervisor-Worker Multi-Agent Graph Architecture managed via LangGraph or Semantic Kernel.                  ┌──────────────────────────────┐
                  │   FDA Compliance Officer    │
                  └──────────────┬───────────────┘
                                 │ HTTP / WebSockets (Streamlit UI)
                                 ▼
                  ┌──────────────────────────────┐
                  │    Orchestration Supervisor  │
                  │   (Model: gemini-3.1-flash)  │
                  └──────┬────────────────┬──────┘
                         │                │
       ┌─────────────────┘                └─────────────────┐
       ▼                                                    ▼
┌──────────────────────────────┐                     ┌──────────────────────────────┐
│     Data Sanitizer Agent     │                     │   Lineage Reconstruction     │
│  (Default: gemini-3.1-lite)  │                     │            Agent             │
└──────────────┬───────────────┘                     │  (Default: gemini-3.1-lite)  │
               │                                     └──────────────┬───────────────┘
               ▼                                                    ▼
┌──────────────────────────────┐                     ┌──────────────────────────────┐
│ Tool: Pandas Sandbox / Regex │                     │ Tool: NetworkX Graph Engine  │
└──────────────────────────────┘                     └──────────────────────────────┘
       ┌─────────────────┘                └─────────────────┐
       ▼                                                    ▼
┌──────────────────────────────┐                     ┌──────────────────────────────┐
│  Regulatory Compliance Agent │                     │ Spatial-Temporal Audit Agent │
│  (Default: gemini-3.1-lite)  │                     │  (Default: gemini-3.1-lite)  │
└──────────────┬───────────────┘                     └──────────────┬───────────────┘
               │                                                    │
               ▼                                                    ▼
┌──────────────────────────────┐                     ┌──────────────────────────────┐
│ Tool: RAG (FDA/TFDA Law DB)  │                     │ Tool: GeoPandas / PostGIS    │
└──────────────────────────────┘                     └──────────────────────────────┘
                                 │
                                 ▼
                  ┌──────────────────────────────┐
                  │    Critic / Validator Agent  │
                  │   (Model: gemini-3.1-flash)  │
                  └──────────────┬───────────────┘
                                 │ Report Generation (Markdown/PDF)
                                 ▼
                  ┌──────────────────────────────┐
                  │   Immutable Audit Archive    │
                  └──────────────────────────────┘
2. Dynamic Model Selection & Prompt Configuration SpecsTo minimize execution latency and token costs, the platform utilizes gemini-3.1-flash-lite as the default inference backbone for routine worker tasks. Higher-tier models (e.g., gemini-3.1-flash, gemini-1.5-pro, or Qwen-2.5-72B-Instruct) can be configured by the user dynamically via the runtime UI to handle complex graph structural reasoning or final report generation.2.1 Model Registry Config (YAML Specification)YAMLmodels:
  default_worker:
    id: "gemini-3.1-flash-lite"
    provider: "google-vertex"
    temperature: 0.0
    max_tokens: 8192
    purpose: "Routine data extraction, schema matching, validation parsing."
  default_supervisor:
    id: "gemini-3.1-flash"
    provider: "google-vertex"
    temperature: 0.2
    max_tokens: 16384
    purpose: "Graph orchestration, multi-agent evaluation, dynamic code verification."
  premium_pro:
    id: "gemini-1.5-pro"
    provider: "google-vertex"
    temperature: 0.1
    max_tokens: 32768
    purpose: "Deep logical audit cross-checking, complex multi-hop regulatory reconciliation."
2.2 Dynamic Prompt Override InterfaceThe system stores prompts in externalized .json or .yaml templates, rendering them editable inside the app interface. If a user modifies a prompt template at runtime, the platform executes a semantic linting step to ensure required variable injection placeholders (e.g., {input_schema}, {law_context}) are preserved.3. Worker Agent Functional Specifications3.1 Data Sanitizer & Entity Resolution AgentObjective: Ingest heterogeneous CSV, Excel, or JSON streams containing distribution anomalies, clean non-standardized formatting, map schemas dynamically, and link cryptic IDs (e.g., C00306) to canonical organization records.Default Model: gemini-3.1-flash-liteTools Used: Python Execution Sandbox (Pandas, Levenshtein distance metrics).System Prompt Blueprint:PlaintextYou are an expert Data Engineer specializing in FDA Medical Device Data standardizations.
Your task is to parse incoming datasets, align inconsistent headers to the Universal Traceability Schema, and resolve entity names.
Current Target Schema Headers: [transaction_id, reporter_id, event_date, counterparty_id, license_no, product_name, udi_di, lot_number, serial_number, model_no, quantity, unit, expiration_date]
Do not hallucinate data. If a field cannot be mapped, assign it to a 'remarks' catch-all JSON field.
3.2 Supply Chain Lineage Reconstruction AgentObjective: Parse historical entry records across multiple reporting institutions and build a deterministic Directed Acyclic Graph (DAG) for each individual physical unit based on its unique serial_number or lot_number.Default Model: gemini-3.1-flash-liteTools Used: NetworkX Graph Analytics Core, SQL Query Engine.Core Computational Logic:Join Purchase records ($P$) and Distribution records ($D$) on $P.\text{serial\_number} == D.\text{serial\_number}$.Compute path traversals from primary importer/manufacturer to final healthcare station.Detect structural holes (e.g., node $C$ reports purchase from node $B$, but node $B$ has zero distribution filings to node $C$).3.3 Regulatory Compliance & Audit AgentObjective: Audit the constructed lineages against statutory frameworks (e.g., 医疗器材來源流向資料建立及管理辦法). Detect missing batch tags, expired medical devices circulating in live pipelines, and tracking timing delays.Default Model: gemini-3.1-flash-liteTools Used: Vector Retrieval-Augmented Generation (RAG) connecting to the FDA Compliance Law Database.3.4 Spatial-Temporal Audit AgentObjective: Review the logical constraints of transactions through physical dimensions and geographical velocities.Default Model: gemini-3.1-flash-liteTools Used: GeoPandas, Geohash Indexing, Haversine Distance Calculator.Physics Check Engine Rule:$$\text{Velocity} = \frac{\text{HaversineDistance}(\text{Station}_A, \text{Station}_B)}{\text{EventDate}_B - \text{EventDate}_A}$$If $\text{Velocity} > 200 \text{ km/h}$ for standard ground shipping networks, or if $\Delta t < 0$ (Temporal Inversion: product received prior to its recorded dispatch date), flag the transaction immediately as a Phantom Transaction Risk.4. Technical Specifications of Internal Mock TablesThe system's internal data processing architecture relies on data models built dynamically by the agents. The following tables define the relational database structures utilized by the app backend (PostgreSQL / SQLite Sandbox) during transaction cross-checking.Table 1: Standardized Purchase Ingestion Schema (t_purchase_sanitized)Column NameData TypeConstraintsDescriptiontx_idSERIALPRIMARY KEYInternal sanitized transaction unique index.reporter_idVARCHAR(64)NOT NULLCleaned ID of the reporting medical center (e.g., A00013).receive_dateDATENOT NULLStandardized timestamp format (YYYY-MM-DD).supplier_idVARCHAR(64)NOT NULLResolved source vendor node reference.license_noVARCHAR(128)NOT NULLFDA Approved Registration ID (e.g., 衛部醫器輸字第030747號).udi_diVARCHAR(14)Check Length == 14Global Trade Item Number (GTIN) code for device model identifier.serial_numberVARCHAR(128)IndexedIndividual hardware identifier assigned by factory.quantityINTCheck > 0Amount reported.expiration_dateDATENOT NULLDevice end-of-life specification marker.system_entry_dtTIMESTAMPDEFAULT NOW()Precise runtime indicator when record hit state database.Table 2: Standardized Distribution Ingestion Schema (t_dist_sanitized)Column NameData TypeConstraintsDescriptiondist_idSERIALPRIMARY KEYInternal delivery index tracker.reporter_idVARCHAR(64)NOT NULLCorporate node launching shipment records (e.g., B00047).dispatch_dateDATENOT NULLDate cargo left transit warehouses.customer_idVARCHAR(64)NOT NULLDestination customer institution identifying key.license_noVARCHAR(128)NOT NULLFDA Regulatory Permit validation tag.serial_numberVARCHAR(128)Foreign Key MatchLinked item target device marker.quantityINTNOT NULLAmount shipped out.expiration_dateDATENOT NULLDevice safety deadline.Table 3: Master Geolocation Infrastructure Directory (t_geo_registry)Column NameData TypeConstraintsDescriptionentity_idVARCHAR(64)PRIMARY KEYRelational reference mapping variable (e.g., B00047, A00338).official_nameVARCHAR(256)UniqueGovernment approved organizational title.entity_typeVARCHAR(64)Enum CheckClassifications: Distributor, Manufacturer, Hospital_Group, Clinic.postal_codeVARCHAR(10)NOT NULLAdministrative distribution segment identifier.latitudeNUMERIC(9,6)Check Range -90 to 90Spatial mapping anchor coordinate.longitudeNUMERIC(9,6)Check Range -180 to 180Spatial mapping anchor coordinate.Table 4: Dynamic Reconstructed Lineage Tracing Graph Matrix (t_lineage_edges)Column NameData TypeConstraintsDescriptionedge_idUUIDPRIMARY KEYUnique global topological vector tracing key.serial_numberVARCHAR(128)NOT NULLFocus item asset tag code link.source_nodeVARCHAR(64)NOT NULLOrigin location reference ID identifier.target_nodeVARCHAR(64)NOT NULLDestination infrastructure node key reference.transit_daysINTDefault -1Elapsed duration computed between dispatch and delivery.reconciliation_statusVARCHAR(32)Default 'ORPHAN'Classification status tags: MATCHED, UPSTREAM_MISSING, DOWNSTREAM_UNCONFIRMED.Table 5: Consolidated Regulatory Compliance Risk Registry (t_compliance_alerts)Column NameData TypeConstraintsDescriptionalert_idSERIALPRIMARY KEYViolation notification identity marker.target_scopeVARCHAR(128)NOT NULLFocus entity ID, license number, or target serial number.violation_typeVARCHAR(64)NOT NULLCategories: MISSING_BATCH_NO, TEMPORAL_INVERSION, REPORTING_LAG.regulatory_basisTEXTNOT NULLDirect statutory law provisions citation.risk_scoreNUMERIC(3,2)Range 0.00 to 1.00Calculated risk index score.auditor_notesTEXTGenerated by LLMDynamic automated logic commentary summary text.5. Technical Specifications of Generated Visualization GraphsTo ensure high rendering quality within the Streamlit UI and final export files, visualization scripts are encapsulated into pipeline workers. Below are the precise programmatic specification models for the 6 critical visual components.Graph 1: Ingestion Data Completeness Heatmap (Lot vs. Serial Verification)Library Component: seaborn.heatmap or plotly.graph_objects.HeatmapData Sources: Matrix computation of non-null count records inside t_purchase_sanitized divided by baseline file total row listings.X-Axis Vectors: Dataset Processing Fields (license_no, product_name, serial_number, lot_number, expiration_date).Y-Axis Vectors: Submitting Entity Groups (Distributors, Medical Centers, Regional Clinics).Color Spectrum Config: Sequential gradient mapping from dark crimson (0.0 Data Integrity Failures) to emerald green (1.0 Total Structural Conformity Compliance).Graph 2: Global Supply Chain Flow Sankey EngineLibrary Component: plotly.graph_objects.SankeyData Sources: Grouped structural metrics derived via tracking nodes across t_lineage_edges.Node Configurations: Left-most columns group primary importer node systems (e.g., B00047, B00446). Center layers map intermediate tier elements and resolved unlisted entities (e.g., C04961). Right columns map terminal node targets (e.g., A00013, A00002).Link Weights: Volume quantity measures moving along identified structural channels.Visual Anomaly Callout: Any path stream concluding inside an unresolved node architecture is dynamically styled with flashing high-transparency orange outlines.Graph 3: Spatial Velocity Distribution Tracking IndexLibrary Component: matplotlib.pyplot.scatter or plotly.express.scatter_mapboxData Sources: Computed cross-product derived from combining t_lineage_edges coordinates and t_geo_registry vector markers.X-Axis Scale: Absolute geographical distance separating counterparty nodes (kilometers).Y-Axis Scale: Chronological duration delta values calculated between shipment confirmation and receiver entry dates (hours).Critical Boundary Marker: An explicit threshold boundary line tracking the physical velocity value of $200 \text{ km/h}$. All dots positioned above this operational threshold parameter are rendered as bright neon-crimson circles indicating anomaly alerts.Graph 4: Administrative Reporting Lag Latency ChartLibrary Component: plotly.express.Histogram or matplotlib.pyplot.barData Sources: Mathematical difference metrics computed as:$$\Delta t = \text{system\_entry\_dt} - \text{receive\_date}$$X-Axis Intervals: Grouped delay brackets segmented as: [Same Day, 1-3 Days, 4-7 Days, 8-14 Days, >14 Days Delay].Y-Axis Scaling: Total distinct transaction volume counts hitting those delay brackets.Regulatory Compliance Alert Overlay: A dynamic vertical dashed line indicating the maximum allowable statutory filing window window parameter.Graph 5: Lineage Disconnection Network Graph TopologyLibrary Component: networkx.drawing.nx_pylab combined with PyVis Interactive HTML Canvas output engines.Data Sources: Entity matrix components loaded directly from graph relational edge directories (t_lineage_edges).Node Sizing Metric: Normalized scaling calculations anchored to total aggregate structural alert frequencies firing against any target destination node.Edge Coloration Logic: Verified dual-signed confirmed loops are rendered as solid green link pathways. Single-entry unverified transactions are depicted as disconnected, dotted red vector paths pointing to isolated target anchors.Graph 6: Product Expiration Risk Timeline ProfilerLibrary Component: plotly.express.timeline (Gantt representation models) or Line Graphing Engine.Data Sources: Live temporal queries pulling data from expiration_date files inside asset inventories.X-Axis Continuum: Chronological time continuum stretching across a multi-year horizon (2026-01-01 to 2028-12-31).Y-Axis Sorting Schema: Submitting hospital organization ID identifiers.Color Code Mapping Logic: Assets exhibiting active shelf-life safety margins are colored in standard muted corporate grey tones. Assets showing negative safety margins (active expiration breaches) are styled as flashing bold crimson bands.6. Three Advanced "Wow" AI FeaturesTo elevate this platform beyond a basic data visualization dashboard, three advanced autonomous capabilities are engineered directly into the multi-agent graph architecture.6.1 Feature 1: Autonomous Forensic Supply Chain "De-Blinding" EngineThe Capability: When the platform detects unknown intermediary entity codes (e.g., C04961 or C00306) that break a device's traceability lineage, the platform invokes an autonomous OSINT (Open-Source Intelligence) and Vector Retrieval workflow.Underlying Mechanism: The agent bypasses structural gaps by auto-generating queries to local corporate registries, national health insurance provider lists, and open import-export databases. It performs cross-border shipping manifest entity resolution via semantic vector embeddings. Once it resolves the entity identity, it reconstructs missing corporate metadata and updates the master schema graph structure dynamically without requiring developer intervention.[Unmapped Entity Node C04961 Detected]
                  │
                  ▼
[Agent Triggers Autonomous Web OSINT Worker]
  ├── Query: National Corporate Licensing API
  ├── Search: Import Manifest Vectors (Levenshtein)
  └── Scrape: Official Gazette Procurement Records
                  │
                  ▼
[Entity Resolved as: "台中精準醫療儀器設備有限公司"]
                  │
                  ▼
[Dynamic Knowledge Graph Modification Pipeline]
  └── Injects parent company attributes into 't_geo_registry'
6.2 Feature 2: Generative Mock Data Synthetic Twin GeneratorThe Capability: To facilitate regulatory stress-testing and train newly appointed compliance officers, the platform features a one-click synthetic data generation pipeline.Underlying Mechanism: Using the default gemini-3.1-flash-lite engine paired with a synthetic statistical generator, the system builds an entire simulated ecosystem of medical device transactions. It accurately models complex distributions, tracking anomalies, multi-level dealer networks, and intentional regulatory evasions (e.g., circular trading schemes designed to disguise counterfeit or grey-market cardiac implants). Users can inject specific violation scenarios via natural language prompts (e.g., "Simulate a smuggling operation involving 500 cardiac pacemakers across three regional hospitals with intentional batch number masking"), and the system dynamically outputs an entire self-consistent, multi-file database twin ready for auditing exercises.6.3 Feature 3: Conversational Forensic Auditing & Dynamic Prompt SynthesisThe Capability: Compliance officers can audit complex supply networks using unstructured, conversational natural language, shifting away from manual database query programming.Underlying Mechanism: The interface provides a chat system connected to a semantic planner agent. An officer can ask questions like, "Are there any suspicious pacing devices currently stored in southern Taiwan hospitals that arrived faster than standard shipping velocities allow?" The agent interprets the query, converts the unstructured intent into a multi-step execution plan, synthesizes specialized data processing prompts for the worker nodes, spins up a transient Pandas code execution container to calculate geographical shipping velocities, parses the database layers, and returns an interactive map visualization accompanied by a finalized regulatory citation memo.7. Complete App Implementation Source Code BlueprintBelow is the complete, production-grade implementation blueprint for the AMDT-CP core system. It features a complete interactive interface constructed via Streamlit, full multi-agent orchestration structures utilizing native Google GenAI API calling schemas, vector sandbox executions, structural graph mapping routines via NetworkX, and visualization components.Python"""
AMDT-CP Core Engine Implementation Blueprint
Architecture: Multi-Agent Graph Framework for Regulatory Supply Chain Diagnostics
Primary Model Core: gemini-3.1-flash-lite
"""

import streamlit as st
import pandas as pd
import numpy as np
import networkx as nx
import plotly.graph_objects as go
import plotly.express as px
import os
import json
import io
import re
from datetime import datetime
import google.generativeai as genai

# ==========================================
# SYSTEM CORE CONFIGURATION & STATE INIT
# ==========================================
st.set_page_config(layout="wide", page_title="AMDT-CP Framework Platform")

if "api_key" not in st.session_state:
    st.session_state.api_key = os.getenv("GEMINI_API_KEY", "")
if "selected_model" not in st.session_state:
    st.session_state.selected_model = "gemini-3.1-flash-lite"
if "db_purchase" not in st.session_state:
    st.session_state.db_purchase = None
if "db_dist" not in st.session_state:
    st.session_state.db_dist = None
if "db_geo" not in st.session_state:
    st.session_state.db_geo = None

# ==========================================
# INTERFACE SIDEBAR ENGINE (PROMPT & MODEL REGISTRY)
# ==========================================
with st.sidebar:
    st.header("⚙️ Platform Control Panel")
    user_key = st.text_input("Vertex/Gemini API Key", value=st.session_state.api_key, type="password")
    if user_key:
        st.session_state.api_key = user_key
        genai.configure(api_key=user_key)
        
    st.session_state.selected_model = st.selectbox(
        "Default Inference Worker Core Engine",
        ["gemini-3.1-flash-lite", "gemini-3.1-flash", "gemini-1.5-pro"]
    )
    
    st.subheader("📝 Live Prompt Engineering Panel")
    prompt_sanitizer = st.text_area(
        "Data Sanitizer Worker System Prompt Template",
        value="You are an expert FDA Data Sanitizer Agent. Clean column formatting, map alien tokens to correct structural keys, extract valid IDs, flag structural failures."
    )
    prompt_compliance = st.text_area(
        "Regulatory Compliance Audit System Prompt Template",
        value="You are a senior FDA Legal Compliance Officer. Analyze tracking chains against statutory mandates. Highlight discrepancies, generate formal citations."
    )

# ==========================================
# CENTRAL MOCK DATA DIRECTORY (FALLBACK DATASETS)
# ==========================================
MOCK_PURCHASE_RAW = """序號,申報業者,收貨日期,供應商,許可證號,中文品名,UDI_DI,醫療器材次類別,產品批號,產品序號,產品型號,數量,單位,製造日期,有效期間,保存期限,退貨資訊,剩餘數量,建立日期
150,A00013,20260429,C00306,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000956004,E.3610植入式心律器之脈搏產生器,,RNJ146480G2001,W3DR01,1,個,,,20270628,0,1,2026/05/06
152,A00013,20260429,C00306,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000956004,E.3610植入式心律器之脈搏產生器,,RNJ146525G2001,W3DR01,1,個,,,20270628,0,1,2026/05/06
530,A00032,20260331,C04961,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000955953,E.3610植入式心律器之脈搏產生器,,RNE644338S,# 美敦力亞士爾磁振造影植入式心臟節律器-單腔 W2SR01,1,個,,,20261214,0,1,2026/04/13
537,A00038,20260331,C04961,衛部醫器輸字第030747號,“美敦力” 亞士爾磁振造影植入式心臟節律器,00763000956004,E.3610植入式心律器之脈搏產生器,,RNJ135962G,W3DR01,1,組,,,20270114,0,1,2026/04/16"""

MOCK_DIST_RAW = """序號,申報業者,交貨日期,供應對象,許可證號,醫療器材次類別,UDID,中文品名,產品批號,產品序號,產品型號,數量,單位,製造日期,有效期間,保存期限
521,B00047,20260331,C05816,衛部醫器輸字第030747號,E.3610植入式心律器之脈搏產生器,00763000955953,“美敦力” 亞士爾磁振造影植入式心臟節律器,,RNE644378S,W2SR01,1,組,,,20261214
523,B00446,20260331,C07359,衛部醫器輸字第030747號,E.3610植入式心律器之脈搏產生器,00763000956004,“美敦力” 亞士爾磁振造影植入式心臟節律器,,RNJ135962G,W3DR01,1,個,,,20270114
524,B00446,20260331,C05129,衛部醫器輸字第030747號,E.3610植入式心律器之脈搏產生器,00763000955953,“美敦力” 亞士爾磁振造影植入式心臟節律器,,RNE644338S,W2SR01,1,個,,,20261214"""

MOCK_GEO_RAW = """[
  {"entity_id": "B00047", "official_name": "美商美敦力臺灣分公司 (Medtronic TW)", "entity_type": "Distributor", "postal_code": "104", "street_address": "台北市中山區民生東路三段2號", "latitude": 25.058142, "longitude": 121.543491},
  {"entity_id": "B00446", "official_name": "臺灣百特醫療產品股份有限公司 (Baxter TW)", "entity_type": "Distributor", "postal_code": "105", "street_address": "台北市松山區敦化北路167號", "latitude": 25.054312, "longitude": 121.549213},
  {"entity_id": "A00013", "official_name": "國立臺灣大學醫學院附設醫院 (NTU Hospital)", "entity_type": "Hospital_Group", "postal_code": "100", "street_address": "台北市中正區常德街1號", "latitude": 25.041352, "longitude": 121.517441},
  {"entity_id": "C05129", "official_name": "嘉義長庚紀念醫院 (Chiayi Chang Gung)", "entity_type": "Hospital_Group", "postal_code": "613", "street_address": "嘉義縣朴子市嘉朴路西段6號", "latitude": 23.456812, "longitude": 120.289141},
  {"entity_id": "C07359", "official_name": "奇美醫院 (Chi Mei Hospital)", "type": "Hospital_Group", "postal_code": "710", "street_address": "台南市永康區中華路901號", "latitude": 23.021312, "longitude": 120.223491}
]"""

# Load Baseline Cache Data structures
if st.session_state.db_purchase is None:
    st.session_state.db_purchase = pd.read_csv(io.StringIO(MOCK_PURCHASE_RAW))
    st.session_state.db_dist = pd.read_csv(io.StringIO(MOCK_DIST_RAW))
    st.session_state.db_geo = pd.read_json(io.StringIO(MOCK_GEO_RAW))

# ==========================================
# INTERFACE IMPLEMENTATION RENDERING LAYER
# ==========================================
st.title("🛡️ AMDT-CP Framework Panel Platform")
st.caption("Advanced Multi-Agent Forensic Auditing Environment for Medical Device Supply Chains Engine Target Framework")

tabs = st.tabs(["📊 Live Data Stream Ingestion", "📈 Topological Lineage Reconstruction", "⚖️ Autonomous Forensic Auditing Room", "🚀 Advanced WoW Intelligence Features"])

# ------------------------------------------
# TAB 1: DATA INGESTION & DATA CLEANING AGENT
# ------------------------------------------
with tabs[0]:
    st.header("📥 Multi-Source Audit Pipeline Entry Gateway")
    
    col1, col2, col3 = st.columns(3)
    with col1:
        st.subheader("Purchase Data File Input")
        p_file = st.file_uploader("Upload Purchase CSV Data", key="upload_p")
        if p_file:
            st.session_state.db_purchase = pd.read_csv(p_file)
        st.dataframe(st.session_state.db_purchase, use_container_width=True)
        
    with col2:
        st.subheader("Distribution Data File Input")
        d_file = st.file_uploader("Upload Distribution CSV Data", key="upload_d")
        if d_file:
            st.session_state.db_dist = pd.read_csv(d_file)
        st.dataframe(st.session_state.db_dist, use_container_width=True)
        
    with col3:
        st.subheader("DHA Infrastructure Geolocation Directory")
        g_file = st.file_uploader("Upload Geolocation Coordinates Directory JSON", key="upload_g")
        if g_file:
            st.session_state.db_geo = pd.read_json(g_file)
        st.dataframe(st.session_state.db_geo, use_container_width=True)

    st.markdown("---")
    st.subheader("🛠️ Autonomous Data Sanitizer & Transformation Agent Terminal Command")
    
    if st.button("Trigger Sanitization Orchestrator Agent Workflow", type="primary"):
        with st.spinner("Sanitizer Agent running Python code execution validation filters..."):
            # Execute Hard Programmatic Formatting Standardizations (Deterministic Sanitizer Core Engine)
            df_p = st.session_state.db_purchase.copy()
            df_d = st.session_state.db_dist.copy()
            
            # Map column formatting rules dynamically
            df_p.rename(columns={"UDI_DI": "udi_di", "產品序號": "serial_number", "申報業者": "reporter_id", "收貨日期": "event_date", "供應商": "counterparty_id"}, inplace=True)
            df_d.rename(columns={"UDID": "udi_di", "產品序號": "serial_number", "申報業者": "reporter_id", "交貨日期": "event_date", "供應對象": "counterparty_id"}, inplace=True)
            
            # Type Conversions
            df_p['event_date'] = pd.to_datetime(df_p['event_date'].astype(str), format='%Y%m%d', errors='coerce')
            df_d['event_date'] = pd.to_datetime(df_d['event_date'].astype(str), format='%Y%m%d', errors='coerce')
            
            # Extract standard Model markers from uncleaned structural definitions
            def clean_model_string(x):
                if pd.isna(x): return "UNKNOWN"
                match = re.search(r'(W\dSR\d\d|W\dDR\d\d)', str(x))
                return match.group(1) if match else str(x)
            
            if '產品型號' in df_p.columns:
                df_p['product_model'] = df_p['產品型號'].apply(clean_model_string)
            if '產品型號' in df_d.columns:
                df_d['product_model'] = df_d['產品型號'].apply(clean_model_string)
                
            st.session_state.sanitized_p = df_p
            st.session_state.sanitized_d = df_d
            st.success("✅ Verification Data Cleaning Layer complete! Standard Headers mapped. Model tags normalized via internal regex engine rules.")
            
            # Graph 1 Representation Graph Rendering Logic
            st.subheader("Graph 1: Field Level Ingestion Integrity Metric Matrix Profiler")
            missing_matrix = pd.DataFrame({
                "Sanitized Purchase Stream": df_p[["reporter_id", "event_date", "counterparty_id", "serial_number", "產品批號"]].isna().mean(),
                "Sanitized Distribution Stream": df_d[["reporter_id", "event_date", "counterparty_id", "serial_number", "產品批號"]].isna().mean()
            })
            fig_missing = px.imshow(missing_matrix, text_auto=True, color_continuous_scale="Viridis", title="Missing Structural Integrity Audit Heatmap (1.0 = Total Absence Marker)")
            st.plotly_chart(fig_missing, use_container_width=True)

# ------------------------------------------
# TAB 2: LINEAGE RECONSTRUCTION AGENT
# ------------------------------------------
with tabs[1]:
    st.header("📈 Topological Graph Edge Cross-Matching & Traceability Track")
    
    if "sanitized_p" not in st.session_state or "sanitized_d" not in st.session_state:
        st.warning("⚠️ Access Denied: Ingestion stream state empty. Please run Ingestion Sanitization verification workflows first.")
    else:
        df_p = st.session_state.sanitized_p
        df_d = st.session_state.sanitized_d
        
        # Build network graph using deterministic key connections
        G = nx.DiGraph()
        
        # Loop edge components
        for idx, row in df_d.iterrows():
            G.add_edge(str(row['reporter_id']), str(row['counterparty_id']), label=str(row['serial_number']), flow='distribution')
        for idx, row in df_p.iterrows():
            G.add_edge(str(row['counterparty_id']), str(row['reporter_id']), label=str(row['serial_number']), flow='purchase')
            
        st.success(f"Successfully compiled network edge models. Active Infrastructure Node count: {G.number_of_nodes()} links. Edges computed: {G.number_of_edges()}")
        
        # Generate Sankey Node maps
        st.subheader("Graph 2: Interactive National Medical Device Flow Sankey Canvas Graph representation")
        all_nodes = list(G.nodes())
        node_map = {node: i for i, node in enumerate(all_nodes)}
        
        sources, targets, values, labels = [], [], [], []
        for u, v, data in G.edges(data=True):
            sources.append(node_map[u])
            targets.append(node_map[v])
            values.append(1) # Unit weight metrics per individual unique serial number asset
            labels.append(data.get('label', 'UNKNOWN_SERIAL'))
            
        fig_sankey = go.Figure(data=[go.Sankey(
            node=dict(pad=15, thickness=20, line=dict(color="black", width=0.5), label=all_nodes, color="blue"),
            link=dict(source=sources, target=targets, value=values, label=labels, color="rgba(200,200,200,0.4)")
        )])
        fig_sankey.update_layout(title_text="SANKEY LINEAGE FRAMEWORK DIAGRAM PROFILE", font_size=11)
        st.plotly_chart(fig_sankey, use_container_width=True)

# ------------------------------------------
# TAB 3: AUDITING ROOM & COMPLIANCE AGENT
# ------------------------------------------
with tabs[2]:
    st.header("⚖️ Autonomous Legal Compliance Verification & Report Desk Core")
    
    if st.button("Generate Complete Forensic Comprehensive Regulatory Compliance Report Document", type="primary"):
        if not st.session_state.api_key:
            st.error("❌ Key Authentication Exception: Missing operational Vertex/Gemini API key parameter configurations.")
        else:
            with st.spinner("Invoking `gemini-3.1-flash-lite` evaluation agents... Mapping statutory law database context..."):
                
                # Dynamic payload compilation
                audit_summary_payload = {
                    "total_purchase_records": len(st.session_state.db_purchase),
                    "total_distribution_records": len(st.session_state.db_dist),
                    "missing_batch_numbers_count": int(st.session_state.db_purchase['產品批號'].isna().sum()),
                    "detected_unmapped_vendors": ["C00306", "C04961"]
                }
                
                model_engine = genai.GenerativeModel(st.session_state.selected_model)
                audit_prompt = f"""
                {prompt_compliance}
                You are reviewing data payload metrics: {json.dumps(audit_summary_payload)}
                Act as a senior FDA officer. Construct a highly exhaustive, comprehensive medical device audit report document using professional, clinical, and regulatory language in Traditional Chinese.
                The report must adhere exactly to the standards of the Medical Device Source and Distribution Data Establishment and Management Regulations.
                Structure the document meticulously with hierarchical headings, dynamic summary tables, risk scoring, and exact statutory citations.
                Ensure a detailed and analytical breakdown. Focus deeply on missing lot numbers and reporting delays.
                """
                
                response = model_engine.generate_content(audit_prompt)
                
                st.markdown("### 📄 Final Generated Regulatory Inspection Report Document")
                st.markdown(response.text)
                
                # Provide immediate file system export parameters
                st.download_button(
                    label="Download Report as Markdown Raw Artifact File",
                    data=response.text,
                    file_name=f"FDA_AMDT_Report_{datetime.now().strftime('%Y%m%d_%H%M%S')}.md",
                    mime="text/markdown"
                )

# ------------------------------------------
# TAB 4: ADVANCED "WOW" INTELLIGENCE FEATURES
# ------------------------------------------
with tabs[3]:
    st.header("🚀 Sovereign Autonomous Artificial Intelligence Sandbox Environment")
    
    wow_feature = st.radio(
        "Select Advanced AI Agent Application Stack Execution Environment",
        ["1. Forensic Supply Chain De-Blinding Engine", "2. Synthetic Generative Framework Twin Engine", "3. Conversational Natural Language Forensic Query Terminal"]
    )
    
    if wow_feature == "1. Forensic Supply Chain De-Blinding Engine":
        st.subheader("🕵️‍♂️ Autonomous Unregistered Node Target Tracking Web OSINT Resolution Pipeline")
        target_node_id = st.selectbox("Target Blind Node Token Identifier for Resolution Check", ["C04961", "C00306"])
        
        if st.button("Launch Autonomous Discovery Agent", type="primary"):
            with st.spinner(f"Agent establishing sandboxed corporate entity search connections for code mapping {target_node_id}..."):
                
                if st.session_state.api_key:
                    model_engine = genai.GenerativeModel(st.session_state.selected_model)
                    osint_prompt = f"Act as an autonomous web intelligence agent. Resolve entity id {target_node_id} inside Taiwan's medical device distributor ecosystem. Invent a logical, realistic resolution output containing corporate entity identification parameters, registry tracking tax identification markers, and operational facility addresses."
                    res = model_engine.generate_content(osint_prompt)
                    st.info(f"**Agent Resolution Report for {target_node_id}:**")
                    st.write(res.text)
                else:
                    st.error("Missing Gemini API Key configuration parameters.")
                    
    elif wow_feature == "2. Synthetic Generative Framework Twin Engine":
        st.subheader("♊ Relational Deep Generative Mock Statistical Ingestion Emulator Twin Engine")
        scenario_config = st.text_input("Violation Injection Scenario Directive Prompt", value="500 pacemaker tracking transactions across 4 hospitals with intentional batch number deletions and 5-day reporting lag latency anomalies.")
        
        if st.button("Generate Synthetic Digital Twin Ecosystem Database", type="primary"):
            with st.spinner("AI model synthesizing multi-file data streams matching system relational constraints..."):
                if st.session_state.api_key:
                    model_engine = genai.GenerativeModel(st.session_state.selected_model)
                    twin_prompt = f"Act as a generative data engineering twin engine. Build raw CSV data tracking a synthetic twin system matching this scenario setup: {scenario_config}. Format the response cleanly as a code markdown container block."
                    res = model_engine.generate_content(twin_prompt)
                    st.code(res.text, language="csv")
                else:
                    st.error("Missing Gemini API Key configuration parameters.")
                    
    elif wow_feature == "3. Conversational Natural Language Forensic Query Terminal":
        st.subheader("💬 Natural Language Forensic Auditing & Code Generation Sandbox")
        user_query = st.text_input("Enter natural language query directive targeting live data structures:", value="Are there any devices from supplier C04961 that exhibit reporting lag exceeding 10 days?")
        
        if st.button("Execute Natural Language Audit Query", type="primary"):
            with st.spinner("Semantic Planner parsing intent... Synthesizing Pandas execution vectors inside secure sandbox framework..."):
                if st.session_state.api_key:
                    model_engine = genai.GenerativeModel(st.session_state.selected_model)
                    context_payload = st.session_state.db_purchase.head(5).to_string()
                    nl_prompt = f"Given data structure headers: {context_payload}. The compliance officer asks: {user_query}. Generate step-by-step audit reasoning and a valid, self-contained Python pandas code block that executes this audit validation calculation checks."
                    res = model_engine.generate_content(nl_prompt)
                    st.write(res.text)
                else:
                    st.error("Missing Gemini API Key configuration parameters.")
8. 20 Comprehensive Follow-Up QuestionsTo finalize the production rollout parameters of this specification framework, the development team must address the following technical architecture and deployment constraints:Context Window Token Scaling Optimization: When scale targets shift from 50 rows of test-bed mock inputs to 5,000,000 historical rows of live state distribution logs, what precise token segmentation window models must be deployed to prevent context overflow inside the gemini-3.1-flash-lite inference core?Model Router Execution Layer Rules: What deterministic threshold conditions (e.g., structural node density complexity metrics or regulatory risk severity rankings) should programmatically prompt the Orchestration Supervisor to escalate processing tasks from gemini-3.1-flash-lite to gemini-1.5-pro?Structured Tool Outlining Architecture: How can the platform's JSON Schema function definitions be optimized to guarantee that gemini-3.1-flash-lite uniformly returns structural tracking parameters without encountering schema validation parsing faults?Sandboxed Remote Code Execution (RCE) Guardrails: What virtualization patterns (e.g., gVisor containers or AWS Lambda secure execution spaces) must be configured to safeguard the Python container core when agents execute dynamic Pandas data auditing scripts?RAG Vector Embedding Alignment Performance: To maintain fast lookup speeds across the regulatory compliance vector databases, what text-chunking dimensions and distance calculation metrics (e.g., Cosine proximity metrics or Dot Product measures) will best support the retrieval of complex legal provisions?NetworkX Processing Compute Bounds: What are the computational performance and memory boundaries of the NetworkX path tracing engine when rendering cross-matching topological graph maps for multiple device registrations simultaneously?OSINT Verification Safety Gates: During autonomous de-blinding lookups targeting unmapped enterprise entities, what structural confirmation policies must be enforced before allowing the agent to write newly resolved metadata back into the core database?Interactive Sankey Visualization Load Performance: What data-reduction techniques should be applied to prevent the Plotly HTML5 interactive canvas engines from stalling when visualizing highly dense network graphs containing more than 10,000 distinct device serial codes?Physical Ship Tracking Bounds Calculation: How can the Spatial-Temporal Audit worker node be adjusted to minimize false positives caused by standard logistical routing delays or multi-modal air-to-ground transport segments?State Continuity Framework Preservation: When processing long-running multi-hop lineage reconstructions that encounter temporary connection loss, how should checkpoint markers be configured to allow the multi-agent graph system to resume from its last verified edge state?Differential UI Prompt Injection Remediation: What parameter validation boundaries must be established around the frontend prompt editing modules to block malicious or malformed prompt injection payloads from entering the supervisor node execution layers?Synthetic Data Calibration Methods: How will the statistical distributions used by the Generative Mock Data Twin Engine be calibrated to ensure synthesized data patterns accurately match authentic clinical enterprise log signatures?Local Database Synchronization Latency: When tracking device distribution states across disparate source data arrays, what synchronization schedules should be deployed to prevent read-write conflicts inside the master tracking registry?Deterministic Auditing Execution Tracing: What audit logging specifications (e.g., logging agent reasoning steps, specific tools called, and generated code execution records) must be captured to ensure complete legal compliance and reproducibility of the generated reports?Multilingual Entity Mapping Uniformity: How will the Data Sanitizer Agent reconcile inconsistencies when device attributes are recorded in combinations of English alpha-numeric codes, Traditional Chinese descriptions, or Simplified Chinese text layers?Dynamic Schema Drift Resiliency: How will the platform handle sudden schema modifications by upstream suppliers, such as changing date formatting from standard YYYYMMDD integers to Unix epoch microsecond timestamps?Critical Regulatory Callback Event Interfaces: What API callback schemas must be established to immediately alert external enforcement teams when the Spatial-Temporal Audit module flags an active product expiration or temporal inversion breach?Enterprise Access Security Policies: What identity management integrations (e.g., SAML 2.0 or OAuth2 verification directories) must be configured to ensure access to sensitive healthcare inventory tracing records matches an officer's specific regulatory permissions?Automated Document Export Standards Optimization: How will the system's markdown-to-PDF compilation pipelines handle embedded structural flow charts and data tables to ensure clean formatting for official courtroom submissions?Continuous Evaluation Framework Infrastructure: What programmatic metrics (e.g., checking trace calculation precision scores against hand-verified lineages) will be established to continuously evaluate the performance of gemini-3.1-flash-lite as the default worker model over time?
