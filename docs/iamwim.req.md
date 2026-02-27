# iamWim · Feature & Technology Specification

> Living document. Sections: What We Are Building · Technology · Development · Components · Visualizer
> Table columns: # · Name · Details · Open Questions · Done In (milestone/sprint)

---

## 0. What We Are Building

> This section explains the concepts, approaches, and tools at the heart of iamWim — what each one is, and why it belongs here.

| # | Name | What It Is | Role in iamWim | Open Questions | Done In |
|---|------|-----------|----------------|----------------|---------|
| 0.1 | **iamWim** | A logic-first, graph-based reasoning system that learns developmentally — starting with a child's vocabulary and expanding through structured experience. LLMs are available but peripheral. | The whole project. Everything below serves this. | — | — |
| 0.2 | **PIERCE** | **P**ipeline for **I**terative and **E**volutionary **R**easoning, **C**ritique and **E**xpansion. A replacement adapter for the LLM inference layer — built on symbolic logic, rule engines, and graph traversal instead of language models. | The reasoning core. PIERCE is what fires when iamWim needs to think — before any LLM is consulted. | Where exactly does PIERCE end and the Orchestrator begin? Define the boundary. | M1 |
| 0.3 | **Iterative Refinement Pipeline** | A processing pipeline where outputs are passed back through validation and improvement steps repeatedly until they meet quality thresholds. Not one-shot — cyclic. | PIERCE's execution model. A conclusion is not accepted until it has passed N refinement cycles without degrading. | How many cycles before accepting? Configurable per context? | M1 |
| 0.4 | **Conjecture–Criticism Cycle** | Inspired by Popper's falsificationism: the system proposes a conclusion (conjecture), then actively tries to break it (criticism) using its own rules and graph structure. Repeat until it survives criticism or is replaced. | iamWim's internal quality loop inside PIERCE. Replaces hallucination-prone one-pass LLM inference with adversarial self-checking. | How is the "critic" role separated from the "proposer" role in the architecture? Separate rule sets? | M1 |
| 0.5 | **Modular Architecture** | Each capability (lexicon building, topic management, output translation, reasoning, memory) is a discrete, replaceable module with a defined interface. Nothing is hardwired. | Allows iamWim to grow without refactoring its core. New reasoning modes, new storage backends, new output formats — all plug in. | Define module contract / interface standard early. | M0 |
| 0.6 | **Unifying Reasoning Modes** | A spectrum of reasoning from pure symbolic/logical (Prolog, rule engines) through probabilistic (weighted graph inference) to generative (LLM consult). All modes are available; the system chooses based on context and confidence. | iamWim does not bet on one reasoning paradigm. Logical checking first. Probabilistic inference second. LLM last resort. | How does the system decide which mode to invoke? Confidence threshold? Rule-based selection? | M2 |
| 0.7 | **Data Integration** | Connecting PIERCE's reasoning outputs back to the persistent lexicon graphs in Memgraph and Supabase. Every concluded fact, link, and weight change is written back to the store — nothing floats free. | Closes the loop between reasoning and memory. iamWim remembers what it concludes. | Transaction model — how do we roll back a bad conclusion? | M2 |
| 0.8 | **First Lexicon (Manual + AI-Assisted)** | The seed lexicon, built by hand with AI assistance. A carefully constructed starting graph to test and stress the architecture before automated lexicon building begins. Reveals what is actually needed in practice. | Proof of concept and calibration tool. Nothing tests a data model like filling it with real data. | What domain for the first lexicon? Child concepts? Basic physical world? | M1 |
| 0.9 | **Child Vocabulary Seed** | iamWim begins with the vocabulary of a toddler-level reader — simple nouns, verbs, relationships, basic questions. Complexity is earned, not assumed. Sourced from graded reader word lists and toddler NLP corpora. | Forces the architecture to handle genuine knowledge growth rather than assuming a pre-loaded adult knowledge base. Mirrors human cognitive development. | Which graded reader corpus? Dolch list? Oxford 3000 foundation? | M1 |
| 0.10 | **Stanza (Stanford NLP)** | Stanford's production NLP library — tokenisation, POS tagging, dependency parsing, named entity recognition, coreference resolution. Accessed via the `stanza-api` project as a local service. | iamWim's linguistic front door. Raw text (questions, human input, documents) is parsed by Stanza before entering the lexicon graph. Stanza turns language into structure; iamWim reasons over the structure. | Does stanza-api run comfortably alongside Memgraph and Ollama on local hardware? Memory budget? | M1 |

---

