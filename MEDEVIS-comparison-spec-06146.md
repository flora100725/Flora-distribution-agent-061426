Technical Specification Document: Agentic AI System for Cross-Referencing TUDID and WHO MeDevIS DatasetsDocument Version: 1.0.0Target Architecture: Multi-Agent Orchestration with Hybrid Search StrategyData Domains: Taiwan Unique Device Identification Database (TUDID) & WHO Medical Devices Information System (MeDevIS)1. Executive Summary & Business ContextIn the global medical device ecosystem, regulatory compliance and clinical procurement are historically siloed operations. National regulatory authorities maintain strict, highly granular item registries—such as Taiwan's Food and Drug Administration (TFDA) Unique Device Identification Database (TUDID). These databases catalog specific stock-keeping units (SKUs) based on exact physical dimensions, prescriptions, and licensing authorities. Conversely, global bodies like the World Health Organization (WHO) maintain macro-level frameworks—such as the Medical Devices Information System (MeDevIS)—which categorize devices by clinical intent, healthcare infrastructure requirements, life courses, and international nomenclatures like EMDN (European Medical Device Nomenclature) and GMDN (Global Medical Device Nomenclature).The core objective of this system is to bridge this gap by deploying an autonomous, Agentic AI platform capable of ingestion, semantic reconciliation, and real-time retrieval across both datasets. When an administrative, medical, or procurement user queries the system using either local terms (e.g., traditional Chinese registration numbers or specific prescription tokens) or global categories, the system autonomously reasons through the structural data disparities. It resolves entities across highly disparate semantic spaces and presents a unified, verified intelligence package.This specification details the structural mechanics, algorithmic routing, machine learning pipelines, and strict safety guardrails required to realize this mission-critical system.2. System Architecture & Agent DesignThe system relies on a decoupled, asynchronous, multi-agent architecture built on top of an enterprise-grade orchestration layer. Single-agent models fail in this domain due to the cognitive load of navigating high-density tables alongside multi-lingual structural transformations.2.1 Component Architecture OverviewThe system is divided into four principal layers:Ingress & Intent Classification Layer: Intercepts user inputs and maps them to explicit execution workflows.Agentic Core Execution Layer (The Multi-Agent Hub): Independent LLM-backed execution instances optimized with specialized runtime tools.Data Retrieval & Knowledge Layer: Managed relational databases, sparse indexers, and dense vector spaces.Guardrail & Fact-Checking Layer: Deterministic filters auditing the outputs of probabilistic LLMs.       +---------------------------------------------------------+
       |                  User Interface / API                   |
       +---------------------------------------------------------+
                                    |
                                    v
       +---------------------------------------------------------+
       |             Orchestration & Routing Agent               |
       +---------------------------------------------------------+
             |                      |                     |
             v                      v                     v
+------------------------+  +-------------------+  +-------------------------+
| Data Harmonization Agt |  | Evaluation Agent  |  | Presentation/Trans Agt  |
+------------------------+  +-------------------+  +-------------------------+
             |                      |                     |
             +----------------------+---------------------+
                                    |
                                    v
       +---------------------------------------------------------+
       |                 Multi-Tool Access Layer                 |
       |  [BM25]   [Vector Search]   [Translation]   [Regex NER] |
       +---------------------------------------------------------+
                                    |
                                    v
       +---------------------------------------------------------+
       |          Enterprise Data Infrastructure                 |
       |  (TUDID PostgreSQL + MeDevIS Vector Store / Qdrant)    |
       +---------------------------------------------------------+
