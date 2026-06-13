MEDTUDI INTELLIGENCE ENGINE (SLEEK INTERFACE)
Enterprise Systems Architecture & Complete Technical Specification
System Version: v4.2.0-Alpha
Target Architecture: Full-Stack Node.js (Vite + Express + React 19) / Highly Portable Analytical Platform
Design Pattern Language: Sleek Interface (High-Contrast Slate-950, Cosmic Indigo Accent, Minimal Geometric Framing)
Document Classification: Advanced Technical Specification & System Design Blueprint
1. Executive Vision & Platform Architecture
1.1 Programmatic Objective
The MedTUDI Intelligence Engine is a high-performance, container-optimized system designed for the ingestion, dynamic transformation, analysis, and cross-reconciliation of disparate global medical device databases. Specifically, the system resolves administrative, semantic, and structural gaps between:
WHO MeDevIS (Medical Devices Information System): A complex, globally diversified nomenclature schema containing high-dimensional metadata (e.g., European Medical Device Nomenclature [EMDN] codes, Global Medical Device Nomenclature [GMDN] codes, healthcare setting levels, and clinical knowledge criteria).
TUDID (Taiwan Unique Device Identifier Database): A structured, strictly formatted national directory containing registration IDs, GS1 UDI-DI codes, model numbers, product descriptions in Traditional Chinese, and clinical characteristics.
By integrating client-side reactive models with an asynchronous server-side routing layer, this architecture enables real-time regulatory auditing, predictive inventory tracking, and dynamic semantic alignment across international boundaries.
1.2 Desktop-First Architectural Flow
The structural layout adheres to the high-density Sleek Interface design pattern. The workspace is formatted into a zero-latency, single-screen structural grid (specifically constrained to 1024px width by 768px height, or dynamically computing fluid screen borders with strict aspect ratio locks). High-frequency layouts utilize a left-hand vertical operations bar, a top global variables controller, a split middle workspace divided into ingestion and data grids, and a lower console featuring a real-time system ledger and an interactive AI Research pane.
code
Code
+--------------------------------------------------------------------------------------------------------+
| [M] MEDTUDI INTELLIGENCE ENGINE          [Engine: Active] [Language: ZH-TW] [Style: Ocean Cobalt]      |
+------------------------------------+-------------------------------------------------------------------+
|  [HIL Operations Bar]              |  [Data Ingestion Hub]                                             |
|  - Ingest Portal                   |  - Paste/Drop Zone for CSV, JSON, TXT (MeDevIS / TUDID)           |
|  - Standardized Grid               |                                                                   |
|  - Unified Analytics               |  +-------------------------------------------------------------+  |
|  - Multi-Agent Console             |  | Standardized Live Preview Grid (Default: 20 Recs)           |  |
|                                    |  | Schema: license_no | product_name | category | udi_di | _extras  |   |
|  [LLM Selector Switch]             |  +-------------------------------------------------------------+  |
|  - Gemini 1.5 Pro / Flash          |                                                                   |
|  - Gemini 3.1 Flash Lite (Default) |  [Interactive Auto-Dash Infographics Workspace]                  |
|                                    |  - Multi-Class Sankey - Network Topology - Time Series- Heatmap   |
+------------------------------------+--------------------+----------------------------------------------+
| [Live HUD Terminal Execution Log (100ms Updates)]       | [AI Console Summary Panel & Magic Suites]    |
| [10:24:02.115] Ingestion parsing buffer...              | - Markdown analysis field & interactive text  |
| [10:24:02.342] Mapping GS1 UDI-DI prefix indexes...     | - 5 Core + 3 Advanced "Wow" AI actions        |
+------------------------------------+--------------------+----------------------------------------------+
2. Ingestion Pipeline & Dynamic Transformation Spec
2.1 File Format Auto-Detection Mechanics
The system features a stream-aligned parsing worker capable of processing file sizes up to 50MB. Rather than relying purely on MIME-type metadata (which is frequently corrupted or generic, such as text/plain for CSV/TSV), the ingestion parser analyzes the incoming raw textual byte string via a heuristic structure pass:
JSON Heuristic: The parser trims outer white-spaces and evaluates the initial and terminal indices of the character string. If the pattern matches ^\{[\s\S]*\}$ or ^\[[\s\S]*\]$, it routes the payload to the JSON parser thread.
Tabular Separation Heuristic: If JSON fails, the system reads the first 10 non-empty rows. It computes the occurrence counts of commas (,), semicolons (;), and tabs (\t) across each line. It applies a standard deviation evaluation algorithm:

where 
 represents the separator count on row 
. The delimiter yielding 
 with the highest median occurrence 
 is designated as the target delimiter.