## 1. Technology

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 1.1 | **Runtime** | Node.js (LTS) — event-driven, non-blocking, native JS ecosystem | Version pinning strategy? `.nvmrc`? | M0 |
| 1.2 | **Forward Chaining / Rules** | [Nools](https://github.com/noolsjs/nools) — Rete-algorithm rule engine in JS. Fires rules against working memory facts. First resort before LLM. | Still maintained? Evaluate vs. hand-rolled Rete | M1 |
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
| 3.1.1 | **Rule Engine** | Nools Rete-based rules fire against the active lexicon graph's working memory. No LLM in the loop unless `consult()` explicitly called | Rule definition format — Nools DSL or hand-rolled Rete? | M1 |
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

## 5. Data Modeling

> All memory, reasoning state, knowledge, and discourse in iamWim is stored as **lexicon graphs**. There is no separate short-term buffer or long-term store — the distinction is structural and topological within the graph system. This section is the authoritative spec for all graph types, node schemas, and link models.

### 5.1 Lexicon Graph — Types

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 5.1.1 | **Domain Lexicon** | Persistent, structured knowledge about a subject domain (physical world, biochemistry, Akkadian grammar, etc.). Built with AI assistance and human curation. Logic-based, tagged, layered, human-readable. NOT statistical. | One Memgraph database per domain or labelled subgraphs? | M1 |
| 5.1.2 | **Discussion Lexicon** | Built on the fly per conversation session. Has a thread subject title. Captures current, planned, and discussed topics; discussion branches; goals, intents, and tasks accumulated per request and per response. Request/response pairs stored as graph entries tied to topic nodes — not raw text. | Persist all discussion lexicons or only flagged ones? | M1 |
| 5.1.3 | **Self Lexicon** | The system's graph of its own state — loaded topics, active rules, recent decisions, confidence levels, known unknowns. Enables self-awareness loop in the orchestrator. | How detailed at M1? Start minimal. | M3 |
| 5.1.4 | **Deep Lexicon** | Fully structured, attributed, weighted, with rich internal topic graphs. Built over time through curation and learning. | Promotion criteria from jotted → deep | M2 |
| 5.1.5 | **Jotted Lexicon** | Lightweight, fast, note-like. Created by the system itself to capture something quickly without full elaboration. Promotable to a deep lexicon later. | File-based (JSON) or Memgraph subgraph? | M1 |

### 5.2 Lexicon Graph — Internal Structure

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 5.2.1 | **Entity Foundation Layer** | The nodes. Entities of any type: concepts, objects, agents, events, facts, intentions, tasks, questions, answers, relations, meta-concepts. Every node has: `id`, `label`, `type`, `confidence` (0–1), `source`, `sourceType`, `tags[]`, `createdAt`, `version` | Final node schema — define before M1 | M1 |
| 5.2.2 | **Link Layer** | Named, bidirectional, weighted edges. Every link has: `type` (named relation), `weight` (signed float — positive affirms, negative contradicts or deprecates), `attributes{}`, `confidence`, `source`, `sourceType`, `tags[]`. A link is not merely a connection — it is a typed, weighted, annotated relationship. | Attribute schema for links — open or closed vocabulary? | M1 |
| 5.2.3 | **Synonym / Equivalence Mappings** | Special link type: `EQUIVALENT_TO`. Captures that "have" = "owns", "big" = "large", etc. Used by the orchestrator to resolve surface variation in questions without LLM. AI-assisted at build time, human-verified. | Store as links in the graph or a separate lookup table? | M1 |
| 5.2.4 | **Topic Graph (Internal)** | Each lexicon contains an internal topic graph — a structured map of subjects, themes, and domains present in that lexicon. Topics are nodes; their relationships (broader, narrower, related, prerequisite) are links. | Topic node schema — separate from entity nodes or same type with a flag? | M1 |
| 5.2.5 | **Huffman Priority Tree** | Each topic graph maintains a priority tree over its nodes. Commonly-needed or high-confidence nodes are shortest path. Updated as access patterns change. Enables fast retrieval without full graph sweep. | Rebuild on access or incremental update? | M2 |

### 5.3 Topic Properties

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 5.3.1 | **Load State** | Each topic has a state: `loaded` (in working context) or `dormant` (persisted, not active). Orchestrator operates only on loaded topics — sparse by design. | LRU eviction, priority-based, or manual only? | M2 |
| 5.3.2 | **Availability Horizon** | Topics tagged as `immediate` (needed now) or `long-run` (background, available but not foregrounded). | How does orchestrator decide horizon? Rule-based or weight-driven? | M2 |
| 5.3.3 | **External Links** | A topic in one lexicon can link to topics in other lexicons without loading them fully — lazy resolution. Cross-lexicon traversal on demand only. | Graph federation in Memgraph — labelled subgraph approach? | M2 |

### 5.4 Question & Answer Modeling

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 5.4.1 | **Question Types** | All incoming questions classified before reasoning: `identity` (who is Jane?), `category` (is Jane a girl?), `quantity` (how many dogs does Jane have?), `property`, `relation`, `existence`, `comparison`, `procedural`. Classification via Stanza parse + rule matching — no LLM. | Full taxonomy of question types needed — define in M1 | M1 |
| 5.4.2 | **Answer Type Expectation** | Every question node carries an expected answer type: `entity`, `boolean`, `number`, `list`, `relation`, `description`. The orchestrator knows before searching what shape the answer should be. | How to handle ambiguous answer types? | M1 |
| 5.4.3 | **Entity–Relation–Entity Triples** | Core reasoning unit. Example: `Jane –[OWNS]→ Spot`, `Spot –[IS_A]→ Dog`. Questions like "how many dogs does Jane have?" decompose to: find entity Jane, follow OWNS (or EQUIVALENT_TO OWNS) links, filter targets by IS_A Dog, count. Pure graph traversal, no LLM. | Triple storage as Memgraph relationships directly — yes | M1 |
| 5.4.4 | **Typo / Variant Tolerance** | "is jane a gril?" should resolve to "is Jane a girl?". Handled at the Stanza pre-processing layer + fuzzy entity lookup in the graph. Not LLM-based. | Levenshtein threshold? Phonetic matching (Soundex)? | M2 |
| 5.4.5 | **LLM Fallback + Learning** | When graph traversal fails to answer, `consult()` is called. But the result is not just returned — it is parsed, mapped back to entity-relation-entity triples, and added to the lexicon as new attributed nodes. iamWim learns from every LLM fallback. | Confidence level for LLM-sourced nodes vs human-sourced? | M2 |

### 5.5 Generalization Model

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 5.5.1 | **Generalization Graph** | A versioned, sliced graph layer that represents abstract patterns — not facts, but structures. Example: "things that have a quantity of X" or "entities that can own other entities of category Y". Updates in real-time as new facts are added. | Separate Memgraph subgraph or overlay on entity layer? | M2 |
| 5.5.2 | **Versioning & Slicing** | Every generalization has a version. Slices allow the system to reason over generalizations valid at a given time or confidence level. Old versions are not deleted — they are superseded with a link. | Version as node property or separate version nodes? | M2 |
| 5.5.3 | **Domain Concept Grasp** | When iamWim reads a domain text (e.g. Biochemistry year 3, Akkadian grammar), it grasps new concepts by connecting them to existing vocabulary and structure — not just words, but typed relationships and categories. New nodes connect into the generalization graph, not just the entity layer. | How is "grasping" triggered? Confidence threshold on new links? | M3 |
| 5.5.4 | **Unknown Domain Bootstrapping** | When asked "what is the mmm of hmf" — iamWim checks: does hmf exist as an entity? Does it have mmm-type links? If not, it notes the unknown, asks for help or consults LLM, and builds the structure from the answer. | How does system signal "I don't know this domain yet"? | M2 |

### 5.6 Meta-Concepts (Predetermined)

| # | Name | Details | Open Questions | Done In |
|---|------|---------|----------------|---------|
| 5.6.1 | **Goals** | Top-level intent nodes. Every discussion lexicon and task context has at least one goal node. Goals have sub-goals. | Goal schema — `id`, `description`, `status`, `priority`, `parentGoal`? | M1 |
| 5.6.2 | **Sub-Goals** | Decomposed goals. A goal may have N sub-goals, each of which may have further sub-goals. Recursive. | Max depth? Or unconstrained? | M1 |
| 5.6.3 | **Tasks** | Concrete actionable units attached to a sub-goal. A task has a status: `pending`, `active`, `done`, `failed`. | Task assignment — to orchestrator, to human, to LLM? | M1 |
| 5.6.4 | **Sequences** | Ordered lists of tasks or sub-goals. A sequence is a meta-node with ordered `NEXT` links between its members. | Parallel sequences (AND) vs alternative sequences (OR)? | M2 |
| 5.6.5 | **States** | Named snapshots of a context's condition at a point in time. Used to track progress, enable rollback, and reason about change. | State as a node snapshot or a diff from previous state? | M2 |
| 5.6.6 | **Scoring Tags** | Metadata tags on any node or link carrying a quality signal: `coherent`, `redundant`, `certain`, `uncertain`, `verified`, `deprecated`, `llm-sourced`, `human-verified`, `inferred`. Used by the Tester component and the Generalization Model. | Closed vocabulary of tags or extensible? | M1 |

---

*iamWim · Features & Spec · Living document — sections will grow*