2.2 Multi-Agent BreakdownRather than processing queries inside a single runtime thread, responsibilities are delegated to autonomous agents using a custom-built state engine (e.g., LangGraph or custom LangChain StateGraphs).2.2.1 The Orchestration & Routing AgentRole: System Gateway.LLM Core Backbone: GPT-4o-mini or Claude 3.5 Sonnet (optimized for strict JSON structural outputs).Behavioral Description: Receives raw text inputs. It analyzes the linguistic morphology and structural characteristics of the input string to classify user intent. It acts as an advanced traffic controller, deciding which specific downstream agents to invoke, what context variables to populate in the shared system state, and how to allocate the compute budget.2.2.2 The Data Harmonization & Entity Resolution AgentRole: Domain Context Matcher.LLM Core Backbone: Claude 3.5 Sonnet (selected for exceptional performance in complex reasoning, mathematical parsing, and abstract semantic layout analysis).Behavioral Description: This agent handles data mismatch problems. When presented with raw database rows from TUDID containing fragmented manufacturing descriptions (e.g., PRECSION1 TOR 30P 850 145 -07.00 125 070), it autonomously extracts core features (Brand: Precision1, Design: Toric/Astigmatism, Quantity: 30 Pack) and translates them into clinical descriptors compatible with MeDevIS taxonomy paths.2.2.3 The Localization & Presentation AgentRole: Multi-lingual Interface Synthesizer.LLM Core Backbone: GPT-4o.Behavioral Description: Translates technical datasets between English and Traditional Chinese without destroying regulatory acronyms. It structures unstructured agent notes into interactive, human-readable UI schemas (Markdown, JSON layouts, or component-ready key-value maps).2.2.4 The Independent Evaluation (LLM-as-a-Judge) AgentRole: Continuous Quality Assurance.LLM Core Backbone: Llama-3-70B-Instruct or specialized fine-tuned models.Behavioral Description: Operates asynchronously alongside or immediately after the main agent cycle. It evaluates the linking proposals generated by the Harmonization Agent against a set of deterministic lookup principles to detect hallucinations before data is returned to the user interface.2.3 Shared Memory ArchitectureTo support continuous conversational debugging (e.g., a procurement officer stating: "Filter the results from my last search to only show reusable options"), the architecture incorporates a bifurcated memory model:Short-Term Ephemeral Memory (StateGraph Channel): A highly structured, state-managed transaction log tracking execution paths, active tool tokens, current data chunks, and inter-agent messages inside a single query-response graph traversal.Long-Term Persisted Memory (Redis-Backed ChatHistory): Stores past interaction frames indexed via session identification keys. The Orchestration Agent leverages this layer using a sliding-window summary mechanism to compress history, preserving context while staying within the LLM's attention span.3. Data Integration, Harmonization & Schema MappingThe core challenge of this agentic system lies in reconciling two fundamentally mismatched data schemas. TUDID tracks local licensing identifiers and SKU configurations; MeDevIS tracks global healthcare infrastructure categories.3.1 Source Dataset Schema AnalysisThe system standardizes ingestion using the following explicit definitions:TUDID Ingestion Schemapermit_id (許可證字號（類型）): Primary regulatory key. String format (e.g., 衛部醫器輸字第034848號).udi_issuing_agency (UDI發碼機構): Categorical string enum (GS1, HIBCC, ICCBBA).basic_di (基本DI): High-level device group identifier code.model_number (型號): Direct manufacture string.product_name_zh (產品中文品名): Localized branding text (e.g., “愛爾康” 水感散光日拋隱形眼鏡).product_description (產品描述): Dense feature-rich raw parameters string (e.g., PRECSION1 TOR 30P 850 145 -07.00 125 070).WHO MeDevIS Ingestion Schemadevice_name: Global standard baseline descriptive term.medevis_version: Control string (e.g., 2025 v2.1).reusable: Boolean flag (Reusable, Single-use).sex: Targeted demographic variable (all, male, female).device_type: Functional taxonomy string (e.g., Surgical instruments, trays and bowls).service_type: Clinical department mapping.knowledge_level: Professional complexity mapping (General clinical, Specialized clinical).delivery_platforms: Tiered infrastructure system.life_courses: Demographic tracking arrays.healthcare_settings: Direct operational environments.nomenclature_code_emdn: Regulatory mapping index.nomenclature_term_emdn: Lexical European standard definitions.nomenclature_code_gmdn: Global medical tracking number.nomenclature_term_gmdn: Structural paragraph definition text.3.2 Dynamic Alignment MatrixBecause direct key joins (e.g., TUDID.ID == MeDevIS.ID) are impossible, the system utilizes a multi-tiered fallback alignment methodology engineered via agentic tool chains:[Level 1: Deterministic Crosswalk Mapping]
                 |
                 +---> (Success) ---> Output Unified Record
                 |
            (No Code Match)
                 |
                 v