2.2 Direct-Inject Paste Buffer
To accommodate workflows where files are not physically present but records reside in clipboard memory, the dashboard exposes an raw-input text area. The input buffer triggers the same auto-detection heuristics as the file drag-and-drop boundary.
2.3 Normalized Schema Definitions
The pipeline processes raw records into a validated Unified Standard Schema. This target model normalizes column layout differences into high-contrast, queryable database entities:
Field Name	Type	Constraints	Description
record_id	UUIDv4	Non-nullable, Unique	Autogenerated system tracking identifier.
source_dataset	Enum	Non-nullable	WHO_MEDEVIS or TUDID or STANDARDIZED.
license_no	String	Nullable, Indexed, Matched	The TFDA permit number format (e.g., 衛部醫器輸字第034848號).
udi_di	String	Nullable, GS1 Validated	Primary Device Identifier (14-digit GTIN standard layout).
product_name	String	Non-nullable	Cleaned Chinese product name or primary trade designation.
product_desc	String	Nullable	Detailed English description, system characteristics, or clinical settings.
category	String	Non-nullable, Indexed	Unified clinical category or nomenclature class code.
model_no	String	Nullable	Alphanumeric manufacturer model identifier.
manufacturer	String	Nullable	Normalized corporate manufacturing body.
is_standardized	Boolean	Non-nullable, Default: true	Status tracking flag indicating successful schema transition.
_extras	JSONB	Non-nullable	Dictionary preserving all non-standard fields for downstream agent querying.
2.4 Algorithmic Transformation Rules
When data is classified as non-standard, the pipeline initializes a multi-pass transformation regex framework:
code
Code
+---------------------------------------------------------------------------------------------------+
|                                 INGESTION & CLEANING STREAM WORKER                               |
+---------------------------------------------------------------------------------------------------+
|  [Stage 1: Header Normalization]                                                                  |
|  - Translate field headers via key dictionary overlays (e.g., "基本DI" -> "udi_di")              |
|  - Convert unicode spacing, strip special punctuation, and apply camelCase formatting             |
+---------------------------------+-----------------------------------------------------------------+
                                  |
                                  v
+---------------------------------+-----------------------------------------------------------------+
|  [Stage 2: Semantic Translation & Extraction Rules]                                               |
|  - Execute TFDA license tracking checks: Extract license numbers via regex expression:             |
|    /衛[署部]醫器[陸輸製]字第\d+號/                                                                |
|  - Extract device model characteristics: Parse the primary design patterns from strings to      |
|    isolate power, curve, cylinder, axes, or axis elements (e.g., "145 -07.00 125 070")             |
+---------------------------------+-----------------------------------------------------------------+
                                  |
                                  v
+---------------------------------+-----------------------------------------------------------------+
|  [Stage 3: Validation Framework]                                                                  |
|  - Inject unmapped keys directly into the "_extras" structured fallback schema                    |
|  - Flag raw anomalies (e.g., null models or malformed UDI sequences) with dynamic warnings        |
+---------------------------------------------------------------------------------------------------+
2.5 Preview Configuration Mechanics
The Standardized Live Preview component is controlled by a reactive container variables state. A secondary input controller (supporting parameters of 20, 50, 100, or 500 rows) triggers slice mutations on the underlying in-memory dataframe:
code
TypeScript
const previewSlice = processedDataset.slice(0, selectViewCount);
During active transformation execution, rows transition down the screen with visual styling representing progress, featuring animated fade-in states driven by layout libraries.
3. Multi-Model LLM Orchestration & Prompt Architecture
3.1 Connection Architecture
To ensure extreme reliability without client-exposure of high-security admin credentials, the platform utilizes an Express-backed server-side API proxy model. All client requests are dispatched through a single routing API /api/agent/dispatch, carrying payload metadata, the designated agent system-prompts, the targeted LLM model, temperature settings, and optional parameter objects.
code
Code
[React Client Workspace]
       |
       |  (Post Request Payload containing Model selection parameters and Target Data)
       v
[/api/agent/dispatch] ----------> [Process Environment Key Discovery]
                                       |
                   +-------------------+-------------------+
                   |                                       |
                   v                                       v
         [GEMINI_API_KEY Found]                 [GEMINI_API_KEY Empty]
                   |                                       |
                   v                                       v
        (Instantiate SDK Layer)                 (Request Secret via UI Pane)
                   |                                       |
                   +-------------------+-------------------+
                                       |
                                       v
       [Connect to Google GenAI SDK using Server-Side Credentials]
                                       |
                                       v
              [Retrieve JSON/Markdown Structured Responses]
