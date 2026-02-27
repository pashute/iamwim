# iamWim · Feature & Technology Specification

> Living document. Sections: Technology · Development · Components · Visualizer
> Table columns: # · Name · Details · Open Questions · Done In (milestone/sprint)

---

## 1. Technology

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 1.1 | **Runtime** | Node.js (LTS) — event-driven, non-blocking, native JS ecosystem | Version pinning strategy? `.nvmrc`? | M0 |
| 1.2 | **Symbolic Logic Engine** | [Tau-Prolog](http://tau-prolog.org/) — full Prolog interpreter in JS. First resort before LLM. Handles unification, backtracking, rule firing | Can it handle graph-scale queries without perf hit? | M1 |
| 1.3 | **Forward Chaining / Rules** | [Nools](https://github.com/noolsjs/nools) — Rete-algorithm rule engine in JS. Fires rules against working memory facts | Still maintained? Evaluate vs. hand-rolled Rete | M1 |
| 1.4 | **Fallback: Local LLM** | [Ollama](https://ollama.com/) — run small models locally (Mistral, Phi-3, Gemma). Called only via `consult()` gate in orchestrator | Which model fits hardware constraints? Quantization level? | M2 |
| 1.5 | **Graph Database** | [Memgraph](https://memgraph.com/) — in-memory graph DB, Cypher query language, fast traversal. Primary persistence for all lexicon graphs | Memgraph vs. Neo4j for local dev? License? | M1 |
| 1.6 | **Relational / Meta Store** | [Supabase](https://supabase.com/) (Postgres) — session metadata, user trust levels, discussion thread index, audit log | Self-hosted or cloud? Row-level security needed? | M1 |
| 1.7 | **RDF / Ontology Layer** | [N3.js](https://github.com/rdfjs/N3.js) — RDF/Turtle/N-Quads parser and reasoner. For importing/exporting structured ontologies into lexicon graphs | Do we need full OWL reasoning or just RDF triples? | M2 |
| 1.8 | **Huffman / Priority Trees** | Custom JS implementation — min-heap or priority queue over topic nodes within each lexicon. Candidate lib: [mnemonist](https://yomguithereal.github.io/mnemonist/) | Build custom or wrap mnemonist heap? | M2 |
| 1.9 | **Inter-process / Messaging** | [BullMQ](https://bullmq.io/) (Redis-backed) — async job queue for background lexicon building, topic loading/unloading, LLM consult calls | Is Redis a dependency too far for local dev? | M2 |
| 1.10 | **Config** | [HOCON](https://github.com/lightbend/config) via `node-hocon` or plain JSON/YAML for system config. HOCON preferred for readability | Standardise on HOCON throughout or only for output? | M0 |

---

## 2. Development

### 2.1 Methodology

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 2.1.1 | **DDD / BDD** | Domain-Driven Design for model boundaries (Lexicon, Topic, Entity, Link, Orchestrator are bounded contexts). BDD with [Cucumber.js](https://cucumber.io/docs/installation/javascript/) + Gherkin scenarios for behaviour specs | Who writes the Gherkin — dev or trusted human collaborator? | M0 |
| 2.1.2 | **TDD** | [Jest](https://jestjs.io/) — unit and integration tests. Test-first on all orchestrator rules and lexicon operations | Coverage threshold? 80%? 90%? | M0 |
| 2.1.3 | **Gitflow** | `main` → `develop` → `feature/*` / `fix/*` / `spike/*`. PRs required into develop. Tags for milestones | Solo dev flow — do we need full PR discipline or lighter? | M0 |
| 2.1.4 | **Semantic Versioning** | `MAJOR.MINOR.PATCH` — MAJOR for breaking schema changes to lexicon graph model | Automate with `semantic-release`? | M1 |

### 2.2 Language & Frameworks

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 2.2.1 | **Language** | JavaScript (ES2022+) — async/await, optional chaining, structuredClone. No TypeScript initially to keep friction low, revisit at M2 | Add TypeScript at M2 for graph schema safety? | M0 |
| 2.2.2 | **Package Manager** | [pnpm](https://pnpm.io/) — fast, disk-efficient, workspace support for monorepo | Monorepo from day one or split later? | M0 |
| 2.2.3 | **Logger** | [Winston](https://github.com/winstonjs/winston) — levelled logging, transports for file + console. Structured JSON logs for machine-readability | Log to Supabase audit table for human review? | M0 |
| 2.2.4 | **HTTP / API Layer** | [Fastify](https://fastify.dev/) — fast, schema-validated REST API. Exposes orchestrator, lexicon queries, and dashboard data | GraphQL instead of REST for graph queries? | M1 |
| 2.2.5 | **Environment / Secrets** | `dotenv` + `.env.local` — never committed. Supabase keys, Ollama endpoint, Memgraph credentials | Secret rotation strategy for multi-user? | M0 |
| 2.2.6 | **Linting / Format** | ESLint + Prettier. Enforced pre-commit via `husky` + `lint-staged` | Agree on style rules upfront | M0 |

### 2.3 Persistence

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 2.3.1 | **Lexicon Graphs** | Memgraph — all entity nodes, link layers, topic graphs, weights and attributes stored as native graph. Cypher for traversal and mutation | Schema constraints in Memgraph? Node labels strategy? | M1 |
| 2.3.2 | **Discussion Threads** | Supabase — thread index, subject titles, session metadata, accumulated request/response pairs (as references to graph nodes, not raw text) | How deep does Supabase go vs Memgraph for discussion state? | M1 |
| 2.3.3 | **Jotted Notes** | Lightweight JSON files or Supabase table — fast-write, low-structure, promotable to full lexicon graph later | File-based vs DB — depends on volume | M1 |
| 2.3.4 | **Audit / Trust Log** | Supabase table — every human-in-the-loop action logged with timestamp, trusted-user ID, action type, affected node/link | Needed from day one for integrity | M0 |

### 2.4 Performance & Benchmarking

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 2.4.1 | **Benchmarking** | [Clinic.js](https://clinicjs.org/) — Node.js performance profiling (flame graphs, bubbleprof, doctor) | Run on every milestone? | M1 |
| 2.4.2 | **Load Testing** | [autocannon](https://github.com/mcollina/autocannon) — HTTP load testing for API layer | Define baseline targets before M1 | M1 |
| 2.4.3 | **Graph Query Perf** | Memgraph's built-in query analyser + EXPLAIN/PROFILE on Cypher. Huffman tree effectiveness measured as mean hops to target node | What's acceptable mean hop count? | M2 |
| 2.4.4 | **LLM Latency Budget** | Track time-to-response for `consult()` calls. Target: LLM should never block the orchestrator thread | Use BullMQ to make all LLM calls async | M2 |

---

## 3. Components

### 3.1 Orchestrator

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 3.1.1 | **Rule Engine** | Nools or Tau-Prolog rules fire against the active lexicon graph's working memory. No LLM in the loop unless `consult()` explicitly called | Rule definition format — Prolog clauses or Nools DSL? | M1 |
| 3.1.2 | **Working Context** | Loaded topic subgraphs held in memory. Orchestrator operates only on what is loaded — sparse by design | Max loaded topic size? Memory ceiling? | M1 |
| 3.1.3 | **consult() Gate** | Single function that calls local LLM. Output is returned as raw text, then parsed and optionally integrated into lexicon as attributed nodes | Trust level for LLM-sourced nodes vs human-sourced? | M2 |
| 3.1.4 | **Self-Awareness Loop** | Orchestrator maintains a self-lexicon — a graph of its own state, loaded topics, recent decisions, and confidence levels | How detailed is the self-model at M1 vs M3? | M3 |

### 3.2 Lexicon Builder

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 3.2.1 | **On-the-Fly Construction** | Builds a new lexicon graph from a discussion session, document import, or human instruction. Populates entity layer and link layer incrementally | Batch vs streaming build? | M1 |
| 3.2.2 | **Topic Analyzer** | Identifies topics within incoming content. Assigns nodes to topic subgraphs within the lexicon. Detects topic overlap with existing lexicons | NLP-free (pattern/rule-based) first, NLP later? | M1 |
| 3.2.3 | **Tester / Quality Gate** | Evaluates new nodes and links on: coherence, relevance, redundancy, certainty, correctness, info source, source type. Assigns confidence weights accordingly | Scoring rubric — numeric 0–1 or categorical? | M2 |
| 3.2.4 | **Link Weighter** | Assigns signed numeric weights and attributes to bidirectional links. Negative weights for contradictions, low-trust relations, deprecated paths | Weight decay over time? | M2 |

### 3.3 Topic Manager

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 3.3.1 | **Load / Unload** | Loads topic subgraphs into orchestrator working context. Unloads when out of scope or deprioritised. Does not delete — just deactivates | LRU eviction? Priority-based? | M2 |
| 3.3.2 | **External Links** | Topics can reference nodes in other lexicons without loading them fully — lazy link resolution | Graph federation strategy with Memgraph | M2 |
| 3.3.3 | **Huffman Tree Maintenance** | Updates the priority tree within each topic graph as node access patterns change | Rebuild on access or incrementally update? | M2 |

### 3.4 Lexicon Adapter

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 3.4.1 | **Query Interface** | Unified interface for graph traversal, pattern matching, and inference queries across any lexicon — discussion, domain, or self | Cypher direct vs abstraction layer? | M1 |
| 3.4.2 | **Cross-Lexicon Query** | Federated queries spanning multiple lexicons via external links and topic bridges | Performance implications — benchmark early | M2 |
| 3.4.3 | **Search** | Semantic-ish search over entity nodes using graph distance and weight, without embedding vectors | Can Huffman + Cypher substitute for vector search adequately? | M2 |

### 3.5 Output Translator

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 3.5.1 | **JSON** | Default structured output from any lexicon query or orchestrator response | Schema versioning for output JSON | M1 |
| 3.5.2 | **HOCON** | Human-friendly config and structured output. Preferred for exporting lexicon configs and system state | Use `node-hocon` or write minimal serialiser | M1 |
| 3.5.3 | **YAML** | For interop with external tools and CI/CD pipelines | YAML only for devops config or also data output? | M1 |
| 3.5.4 | **Natural Language** | Plain-English output from orchestrator for human-in-the-loop interface. Template-based first, LLM-assisted later | Template engine — Handlebars? Nunjucks? | M2 |

---

## 4. Visualizer & Dashboard

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 4.1 | **[Cytoscape.js](https://js.cytoscape.org/)** | Mature graph visualisation library. Layouts, styling, interaction. Best for structured lexicon graph display. Large community | Bundling with main app or separate dashboard app? | M2 |
| 4.2 | **[Sigma.js](https://www.sigmajs.org/)** | WebGL-accelerated, handles very large graphs. Better than Cytoscape at scale. Less flexible styling | Use Sigma for large lexicons, Cytoscape for small? | M3 |
| 4.3 | **[D3.js](https://d3js.org/)** | Maximum control, maximum effort. Good for custom link-weight visualisation and Huffman tree display | Only for bespoke views where Cytoscape falls short | M3 |
| 4.4 | **Dashboard Shell** | Minimal browser UI — [Vite](https://vitejs.dev/) + vanilla JS or lightweight framework. Shows active lexicon, loaded topics, working context, discussion thread, orchestrator log | React/Svelte/vanilla — preference? | M2 |
| 4.5 | **Topic Graph View** | Dedicated view for topic subgraph within a lexicon. Shows load/unload state, Huffman priority highlights, external links | Colour-code by confidence weight | M2 |
| 4.6 | **Discussion Lexicon View** | Live view of current discussion lexicon building as conversation unfolds. Nodes appear as topics are identified | Real-time via WebSocket from Fastify | M3 |
| 4.7 | **Human-in-the-Loop Panel** | UI panel for trusted human actions: assert node, adjust weight, load/unload topic, write note, review audit log | Auth — who is trusted? Simple token or Supabase auth? | M2 |

---

*iamWim · Features & Spec · Living document — sections will grow*