[Level 2: Semantic Translation & Property Graph Routing]
                 |
                 +---> (Success) ---> Output Unified Record
                 |
          (Low Confidence)
                 |
                 v
[Level 3: Probabilistic Attribute Extraction & Inferences]
                 |
                 +------------------> Final Verification Audit
Level 1: Deterministic Crosswalk Mapping. If a TUDID record contains a valid basic_di or an extracted GMDN code via metadata extraction, the system queries a core translation mapping database linking GMDN to EMDN. If matched, it pulls the associated MeDevIS record directly.Level 2: Semantic Translation & Property Graph Routing. The system converts the Traditional Chinese product descriptions and titles into standardized English clinical descriptors. It passes these terms through a Neo4j medical property graph mapping entities like Contact Lenses $\rightarrow$ Ophthalmic Devices $\rightarrow$ Optical Correction Instruments.Level 3: Probabilistic Attribute Extraction & Inference. If structural metadata remains unavailable, the Data Harmonization Agent uses tokenized vector search weights across the product_description parameters to determine matching medical intent.4. Search Infrastructure & Hybrid Retrieval StrategyTo handle both exact matching (regulatory number searches) and conceptual matching (clinical search workflows), the data platform couples structured transactional search systems with high-dimensional vector mathematical indexing.4.1 Hybrid Database InfrastructureThe system uses a unified, multi-engine architecture:Relational Store (PostgreSQL with pgvector): Holds the raw ingestion matrices for both TUDID and MeDevIS. It handles strict relational mappings and executes structured metadata queries.Dense Vector Search Core (Qdrant Database Cluster): Indexes multi-lingual embeddings generated from product text combinations to facilitate neural context discovery.4.2 Embedding and Sparse Representation ModelsText entries are processed using a dual-representation strategy:Sparse Text Indexing: BM25 (Best Matching 25) algorithmic search over tokenized character terms, optimized with customized dictionaries for stop-word management (stripping structural prefixes like 衛部醫器輸字第).Dense Semantic Embedding: text-embedding-3-large or multilingual-e5-large. The system processes multi-lingual inputs by mapping both the traditional Chinese metadata and the English WHO text fields into a unified 1536-dimensional hyper-space.4.3 Algorithmic Execution FormulaThe retrieval engine combines sparse retrieval scores and dense similarity metrics using Reciprocal Rank Fusion (RRF). This ensures exact-match identifiers are prioritized without discarding semantic relationships.$$RRF\_Score(d \in D) = \sum_{m \in M} \frac{1}{k + r_m(d)}$$Where:$D$ represents the complete document space composed of intersecting records.$M$ represents the search methodologies utilized (Sparse BM25 and Dense Neural Embedding).$r_m(d)$ represents the direct ordinal rank assigned to document $d$ by search strategy $m$.$k$ represents a smoothing constant optimized at $60$ to mitigate disproportionate skewing caused by outlier high rankings from a single system channel.4.4 Chunking & Metadata Enrichment ProtocolsTo optimize context retrieval, records are structured as specialized document blocks. For instance, the TUDID dataset chunk combines explicit string interpolations:=== ENRICHED TUDID SEARCH CHUNK ===
Document ID: TUDID_034848_00730822294652
Source Permit: 衛部醫器輸字第034848號
Manufacturer Branding: "愛爾康" 水感散光日拋隱形眼鏡 (Alcon)
Technical Parameters: PRECSION1 TOR 30P 850 145 -07.00 125 070
Extracted Tokens: Precision1, Toric, Astigmatism, Daily Disposable, Soft Contact Lens
Global Identification Code: GS1-00730822294652
Correspondingly, MeDevIS blocks are compiled down to highly dense knowledge descriptions:=== ENRICHED MEDEVIS KNOWLEDGE CHUNK ===
Nomenclature Term: CONTACT LENSES, CORRECTIVE, DAILY DISPOSABLE
EMDN Code: Q02010101 | GMDN Code: 35085
Clinical Setting: Outpatient care, Specialized ophthalmology treatment
Operational Parameters: Single-use, Demographics: All, System Platform: First-level/Specialized clinic
5. Agentic Workflows & Tool SpecificationsThe system operates via an event-driven loop where the Orchestration Agent receives the payload and issues function calls to dedicated tools.+-----------------------------------------------------------------------+
|                       CORE USER SEARCH QUERY                          |
|    "Find 衛部醫器輸字第034848號 and display its WHO clinical profile"  |
+-----------------------------------------------------------------------+
                                    |
                                    v
