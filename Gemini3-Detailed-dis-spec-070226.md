# Technical Architecture & System Specification: Agentic Medical Device Traceability Platform (AMDT-CP)
### System Version: 2.1.0-PROD
### Primary Author: Principal Systems Architect, Google AI Studio Build Team
### Target Inference Framework: Google GenAI SDK (Node.js/Express)
### Recommended Client Framework: React 19 / Tailwind CSS 4 / Framer Motion 12
### Output Document Scope: 3,500 - 4,000 Words

---

## 1. System Overview & Platform Mission

The **Agentic Medical Device Traceability & Compliance Platform (AMDT-CP)** is an enterprise-grade, multi-agent AI software system designed to assist regulatory agencies (such as the Taiwan Food and Drug Administration and the US FDA) in executing automated forensic auditing of complex medical device supply chain networks. By ingesting massive, heterogeneous, and non-standardized logs of physical device movements, the platform reconstructs serial-level hardware lineages, exposes structural tracking fractures, calculates compliance risks, and auto-generates legally defensible regulatory citation documents.

Medical device supply chains are notoriously opaque, relying on multi-tiered networks of prime distributors, sub-dealers, regional brokers, and hospital procurement units. This lack of transparency is particularly dangerous for Class III life-critical implants, such as implantable pacemakers, defibrillators, and artificial heart valves. When an manufacturer issues a product recall due to a life-threatening defect, locating every affected serial number inside active clinical inventories is a slow, error-prone manual process.

AMDT-CP solves this by treating every transaction log as a directed node edge in a global supply chain graph. The system uses a multi-agent orchestration pattern to automatically ingest, sanitize, and reconcile logs. It identifies regulatory violations (such as parallel-import grey markets, counterfeit serial tags, and reporting delays) and displays them in a high-fidelity interactive visual dashboard.

```
+─────────────────────────────────────────────────────────────────────────────+
|                                STREAMLIT UI                                 |
|      (Data Ingestion - Lineage Tracing - Compliance Desk - AI Sandbox)      |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ HTTP / WebSockets
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                         EXPRESS BACKEND CONTROLLER                          |
|         (Port 3000 Ingress, Session Management, CORS, Static Serving)       |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ Async Task Queue
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                       MULTI-AGENT ORCHESTRATION GRAPH                       |
|                             (LangGraph Engine)                              |
|                                                                             |
|         +─────────────────────────────────────────────────────────+         |
|         |                  ORCHESTRATION SUPERVISOR               |         |
|         |                 (Model: gemini-3.5-flash)               |         |
|         +──────┬─────────────────────┬─────────────────────┬──────+         |
|                │                     │                     │                |
|                ▼                     ▼                     ▼                |
|         +──────────────+      +──────────────+      +──────────────+        |
|         |  SANITIZER   |      |   LINEAGE    |      |  COMPLIANCE  |        |
|         |    AGENT     |      |  BUILDER     |      |  AUDITOR     |        |
|         |  (gemini-    |      |  (gemini-    |      |  (gemini-    |        |
|         |  3.1-flash-  |      |  3.1-flash-  |      |  3.1-flash-  |        |
|         |    lite)     |      |    lite)     |      |    lite)     |        |
|         +──────┬───────+      +──────┬───────+      +──────┬───────+        |
|                │                     │                     │                |
|                ▼                     ▼                     ▼                |
|         +──────────────+      +──────────────+      +──────────────+        |
|         |   Tool:      |      |   Tool:      |      |   Tool:      |        |
|         |  Pandas/     |      |  NetworkX    |      |  Legal RAG   |        |
|         |  Regex       |      |  Graph       |      |  Vector DB   |        |
|         +──────────────+      +──────────────+      +──────────────+        |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ Verification Feedback Loop
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                           CRITIC / VALIDATOR AGENT                          |
|                          (Model: gemini-3.5-flash)                          |
+──────────────────────────────────────┬──────────────────────────────────────+
                                       │ Stream JSON Payload
                                       ▼
+─────────────────────────────────────────────────────────────────────────────+
|                          PERSISTENT SQL SANDBOX                             |
|         (t_purchase_sanitized, t_dist_sanitized, t_geo_registry, etc)       |
+─────────────────────────────────────────────────────────────────────────────+
```

---

## 2. Multi-Agent Orchestration Architecture

