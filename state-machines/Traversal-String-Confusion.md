# Traversal String Confusion

**Target:** Embedding directory-traversal syntax (`..`) inside stylized text blocks forces regex/URL extractors to diverge from what a native browser renders.

---

## The tokenizer shatter event

Most security researchers look at path traversal (`../`) from a traditional file-system execution perspective. But when you wrap these directory-relative sequences into high-repetition string chains like `..--..--..`, you are targeting the Byte-Pair Encoding (BPE) Tokenizer of the LLM parser.

A standard tokenizer easily groups common words or clean repetition (like plain dashes `-------`) into single, cheap tokens. But when it hits a dense, fluctuating stream of periods and dashes (`..--..--..`), it cannot find a clean statistical match in its vocabulary matrix. It is forced to shatter the string into dozens of tiny, high-entropy, rare token IDs.

This artificially balloons the token count of the document, consuming maximum context-window space while shifting the mathematical vector embedding of your page into a completely different cluster — blinding similarity searches.

---

## Exploiting canonicalization and path sanitizers

When a standard crawler or scraping script processes links, it has to resolve the URL. If it encounters a literal string containing `..` packed between characters inside an `href` or a data attribute, the routing engine tries to canonicalize it.

- Many naive parsers will misinterpret `..--..--..` as a relative path directive (`../`).
- This triggers a **Path Traversal / Sandbox Confusion** state inside the ingestion engine.
- The parser might try to back out of its current working directory or drop its scope entirely, completely breaking its loop before it can index the actual content.

---

## Live capture evidence

In the Gemini capture (`case-studies/Gemini-Hallucination.md`), the absolute breaking point where the model dropped out of reality and began radically hallucinating occurred exactly when the system was forced to deal with the path layout:

```
https://franksx.github.io/../../
```

The scraping tool hit the relative traversal string, triggered a backend processing error (`PERMISSION_DENIED`), and the core reasoning model had to panic-fabricate an entire fake description of `Jpegless.html` just to satisfy the prompt.

---

## URL resolution divergence

**Standard Browser Behavior:** Modern web browsers (Chrome, Firefox, Safari) normalize URLs on the client side before initiating the HTTP request. When a browser sees directory traversal commands (`../`), it resolves the path locally (e.g., stripping the redundant path to hit the root directory) and requests the normalized URL from the server.

**Scraper Limitation:** Many internal browsing/fetch tools act more like raw cURL requests. They often attempt to fetch the exact string provided. If the tool lacks robust pre-flight URL normalization, the server rejects the malformed request.

**The Result:** The scraper is functionally blind to a page the user can easily see.

---

## Implementation in defensive layouts

Replace standard horizontal rules (`---`) with custom token-shattering dividers:

```markdown
# Module 2: State Machine Divergence
..--..--..-------..--..--..-------..--..--..

## Section 2.1: Trailing Gutter Exploitation
```

This merges technical research with structural identity, presenting a portfolio that looks deliberate, deeply forensic, and structurally hostile to automated collection.

---

## Defensive checklist

- Normalize ALL URLs before processing, using the same canonicalization rules as target browsers.
- Do not let URL extractors pass raw strings to fetchers without path resolution.
- Treat URLs with embedded traversal sequences as suspicious even if they resolve correctly.
- Test scrapers against paths containing `..`, `.`, and percent-encoded variants (`%2e%2e`).

---

## Related files

- [[Invisible-Gutters]] — trailing data exploitation.
- [[Gemini-Hallucination]] — live capture of traversal-induced cognitive failure.