+-----------------------------------------------------------------------+
| 1. INTENT CLASSIFICATION & PARSING                                    |
|    - Routing Agent detects exact TUDID registration key format.       |
|    - Extracts token: "衛部醫器輸字第034848號"                           |
+-----------------------------------------------------------------------+
                                    |
                                    v
+-----------------------------------------------------------------------+
| 2. TUDID DATABASE ROUTING                                             |
|    - Invokes tool: `query_tudid_relational_database`                  |
|    - Retrieves 45 SKU records (Precision1 Toric daily parameters).     |
+-----------------------------------------------------------------------+
                                    |
                                    v
+-----------------------------------------------------------------------+
| 3. FEATURE EXTRACTION & TRANSLATION                                   |
|    - Invokes tool: `regex_attribute_parser`                           |
|    - Harmonization Agent receives content. Extracts Chinese brand     |
|      "愛爾康" -> Alcon, and parameter "TOR" -> Toric.                 |
+-----------------------------------------------------------------------+
| 4. SEMANTIC MEDEVIS CROSS-REFERENCE                                   |
|    - Invokes tool: `vector_search_medevis`                            |
|    - Queries "Alcon Toric Daily Disposable Contact Lenses".            |
|    - Matches MeDevIS term: "Corrective Contact Lenses" (EMDN Q0201).  |
+-----------------------------------------------------------------------+
                                    |
                                    v
+-----------------------------------------------------------------------+
| 5. AGGREGATION & GRAPH VALIDATION                                     |
|    - Collates structural WHO data (Outpatient care, Single-use).       |
|    - Validates link using the Evaluation Agent.                       |
+-----------------------------------------------------------------------+
                                    |
                                    v
+-----------------------------------------------------------------------+
| 6. PRESENTATION GENERATION                                            |
|    - Formats output matrix. Generates structural UI layout.            |
+-----------------------------------------------------------------------+
5.1 Tool Definition ManifestsThe following execution primitives are bound directly to the agent environments:query_tudid_relational_databaseParameters: search_term (String), target_column (Enum: permit_id, basic_di, model_number, all).Description: Performs structured SQL string matches across index tables.Return Format: JSON array containing raw strings matching target rows.vector_search_medevisParameters: query_vector (Array of floats), top_k (Integer), filters (JSON Object mapping categorical variables like reusable).Description: Performs vector cosine distance computations against the Qdrant MeDevIS store.Return Format: List of nested dictionaries containing matching WHO documents alongside similarity coefficients.regex_attribute_parserParameters: target_string (String).Description: Applies predefined regulatory expression matrices to pull out base elements like base curve parameters, spherical powers, cylinder markings, and packaging quantities from raw text blocks.Return Format: Formatted key-value maps.llm_semantic_translationParameters: source_text (String), direction (Enum: zh_to_en, en_to_zh).Description: Specialized low-latency translation service preserving contextual medical device vocabulary.Return Format: Standard string.6. Implementation Architecture & Data PipelinesThe engineering implementation utilizes Containerized Services coordinated via a high-performance orchestration plane.                                  +-----------------------+
                                  |   TFDA / WHO Sources  |
                                  +-----------------------+
                                              |
                                              v
+-----------------------+         +-----------------------+
| User Interface        |         | Ingestion Pipeline    |
| (Next.js / React)     |         | (Apache Airflow / S3) |
+-----------------------+         +-----------------------+
     |                                        |
     | (REST / WebSocket)                     v
     v                            +-----------------------+
+-----------------------+         | Database Engine Layer |
| Core Agent Service    |-------->| - PostgreSQL          |
| (FastAPI / LangGraph) |         | - Qdrant Vector DB    |
+-----------------------+         +-----------------------+
     |
     +---> External LLM Foundation Layer (OpenAI / Anthropic APIs)
