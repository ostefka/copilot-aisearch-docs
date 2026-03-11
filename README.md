# Intelligent Document Search with Azure AI Search & M365 Copilot

> **Enterprise knowledge retrieval that you can tune — unlike SharePoint's out-of-the-box search.**

---

## The Challenge

Organizations store critical knowledge in documents — product specifications, pricing sheets, internal processes, methodologies. Employees need fast, accurate answers from this content through natural language. 

**SharePoint Online search** is a fully managed service. It works — but it works *as-is*. There is no way to influence how documents are chunked, how search relevance is scored, what content is extracted from images and tables, or how queries are interpreted. When it doesn't find the right answer, there is **no lever to pull**.

**Our solution** gives you full control over every stage of the pipeline — from how documents are ingested and processed to how search queries are understood and results are ranked. When something doesn't work, it **can be fixed**.

---

## What We Built

An end-to-end system that connects **M365 Copilot declarative agents** to enterprise documents through **Azure AI Search**, with full control over ingestion quality and search accuracy.

```
┌──────────────┐                    ┌──────────────┐                    ┌──────────────────┐
│   DOCUMENTS  │                    │  INTELLIGENT │                    │  M365 COPILOT    │
│              │  ── Ingestion ──▶  │    SEARCH    │  ◀── Queries ───  │  DECLARATIVE     │
│ Word, PDF,   │     Pipeline       │    INDEX     │     via OpenAPI   │  AGENT           │
│ PowerPoint   │                    │              │     Plugin        │                  │
│              │                    │  Azure AI    │                    │  Natural language │
│ SharePoint / │                    │  Search      │                    │  answers with     │
│ Local        │                    │              │                    │  citations        │
└──────────────┘                    └──────────────┘                    └──────────────────┘
```

### Three key layers:

| Layer | What It Does | Why It Matters |
|---|---|---|
| **Ingestion Pipeline** | Extracts, processes, and indexes document content | Garbage in → garbage out. Quality ingestion is the foundation |
| **Azure AI Search** | Stores and retrieves documents using semantic + hybrid search | Fine-tunable relevance, vector search, per-user security |
| **M365 Copilot Integration** | Connects the index to a declarative agent via OpenAPI plugin | Users get answers in natural language, not search results |

---

## Layer 1: Ingestion Pipeline — Why It's the Most Important Step

> *The quality of search results is determined **before the first query is ever asked.***

Most projects focus on the search query side. But if documents aren't processed correctly — if tables are garbled, images are ignored, content is chunked in the wrong places — no amount of query tuning will fix it.

### What our pipeline does

| Capability | Description |
|---|---|
| **Azure Document Intelligence** | Uses the `prebuilt-layout` model to extract structured content from Word, PDF, and PowerPoint — including tables, headers, and reading order |
| **Table preservation** | Tables are extracted as Markdown — columns and rows stay intact, not flattened into unreadable text |
| **Image description (GPT vision)** | Embedded images — screenshots, flowcharts, pricing tables captured as images — are described by GPT vision models in the document's language |
| **Smart chunking** | Content is split into semantically meaningful sections by heading structure, with configurable overlap to preserve cross-section context |
| **Vector embeddings** | Each chunk gets a 3072-dimensional embedding (`text-embedding-3-large`) for semantic similarity search |
| **Metadata enrichment** | Category, source document, section title, and document provenance are preserved with each chunk |
| **Multi-format extraction** | Native image extraction from DOCX (including EMF/WMF conversion), PDF (PyMuPDF), and PPTX (python-pptx) |

### Why this matters — a real example

A customer's product pricing methodology contained critical information **only in embedded images** — time zone pricing tables, tariff percentage charts. SharePoint search would index the surrounding text but completely ignore the image content.

Our pipeline:
1. Extracts each embedded image
2. Sends it to GPT vision with a domain-specific prompt
3. Converts the visual content into searchable text
4. Indexes it alongside the document text

**Result:** Employees can ask *"What discount applies in the early morning time zone?"* and get the answer — even though that information only existed as a screenshot of a table.

### Quality controls

- **Hallucination filtering** — Vision model responses are post-processed to detect and remove fabricated "no image" responses
- **SKIP detection** — Truly unreadable images (decorative borders, blank placeholders) are identified and excluded
- **Automatic format conversion** — Legacy formats (EMF, WMF) are converted to PNG before vision processing
- **Per-chunk validation** — Minimum token thresholds prevent noise chunks from polluting the index

---

## Layer 2: Azure AI Search — Tunable Relevance

Unlike SharePoint's managed search, every aspect of Azure AI Search can be configured:

| Capability | What You Control | Impact |
|---|---|---|
| **Semantic ranking** | AI-powered reranking of results by meaning, not just keywords | Understands that "discount" and "price reduction" mean the same thing |
| **Hybrid search** | Combine keyword match + vector similarity + semantic ranking | Best of all three approaches — highest accuracy |
| **Language analyzers** | Language-specific text analysis (Czech, English, etc.) | Proper handling of declension, conjugation, compound words |
| **Scoring profiles** | Boost recent documents, specific categories, or metadata fields | Business-relevant results ranked higher |
| **Vector search** | 3072-dimensional embeddings for meaning-based retrieval | Find relevant content even when no keywords match |
| **Query rewriting** | AI-powered query expansion with synonyms, acronyms, and bilingual terms | "solar energy" also finds "fotovoltaika" and "solární panely" |
| **Per-user security** | RBAC integration with Entra ID — users only see what they're authorized to | Enterprise-grade document security at the index level |

### SharePoint Search vs. Azure AI Search

| Aspect | SharePoint Online Search | Azure AI Search (Our Solution) |
|---|---|---|
| **Chunking strategy** | Managed, no control | Configurable — by heading, token count, overlap |
| **Image content** | Not indexed | GPT vision extraction, fully searchable |
| **Table handling** | Flattened text | Markdown tables — structure preserved |
| **Relevance tuning** | None — works as-is | Scoring profiles, semantic config, boosting |
| **Language support** | Basic | Dedicated analyzers (e.g., `cs.lucene` for Czech) |
| **Vector search** | Not available | Full vector + hybrid search |
| **Query optimization** | None | AI query rewriting with synonym/acronym expansion |
| **Security model** | SharePoint permissions (good) | Entra ID RBAC — per-index or per-document |
| **Metadata control** | Limited | Custom fields, facets, filters |
| **When it fails** | No recourse | Full diagnostics, tunable at every stage |

---

## Layer 3: M365 Copilot Integration — OpenAPI Plugin

The search index is exposed to **M365 Copilot** through a **declarative agent** with an **OpenAPI plugin**. The agent calls standard REST endpoints — no custom protocol needed.

### REST API Endpoints

| Endpoint | Purpose |
|---|---|
| `POST /api/search` | Keyword, semantic, or hybrid search with filters |
| `POST /api/smart-search` | AI-powered multi-query search with query rewriting |
| `GET /api/documents/{id}` | Retrieve a specific document by ID |
| `GET /api/categories` | List available document categories |
| `GET /api/openapi.yaml` | Self-describing API spec for plugin registration |

### How it works

1. The **declarative agent** is registered in M365 Copilot with an OpenAPI plugin pointing to the REST API
2. When a user asks a question, M365 Copilot invokes the appropriate API endpoint
3. The API runs the search against Azure AI Search using the user's identity (On-Behalf-Of flow)
4. Results are returned with citations — the agent synthesizes a natural language answer

### Why OpenAPI plugin

| Advantage | Detail |
|---|---|
| **Standard REST** | No custom protocols — any HTTP client can call the API |
| **Self-describing** | OpenAPI spec at `/api/openapi.yaml` — the agent auto-discovers available operations |
| **Per-user security** | OAuth 2.0 + On-Behalf-Of — each query runs with the user's own permissions |
| **Shared backend** | Same search service, same logic — MCP support can be added later if needed |

---

## Architecture Overview

```
    ┌─────────────────────┐
    │  M365 Copilot        │
    │  Declarative Agent   │
    │  (OpenAPI Plugin)    │
    └──────────┬──────────┘
               │ POST /api/*
               │ (REST + OAuth)
               ▼
    ┌────────────────────────┐
    │     APIM Gateway       │
    │  OAuth + IP Filtering  │
    └──────────┬─────────────┘
               │
               ▼
    ┌─────────────────────────────────────┐
    │        Container App (Node.js)       │
    │                                      │
    │         REST API (OpenAPI)            │
    │              │                        │
    │        SearchService                  │
    │    (search, smart-search, etc.)       │
    └──────────────────┬───────────────────┘
                       │ On-Behalf-Of
                       │ (per-user token)
                       ▼
              ┌─────────────────┐
              │  Azure AI Search │
              │  (RBAC, no keys) │
              └─────────────────┘
```

**Key design decisions:**
- **APIM offloads OAuth** — the server never validates tokens itself; APIM does it and forwards trusted headers
- **On-Behalf-Of flow** — each user's query runs with their own permissions; the system doesn't use a service account
- **No API keys** — Azure AI Search is accessed exclusively via Managed Identity + RBAC
- **OpenAPI-first** — standard REST API with self-describing spec; declarative agent registers it as a plugin