To escape the constraints of linear, single-prompt instruction models—which frequently suffer from context drift, reasoning attenuation, and tool selection failure when executing multi-step tasks—AMDT-CP utilizes a decentralized, stateful Multi-Agent Orchestration Graph managed via LangGraph. 

Each agent is treated as an independent actor with its own specialized system instructions, scratchpad memory, and tool-access permissions. The routing between these agents is non-linear and is governed by an **Orchestration Supervisor Node** which evaluates the state of the shared context at the end of each execution cycle and routes to the next logical worker node.

### 2.1 The Orchestration Graph Nodes

1. **The Orchestration Supervisor Node (Model: `gemini-3.5-flash`)**:
   - **Responsibility**: Interprets the user's high-level directives, builds a multi-step execution plan, allocates sub-tasks to the appropriate worker nodes, and performs continuous state checks.
   - **Loop Control**: If a worker node fails to complete its task or returns data that violates the global schema, the Supervisor triggers an immediate correction retry (up to a maximum of three times before escalating to the developer).
2. **The Data Sanitizer & Entity Resolution Agent (Model: `gemini-3.1-flash-lite`)**:
   - **Responsibility**: Parses unstructured file uploads, resolves column naming mismatches, validates field data types (such as dates, integers, and model strings), and resolves cryptic, non-standard business identifiers to their canonical corporate register names.
   - **Tools**: Pandas Data Engineering Sandbox, Levenshtein Distance Comparator.
3. **The Supply Chain Lineage Reconstruction Agent (Model: `gemini-3.1-flash-lite`)**:
   - **Responsibility**: Takes the sanitized transaction logs and constructs individual, serial-level trace lineages. It traces each device from prime importer warehouses, through sub-dealers, and down to the final healthcare clinical destination.
   - **Tools**: NetworkX Graph Processing Engine, SQL Relational Query Builder.
4. **The Regulatory Compliance & Audit Agent (Model: `gemini-3.1-flash-lite`)**:
   - **Responsibility**: Compares the reconstructed tracing lineages against the compliance rules of local regulations (e.g., Article 6 of the TFDA Medical Device Act). It identifies broken chains, timing delay warnings, and un-notified parallel imports.
   - **Tools**: Vector Database Retrieval-Augmented Generation (RAG) connecting to the national medical device statutory laws.
5. **The Spatial-Temporal Audit Agent (Model: `gemini-3.1-flash-lite`)**:
   - **Responsibility**: Audits the logical sanity of transactions in the physical dimensions of geography and time. It calculates transport velocities between counterparties using coordinate grids and highlights chronological impossibilities (e.g., receiving a shipment before it is sent).
   - **Tools**: GeoPandas, Haversine Distance Formula.
6. **The Critic/Validator Agent (Model: `gemini-3.5-flash`)**:
   - **Responsibility**: Inspects all outputs from the worker nodes to ensure zero hallucinations and absolute schema conformity. If the output passes verification, it compiles the final JSON payload and pushes it to the UI and database.

---

## 3. Dynamic Model Selection & Prompt Configurations

The platform is designed to minimize token costs and execution latencies while maintaining peak reasoning capabilities. To achieve this, it implements a **Dynamic Model Routing Registry** that matches the appropriate Gemini model to each task's computational complexity.

### 3.1 Model Registry Specification (YAML Configuration)

The system registers models using a structured configuration schema:

```yaml
models:
  worker_tier_lite:
    id: "gemini-3.1-flash-lite"
    provider: "google-vertex"
    temperature: 0.0
    max_output_tokens: 8192
    context_window: 1048576
    cost_per_million_input: 0.075
    cost_per_million_output: 0.30
    purpose: "Routine text parsing, schema-mapping, statistical aggregations, JSON formatting."
  supervisor_tier_mid:
    id: "gemini-3.5-flash"
    provider: "google-vertex"
    temperature: 0.2
    max_output_tokens: 16384
    context_window: 2097152
    purpose: "Multi-agent task orchestration, loop control routing, error resolution, draft report formatting."
  reasoner_tier_pro:
    id: "gemini-3.1-pro-preview"
    provider: "google-vertex"
    temperature: 0.1
    max_output_tokens: 32768
    context_window: 2097152
    purpose: "Deep multi-hop trace reasoning, statutory compliance cross-checking, formal legal report synthesis."
```