6.1 Ingestion Flow & Transformation PipelineTo capture ongoing updates from both the Taiwan Food and Drug Administration (TFDA) and the World Health Organization, the ingestion backend operates on a automated schedule run via Apache Airflow.Extraction: Python hooks scrape or extract structural updates via authorized source interfaces, saving source dumps directly to an encrypted S3 landing bucket.Sanitization & Standardization: Pandas and PySpark workers validate character sets, enforce UTF-8 encodings, strip corrupt delimiter rows, and standardize identification layouts.Vector Store Seeding: New records are routed to an internal processing pipe where text clusters are transformed into vector metrics and written directly into the Qdrant indexing engine.6.2 Service Runtime LayerThe runtime engine is built as a FastAPI service containerized using Docker and deployed on an AWS Elastic Kubernetes Service (EKS) infrastructure cluster.Concurrency Paradigm: Highly asynchronous processing paths handle downstream LLM requests via asynchronous worker routines (asyncio).Caching Subsystems: Redis caches store high-frequency search lookups. If a user queries an identical structural regulatory number, the database routing phase is bypassed entirely, serving the complete entity map from memory.7. Guardrails, Safety, Validation & Performance EvaluationIn medical device data management, AI hallucinations pose significant risks to operational safety and procurement accuracy. The system requires multiple validation steps to ensure reliability.7.1 Multi-Tier Guardrail ConfigurationThe system features real-time validation layers that check data at key points during processing:[ Incoming User Query ]
          |
          v
===================================================
 GUARDRAIL LAYER 1: Inbound Input Validation
  - Evaluates security constraints (Prompt Injection)
  - Rejects malformed scripts or injection patterns
===================================================
          |
          v
[ Core System Processing / Agent Execution Loop ]
          |
          v
===================================================
 GUARDRAIL LAYER 2: Outbound Structured Verification
  - Intercepts generated data packets
  - Checks extracted text against real DB IDs
  - Confirms regulatory terms match real sources
===================================================
          |
          v
===================================================
 GUARDRAIL LAYER 3: Evaluation Agent (LLM-as-a-Judge)
  - Scores alignment reasoning
  - Blocks responses below a 90% confidence score
===================================================
          |
          v
