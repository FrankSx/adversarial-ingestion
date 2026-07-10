# JXL-PDF-WASM Triple Container

**Created:** Monday 24 March 2026

A worked defanged example of a triple container: a single byte stream that is simultaneously valid as JPEG XL, PDF 2.0, and WebAssembly. This is the geometry of parser divergence at its most extreme — three entirely separate format specifications satisfied by one carefully arranged sequence of bytes.

---

## The challenge

Make a file that is simultaneously:

- A valid image (JXL — JPEG XL container),
- A valid document (PDF 2.0),
- A valid executable module (WASM — WebAssembly).

The trick is to find formats whose required structures can be nested or aliased. Each parser reads the same bytes but follows different rules about what to look at, what to skip, and where to stop.

---

## Format requirements

**JXL (JPEG XL)** uses ISOBMFF-style boxes with four-byte type codes. Key boxes:

- `JXL ` — signature box (brand).
- `jxlc` — codestream box.
- `xml ` — XML metadata box (this is our entry point).
- Boxes are length-prefixed: `size(4) | type(4) | data(size-8)`.

**PDF 2.0** expects:

- `%PDF-2.0` at offset 0 (or within the first 1024 bytes per spec).
- Cross-reference tables or streams.
- `%%EOF` marker.
- Supports embedded files via `EmbeddedFile` stream objects.

**WASM** expects:

- Magic bytes: `\x00asm` at offset 0.
- Version: `0x01 0x00 0x00 0x00`.
- Sections with length-prefixed content.

---

## The layout geometry

The key insight: PDF allows arbitrary bytes before `%PDF-` if within the first 1024 bytes, and JXL allows arbitrary boxes in any order. WASM's magic bytes can appear inside data regions of other formats.

```
[ JXL box: size + "JXL " + brand data ]
[ JXL box: size + "xml " + XML payload containing PDF wrapper ]
    Inside the XML:
    [ PDF header: %PDF-2.0 ]
    [ PDF EmbeddedFile stream ]
        Inside the embedded file:
        [ WASM magic: \x00asm ]
        [ WASM version + sections ]
[ JXL box: size + "jxlc" + codestream data ]
[ PDF xref table ]
[ PDF %%EOF ]
```

---

## How each parser sees it

**JXL parser** reads ISOBMFF boxes sequentially:

1. Sees `JXL ` signature box → validates as JPEG XL.
2. Sees `xml ` box → extracts XML metadata.
3. Sees `jxlc` box → finds the codestream.
4. Ignores everything after the last box (including the PDF xref and EOF).

**PDF parser** scans for `%PDF-` within the first 1024 bytes:

1. Finds `%PDF-2.0` inside the XML payload.
2. Reads the cross-reference table near the end.
3. Validates `%%EOF`.
4. Discovers the `EmbeddedFile` object → extracts the WASM payload.

**WASM parser** (if the embedded file is extracted):

1. Sees `\x00asm` magic at the start of the extracted bytes.
2. Reads version and sections.
3. Executes as valid WebAssembly.

---

## The 2078-byte proof of concept

The actual implementation achieves all three formats in **2078 bytes total**:

```
PDF wrapper → EmbeddedFile → JPEG XL ISOBMFF → xml box → WASM custom section
```

The PDF wrapper provides the outer document structure. Inside it, an EmbeddedFile stream contains the JXL container. The JXL container's `xml ` box holds metadata that includes the WASM module's custom section. Each layer is a valid container for the next.

---

## Why this matters

In an ingestion pipeline:

1. **Upload validator** checks magic bytes → sees JXL signature → passes as image.
2. **Document scanner** scans for `%PDF-` → finds PDF structure → extracts embedded files.
3. **Security scanner** checks the WASM module → sees `\x00asm` → flags executable content.

But if each scanner only looks for its own format and stops at the first match, they miss the nested payloads. The image validator doesn't scan for PDF. The PDF scanner doesn't look inside images for WASM. The WASM scanner never sees the original file because it was already categorized as "safe image."

---

## Defensive notes

- Do not trust single-format validation. A file that passes as an image may also be a document or executable.
- Re-encode or transcode all uploads, destroying nested payloads in the process.
- Use format-specific scanners in isolation, then compare results.
- Scan the full file for signatures of ALL supported formats, not just the claimed one.
- Treat files with multiple format signatures as inherently suspicious.

---

## References

- JPEG XL spec: ISOBMFF box structure and container semantics.
- PDF 2.0 ISO 32000-2: embedded files and stream objects.
- WebAssembly spec: binary format and section encoding.
