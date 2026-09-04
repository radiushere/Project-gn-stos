# Project Agnostos: Forensic Canvas
**An Information Retrieval & NLP System for Crime Data Analysis**

## 1. Abstract
Project Agnostos: Forensic Canvas is a full-stack Information Retrieval (IR) system that collects, processes, and visualizes crime and missing persons data from publicly available sources. The project demonstrates a complete, end-to-end IR pipeline from raw, unstructured web content through to a richly interactive visual interface.

The system was built in four stages: a focused web crawler harvests approximately 1,500 cases; a Natural Language Processing engine extracts structured forensic entities from raw narrative text; a RESTful API serves the structured data; and a React.js-powered detective evidence board visualizes relationships between all extracted entities.

Key IR contributions include wrapper-based information extraction, Named Entity Recognition (NER) with role classification, knowledge graph construction, and interactive graph-based document visualization. The system transforms unstructured case narratives into a queryable, navigable forensic knowledge base.

## 2. Introduction

### 2.1 Background & Motivation
Crime data and missing persons records are scattered across multiple online sources in largely unstructured textual form. Investigators and researchers face significant manual effort when trying to locate patterns, identify relationships between entities, and cross-reference cases across different data repositories. No single tool currently brings this information together in a structured, visually navigable format.

Project Agnostos was conceived to address this gap building an automated pipeline that collects, understands, and presents this data in a way that reveals the connections hidden within it.

### 2.2 Project Objectives
The system was designed to achieve the following objectives:
* **Data Collection** - Harvest raw crime and missing persons case data from the FBI's public REST API and The Charley Project missing persons archive via targeted web scraping.
* **NLP Processing** - Apply Named Entity Recognition and rule-based classification to extract structured forensic entities people, locations, vehicles, evidence, and digital footprints from unstructured text.
* **Schema Transformation** - Normalize all extracted data into a unified 13-field forensic schema, merging heterogeneous source formats into a single master database.
* **Retrieval & Visualization** - Serve the structured data through a RESTful API and render it as an interactive, graph-based detective evidence board with physics-simulated entity relationships.

## 3. Literature Review
Information Retrieval research has long explored the challenge of transforming unstructured web content into structured, queryable knowledge. Wrapper induction techniques (Kushmerick, 1997) formalize the extraction of structured data from HTML templates a principle directly applied in this project's web collection layer, which exploits known DOM structure to extract semi-structured data from targeted web pages.

Named Entity Recognition (NER) has evolved from purely rule-based systems to statistical and neural models. The spaCy NLP framework uses convolutional neural networks trained on OntoNotes 5 to detect entities such as PERSON, GPE (geopolitical entities), DATE, and ORG from general English text with high accuracy.

Knowledge Graphs (Ehrlinger & Wöß, 2016) represent entities and their typed relationships as graph structures, enabling significantly richer retrieval than flat document indexes. This project's connection graph implements a domain-specific knowledge graph over forensic entities. Prior work in investigative analytics (Stasko et al., 2008) demonstrated that spatial node-link diagrams improve an analyst's ability to identify co-occurrences and relationships across cases a finding that directly motivated the project's visual design.

COPLINK (Chen et al., 2003) demonstrated the value of structured criminal justice databases for law enforcement analysis. Project Agnostos extends this spirit with modern NLP techniques and a web-based interactive visualization layer accessible without specialized tooling.

## 4. System Architecture
The system is organized as a four-stage linear pipeline. Each stage produces a well-defined output artifact consumed by the next, with clear separation of concerns between data acquisition, processing, serving, and presentation.

| Stage | Name | Output Artifact |
|---|---|---|
| **Stage 1** | Data Collection | Two raw JSON datasets (~1.8 MB combined) |
| **Stage 2** | NLP Processing | Master forensic database (9.5 MB, 13 fields/case) |
| **Stage 3** | Backend API | RESTful endpoints serving structured case & graph data |
| **Stage 4** | Frontend Visualization | Interactive detective evidence board (React + React Flow) |

### 4.1 Data Flow Overview
* **Collection** → Two independent scrapers run in parallel, targeting the FBI REST API (500 cases) and The Charley Project HTML pages (1,000 cases). Both output raw JSON files.
* **Processing** → A Google Colab NLP notebook processes both raw files through the full extraction pipeline. The resulting forensic databases are merged into a single master database.
* **Serving** → A FastAPI backend loads the master database and exposes three endpoints: Case index data, graph data for individual cases, and an image proxy for CORS-restricted portrait images.
* **Visualization** → The React frontend fetches graph data per case and renders it as an interactive node-edge canvas with physics-simulated connections, draggable tools, and a sliding case navigation drawer.