3.2 Supported Model Inventory
Gemini 3.1 Flash Lite (Default): Assigned to rapid semantic table matching, standard entity parsing, and low-latency log generation.
Gemini 3.1 Pro: Designated for high-complexity regulatory cross-referencing, multi-language context matching, and multi-step inference chains.
Gemini 2.5 Pro: Assigned to processing complex tabular structures, schema transformation runs, and structured output formatting.
Gemini 2.5 Flash: Selected for high-throughput translation tasks and formatting large exports.
3.3 Dynamic In-App Live Activity Log Terminal
To capture user attention during multi-step analysis runs, the dashboard includes a terminal pane simulating automated systems logs. This panel updates every 100 milliseconds using randomized execution delays.
Simulated Running Ledger Output Example:
code
Text
[14:24:02.102] [SYSTEM] Initializing Ingestive Stream Parser on TUDID_Upload_06-13.csv...
[14:24:02.205] [PARSER] Extracted 45 unique contact lens entries under TFDA license 衛部醫器輸字第034848號.
[14:24:02.311] [STANDARDIZATION] Standardizing row structural keys: Mapping "基本DI" to schema key "udi_di".
[14:24:04.415] [GATEWAY] Routing context data to Gemini 3.1 Flash Lite for compliance check...
[14:24:04.608] [AGENT] Executing Core Wow Feature 2: Compliance Gap & Risk Radar...
[14:24:05.112] [AGENT] Anomaly discovered: Model series 000000000010165375 lacks localized EMDN mapping.
[14:24:05.356] [SYSTEM] Export buffers finalized. Ready for JSON/XML conversion.
4. The 5+3 Cognitive "Wow" AI Features Spec
The system provides eight dedicated, interactive "Wow" features constructed to automate, enrich, and secure data workflows. Clicking any tool modifies the current application state, triggers the live logger, updates the workspace grid, and generates structured markdown analyses.
code
Code
+-------------------------------------------------------------------+
       |                    DATASET METADATA REPOSITORY                    |
       +---------------------------------+---------------------------------+
                                         |
         +-------------------------------+-------------------------------+
         |                                                               |
         v                                                               v
 [5 Primary Core AI Magics]                              [3 Advanced Breakthroughs]
- Supply Allocation Anomaly Pulse                       - Nomenclature Alignment Tunnel
- Active Regulatory Risk Radar                          - Pathological Adverse Predictor
- Arbitrage & Price Velocity Planner                    - Bilingual Semantic Translator
- Spatial-Temporal Demand Forecaster
- Patient-Device Matching Engine
                                         |
                                         +-------------------------------+
                                                                         |
                                                                         v
                                                       [Dynamic Large Language Model]
                                                                         |
                                                                         v
                                                       [Final Interactive MD Summary]
4.1 Feature 1: Dynamic Supply & Allocation Anomaly Pulse
Operational Mechanism: Analyzes statistical distribution parameters across product model distributions. If a single product model contains identical base specifications but presents divergent UDI-DI tracking fields, this tool alerts the operator to a potential gray-market routing deviation.
Algorithmic Prompt Logic:
code
Text
[SYSTEM CONTEXT]
You are an expert Forensic Medical Logistics Inspector. Analyze the provided standardized logistics trace for discrepancies indicating stock-piling, cargo diversion, or illegal distribution splits.

[DATA PAYLOAD]
{dataset_json}

[OUTPUT INSTRUCTION]
Identify records that deviate statistically from average shipment frequencies. Produce a diagnostic table outlining anomaly risk scores (0-100%) and pinpointing critical tracking points.
4.2 Feature 2: Active Regulatory Compliance Gap & Risk Radar
Operational Mechanism: Validates whether the active license number meets the strict national registry structure, checks if UDI codes adhere to GS1 specifications, and determines whether the active lifecycle is within range.
Algorithmic Prompt Logic:
code
Text
[SYSTEM CONTEXT]
You are a Senior Regulatory Auditor specialized in TFDA/WHO medical device licensing systems.

[DATA PAYLOAD]
{dataset_json}

[OUTPUT INSTRUCTION]
Scan all entries. Check license patterns for structure-check values: "衛部醫器" + ("輸"|"製"|"陸") + "字第" + 6 digits + "號". Identify entries that fail this validation. Flag high-risk records with a detailed tracking report outlining recommended corrective administrative actions.
4.3 Feature 3: Cross-Regional Arbitrage & Price Velocity Planner
Operational Mechanism: Evaluates pricing attributes embedded in clinical datasets against regional demand, calculating logistical price volatility indexes.
Algorithmic Prompt Logic:
code
Text
[SYSTEM CONTEXT]
You are an Healthcare Procurement Economist tracking global distribution pricing structures.

[DATA PAYLOAD]
{dataset_json}