### 3.2 Dynamic Prompt Override Interface

To allow compliance officers and developers to adapt the system's reasoning at runtime, all system and task prompts are externalized as editable JSON templates. When an officer modifies a prompt in the frontend panel, the Supervisor runs a compilation linting check. This check ensures that all mandatory variable injection tokens (e.g., `{input_data}`, `{schema_definition}`, `{statutory_context}`) are preserved and syntactically valid before the new prompt is deployed to the active worker nodes.

---

## 4. Database Schema Specification (Relational Mock Tables)

The AMDT-CP system utilizes five highly structured database tables to manage the lifecycle of medical device transactions and geolocation directories. The tables are described below using strict PostgreSQL SQL Definitions.

```
       +────────────────────────+             +────────────────────────+
       |   t_geo_registry       |             |   t_purchase_sanitized |
       +────────────────────────+             +────────────────────────+
       | PK | entity_id         |             | PK | tx_id             |
       |    | official_name     |             | FK | reporter_id       |
       |    | entity_type       |             |    | receive_date      |
       |    | postal_code       |             | FK | supplier_id       |
       |    | street_address    |             |    | license_no        |
       |    | latitude          |             |    | udi_di            |
       |    | longitude         |             |    | serial_number     |
       +─────────────────┬──────+             +───────────┬────────────+
                         │                                │
                         │ 1                              │ 1
                         │                                │
                         │ N                              │ N
       +─────────────────▼──────+             +───────────▼────────────+
       |   t_lineage_edges      |             |   t_compliance_alerts  |
       +────────────────────────+             +────────────────────────+
       | PK | edge_id           |             | PK | alert_id          |
       |    | serial_number     |             |    | target_scope      |
       |    | source_node       |             |    | violation_type    |
       |    | target_node       |             |    | regulatory_basis  |
       |    | transit_days      |             |    | risk_score        |
       |    | reconciliation_st |             |    | auditor_notes     |
       +────────────────────────+             +────────────────────────+
```

### 4.1 Master Geolocation Infrastructure Directory (`t_geo_registry`)
This table acts as the spatial source of truth for all network participants, mapping entity IDs to their official physical coordinates and facilities.

