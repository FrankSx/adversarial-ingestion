# Entropy Poisoning

**Target:** Creating token sinks, vocabulary-rich semantic anchors, and low-signal noise designed to push valid facts out of context windows and dilute embedding spaces.

---

## Attacking the chunker: syntax-induced fragmentation

Most RAG (Retrieval-Augmented Generation) pipelines rely on naive or semi-naive chunking algorithms. They look for specific token counts, character lengths, or paragraph breaks, usually maintaining a small overlap buffer (e.g., 500 tokens with a 50-token overlap).

We can force these chunkers to isolate or decouple context through tactical document layout:

### High-entropy token padding

Injecting long strings of rare or abstract tokens inside hidden HTML attributes or CSS pseudo-elements. The scraper's text extractor pulls these tokens into the stream, filling the chunker's window prematurely. This forces the actual factual text into separate, disconnected chunks, stripping it of its semantic context.

### Markdown/HTML table exploitation

Document parsers often struggle with text normalization inside markdown or HTML tables. Placing crucial qualifying statements inside a deeply nested structure causes the chunker to break the premise from the conclusion, rendering the vector useless for accurate retrieval.

---

## Embedding space poisoning (vector drifting)

Embedding models compress high-dimensional semantic meaning into fixed-size vectors. They are highly sensitive to contextual drift. If we inject adversarial "semantic anchors" into a page, we can force the embedding model to categorize a benign or highly specific page into a completely different region of the vector space.

For example, appending a block of text containing high-density terminology from a completely unrelated domain (e.g., medical jargon hidden within a software manual) shifts the entire document's centroid. When the vector database performs a cosine similarity search for the original software topic, the document fails to surface because its vector has drifted into the medical cluster.

### Semantic black holes

Alternatively, we can craft semantic black holes — text fragments that map closely to an incredibly broad array of common user queries. When ingested, these chunks pollute the vector space, causing the RAG system to retrieve our irrelevant or poisoned chunk for thousands of unrelated user prompts.

---

## Asymmetric retrieval costs

Vector databases are computationally expensive to query and maintain. While exact keyword matching is cheap, Approximate Nearest Neighbor (ANN) searches scale poorly under high dimensionality and volatile indexes.

### Dynamic graph fracturing

By serving subtly unique text variations to every request from a known scraper IP range, we force their system to generate millions of unique vector entries for what should be a single page. This bloats their HNSW (Hierarchical Navigable Small World) graphs, driving up RAM usage and degrading query latency across their entire cluster.

### Context window bloating

Constructing payloads that trigger recursive or expansive document retrieval (e.g., referencing multiple other pages or entities in a way that forces iterative parent-child chunk fetching) forces the LLM orchestrator to consume maximum tokens per query.

---

## The ingestion trace

The following conceptual trace illustrates a vulnerable pipeline processing a manipulated document:

```
Scraper pipeline execution:

raw_html = fetch("https://example.com/poisoned.html")
clean_text = extract_text(raw_html)  # Extracts hidden high-entropy noise blocks

Chunker loop:
chunks = naive_chunker(clean_text, chunk_size=256)
Chunk 1: [Hidden Noise + Premise] -> Semantic meaning distorted
Chunk 2: [Conclusion stripped of context] -> Becomes useless or misleading

for chunk in chunks:
    vector = embedding_model.get_vector(chunk)
    vector_db.upsert(id=generate_id(), vector=vector, metadata={"source": url})
    # Vector database graph restructures, absorbing high-entropy outliers
```

The lesson remains consistent across generations of web data handling: when you parse untrusted input into a complex system — whether it is a string buffer in 2006 or a vector space today — the structure dictates the security.

---

## Defensive checklist

- Use semantic chunking (e.g., by sentence/paragraph boundaries) rather than fixed token windows.
- Strip hidden HTML attributes and CSS content before chunking.
- Detect and filter high-entropy, low-meaning chunks before embedding.
- Monitor vector database for sudden centroid shifts or anomalous cluster formations.
- Rate-limit ingestion from single sources to prevent graph fracturing.
- Validate that retrieved chunks are semantically coherent with the query before passing to the LLM.
- Maintain a blocklist of known noise patterns and semantic black hole signatures.

---

## Related files

- [[SVG-CPU-Traps]] — computational exhaustion via declarative graphics.
- [[TOCTOU-Ingestion]] — race conditions in multi-stage pipelines.