[OUTPUT INSTRUCTION]
Estimate price velocity risks for each product class. Identify potential pricing discrepancies across different registries where price-to-volume ratios exceed standard deviation limits. Recommend optimal procurement structures.
4.4 Feature 4: Proactive Spatial-Temporal Demand Forecasting Pipeline
Operational Mechanism: Extrapolates seasonal parameters based on hospital usage trends, preventing inventory shortfalls of critical components.
Algorithmic Prompt Logic:
code
Text
[SYSTEM CONTEXT]
You are an Advanced Demand Planner tracking medical device depletion variables.

[DATA PAYLOAD]
{dataset_json}

[OUTPUT INSTRUCTION]
Construct a chronological demand forecast. Identify product categories with potential supply deficits within the next 60 days. Structure findings into a forecasting table.
4.5 Feature 5: Patient-Device Matching Optimization Protocol
Operational Mechanism: Reviews detailed device specifications (such as lens cylinder and axis values parsed from desc field PRECSION1 TOR -07.00 125 070) to confirm matching accuracy against standardized prescription files.
Algorithmic Prompt Logic:
code
Text
[SYSTEM CONTEXT]
You are a Clinical Biomedical Safety Engineer verifying precise lens prescription compatibility metrics.

[DATA PAYLOAD]
{dataset_json}

[OUTPUT INSTRUCTION]
Parse specific cylinder, sphere, and axis configurations from describing text fields. Identify any structural specification anomalies. Generate a clinical verification scorecard.
4.6 Advanced Feature 6: Nomenclature Alignment Tunnel
Operational Mechanism: Resolves semantic mapping issues between WHO GMDN/EMDN nomenclature tags and local Taiwan products.
Algorithmic Prompt Logic:
code
Text
[SYSTEM CONTEXT]
You are an International Healthcare Terminologist bridging local registration items with global nomenclatures.

[DATA PAYLOAD]
{dataset_json}

[OUTPUT INSTRUCTION]
Analyze the localized product names and mapping strings. Link them to the closest international classification codes (e.g., EMDN Class L or A codes). Output a translation map outlining matching confidence scores.
4.7 Advanced Feature 7: Pathological Adverse Predictor & Risk Simulator
Operational Mechanism: Maps active devices to historical international recall vectors (such as global FDA warnings on high-risk implants), predicting potential clinical liabilities.
Algorithmic Prompt Logic:
code
Text
[SYSTEM CONTEXT]
You are an Epidemiological Risk Assessment Expert analyzing international clinical alert vectors.

[DATA PAYLOAD]
{dataset_json}

[OUTPUT INSTRUCTION]
Simulate potential clinical liabilities for each active category. Recommend urgent inspection intervals and outline safety metrics.
4.8 Advanced Feature 8: Bilingual Semantic Translator & Localization Tunnel
Operational Mechanism: Provides zero-loss semantic translations of technical clinical terms (e.g., translating "散光日拋隱形眼鏡" to "Toric Daily Disposable Contact Lenses") without altering critical specification data.
Algorithmic Prompt Logic:
code
Text
[SYSTEM CONTEXT]
You are an Expert Biomedical Translator localizing technical databases between English and Traditional Chinese.

[DATA PAYLOAD]
{dataset_json}

[OUTPUT INSTRUCTION]
Perform translations on text structures. Keep alphanumeric tracking characters, model numbers, and metric coordinates intact. Provide a side-by-side localization map.
5. Visual Dashboard and Plotly Dynamic Interactive Chart Specification
The user is equipped with six complex, interactive visualization components to analyze logistics networks. Clicking on interactive elements within these charts exposes analytical metrics.
code
Code
+----------------------------------------------------------------------------------------------------+
|                                      VISUALIZATION DASHBOARD                                       |
+------------------------------------+---------------------------------------------------------------+
|  Multi-Class Sankey-Flow           |  - Maps flow pathways from suppliers to target classifications |
|  Topological Dynamic Network       |  - Represents linkages between active licenses and components |
|  Dual-Axis Temporal Trends         |  - Evaluates shipment events against aggregate asset volume   |
|  Demographic Density Heatmaps      |  - Matrix visualization highlighting regional supply surges   |
|  Clinical Expiry Risk Cohorts      |  - Organizes devices by remaining shelf-life risk tiers        |
|  Symmetrical Comparative Metrics   |  - Parallel charts for distribution and consumption volumes   |
+------------------------------------+---------------------------------------------------------------+
5.1 Interactive Plotly Chart Specifications
The interactive dashboard uses six customized Plotly chart configurations, styled to match the dark high-contrast slate aesthetics of the theme.
Chart 1: Multi-Class Sankey-Flow Diagram
Computational Goal: Tracks structural pathways to illustrate device distributions, tracing volumes from initial suppliers through regulatory classifications to end hospitals.
Python Engine Structure & Plotly Parameters:
code
Python
import plotly.graph_objects as go

