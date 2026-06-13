Developing an agentic AI system that bridges local regulatory systems like Taiwan's **TUDID** (Taiwan Unique Device Identification Database) with global healthcare frameworks like the WHO's **MeDevIS** (Medical Devices Information System) presents a fascinating set of challenges.

To help you map out your architecture, data pipelines, agent logic, and compliance strategy, here are 50 comprehensive questions broken down into structural domains.

---

## 1. System Architecture & Agent Design (Questions 1–10)

1. **Agentic Framework:** Will you use a single-agent architecture with complex tools, or a multi-agent workflow (e.g., a routing agent, a data harmonization agent, and a presentation agent)?
2. **Autonomy Level:** Should the agent autonomously decide how to query both databases simultaneously, or execute a strict, pre-defined step-by-step pipeline when a user inputs a query?
3. **Memory Management:** How will the agent retain conversational context if a user starts filtering down a search (e.g., "Show me only the soft contact lenses from my last search")?
4. **Tool Use Strategy:** What specific tools will you expose to the LLM agent (e.g., `search_tudid_db`, `vector_search_medevis`, `entity_resolution_tool`)?
5. **Fail-safe Mechanisms:** If the agent encounters a TUDID record that has absolutely no logical counterpart in MeDevIS, how should it gracefully report this to the user without breaking execution?
6. **Query Intent Classification:** How will the agent determine if a user's query is an exact match identifier search (like a Basic UDI-DI or Model Number) versus a semantic text search (like "daily contact lenses")?
7. **Latency Optimization:** Since Agentic AI steps can introduce latency, what is your target response time for an end-user search query, and how will you optimize it?
8. **Asynchronous Processing:** Should the agent stream search results incrementally as it resolves links, or wait until all cross-database joins are fully completed?
9. **Agent Evaluation (LLM-as-a-Judge):** How will you evaluate the accuracy of the agent's linking decisions? Will you curate a golden evaluation dataset?
10. **Fallback Logic:** If a critical database API times out, can the agent fall back to a cached local copy or static semantic search embedding?

## 2. Data Mapping & Cross-Database Linking (Questions 11–20)

11. **The Missing Link Challenge:** TUDID uses UDI-DI tracking numbers (e.g., `00730822294652`) for specific SKUs, while MeDevIS maps general device categories (e.g., GMDN/EMDN codes). How will your agent map a highly specific SKU to a high-level nomenclature term?
12. **Nomenclature Bridging:** How will the system link a TUDID record to MeDevIS if the TUDID record does not contain an explicit EMDN or GMDN code?
13. **Semantic String Matching:** The TUDID sample shows traditional Chinese names like `“愛爾康” 水感散光日拋隱形眼鏡`. How will the agent cross-reference this with English-dominated MeDevIS fields? Will you embed a real-time translation agent layer?
14. **Parsing Product Descriptions:** TUDID descriptions contain raw parameter strings (e.g., `PRECISION1 TOR 30P 850 145 -07.00 125 070`). Does the agent need a dedicated regex or NER (Named Entity Recognition) tool to extract these attributes before searching MeDevIS?
15. **Handling Many-to-Many Relationships:** Multiple TUDID records (different prescription parameters) point to the exact same device type. How will the agent group these results so the user isn't overwhelmed by duplicate MeDevIS context blocks?
16. **Version Drift:** MeDevIS is explicitly versioned (e.g., `2025 v2.1`). How will the agent handle instances where a previously linked code is deprecated or changed in a newer MeDevIS release?
17. **Data Ingestion Frequency:** How often will the agent's underlying databases sync with official updates from Taiwan's TFDA (TUDID) and the WHO (MeDevIS)?
18. **Confidence Scoring:** Will the agent provide a match confidence percentage (e.g., "95% match based on GMDN alignment") when displaying linked WHO data?
19. **Ambiguity Resolution:** If a text search matches two entirely different device types in MeDevIS, how will the agent prompt the user for clarification?
20. **Handling Missing Metadata:** MeDevIS includes clinical variables like `Sex`, `Life courses`, and `Healthcare settings`. If TUDID records lack these fields entirely, will the agent attempt to infer them based on the device type?

## 3. Search Mechanics & Vector Embeddings (Questions 21–30)