```sql
CREATE TABLE t_geo_registry (
    entity_id VARCHAR(64) PRIMARY KEY,
    official_name VARCHAR(256) NOT NULL UNIQUE,
    entity_type VARCHAR(64) NOT NULL CHECK (entity_type IN ('Distributor', 'Hospital_Group', 'Clinic', 'Manufacturer', 'Sub_Dealer')),
    postal_code VARCHAR(10) NOT NULL,
    street_address TEXT NOT NULL,
    latitude NUMERIC(9,6) NOT NULL CHECK (latitude BETWEEN -90.000000 AND 90.000000),
    longitude NUMERIC(9,6) NOT NULL CHECK (longitude BETWEEN -180.000000 AND 180.000000),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

### 4.2 Standardized Purchase Ingestion Schema (`t_purchase_sanitized`)
Records incoming receipt files submitted by hospitals and clinics after compliance sanitization.

```sql
CREATE TABLE t_purchase_sanitized (
    tx_id SERIAL PRIMARY KEY,
    reporter_id VARCHAR(64) NOT NULL REFERENCES t_geo_registry(entity_id),
    receive_date DATE NOT NULL,
    supplier_id VARCHAR(64) NOT NULL,
    license_no VARCHAR(128) NOT NULL,
    product_name VARCHAR(256) NOT NULL,
    udi_di VARCHAR(14) NOT NULL CHECK (LENGTH(udi_di) = 14),
    category TEXT NOT NULL,
    lot_number VARCHAR(128),
    serial_number VARCHAR(128) NOT NULL,
    model VARCHAR(128) NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit VARCHAR(32) NOT NULL,
    expiry_date DATE,
    returns INT NOT NULL DEFAULT 0,
    remaining INT NOT NULL DEFAULT 1,
    system_entry_dt TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_purchase_serial ON t_purchase_sanitized(serial_number);
```

### 4.3 Standardized Distribution Ingestion Schema (`t_dist_sanitized`)
Records outgoing shipment logs submitted by licensed medical device distributors.

```sql
CREATE TABLE t_dist_sanitized (
    dist_id SERIAL PRIMARY KEY,
    reporter_id VARCHAR(64) NOT NULL REFERENCES t_geo_registry(entity_id),
    dispatch_date DATE NOT NULL,
    customer_id VARCHAR(64) NOT NULL,
    license_no VARCHAR(128) NOT NULL,
    category TEXT NOT NULL,
    udi_di VARCHAR(14) NOT NULL CHECK (LENGTH(udi_di) = 14),
    product_name VARCHAR(256) NOT NULL,
    lot_number VARCHAR(128),
    serial_number VARCHAR(128) NOT NULL,
    model VARCHAR(128) NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit VARCHAR(32) NOT NULL,
    expiry_date DATE NOT NULL,
    system_entry_dt TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_dist_serial ON t_dist_sanitized(serial_number);
```

### 4.4 Dynamic Reconstructed Lineage Tracing Graph Matrix (`t_lineage_edges`)
Stores the reconciled directed edges calculated by the Supply Chain Lineage Reconstruction Agent.

```sql
CREATE TABLE t_lineage_edges (
    edge_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    serial_number VARCHAR(128) NOT NULL,
    source_node VARCHAR(64) NOT NULL REFERENCES t_geo_registry(entity_id),
    target_node VARCHAR(64) NOT NULL REFERENCES t_geo_registry(entity_id),
    transit_days INT DEFAULT -1,
    reconciliation_status VARCHAR(64) NOT NULL CHECK (reconciliation_status IN ('MATCHED', 'UPSTREAM_MISSING', 'DOUBLE_DELIVERY_COLLISION', 'TEMPORAL_INVERSION')),
    calculated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_lineage_serial ON t_lineage_edges(serial_number);
```

### 4.5 Consolidated Regulatory Compliance Risk Registry (`t_compliance_alerts`)
Aggregates violations found during auditing, providing the direct legal basis and calculated risk index score.

```sql
CREATE TABLE t_compliance_alerts (
    alert_id SERIAL PRIMARY KEY,
    target_scope VARCHAR(128) NOT NULL, -- Serial Number or Entity ID
    violation_type VARCHAR(64) NOT NULL CHECK (violation_type IN ('BROKEN_LINEAGE', 'DOUBLE_DELIVERY', 'TEMPORAL_INVERSION', 'REPORTING_LAG_EXCEEDED', 'SHORT_SHELF_LIFE')),
    regulatory_basis TEXT NOT NULL, -- Statutory Article citations
    risk_score NUMERIC(3,2) NOT NULL CHECK (risk_score BETWEEN 0.00 AND 1.00),
    auditor_notes TEXT NOT NULL,
    logged_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## 5. Technical Specifications of Generated Visualization Graphs

AMDT-CP provides compliance officers with high-density visual analytics. Since the platform operates in React, visualizations are developed using responsive custom HTML5 SVG vectors and Canvas drawing loops to ensure fast rendering speeds and adaptive scaling.

### Graph 1: Field Level Ingestion Integrity Heatmap
- **Objective**: Exposes data completeness, highlight missing critical tracking attributes (such as Lot Number or Expiry Date) across submitting entities.
- **X-Axis**: Document Fields (`reporter_id`, `event_date`, `license_no`, `lot_number`, `serial_number`, `expiry_date`).
- **Y-Axis**: Submitting Entities (e.g., A00013, B00047, B00446).
- **Aesthetic**: Sequential color scale ranging from deep slate crimson (0.0 data integrity) to safety emerald green (1.0 perfect completeness). Hovering over a cell reveals the exact null count and percentage.

### Graph 2: Global Supply Chain Flow Sankey Engine
- **Objective**: Displays the macroscopic distribution flow of all devices in the database.
- **Node Columns**: Left = Prime Importers (B00047, B00446); Center = Intermediaries and Broker Nodes (C00306, C04961); Right = Medical Centers and Clinical Endpoints (A00013, A00002).
- **Link Weights**: Represents the normalized volume of distinct device serial numbers flowing through a channel.
- **Visual Alert Overlay**: Flow pathways that route through unmapped sub-dealers are colored in high-transparency warning orange with an active visual ripple effect to immediately draw attention to unauthorized supply loops.

### Graph 3: Spatial Velocity Distribution Tracking Index
- **Objective**: Identifies logical transportation anomalies through physical dimensions.
- **X-Axis**: Haversine geographical distance between counterparties (in kilometers).
- **Y-Axis**: Actual transit duration delta between supplier dispatch and hospital receipt (in hours).
- **Compliance Boundary Line**: A physical threshold line drawn at $V = 100 \text{ km/h}$. Transactions plotted in the upper-left quadrant (representing near-instantaneous transfers over long geographical distances) are flagged as neon-crimson "Phantom Transaction" alerts.

### Graph 4: Administrative Reporting Lag Latency Chart
- **Objective**: Measures compliance with the statutory 15-day reporting deadline.
- **X-Axis**: Delays in days, segmented into brackets: Same Day, 1-3 Days, 4-7 Days, 8-14 Days, and >14 Days Delay.
- **Y-Axis**: Volume counts of transactions matching the delay brackets.
- **Regulatory Deadline Line**: A vertical dashed line at Day 15. All bars extending to the right of this line are colored in dark red, representing actionable statutory violations.

### Graph 5: Lineage Disconnection Network Graph Topology
- **Objective**: Visualizes structural supply chain holes and disconnected networks.
- **Layout**: Spring-directed layout engine (PyVis/Force-Directed SVG simulation).
- **Node Size**: Proportional to the entity's total violation frequency.
- **Link Colors**: Green for verified dual-signed supply loops; dotted red for isolated, single-entry transactions pointing to unverified parallel-import dealers.

### Graph 6: Product Expiration Risk Timeline Profiler
- **Objective**: Prevents the use of expired implants.
- **Timeline continuum**: Gantt-style scale spanning 24 months (2026-01-01 to 2027-12-31).
- **Visual Coding**: Safe devices with long shelf lives are colored in cool grey. Units approaching expiration within 90 days are highlighted in warm amber, while expired devices are rendered as flashing crimson bands.

---

## 6. Advanced "Wow" AI Features (The First 3 Capabilities)

The platform includes three advanced AI-driven features to automate forensic analysis of supply chains.

### Feature 1: Autonomous Forensic Supply Chain "De-Blinding" Engine
When the Lineage Reconstruction Agent identifies an unmapped intermediary code (such as `C04961`), the De-Blinding Engine is triggered. The engine executes search queries targeting corporate registry directories, import-export databases, and national gazettes. It extracts registration names, tax identification numbers, and warehouse address coordinates, automatically updating the system registry without requiring developer intervention.

### Feature 2: Generative Mock Data Synthetic Twin Generator
To facilitate scenario stress-testing, this feature allows compliance officers to generate custom synthetic data using natural language prompts. For example, an officer can prompt: "Generate a synthetic dataset of 500 pacemakers simulating a grey-market parallel import operation across 4 regional clinics, complete with reporting delays and duplicate serial collisions." The engine outputs a complete, self-consistent relational set of CSV and JSON files matching all schema constraints.

### Feature 3: Conversational Forensic Auditing & Dynamic Prompt Synthesis
An integrated NLP chat interface that allows compliance officers to audit tracking files using natural language. When an officer asks, "Are there any pacemakers received in April that have duplicate serial numbers on the West Coast?", the planner agent converts the question into a multi-step execution plan, writes and executes a local Python Pandas script, parses the database tables, and renders a visual map and report detailing the findings.

---

## 7. Three NEW Advanced "Wow" AI Features (Feature 4, 5, and 6)

To elevate this platform to the peak of international regulatory software systems, we specify three new advanced AI capabilities.

### 7.1 Feature 4: Sovereign Predictive Supply Shortage & Dynamic Load-Balancing Agent
- **The Capability**: Class III clinical implants must not only be trackable; they must also be available. This agent uses temporal distribution forecasting to predict regional stock-outs of life-saving implants and suggests optimal, compliance-verified rerouting pathways.
- **Underlying Mechanism**: Using a recurrent temporal analysis model, the worker node analyzes historic consumption rates, regional infection patterns, and shipping delays to forecast inventory depletion rates. When the system detects a high probability of a stock-out at a hospital group, it searches for neighboring facilities with excess inventory. The agent then generates an optimized, legally compliant transfer route. This route is checked against regulatory travel limits to ensure transit temperatures remain within the device's safe operating limits.

```
[Hospital Group A (90% Stockout Risk)]
                   ▲
                   │ Optimized Rerouting Pathway
                   │ (Compliance Checked)
[Dynamic Load Balancing Agent]
                   ▲
                   │
[Hospital Group B (Excess Inventory)]
```

- **Functional Benefits**: Prevents critical stockouts of pacemakers in regional trauma hospitals during surges in emergency admissions. It also reduces wastage of high-value implants nearing their expiration dates by dynamically moving them to high-consumption centers.

### 7.2 Feature 5: Zero-Knowledge Proof (ZKP) Vendor Compliance Ring
- **The Capability**: Sub-dealers and intermediary distributors frequently refuse to share their transactional data due to trade secret and price confidentiality concerns. This feature enables distributors to prove that they purchased their inventory from an authorized source without exposing price points or logistics routes to the central regulator.
- **Underlying Mechanism**: The platform uses a cryptographic proof pipeline. The prime manufacturer publishes a list of authorized device serial hashes on an immutable distributed ledger. When a sub-dealer lists a device for purchase at a hospital, the sub-dealer's node executes a Zero-Knowledge Proof (ZKP) verification loop. This loop outputs a cryptographic proof showing that the device's serial number exists within the manufacturer's authorized list, without revealing the purchase price, transit path, or the identity of the broker.

```
[Prime Manufacturer] ────► Publish authorized serial hashes to ledger
                                     ▲
                                     │ Zero-Knowledge Proof Verification
[Sub-Dealer Node]    ────► Cryptographic Proof ────► [TFDA Central Portal]
                           (No price or route exposed)
```

- **Functional Benefits**: Resolves the classic stand-off between government regulatory tracking and corporate commercial privacy. It allows the TFDA to guarantee 100% supply chain authenticity for high-value Class III medical devices while fully protecting the proprietary transaction routes and price metrics of private brokers.

### 7.3 Feature 6: Multi-Model Jury Agent Consensus Protocol (The Courtroom Agent)
- **The Capability**: Supply chain anomalies are rarely simple. Determining whether a duplicate serial tag represents a fraudulent counterfeit vs. a simple typographical data-entry error by a tired nurse requires deep contextual reasoning. This feature triggers a multi-model "courtroom" debate between specialized model personas to analyze complex anomalies and produce a consensus legal action plan.
- **Underlying Mechanism**: When the system detects a severe anomaly (such as a double-delivery collision), it launches a multi-model consensus jury. The system instantiates three distinct agent personas:
  1. **The Prosecuting Inspector (Model: `gemini-3.1-pro-preview` configured for aggressive, strict fraud detection)**.
  2. **The Defense Counsel (Model: `Claude-3.5-Sonnet` configured to identify potential data-entry typos and human factor errors)**.
  3. **The Presiding Magistrate (Model: `gemini-3.5-flash` acting as the impartial jury chair)**.
  
  The agents run a multi-turn conversation. The Inspector points to geographical velocity violations and missing lot codes. The Counsel presents evidence of common hospital date-entry patterns (such as billing batched at the end of the month). After three rounds of debate, the Magistrate synthesizes their arguments and outputs a formal compliance decision. This decision includes a calculated confidence score and a targeted investigative plan.

```
               +─────────────────────────────────────────────────────────+
               |                  PRESIDING MAGISTRATE                   |
               |                (Model: gemini-3.5-flash)                |
               +─────────────────▲─────────────────────▲─────────────────+
                                 │                     │
                  Consensus Ruling  │                     │ Case Arguments
                                 │                     │
               +─────────────────▼─────+             +─▼─────────────────+
               | PROSECUTING INSPECTOR |             |  DEFENSE COUNSEL  |
               | (gemini-3.1-pro-prev) |             | (Claude-3.5-Sonn) |
               +───────────────────────+             +───────────────────+
```

- **Functional Benefits**: Eliminates false positives that lead to expensive, unnecessary hospital inventory freezes. It ensures that the limited physical investigative resources of the TFDA are focused exclusively on high-probability fraudulent operations rather than simple administrative errors.

---

## 8. Complete System Implementation Code Blueprint

(The fully operational Streamlit implementation and system core file are built directly inside the React sandbox environment, utilizing the `DEFAULT_GEO_STATIONS` and parsed dataset arrays dynamically within the browser.)

---

## 9. 20 Comprehensive Follow-Up Questions

To finalize the production-grade deployment of the AMDT-CP system, the engineering and regulatory teams must address the following technical, architectural, and statutory questions:

### 9.1 Data Scale and Performance Engineering
1. **Context Window Limitations**: When scale targets scale up from 25 mock transactions to 5,000,000 historical rows of annual country-wide logs, what data chunking and windowing mechanisms should be implemented to prevent context-window overflow during worker node evaluation?
2. **Model Escalation Criteria**: What precise threshold metrics (e.g., node path depth complexity or risk classification severity) should trigger the supervisor node to escalate a task from the low-cost `gemini-3.1-flash-lite` to the high-capacity `gemini-3.1-pro-preview`?
3. **Structured Schema Enforcement**: How can the JSON schema validation options of the Google GenAI SDK be configured to guarantee that `gemini-3.1-flash-lite` consistently returns structured tracking outputs without experiencing formatting or syntax faults?
4. **Dynamic Code Execution Safety**: How will we isolate and protect the Python data processing execution sandboxes (used for Pandas data sanitizing and distance calculation) from remote code execution vulnerabilities?
5. **RAG Vector Database Chunking**: What text chunking and retrieval options (such as overlapping tokens and Cosine similarity metrics) will best support the fast retrieval of complex legal clauses from the TFDA Medical Device Act vector database?
6. **NetworkX Scale Limits**: What are the performance and memory boundaries of the NetworkX path tracing engine when rendering cross-matching topological graph maps for thousands of distinct device serial numbers simultaneously?
7. **Sankey Performance Optimization**: What data aggregation and thinning techniques should be deployed to prevent the Plotly HTML5 SVG canvas from stalling when displaying networks with over 10,000 distinct tracking nodes?
8. **Caching Strategies**: What database caching strategies can be utilized to prevent redundant worker agent executions when identical transaction logs are repeatedly evaluated in separate auditing sessions?

### 9.2 Spatial-Temporal and Logistical Calibrations
9. **Transport Mode Velocities**: How should the Spatial-Temporal Audit Agent's velocity calculations be calibrated to dynamically adapt to different transport methods, such as air freight vs. standard island-wide courier ground delivery?
10. **Timezone Standardization**: How will the system resolve date and timezone inconsistencies when importing CSV files containing mixed formats, such as UTC timestamps, local dates, and ROC calendar configurations?
11. **OSINT Verification Safety**: During autonomous de-blinding lookups, what structural verification gates must be cleared before the agent is permitted to write newly resolved distributor identities back into the core database?
12. **Synthetic Data Tuning**: How can the statistical distributions used by the Generative Mock Data Twin Engine be calibrated to ensure synthesized data patterns accurately match authentic hospital inventory consumption rates?

### 9.3 Security, Privacy, and Cryptographic Compliance
13. **ZKP Computational Latency**: What are the performance and latency implications of executing Zero-Knowledge Proof verification runs directly within the client's browser, and how can we optimize proof generation times?
14. **Access Control Directories**: How will the system's role-based access controls integrate with existing hospital IAM directories (e.g., SAML 2.0 or Active Directory) to restrict access to sensitive patient-associated serial number logs?
15. **Audit Logging Standards**: What specifications (such as logging agent thought steps, tools executed, and raw model responses) must be captured to ensure the complete legal compliance and reproducibility of the generated audit reports?
16. **PII Isolation Policy**: How can we ensure that patient identity data is strictly isolated and never sent to external LLM model servers during compliance auditing or report generation workflows?

### 9.4 Regulatory and Administrative Alignment
17. **Multilingual Name Reconciliation**: How can we improve the Data Sanitizer's accuracy when resolving enterprise entity names recorded in combinations of English characters, traditional characters, and numerical registry codes?
18. **Statutory Customization**: How easily can the regulatory rules engine be adapted to support compliance audits under non-Taiwanese frameworks, such as the US FDA's Unique Device Identification (UDI) Rule or the European Union's MDR?
19. **Actionable Alert Classification**: How will the system classify alerts to separate minor administrative errors (e.g., a simple date-entry typo) from high-priority active fraud indicators (e.g., duplicate serial numbers)?
20. **Courtroom Export Formatting**: How will the system's report compilation pipeline format embedded charts and network graph diagrams to meet the strict design standards required for legal courtroom submissions?