# Dynamic Aggregation Logic
df_agg = df_clean.groupby(['supplier_id', 'category', 'license_no'])['quantity'].sum().reset_index()

# Extract Unique Nodes
nodes = list(set(df_agg['supplier_id'].unique()) | set(df_agg['category'].unique()) | set(df_agg['license_no'].unique()))
node_map = {node: i for i, node in enumerate(nodes)}

# Assemble Source-Target Vectors
sources = [node_map[row['supplier_id']] for _, row in df_agg.iterrows()]
targets = [node_map[row['category']] for _, row in df_agg.iterrows()]
values = [row['quantity'] for _, row in df_agg.iterrows()]

# Append Next Hop (Category -> License)
sources.extend([node_map[row['category']] for _, row in df_agg.iterrows()])
targets.extend([node_map[row['license_no']] for _, row in df_agg.iterrows()])
values.extend([row['quantity'] for _, row in df_agg.iterrows()])

# Visual Styling Parameters
fig = go.Figure(data=[go.Sankey(
    node=dict(
        pad=15, thickness=20, line=dict(color="#1e293b", width=1),
        label=nodes, color="#3b82f6"
    ),
    link=dict(
        source=sources, target=targets, value=values,
        color="rgba(59, 130, 246, 0.25)"
    )
)])
fig.update_layout(
    paper_bgcolor="rgba(0,0,0,0)", plot_bgcolor="rgba(0,0,0,0)",
    font_color="#e2e8f0", margin=dict(t=30, b=10, l=10, r=10)
)
Chart 2: Topological Dynamic Network Visualization
Computational Goal: Formulates relationships between manufacturer units, active licenses, and tracking codes.
Python Engine Structure & Plotly Parameters:
code
Python
# Create Coordinate Projections using Fruchterman-Reingold Force Layout
# Nodes representing License files are colored Emerald (#10b981)
# Nodes representing UDI identifiers are colored Cobalt (#3b82f6)
# Edges represent active registration matches
edge_trace = go.Scatter(
    x=edge_x, y=edge_y, line=dict(width=1, color="rgba(148, 163, 184, 0.3)"),
    hoverinfo='none', mode='lines'
)
node_trace = go.Scatter(
    x=node_x, y=node_y, mode='markers',
    marker=dict(
        showscale=True, colorscale='Viridis', color=node_colors,
        size=14, line=dict(color="#1e293b", width=2)
    )
)
Chart 3: Dual-Axis Temporal Trends
Computational Goal: Displays daily transaction indices overlaying the aggregate volume, tracking distribution stability.
Python Engine Structure & Plotly Parameters:
code
Python
fig = go.Figure()
fig.add_trace(go.Bar(
    x=df_time['date'], y=df_time['volume'], name="Shipment Volume",
    marker_color="#3b82f6", yaxis="y"
))
fig.add_trace(go.Scatter(
    x=df_time['date'], y=df_time['tx_count'], name="Transaction Events",
    line=dict(color="#10b981", width=2), yaxis="y2"
))
fig.update_layout(
    yaxis=dict(title="Volume (Units)", titlefont=dict(color="#3b82f6")),
    yaxis2=dict(title="Transaction Events", titlefont=dict(color="#10b981"), overlaying="y", side="right"),
    paper_bgcolor="rgba(0,0,0,0)", plot_bgcolor="rgba(0,0,0,0)"
)
Chart 4: Demographic Density Heatmaps
Computational Goal: Highlights regional anomalies by plotting product classifications against local registration nodes.
Python Engine Structure & Plotly Parameters:
code
Python
fig = go.Figure(data=go.Heatmap(
    z=pivot_matrix.values,
    x=pivot_matrix.columns,
    y=pivot_matrix.index,
    colorscale=[[0, '#0f172a'], [0.5, '#1e3a8a'], [1, '#3b82f6']],
    showscale=True
))
Chart 5: Clinical Expiry Risk Cohorts
Computational Goal: Groups products by remaining shelf-life safety windows to flag expiring stock.
Python Engine Structure & Plotly Parameters:
code
Python
fig = go.Figure(data=[go.Pie(
    labels=['Urgent Recall (<30 days)', 'Critical Watch (30-90 days)', 'Monitoring (91-180 days)', 'Stable (>180 days)'],
    values=cohort_counts,
    marker=dict(colors=['#f43f5e', '#f59e0b', '#3b82f6', '#10b981']),
    hole=0.4
)])
Chart 6: Symmetrical Comparative Metrics Bar Graphs
Computational Goal: Displays local procurement values against production capacities to analyze supply imbalances.
Python Engine Structure & Plotly Parameters:
code
Python
fig = go.Figure(data=[
    go.Bar(name='Taiwan Registration Units', x=categories, y=tw_counts, marker_color='#3b82f6'),
    go.Bar(name='WHO Global Database Count', x=categories, y=who_counts, marker_color='#6366f1')
])
fig.update_layout(barmode='group')
6. CSS Engine, Theme Matrix, and Pantone Color System Specification
The design system incorporates a runtime CSS compiler. This theme engine ensures proper accessibility ratios and maintains styling integrity under both light and dark modes across ten color profiles.
6.1 Theme Configurations Map
code
Code
+---------------------------------------------------------------------------------------------------------+
|                                         PANTONE THEME STYLES                                            |
+---------------------+----------------------++----------------------+------------------------------------+
|  Vibrant Coral      |  (PANTONE 16-1546)   ||  Crimson Bold        |  (PANTONE 19-1761)                 |
|  Tech Indigo        |  (PANTONE 19-4052)   ||  Wisteria Frost      |  (PANTONE 17-3930)                 |
|  Slate Mint         |  (PANTONE 16-5919)   ||  Forest Jade         |  (PANTONE 19-6026)                 |
|  Amber Bloom        |  (PANTONE 14-0848)   ||  Teal Pulse          |  (PANTONE 17-5126)                 |
|  Cerulean Calm      |  (PANTONE 15-4020)   ||  Tangerine Glow      |  (PANTONE 15-1157)                 |
+---------------------+----------------------++----------------------+------------------------------------+
The system map outlines variables for primary colors, accents, and contrasting text elements under both dark and light variations:
code
CSS
:root {
  /* Dynamic Palette Configurator Class System Map */

  /* 01. VIBRANT CORAL: PANTONE 16-1546 Living Coral */
  --coral-primary: #FF6F61;
  --coral-secondary: #1E2A38;
  --coral-accent: #3A8df7;
  
  /* 02. TECH INDIGO: PANTONE 19-4052 Classic Blue */
  --indigo-primary: #0F4C81;
  --indigo-secondary: #0F172A;
  --indigo-accent: #20B2AA;

  /* 03. SLATE MINT: PANTONE 16-5919 Coral Green */
  --slate-mint-primary: #16A085;
  --slate-mint-secondary: #2C3E50;
  --slate-mint-accent: #8E44AD;

  /* 04. AMBER BLOOM: PANTONE 14-0848 Mimosa */
  --amber-bloom-primary: #F5B041;
  --amber-bloom-secondary: #1C2833;
  --amber-bloom-accent: #3498DB;

  /* 05. CERULEAN CALM: PANTONE 15-4020 Cerulean */
  --cerulean-calm-primary: #98B2D1;
  --cerulean-calm-secondary: #17202A;
  --cerulean-calm-accent: #E74C3C;

  /* 06. CRIMSON BOLD: PANTONE 19-1761 Tango Red */
  --crimson-bold-primary: #C0392B;
  --crimson-bold-secondary: #1A252F;
  --crimson-bold-accent: #F1C40F;

  /* 07. WISTERIA FROST: PANTONE 17-3930 Ultra Violet */
  --wisteria-frost-primary: #5B2C6F;
  --wisteria-frost-secondary: #1B2631;
  --wisteria-frost-accent: #2ECC71;

  /* 08. FOREST JADE: PANTONE 19-6026 Forest Green */
  --forest-jade-primary: #1E8449;
  --forest-jade-secondary: #111;
  --forest-jade-accent: #F39C12;

  /* 09. TEAL PULSE: PANTONE 17-5126 Deep Teal */
  --teal-pulse-primary: #148F77;
  --teal-pulse-secondary: #2C3E50;
  --teal-pulse-accent: #E67E22;

  /* 10. TANGERINE GLOW: PANTONE 15-1157 Flame Gold */
  --tangerine-glow-primary: #E67E22;
  --tangerine-glow-secondary: #1A252C;
  --tangerine-glow-accent: #9B59B6;
}
6.2 Light/Dark Mode Variable Mapping
The theme context wrapper maps layout dimensions, backgrounds, typography colors, border variables, and interactive feedback highlights automatically programmatically using CSS styling classes:
code
CSS
/* Sleek Interface Theme: Dark System Defaults */
.theme-dark {
  --bg-main: #020617;        /* Slate 950 */
  --bg-card: #0f172a;        /* Slate 900 */
  --border-main: #1e293b;    /* Slate 800 */
  --text-primary: #f8fafc;   /* Slate 50 */
  --text-muted: #64748b;     /* Slate 500 */
  --accent-neon: #3b82f6;     /* Light blue cobalt */
}