21. **Hybrid Search Structure:** Will the system rely on traditional keyword search (BM25) for regulatory numbers, vector search for descriptions, or a dense-sparse hybrid approach?
22. **Embedding Model Choice:** What embedding model will you use to handle bilingual inputs (Traditional Chinese medical terms vs. English WHO guidelines)?
23. **Chunking Strategy:** Since MeDevIS records have dense technical definitions (e.g., the definition for GMDN code 64085), how will you chunk these records for optimal vector retrieval?
24. **Search Routing:** If a user types a query in Chinese, should the agent search TUDID first, grab the English trade name/attributes, and use *that* to query MeDevIS?
25. **Metadata Filtering:** How can users explicitly filter down results using MeDevIS fields (like selecting only "Reusable" or specific "Delivery platforms") while reviewing TUDID entries?
26. **Synonym & Acronym Expansion:** How will the agent recognize that "TOR" in the TUDID description stands for "Toric" (astigmatism correction) and align it with clinical terms?
27. **Token Limit Constraints:** If a TUDID permit has hundreds of model numbers associated with it, how will the agent safely pass this data to the LLM context window without running out of tokens?
28. **Re-ranking Pipeline:** Will you use a cross-encoder model (like Cohere Rerank or BGE-Rerank) to audit the top search results before presenting them to the user?
29. **Stop-Word Strategy:** How will the agent filter out non-informative regulatory prefixes like `衛部醫器輸字第` during semantic vector indexing?
30. **Exact vs. Fuzzy Matching Routing:** Can the agent recognize when a user types an exact 14-digit Global Trade Item Number (GTIN/UDI) and completely bypass semantic search for an exact DB look-up?

## 4. User Interface, Experience & Safety (Questions 31–40)

31. **Dual-Database Visualization:** How should the UI distinguish between an absolute fact derived directly from TUDID vs. an AI-linked recommendation from MeDevIS?
32. **Language Switching:** Should the interface display the English WHO data side-by-side with localized Chinese translations generated on-the-fly by the agent?
33. **Disclaimer Handling:** The WHO dataset contains a strict nomenclature disclaimer notice. How will the agent display this to ensure regulatory compliance?
34. **Interactive Refining:** Can the user click on a MeDevIS attribute (e.g., `Specialized clinical`) to trigger a brand-new agent workflow that surfaces all corresponding TUDID devices in that category?
35. **Explainable AI (XAI):** Can the user ask the agent: *"Why did you link this specific contact lens to this WHO service type?"* and get an audible line of reasoning?
36. **Error Notification:** If the system makes an obviously incorrect cross-reference link, what UI mechanism allows medical professionals to report or flag the error?
37. **Export Capability:** Can the agent compile the combined TUDID/WHO results into an exportable standard CSV, JSON, or hospital-procurement-ready PDF format?
38. **Zero-Result Guidance:** If a search yields zero matches in both datasets, what dynamic suggestions will the agent generate to guide the user?
39. **Security & Access Control:** Does this agentic system process any patient data, or is it strictly an open-access device procurement/regulatory research tool?
40. **Accessibility Standards:** How will the agent present tabular data to meet accessibility needs (e.g., screen readers parsing complex columns from two joined datasets)?

## 5. Medical & Regulatory Domain Knowledge (Questions 41–50)

41. **Risk Classification Alignment:** TUDID operates under Taiwan's TFDA risk classes (Class I, II, III). MeDevIS maps to clinical settings. How will the agent leverage risk classes to filter out safe vs. highly regulated devices?
42. **Procurement Support Integration:** MeDevIS is designed to assist global health procurement. Can the agent use the `Healthcare settings` and `Service type` fields to help hospital administrators build a list of missing mandatory equipment?
43. **Device Reusability Tracking:** If a TUDID entry doesn't clearly state if a device is single-use or reusable, how can the agent use the MeDevIS `Reusable` field to infer or verify safety rules?
44. **Manufacturer Verification:** TUDID separates local agents from foreign manufacturers. How will the agent trace device records if the manufacturer name differs slightly between local registration and global WHO registries?
45. **Clinical Context Enrichment:** When a nurse searches for a sterile tool, can the agent fetch the exact definition from GMDN/EMDN via MeDevIS to show them precisely what the device was built to do?
46. **Nomenclature Cross-Walking:** Will your agentic system feature a static crosswalk table mapping GMDN directly to EMDN codes, or will it dynamically compute similarities?
47. **Clinical Use Limitations:** MeDevIS lists limitations based on `Life courses` (e.g., pediatric vs. adult). How will the agent warn a user if a TUDID device is being searched for an off-label demographic?
48. **Handling Regulatory Recalls:** Could this agent be extended to cross-reference search results with global or local medical device adverse event/recall databases?
49. **Standardization Alignment:** TUDID relies on GS1/HIBCC standards. Will the agent validate the checksums of these UDI codes before making API calls to external data stores?
50. **System Evolution:** As more global regulatory bodies adopt IMDRF (International Medical Device Regulators Forum) guidelines, how easy will it be to adapt your agent's schema to include datasets beyond TUDID and MeDevIS?

---

### How would you like to begin?

We can tackle these questions systematically by choosing a specific domain to focus on first.

* Would you like to start designing the **data pipeline and schema matching strategy** (Section 2), or would you prefer to map out the **agentic framework and tool architecture** (Section 1)?