---

## Structured Data: Dataverse as a Complementary Solution

Not all enterprise data lives in documents. Product catalogs, pricing tables, FAQ databases, and CRM records are **structured data** — rows and columns, not paragraphs.

For structured data, **Dataverse** (the Power Platform data store) provides a complementary search path:

| Approach | Best For | Search Engine |
|---|---|---|
| **Azure AI Search** (this solution) | Unstructured documents — Word, PDF, PowerPoint | Dedicated AI Search with full tuning |
| **Dataverse Tables + Copilot Knowledge** | Structured data — product catalogs, FAQs, pricing | Dataverse Search (Azure Search under the hood) |
| **Combined** | Both document knowledge and structured data | Agent uses both sources based on the question |

### How Dataverse fits in

| Capability | Detail |
|---|---|
| **Store structured data** | Create tables for products, FAQs, processes — proper columns and data types |
| **Search** | Dataverse Search (powered by Azure Search) indexes all searchable columns automatically |
| **Copilot integration** | Add Dataverse tables as knowledge sources — agent can query them natively |
| **Security** | Dataverse security roles, business units, row-level security |
| **No custom code** | Data import via Excel, Power Apps forms; search is built-in |

### When to combine both approaches

A **Copilot agent** can connect to **multiple knowledge sources simultaneously**:

- *"What products do we offer?"* → Dataverse product catalog (structured)
- *"What's the process for changing a customer's product?"* → Azure AI Search documents (unstructured)
- *"What's the pricing for product X including the latest methodology changes?"* → Both sources

This gives the agent comprehensive coverage — structured data for lookups and precise answers, document search for detailed methodologies and policies.

---

## What Makes This Different

| | SharePoint Search | Our Solution |
|---|---|---|
| **Control** | None — fully managed black box | Full control at every stage |
| **Image content** | Ignored | Extracted and searchable |
| **Tables** | Garbled | Preserved as structured data |
| **Query understanding** | Basic keyword matching | AI rewriting + synonym expansion + bilingual |
| **When it fails** | Accept it | Diagnose and fix it |
| **Relevance tuning** | Not possible | Scoring profiles, semantic config, boosting |
| **Integration** | SharePoint-only | OpenAPI plugin — any AI agent |
| **Structured data** | N/A | Dataverse for tables + catalogs |

### The bottom line

**SharePoint search is a managed convenience.** It works for basic document retrieval across an organization.

**Our solution is a tunable platform.** Every failure point can be diagnosed, every parameter can be adjusted, every edge case can be addressed. When an employee asks a question and doesn't get the right answer, we can trace exactly why — was it a chunking issue? A missing image description? A keyword mismatch? — and fix it.

For organizations where **knowledge retrieval accuracy matters**, this controllability is the difference between a tool employees trust and one they abandon.

---

## Validated Results

The solution was validated against real enterprise documents — 23 internal documents across 3 categories (product catalogs, pricing methodologies, operational processes), producing 248 indexed chunks.

**40 test queries** were designed covering:
- Product catalog lookups
- Cross-category comparisons  
- Process knowledge questions
- Dynamic pricing methodology
- Structured data extraction (from images)
- Semantic paraphrase queries (testing meaning understanding, not keyword matching)

**Result: 40/40 queries returned correct, relevant answers — 100% pass rate.**

Key challenges that were solved during validation:
- Czech language declension (word form changes) → language-specific analyzers + word stem matching
- Content locked inside images → GPT vision extraction
- Unicode character handling → proper encoding throughout the pipeline
- Cross-document synthesis → multi-query search with AI query rewriting

---

## Technology Stack

| Component | Service | Purpose |
|---|---|---|
| **Ingestion** | Azure Document Intelligence | Extract structured content from documents |
| **Vision** | Azure OpenAI (GPT vision) | Describe embedded images in documents |
| **Embeddings** | Azure OpenAI (text-embedding-3-large) | 3072-dim vectors for semantic search |
| **Search** | Azure AI Search | Hybrid search, semantic ranking, RBAC |
| **Gateway** | Azure API Management | OAuth validation, IP filtering, routing |
| **Compute** | Azure Container Apps | Host the REST API server |
| **Auth** | Microsoft Entra ID | OAuth 2.0, On-Behalf-Of, per-user RBAC |
| **Agent** | M365 Copilot (Declarative Agent) | Conversational AI interface for end users |
| **Structured data** | Dataverse (optional) | Product catalogs, FAQs, structured knowledge |

---

*Built with Azure AI services and M365 Copilot. For technical implementation details, contact the project team.*