## 5. Module Descriptions

### 5.1 Module 1 - Data Collection
**FBI Most Wanted Scraper**
The first collection component targets the FBI's public REST API. It paginates through up to 500 most-wanted cases, handling rate limiting with a built-in delay between requests. For each case, it extracts the case identifier, title, full narrative details, remarks, warning messages, distinguishing marks, known aliases, location references, and the highest-resolution portrait image available.

*Key features of the FBI scraper:*
* Pagination through up to 500 cases across multiple API pages
* Built-in request delays to respect rate limits and avoid blocking
* Automatic extraction of the highest-resolution portrait image URL per case
* Output stored in a structured raw JSON format ready for NLP processing

**Charley Project Spider**
The second collection component is a focused web crawler targeting The Charley Project, a comprehensive missing persons archive. It discovers individual case URLs by paginating through the site's archive index, then visits each case page individually.

*Data extracted per case:*
* Case title and subject name
* Portrait image from the case photo section
* Demographic bullet points: age, sex, race, last known location, date of disappearance
* Full narrative text from the "Details of Disappearance" section

*Reliability features:*
* Browser-mimicking request headers to prevent server-side blocking
* Randomized delays between requests to simulate human browsing patterns
* Incremental progress saving every 10 cases for crash recovery

> *IR Relevance:* These components implement document acquisition and focused web crawling - the first stage of any IR pipeline. The HTML-based extractor applies the wrapper induction principle, exploiting known DOM structure to reliably extract semi-structured data from otherwise unstructured web pages.

### 5.2 Module 2 - NLP Processing & Information Extraction
The NLP module is the core information extraction engine of the system. It runs in Google Colab (T4 GPU environment) and processes both raw datasets into a unified 13-field forensic schema using spaCy's NER pipeline and several layers of rule-based post-processing.

#### The 13-Field Forensic Schema
Every processed case is stored as a structured object with the following 13 fields:

| Field | Description |
|---|---|
| **Case Core Metadata** | Case ID, title, status, severity classification, source agency |
| **Temporal Data** | Incident date, estimated time window, last known contact date |
| **Geospatial Data** | Primary location name, coordinates, secondary location references |
| **Victim Profiles** | Name, demographics, current status, portrait image URL |
| **Suspects & POIs** | Name, legal status, known associates, alibi references, image URL |
| **Witnesses & Informants** | Name, relationship to case, summarized statement |
| **Vehicles Involved** | Make, model, color, plate reference, distinguishing features |
| **Physical & Forensic Evidence** | Category, description, collection location |
| **Digital & Financial Footprints** | Type (phone/ATM/card), description, associated timestamp |
| **Investigative Theories** | Theory title, description, supporting evidence references |
| **Legal & Court Data** | Jurisdiction, charges filed, case verdict or status |
| **Red String Connections** | Source entity, target entity, relationship label, edge color |
| **IR Provenance & Source Metrics** | Origin URL, source platform, retrieval quality scores |

#### NLP Processing Pipeline
The pipeline processes each case through five sequential steps:
1. **Step 1 - Named Entity Recognition:** spaCy's pre-trained NER model is applied to the case narrative to detect PERSON, GPE (location), DATE, and ORG entities. A 35-character length guard prevents sentence fragments from being misidentified as person names, and an exclusion list filters out known organizational keywords.
2. **Step 2 - Smart Naming Router:** Each extracted person is classified as a Victim, Suspect, or Witness based on the sentence context in which their name appears. The classifier scans surrounding sentences for role-indicating keywords (e.g., *arrested, charged* trigger Suspect; *witnessed, stated* trigger Witness).
3. **Step 3 - Victim Deduplication:** To prevent the primary case subject from appearing in multiple role categories, the system performs substring matching between each extracted name and the case title.
4. **Step 4 - Evidence, Vehicle & Digital Footprint Extraction:** Physical features and distinguishing marks are parsed from dedicated case fields. Vehicles and digital footprints are detected via keyword matching across terminology (e.g. *cell phone, ATM, credit card*).
5. **Step 5 - Red String Connection Graph Generation:** After all entities are extracted, the system automatically generates typed relationship edges connecting each entity back to the primary subject.

#### Edge Color Coding (Red String Connections)
| Relationship Type | Color |
|---|---|
| Suspect → Victim (Investigated POI) | Red |
| Witness → Victim (Witness Statement) | Yellow |
| Vehicle → Victim (Associated Vehicle) | Gray |
| Digital Footprint → Victim | Blue |
| Physical Evidence → Victim | Green |
| Location → Victim (Last Known Location) | Stone / Warm Gray |