/* Sleek Interface Theme: Light System Overrides */
.theme-light {
  --bg-main: #f8fafc;        /* Slate 50 */
  --bg-card: #ffffff;        /* Crisp white background */
  --border-main: #e2e8f0;    /* Slate 200 */
  --text-primary: #0f172a;   /* Slate 900 */
  --text-muted: #94a3b8;     /* Slate 400 */
  --accent-neon: #1d4ed8;     /* Concentrated blue marker */
}
7. Export Pipelines and XML/JSON/CSV Schema Specs
The system contains dedicated client-side export workers to allow users to output transformed datasets into standard hospital databases or global administrative portals.
7.1 XML Schema Definition Structuring
code
Xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified">
  <xs:element name="MedTUDIEngineExport">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="platform_meta">
          <xs:complexType>
            <xs:sequence>
              <xs:element name="system_version" type="xs:string"/>
              <xs:element name="timestamp" type="xs:dateTime"/>
              <xs:element name="export_count" type="xs:integer"/>
              <xs:element name="compliance_status_summary" type="xs:string"/>
            </xs:sequence>
          </xs:complexType>
        </xs:element>
        <xs:element name="device_collection">
          <xs:complexType>
            <xs:sequence>
              <xs:element name="device_item" maxOccurs="unbounded">
                <xs:complexType>
                  <xs:sequence>
                    <xs:element name="record_id" type="xs:string"/>
                    <xs:element name="license_no" type="xs:string"/>
                    <xs:element name="udi_di" type="xs:string"/>
                    <xs:element name="product_name" type="xs:string"/>
                    <xs:element name="product_desc" type="xs:string"/>
                    <xs:element name="category" type="xs:string"/>
                    <xs:element name="model_no" type="xs:string"/>
                    <xs:element name="manufacturer" type="xs:string"/>
                    <xs:element name="is_standardized" type="xs:boolean"/>
                    <xs:element name="extra_fields_bag" type="xs:string" />
                  </xs:sequence>
                </xs:complexType>
              </xs:element>
            </xs:sequence>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
