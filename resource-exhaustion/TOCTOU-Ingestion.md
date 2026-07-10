# TOCTOU Ingestion

**Target:** Time-of-check to time-of-use vulnerabilities within transient container ingestion paths, detailing how file-handle reuse can lead to pipeline cache poisoning.

---

## The TOCTOU problem in ingestion pipelines

Traditional TOCTOU (Time-of-Check to Time-of-Use) vulnerabilities occur when a program checks a resource's state, then uses it based on that check, but the resource changes between the two operations. In ingestion pipelines, this manifests when:

1. A file is validated (check: magic bytes, size, MIME type).
2. The file is processed (use: parsed, chunked, embedded, indexed).
3. Between these steps, the file content changes — or a different file is substituted.

---

## File-handle reuse in containerized pipelines

Modern ingestion pipelines often use ephemeral containers:

```
Upload → Temp Storage → Validator → Parser → Chunker → Embedder → Vector DB
```

Each stage may run in a separate container or process, communicating via:
- Shared filesystem paths (`/tmp/uploads/file.jpg`).
- Object storage URLs with short-lived tokens.
- Message queue references.

### The race condition

If two uploads receive the same temporary path (e.g., via predictable filename generation or path recycling):

1. **T0**: Upload A (safe PNG) → stored at `/tmp/uploads/12345.png`.
2. **T1**: Validator checks `/tmp/uploads/12345.png` → passes as PNG.
3. **T2**: Upload B (malicious polyglot) → overwrites `/tmp/uploads/12345.png`.
4. **T3**: Parser opens `/tmp/uploads/12345.png` → processes the malicious file.

The validator saw a safe file. The parser processed a malicious one. Same path, different content.

---

## Pipeline cache poisoning

Multi-stage pipelines often cache intermediate results:

- Parsed DOM trees cached by URL.
- Extracted text cached by content hash.
- Vector embeddings cached by chunk hash.

If an attacker can control the cache key while changing the content:

1. First request: benign content → parsed, cached.
2. Second request: same cache key, malicious content → parser returns cached benign result.
3. The malicious content bypasses parsing entirely.

---

## Object storage token races

When pipelines use presigned URLs for object storage:

1. Attacker uploads file A, gets presigned URL for file B (different object).
2. Pipeline fetches from the presigned URL → gets file B.
3. But the pipeline's metadata says it's file A (based on the upload handler's record).
4. File B is processed with file A's permissions and classification.

---

## Defensive checklist

- Use content-addressed storage (path = hash of content) to prevent path collisions.
- Validate file content immediately before EACH processing stage, not just at upload.
- Never trust filesystem paths as identifiers; always verify content matches expectations.
- Use immutable references in pipeline messages (content hash, not path).
- Implement pipeline stage versioning: each stage signs its output, next stage verifies.
- Set strict timeouts on presigned URLs and single-use tokens.
- Monitor for path reuse and cache key collisions in pipeline telemetry.

---

## Related files

- [[Entropy-Poisoning]] — vector space and chunking attacks.
- [[Wrapping-Wrappers]] — polyglot format exploitation.