> *IR Relevance:* This module implements the full information extraction stack - NER for entity detection, relation extraction via the Smart Naming Router, schema-based document indexing, and knowledge graph construction. Together, these convert unstructured crime narratives into structured, retrievable, and traversable knowledge.

### 5.3 Module 3 - Backend API & Retrieval Layer
The backend API acts as the retrieval engine of the system. It loads the master forensic database and exposes three REST endpoints that the frontend consumes:

* **Case Index Endpoint:** Returns a lightweight list of all cases in the database, including case ID, title, and primary location. This powers the case navigation drawer in the frontend.
* **Graph Retrieval Endpoint:** Accepts a case ID and returns a fully computed React Flow-compatible node-edge graph for that case. The backend transforms the 13-field schema into spatially positioned nodes (victims anchor the center, suspects appear adjacent, witnesses sit below, etc.).
* **Image Proxy Endpoint:** An asynchronous proxy that relays FBI portrait images through the backend to bypass CORS restrictions. A transparent 1x1 PNG fallback prevents broken image icons in the UI.

> *IR Relevance:* The backend implements structured retrieval by ID, a document catalog for browsing, and a graph transformation layer that converts stored document records into graph-based knowledge representations.

### 5.4 Module 4 - Frontend: Interactive Evidence Board
The frontend is built with React 19, Vite, React Flow, and TailwindCSS. It renders the retrieved graph data as an immersive detective evidence board: a cork-board-textured canvas where entities appear as physical evidence artifacts connected by physics-simulated yarn strings.

**Evidence Node Types**
* **Persons (Victim / Suspect):** Polaroid-style photo card with a grayscale portrait, thumbtack pin, and case label.
* **Witnesses:** Yellow sticky note with handwritten-style font and relationship tag.
* **Physical Evidence:** Newspaper clipping style with masking tape accent and evidence category label.
* **Location:** Purple-bordered card with a map pin icon and last-known-location label.
* **Digital Footprints:** Dark navy background card with telemetry-style typography.
* **Vehicles:** Gray-toned card with a car icon and vehicle details.

**Interactive Features**
* **Red String Connections:** Physics-simulated bezier curves that sag under simulated gravity to mimic real yarn or string.
* **Case File Drawer:** A sliding filing cabinet drawer containing all cases with search and alphabetical tab filtering.
* **Desk Lamp:** A draggable desk lamp with an elastic SVG power cable that casts a dynamic conic-gradient spotlight.
* **Magnifying Glass:** A draggable 1.5x zoom tool that clones the underlying board content and applies a magnification transform.
* **Sticky Notes & Props:** Peelable sticky notes that can be placed and edited on the board, alongside decorative investigator props.
* **MiniMap:** A color-coded overview map of the full graph layout for navigation.

## 6. IR Concepts Demonstrated
The following table maps core Information Retrieval concepts to their concrete implementations within the system:

| IR Concept | Implementation in Project |
|---|---|
| **Document Acquisition** | Web scraping (Charley spider) + API consumption (FBI Most Wanted) |
| **Focused Web Crawling** | Pagination with rate limiting and anti-blocking request headers |
| **Information Extraction** | spaCy NER applied to unstructured case narrative text |
| **Named Entity Recognition** | PERSON, GPE, DATE, ORG detection with a hallucination guard |
| **Relation Extraction** | Smart Naming Router classifying persons via sentence context analysis |
| **Document Indexing** | 13-field forensic schema as structured document representation |
| **Structured Retrieval** | Case lookup by ID via dedicated REST endpoints |
| **Knowledge Graph** | Typed entity relationship graph (red string connections) |
| **Query Interface** | Case file drawer with text search and alphabetical filtering |
| **Deduplication** | Substring name matching to prevent victim duplication across roles |
| **Data Fusion** | Merging heterogeneous FBI and Charley sources into one schema |

## 7. Results & Evaluation

### 7.1 Quantitative Results
| Metric | Value |
|---|---|
| **Total cases collected** | ~1,500 (1,000 missing persons + 500 FBI Most Wanted) |
| **Raw data volume** | ~1.8 MB (combined from both sources) |
| **Processed database size** | 9.5 MB (master forensic database) |
| **Schema fields per case** | 13 structured fields |
| **Entity types extracted** | 6 types: persons, locations, vehicles, evidence, digital, witnesses |
| **Graph edge color types** | 6 color-coded relationship types |
| **Backend endpoints** | 3 (case index, graph by ID, image proxy) |
| **Frontend components** | 8+ including board, nodes, edges, drawer, lamp, magnifier, minimap |

