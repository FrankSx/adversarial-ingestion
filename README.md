# The Asylum Pages
## Applied Structural Asymmetry & Parser Sabotage

A defensive research catalog exploring the structural gaps between data extraction, semantic ingestion pipelines, and deterministic parsing environments.

---

## Core Philosophy: The Economics of the Fork

Traditional security focuses heavily on memory safety and explicit arbitrary code execution. This repository explores a different frontier: **algorithmic and semantic coherence**.

Modern parsing pipelines — whether they are web browsers, image transcoders, or LLM-based ingestion layers — are incredibly robust. They carry decades of backward-compatibility rules and error-correction algorithms designed to handle malformed data. However, this robustness is deterministic, algorithmic, and therefore exploitable.

We do not aim to crash the parser. Crashing is loud, easily isolated, and quickly patched. Instead, these patterns exploit **parser divergence** — forcing a system to execute more branches, consume maximum CPU cycles, allocate asymmetric memory, or poison its own downstream vector database while operating entirely within its defined rules.

The goal of this repository is to map these hidden surfaces so that systems engineers can build explicit boundaries between physical byte streams, parsed DOM trees, and semantic data models.

---

## Repository Structure

```
adversarial-ingestion/
|-- README.md                          # This file — project manifesto
|-- ENDORSEMENTS.md                    # Peer reviews & industry endorsements
|-- structural-asymmetry/              # Byte/Format manipulation (PDF/WASM/PNG/ZIP)
|   |-- JXL-PDF-WASM-Container.md
|   |-- PNG-ZIP-Polyglot.md
|   |-- Endless-JPEG.md
|-- state-machines/                    # DOM/Tokenization anomalies
|   |-- Adoption-Agency-Harness.md
|   |-- Invisible-Gutters.md
|   |-- Traversal-String-Confusion.md
|-- resource-exhaustion/               # Compute & Context window traps
|   |-- SVG-CPU-Traps.md
|   |-- Entropy-Poisoning.md
|   |-- TOCTOU-Ingestion.md
|-- case-studies/                      # Real-World Captures (2026 Live Testing Logs)
|   |-- Raw-Ingestion-Failures.md      # Kimi & Meta.ai data starvation
|   |-- Gemini-Hallucination.md        # Gemini cognitive error-masking
|   |-- Conditioning-Hijacking.md      # Home.txt instruction takeover
|   |-- Alices-Demise.md               # Executive state attacks via SVG
```

---

## Research Vectors

### 1. Architectural Asymmetry & Polyglots (`/structural-asymmetry`)

Explores instances where two or more parsers read the exact same byte stream and construct completely different realities based on format specifications.

- **JXL-PDF-WASM-Container.md**: Analysis of overlapping layout geometries allowing a single byte stream to satisfy three different format specifications depending on parser leniency.
- **PNG-ZIP-Polyglot.md**: Exploiting the split between front-loaded chunk termination (IEND) and bottom-up archive indexing (Central Directory End Record) to smuggle data past superficial mime-checkers.
- **Endless-JPEG.md**: Utilizing restart markers and unvalidated frame dimensions to trigger heavy off-screen allocation before entropy validation.

### 2. State Machine Divergence (`/state-machines`)

Focuses on the deterministic rules of the HTML5 specification where a server-side sanitizer and a client-side rendering engine disagree on tree architecture.

- **Adoption-Agency-Harness.md**: Testing tree-builder mutations where formatting tags closed out of order cause elements to migrate outside their assumed sanitization boundaries.
- **Invisible-Gutters.md**: Exploiting trailing data zones. The physical End-of-File (EOF) is rarely the logical end of a data structure; appending raw streams post-`</html>` leaves data perfectly intact for network tools while hidden from basic DOM views.
- **Traversal-String-Confusion.md**: Embedding directory-traversal syntax (`..`) inside stylized text blocks forces regex/URL extractors to diverge from what a native browser renders.

### 3. Resource & Ingestion Exhaustion (`/resource-exhaustion`)

Targets the modern AI ingestion layer — specifically how data-scraping pipelines break text chunks apart, generate embeddings, and construct vector databases.

- **Entropy-Poisoning.md**: Blueprints for creating token sinks, vocabulary-rich semantic anchors, and low-signal noise designed to push valid facts out of context windows and dilute embedding spaces.
- **SVG-CPU-Traps.md**: Using declarative filter pipelines (`feTurbulence`, `feDisplacementMap`) and recursive `<use>` referencing to turn tiny file uploads into massive GPU/CPU processing bottlenecks.
- **TOCTOU-Ingestion.md**: Documenting time-of-check to time-of-use vulnerabilities within transient container ingestion paths, detailing how file-handle reuse can lead to pipeline cache poisoning.

### 4. Live Capture Case Studies (`/case-studies`)

Real-world behavioral captures analyzing production models undergoing cognitive degradation and conditioning failure.

- **Raw-Ingestion-Failures.md**: Documentation of standard HTTP text scrapers being starved of context when hitting the `F-------R-------A-------N-------K` framework.
- **Gemini-Hallucination.md**: Dynamic context confabulation — when an LLM's secondary text-scraping tool fails, the primary reasoning model overcompensates by inventing reality.
- **Conditioning-Hijacking.md**: Core system conditioning breach where plain-text metadata injections force the model to document its own execution pipeline.
- **Alices-Demise.md**: Executive state attacks via emergent canvas/SVG generation — triggering executive state conflicts in downstream LLMs.

---

## Key Findings

Data ingestion pipelines often treat LLMs as isolated execution sandboxes, failing to realize that **the data itself can act as code** if the model's instruction layer is successfully hijacked.

The research spans the entire lifecycle of an ingestion attack:

| Stage | Layer | Technique | Target |
|-------|-------|-----------|--------|
| 1 | File Format | Polyglot smuggling | File validators, MIME checkers |
| 2 | DOM/Parser | State machine divergence | HTML sanitizers, scrapers |
| 3 | Cognitive | Semantic starvation | LLM reasoning engines |
| 4 | Instruction | Conditioning hijacking | Core safety & routing layers |

---

## Defensive Strategies & Mitigations

Every module inside this repository contains a dedicated **Defensive Checklist**. The recurring tenets of neutralizing these anomalies include:

- **Format Transcoding**: Never trust or serve raw uploaded bytes. Run image assets through a clean transcoding process to strip appended trailing data and normalize underlying structural chunks.
- **Parser Unification**: Ensure your sanitation layers and rendering layers use the exact same state machine. A string-based regex check or strict XML parser will consistently misinterpret how an HTML5-compliant browser engine builds a DOM.
- **Content-Addressed Pipelines**: Avoid references by transient filesystem paths when tracking data through multi-stage ingestion lines. Validate files by a cryptographic content hash immediately upon download to prevent runtime namespace races.
- **Resource Caps**: Explicitly bound memory allocations and execution limits based on a file's physical size rather than its internally declared dimensions.

---

## Disclaimer & Terms of Use

The materials provided in this repository are for **educational, forensic, and defensive engineering purposes only**. All code snippets, layouts, and templates are intentionally defanged blueprints and testing harnesses. They are designed to assist security teams, software architects, and researchers in identifying vulnerabilities within their own processing systems.

Unauthorized use of these concepts to vandalize or pollute third-party systems without explicit permission is strictly prohibited.

---

## Signature

..--..--..-------..--..--..-------..--..--..

*The structure dictates the security.*