7.2 Compliant JSON Output Instance
code
JSON
{
  "exportMetadata": {
    "systemIdentifier": "MEDTUDI_INTELLIGENCE_ENGINE",
    "exportTimestamp": "2026-06-13T10:24:02Z",
    "exportCount": 420,
    "systemHealthStatus": "STABLE"
  },
  "datasetRecords": [
    {
      "record_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
      "source_dataset": "TUDID",
      "license_no": "衛部醫器輸字第034848號",
      "udi_di": "00730822294652",
      "product_name": "“愛爾康” 水感散光日拋隱形眼鏡",
      "product_desc": "PRECSION1 TOR 30P 850 145 -07.00 125 070",
      "category": "Ophthalmic Device - Contact Lenses",
      "model_no": "000000000010165367",
      "manufacturer": "Alcon Laboratories Inc.",
      "is_standardized": true,
      "extraFields": {
        "udi_issuer": "GS1",
        "optical_power_sphere": "-07.00",
        "optical_cylinder": "1.25",
        "optical_axis": "070"
      }
    }
  ]
}
7.3 CSV Format Specifications
The CSV output utilizes a standard separation layout to avoid cell truncation issues. Row strings containing escape quotes are encapsulated inside standard text markers:
code
CSV
"record_id","source_dataset","license_no","udi_di","product_name","product_desc","category","model_no","manufacturer","is_standardized","extra_fields"
"9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d","TUDID","衛部醫器輸字第034848號","00730822294652","“愛爾康” 水感散光日拋隱形眼鏡","PRECSION1 TOR 30P 850 145 -07.00 125 070","Ophthalmic Device - Contact Lenses","000000000010165367","Alcon Laboratories Inc.","true","{""optical_power_sphere"": ""-07.00""}"
8. Scale, Security, Performance, and Error Mitigation Frameworks
Designing systems for container runtimes requires proactive safety layouts to prevent resource leaks during large-scale ingestion operations.
code
Code
+-------------------------------------------------------------------+
       |                     INGESTION DATA PRE-FILTERER                   |
       +---------------------------------+---------------------------------+
                                         |
         +-------------------------------+-------------------------------+
         | If file size < 5MB            | If file size >= 5MB           |
         v                               v                               v
[Standard Memory Parser]     [File Partition Splitter (Chunking)] [Streaming UI Render Worker]
         |                               |                               |
         +-------------------------------+-------------------------------+
                                         |
                                         v
                      [System Resource Allocation Check]
                                         |
                                         v
                     [Active UI State Grid Synchronization]