### 7.2 Screenshots
<img width="1429" height="797" alt="image" src="https://github.com/user-attachments/assets/a899395a-c746-44e9-b0de-ed677ed30c8f" />
<img width="1433" height="787" alt="image" src="https://github.com/user-attachments/assets/2f02db07-d68b-4dcf-a901-23ddcf6ca509" />
<img width="1430" height="804" alt="image" src="https://github.com/user-attachments/assets/fbf9ef58-e728-4e71-8ca1-311fb2c0076a" />
<img width="1434" height="793" alt="image" src="https://github.com/user-attachments/assets/c7bbe316-13bc-4245-a225-7a461277046c" />
<img width="1010" height="847" alt="image" src="https://github.com/user-attachments/assets/b54f0632-2a76-4e0d-9d74-1c2d53a242d3" />
<img width="941" height="838" alt="image" src="https://github.com/user-attachments/assets/87c8d23d-b93d-4e57-8ab4-fb4d9c133558" />
<img width="1469" height="836" alt="image" src="https://github.com/user-attachments/assets/4763e1fa-d599-4da0-9824-e17d4dc4bfd8" />



## 8. Future Work
Several enhancements are planned or recommended for future iterations of the system:
1. **BM25/TF-IDF Full-Text Search:** Add ranked full-text search across all case narratives, allowing investigators to find relevant cases by keyword with relevance scoring.
2. **Vector Similarity Search:** Embed case descriptions using sentence transformers to enable semantic similarity search finding cases that are conceptually related even without shared keywords.
3. **Geospatial Mapping:** Plot extracted location coordinates on an interactive geographic map to reveal spatial clustering patterns and hotspot analysis.
4. **Timeline View:** A chronological visualization of case events derived from the temporal data field.
5. **Cross-Case Analysis:** Automated detection of shared suspects, locations, or behavioral patterns linking multiple cases.
6. **Exportable Case Reports:** A feature to generate printable PDF case summaries from the current evidence board state.

## 9. Conclusion
Project Agnostos: Forensic Canvas successfully demonstrates a complete, end-to-end Information Retrieval pipeline applied to a real-world domain. Starting from raw, unstructured web content, the system automatically acquires, processes, indexes, and visualizes approximately 1,500 cases through a well-defined chain of IR stages.

The project bridges academic IR concepts information extraction, knowledge graphs, and structured retrieval with a polished, immersive user interface that makes the underlying data genuinely accessible and explorable. The detective evidence board metaphor, with its physics-simulated red strings, draggable tools, and dynamic lighting, transforms raw forensic data into a navigable spatial knowledge base.

The system demonstrates that modern IR techniques NER, knowledge graphs, and graph-based retrieval can be meaningfully combined with thoughtful interface design to create tools that are both technically rigorous and practically useful.

> *"In a field where information is scattered, unstructured, and hard to navigate, a well-designed IR system does not merely store data - it reveals the connections hidden within it."*

## 10. References
1. Chen, H., Zeng, D., Atabakhsh, H., Wyzga, W., & Schroeder, J. (2003). COPLINK: Managing law enforcement data and knowledge. Communications of the ACM, 46(1), 28-34.
2. Ehrlinger, L., & Wöß, W. (2016). Towards a definition of knowledge graphs. SEMANTICS (Posters, Demos, SUCCESS), Klagenfurt, Austria.
3. Honnibal, M., & Montani, I. (2017). spaCy 2: Natural language understanding with Bloom embeddings, convolutional neural networks and incremental parsing. To appear.
4. Kushmerick, N. (1997). Wrapper induction for information extraction. PhD Thesis, University of Washington.
5. Stasko, J., Görg, C., & Liu, Z. (2008). Jigsaw: Supporting investigative analysis through interactive visualization. IEEE Transactions on Visualization and Computer Graphics, 14(6), 1576-1583.
6. FBI Most Wanted API. Federal Bureau of Investigation. https://api.fbi.gov/wanted/v1/list
7. The Charley Project. Missing Persons Archive. https://charleyproject.org

**Contributors**

| <a href="https://github.com/Najaf-Ali-Imran"><img src="https://github.com/Najaf-Ali-Imran.png" width="50" height="50" alt="Najaf-Ali-Imran"/></a> | <a href="https://github.com/Musamehar"><img src="https://github.com/Musamehar.png" width="50" height="50" alt="Musamehar"/></a> |
| :---: | :---: |
| [@Najaf-Ali-Imran](https://github.com/Najaf-Ali-Imran) | [@Musamehar](https://github.com/Musamehar) |
