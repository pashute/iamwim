Document name: iamwim.bg.md
Version: 058


Background discussion with Claude

# iamWim
### Integrated Accumulation Machine with Intelligent Mind
*"I am what I M"*

> A logic-based orchestrator with a developmental learning model, built from the ground up on lexicon graphs — where LLMs play a back-seat supporting role rather than sitting at the centre.

---

## 1. Assessment: What Does This Vision Get Right?

The core instinct is sound. The critique of LLMs — that they bring everything up at once, lack genuine memory architecture, and have no principled developmental path — is legitimate and shared by serious researchers. Key strengths of the proposed vision:

- **Sparse activation** — connections exist in the graph but are not all retrieved at once. This is how the brain actually works and is conspicuously absent from transformer-based systems.
- **Developmental trajectory** — starting with child-level vocabulary and expanding outward, anchored by what is already known. This mirrors constructivist learning theory (Piaget, Vygotsky).
- **Competing sub-models** — multiple components with specialised responsibilities, mapping to Minsky's Society of Mind and modular cognitive architecture research.
- **Memory as a first-class architectural primitive**, not a prompt engineering trick. In iamWim, both working and persistent memory are lexicon graphs — not buffers, not vector stores, not token windows.
- **Human-in-the-loop as trusted teacher** rather than RLHF-style rater — a meaningfully different, and more honest, relationship between human and machine.

### A Note on Creativity

The combinatorial surprise in LLMs is not magic — it is statistical interpolation. A well-connected lexicon graph with good inference rules can generate novel combinations too, just more auditably. The system can always consult an LLM when genuine generative breadth is needed. Creativity is not lost — it is delegated appropriately.

---

## 2. Is It Already Baked? Prior Art Survey

The ingredients exist in scattered form. Nothing quite matches the full assembled vision. Here is the landscape:

### 2.1 Closest Match: OpenCog / AtomSpace
A knowledge hypergraph with multiple cognitive processes and explicit developmental intent. Sprawling, primarily academic, and difficult to run in practice. Does not use a unified lexicon graph model across all memory layers.

### 2.2 Classical Cognitive Architectures: SOAR and ACT-R
Both include memory separation and learning loops. Designed for narrow task domains, no modern graph integration, and no developmental arc. Their chunking mechanisms are instructive but limited.

### 2.3 Cyc
Forty years of logic-based common-sense knowledge. Closed, brittle, and not built for dynamic on-the-fly graph construction.

### 2.4 Neuro-Symbolic AI
Active research at IBM, MIT, and elsewhere. These approaches typically still place neural nets at the centre — the opposite of iamWim's architecture.

### 2.5 HTM (Numenta)
Hierarchical Temporal Memory handles sparse temporal representations in a biologically-inspired way. Useful reference for the sparse activation component.

### 2.6 Verdict
**Half-baked. The ingredients exist. Nobody has assembled them into a coherent developmental system with a trusted human-teacher loop and a unified lexicon graph memory model. The recipe is yours.**

---

## 3. Core Architecture: The Lexicon Graph Model

All memory, context, knowledge, and discourse in iamWim is represented as **lexicon graphs**. There is no separate short-term buffer or long-term store in the traditional sense — the distinction is structural and topological within the graph system itself.

### 3.1 Lexicon Graph Structure

Every lexicon graph has two foundational layers:

1. **Entity Foundation Layer** — the nodes. Entities of any type: concepts, objects, agents, events, facts, intentions, tasks.
2. **Link Layer** — named, bidirectional, weighted edges between entities. Weights are signed (positive or negative) and carry attributes. A link is not merely a connection — it is a typed, weighted, annotated relationship.

### 3.2 Topic Graphs (Internal)

Each lexicon graph contains an internal **topic graph** — a structured map of the subjects, themes, and domains present within that lexicon. Topics within a lexicon can:

- Be **loaded or unloaded** — active in working context or dormant but accessible
- Be available for **immediate use or long-horizon planning**
- Carry **external links** to topics in other lexicons
- Maintain a **Huffman tree** for rapid access to high-priority or frequently-needed nodes — the most important or most-used paths are shortest

### 3.3 Discussion Lexicons (On-the-Fly)

Each conversation or discussion session generates its own **discussion lexicon**, built dynamically as the exchange unfolds. This lexicon captures:

- **Current, planned, and discussed topics** — with a thread subject title
- **Discussion branches** — divergences, tangents, and returns
- **Goals, intents, and tasks** — accumulated per request and per response, building a semantic thread of the entire discussion
- **Accumulated request/response pairs** — not as raw text but as structured graph entries tied to topic nodes

The discussion lexicon is not discarded at session end — it can be persisted, summarised, or merged into a broader lexicon depending on its value.

### 3.4 Lexicon Depth and Note-Taking

Lexicons exist on a spectrum of depth:

- **Deep lexicons** — fully structured, attributed, weighted, with rich internal topic graphs
- **Jotted lexicons** — lightweight, fast, note-like; created by the system itself when it needs to capture something quickly without full elaboration

The system writes its own notes. It can re-read them, expand them, or promote a jotted lexicon into a deeper one over time.

### 3.5 Persistence and Growth

Lexicons persist. They grow. They can be linked to one another across domains and sessions. The system does not "forget" by default — it manages attention by loading and unloading topic subgraphs, not by discarding them. The Huffman tree structure within each topic graph ensures that commonly-needed or high-importance material is always near the surface without requiring a full retrieval sweep.

---

## 4. LLM as Isolated Consultant

LLM calls are isolated behind a `consult()` function. The orchestrator decides when to invoke an LLM — not the other way around. The LLM is a tool the system reaches for, not the engine driving it. All LLM outputs that are retained get integrated back into the lexicon graph as attributed, weighted nodes — they do not float free as raw text.

---

## 5. Human-in-the-Loop

Trusted humans can:

- **Teach** — assert new nodes and links directly into a lexicon
- **Correct** — adjust weights, remove links, re-type relationships
- **Redirect** — shift the active topic graph or load a different lexicon into working context
- **Review** — inspect the system's written notes and accumulated discussion lexicons

This is a teacher relationship, not a rating relationship.

---

## 6. Prototype Scope (Codespaces)

A runnable skeleton demonstrating the shape of the architecture is achievable in a few hours. The two decisions that must be made first, as they shape everything else:

1. **Node schema** — what fields an entity node carries, how links are typed, how weights and attributes are stored
2. **Rule format** — how the orchestrator's rules are declared, how they pattern-match against the graph, and how they fire

With those agreed, the scaffold — graph, orchestrator, discussion lexicon builder, note-writer, and `consult()` stub — can be running within the first session.

---

*iamWim · Working document · More architectural details to follow*