8.1 UI Rendering and Memory Optimization
CSV Ingestion Optimization: Uploaded datasets with more than 10,000 rows are processed in the background using streaming chunks. The UI grid updates rows incrementally, preventing browser lag.
Chart Rendering Optimization: Plotly graphs bypass static layout recalculations on resize events by using a centralized ResizeObserver listener. Layout updates are debounced by 200 milliseconds to preserve system processing resources.
8.2 Client-Side Security Rules
API Key Isolation Layer: Key inputs are stored in system memory rather than physical storage. The input forms match styling configurations to prevent key exposure in logs.
Data Sanitization: Script markers (such as <script> structures) are removed from text pastes prior to analysis, preventing security risks in generated markdown views.
9. System State Management Blueprint
To ensure data persistence during layout changes or language swaps, application configurations are organized within a structured context wrapper:
code
TypeScript
interface MedTUDIEngineState {
  sys_preferences: {
    theme_mode: 'light' | 'dark';
    color_palette: string; // 10 Pantone profiles
    language: 'en' | 'zh_tw';
    preview_records: number;
  };
  credentials: {
    OPENAI_API_KEY: string | null;
    GEMINI_API_KEY: string | null;
    ANTHROPIC_API_KEY: string | null;
    GROK_API_KEY: string | null;
  };
  dataset: {
    raw_input_buffer: string;
    imported_records: Array<StandardizedRecord>;
    filtered_records: Array<StandardizedRecord>;
    active_search_query: string;
  };
  ai_execution_terminal: {
    live_logs: Array<string>;
    current_markdown_briefing: string;
    is_streaming: boolean;
  };
}
10. Comprehensive Architecture Follow-Up Questions
To finalize the production development path of the MedTUDI Intelligence Engine, the system architect proposes a series of development and operational questions to the project team:
10.1 Data Ingestion & Transformation Questions
Parsing Edge Cases: If an uploaded dataset uses mixed line encodings (e.g., standard carriage-returns \r\n interspersed with standard UNIX \n line-breaks), how should the pre-filtering worker normalize row splits to prevent schema row offset errors?
Missing Key Standardizations: If a user uploads a TUDID dataset slice but the critical structural label 基本DI (representing the primary UDI ID tracking column) is completely empty, should the mapping parser prioritize generating UUID keys, or should it search unmapped tracking columns for replacements?
JSON Validation Loops: During the parsing of multi-layered nested JSON structures, what is the maximum depth index the system validation worker should target to prevent memory allocation faults?
License Key Reconciliation: How should the normalization pipeline handle older license codes (such as those matching historical regulatory patterns) that do not conform to modern national classification layouts?
10.2 AI Orchestration and LLM Security Questions
Multi-User Context Overlaps: If multiple users send concurrent API requests to the server routing layer /api/agent/dispatch, how should the platform manage concurrent sessions to prevent key limits from being exceeded?
Prompt Size Mitigation: When running deep checks on large datasets via Gemini models, what strategy should we use to segment large datasets into safe token windows to prevent model limits from being reached?
Parsing Unstructured Specifications: If clinical attributes in the product description fields are written in highly non-standard layouts, should we use targeted regular expressions or rely on fine-tuned LLM parsing models to extract variables?
Output Validation Mechanisms: What safety thresholds should be placed on model-generated JSON files to identify and reject malformed outputs before they are processed by the visual layout modules?
10.3 Dynamic Graphic Layouts & Performance Questions
Rendering High-Density Plots: When displaying network topologies with more than 5,000 node linkages, what strategy should we use inside the Plotly container to maintain responsive performance?
Data Synchronization Layouts: How should we structure the chart updates so that when a user applies a search query, the active sub-charts update their data points smoothly without restarting full page rendering?
Responsive Mobile Charts: How should touch input and zoom controls be mapped within the complex Sankey flow charts on mobile touchscreens to maintain navigation usability?
Custom Chart Formats: Should the visualization engine rely on inline canvas renderings or utilize SVG pipelines to maintain crisp text scaling under different font sizes?
10.4 UI/UX Theming and Pantone Matrix Questions
Accessibility Verification Checklists: Do all ten Pantone color themes maintain a minimum contrast ratio of 4.5:1 against both light (#FFFFFF) and dark (#020617) backdrops, particularly for system text elements?
Dynamic Language Adjustments: When switching languages between English and Traditional Chinese, how does our visual column container handle layout shifts to prevent text truncation issues?
Hysteresis Timing Controls: What delay timings are optimal for animated layout elements to prevent flashing during fast actions?
User Preferences Tracking: Should chosen Pantone color profiles and light/dark theme settings be stored in client cookies or linked directly to database profile records?
10.5 Deployment & Operational Performance Questions
File Execution Controls: What safety restrictions should be placed on file uploads to detect and block malicious attachments before processing?
Web Assembly Optimizations: If dataset sizes grow over 100MB, would compiling core parsing routines into WebAssembly modules provide better performance compared to standard browser execution?
Automated Verification Coverage: What testing approaches should be implemented to verify regulatory compliance parsing across different language configurations and datasets?
Offline Data Persistence: Under network failure scenarios, what browser storage strategies should the system use to protect active, unexported work from data loss?