[ Output to User Interface ]
Inbound Input Validation: Evaluates security constraints to verify the string is safe, blocking potential prompt injections, malicious scripts, or broken character payloads.Outbound Structured Verification: Intercepts data packets before they are delivered to the UI. The engine extracts the proposed cross-referenced IDs and checks them against the core databases. If the model references an ID or nomenclature term that does not exist in the source tables, the response is blocked, and a fallback data lookup is executed.Evaluation Agent (LLM-as-a-Judge): Analyzes the reasoning path behind a proposed link and scores its alignment confidence. If the evaluation score drops below a 90% threshold, the output is flagged with a warning indicator, or the system requests an alternate search strategy.7.2 Performance Evaluation MetricsSystem performance is continuously measured across three main vectors:Retrieval Quality (Search Metrics)Mean Reciprocal Rank (MRR): Measures how close the first relevant match is to the top of the search results for exact identifier queries.Normalized Discounted Cumulative Gain (NDCG@10): Evaluates the relevance ranking profile of combined hybrid search outputs across semantic clinical searches.Alignment Accuracy (Entity Resolution Metrics)Precision: $\frac{\text{True Positive Linked Records}}{\text{True Positive Linked Records} + \text{False Positive Linked Records}}$Recall: $\frac{\text{True Positive Linked Records}}{\text{True Positive Linked Records} + \text{False Negative Unlinked Records}}$Operational & System QualityTime-to-First-Token (TTFT): Target latency is less than 400ms for conversational workflows.Complete Graph Traversal Latency: Full-scale database matching, extraction, verification, and formatting operations must resolve in less than 3.5 seconds per user transaction cycle.8. Exporting and Reporting SubsystemsThe presentation layer includes functional utilities that convert complex, joined multi-database records into highly polished, structured, and review-ready document downloads.8.1 Export Format ImplementationsJSON Schema Payload: Complete system dumps containing complete operational traces, confidence metrics, and matching source database rows designed for integration into hospital ERP platforms.Tabular Formats (CSV / XLSX): Generates structured data tables that align TUDID rows side-by-side with corresponding MeDevIS tracking parameters, complete with cell formatting, search metadata headers, and localized column names.Procurement-Ready PDF Dossier: Automatically generates high-quality PDF files from HTML/CSS layouts via WeasyPrint. These reports feature clear page margins, standard headers, structured tables, and bold cross-referencing summaries suitable for administrative reviews or regulatory filings.9. Comprehensive System Follow-Up QuestionsTo ensure this technical specification aligns with your production environment, operational needs, and long-term product vision, please consider the following 20 systemic questions:Architecture & Agent FrameworkAgent Orchestration Framework: Do you prefer building the agent state loops using an open-source framework like LangGraph (Python/JS), or should we design a lightweight, deterministic custom state machine tailored to this data schema?LLM Infrastructure Hosting: Will this platform rely on commercial cloud endpoints (e.g., Anthropic Claude API, OpenAI API), or must it operate completely on-premises using open-weights models (e.g., Llama-3, Mistral) due to hospital data privacy policies?Concurrency and Scalability Foundations: What is the anticipated peak load of concurrent medical procurement officers or researchers using this system during high-volume procurement windows?Data Mapping, Semantics & LocalizationTranslation Management: Should the system translate traditional Chinese terms to English before performing vector similarity matches against MeDevIS, or should we index both datasets using a natively multi-lingual embedding model (e.g., multilingual-e5-large)?Parsing Unstructured Discrepancies: The product description strings contain cryptic manufacturing abbreviations (e.g., 30P, TOR, 850). Should we maintain a verified dictionary of these terms, or rely on the LLM's clinical reasoning to parse them during search preprocessing?Managing Hierarchical Disparities: A single TUDID permit covers dozens of individual SKU models with varying dimensions or prescriptions. When presenting results, should the user see a list of every matching model number, or should the system collapse them into a single parent category aligned with MeDevIS?Mapping Unlinked Items: When the agent cannot find a matching product category in MeDevIS for a highly specialized TUDID medical device, should it flag the item as "Unlinked" or try to find the closest parent category based on high-level EMDN classifications?Search Engineering & PerformanceWeighting Hybrid Search: How should the retrieval system balance exact alphanumeric keyword matches (e.g., explicit permit numbers or GTIN codes) against abstract semantic descriptions? Is a standard 70/30 or 50/50 balance preferred?Embedding Update Workflows: How frequently do the source TUDID and WHO MeDevIS datasets change, and what is your target timeline for updating the vector database indices after an upstream data release?Vector Index Management: As the dataset grows over time, will you require specialized indexing configurations (such as HNSW clusters in Qdrant) to maintain low search latencies?User Experience & InterfaceVisualizing Results: How should the user interface display linked records? Do you prefer a side-by-side view separating local regulatory facts from global WHO information, or a single combined dashboard?Context Preservation Limits: How long should a user's conversational search history remain active in long-term memory for contextual follow-up queries?Data Export Options: For spreadsheet exports, should the system generate a single flattened table or an organized workbook with separate tabs for search data, overview summaries, and regulatory reference links?Safety, Guardrails & ComplianceEnforcing Trust Thresholds: What minimum accuracy or confidence score should the Evaluation Agent require before a linked search result is displayed to users without a system warning?Handling Hallucination Events: If the system detects a mismatch during its safety checks, should it show the user the raw unlinked TUDID row with an error message, or run an alternate search query?User-Driven Corrections: Do you need a feature that allows expert users to manually fix or confirm an AI-generated link, saving that choice to a permanent database override table?Medical Domain & Compliance ContextIntegrating Risk Classes: Should the system map Taiwan TFDA's risk classifications (Class I, II, III) to international regulatory frameworks inside the unified interface?Tracking Device Reusability: If a local TUDID entry is missing explicit data about whether an item is single-use or reusable, can the system use the MeDevIS reusable field to infer this safety rule, or must it remain unverified?Clinical Workflow Extensions: Will this system eventually connect to hospital inventory networks (like Epic, Cerner, or localized HIS platforms) to cross-reference procurement logs with global WHO guidelines?Future Schema Extensions: Do you plan to add other national medical device registries (e.g., US FDA GUDID, European EUDAMED) to this agentic system in future development phases?
