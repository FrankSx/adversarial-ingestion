# Wrapping Wrappers

**Created:** Monday 18 May 2026

Polyglot files are documents that are valid enough for two or more parsers. The classic examples — GIFAR, JAR/JPG, ZIP-over-PNG — work because the parsers look at different offsets or different magic numbers. A file can be a valid image to a browser and a valid archive to `unzip` at the same time.

---

## Why wrappers work

Most file formats are not exclusive. They leave room for:

- **Comments and ignored chunks** (PNG ancillary chunks, JPEG APP segments).
- **Appended or prepended data** (ZIP files can have data before the first local file header).
- **Block-structured containers** where the parser jumps via offsets (PDF, MP4, JXL, PNG chunks).
- **Magic bytes at fixed offsets** that do not overlap.

If format A allows arbitrary trailing bytes and format B allows arbitrary leading bytes, you can concatenate them. If both use length-prefixed blocks, you can interleave them.

---

## ZIP-over-PNG

A PNG file starts with the 8-byte signature `89 50 4E 47 0D 0A 1A 0A` and continues with length-type-data-crc chunks. A ZIP archive starts with local file headers but can tolerate arbitrary bytes before the first header, as long as the central directory at the end points to the right offsets.

So you can:

1. Write a valid PNG.
2. Append a ZIP local file header and compressed data.
3. Append a ZIP central directory whose offsets account for the PNG prefix.

A browser or image viewer sees a PNG. `unzip -l` sees the archive. The same bytes serve two masters. This is the basis of many steganography and delivery tricks.

---

## GIFAR and JAR/JPG

- **GIFAR**: concatenate a GIF and a JAR. Old browsers/applets treated the file as both an image and an archive, allowing same-origin policy abuse.
- **JAR/JPG**: a JAR file prepended with JPEG APP data, or a JPEG with a ZIP central directory appended, could be uploaded as an image and loaded as an applet.

These specific vectors are mostly historical because applet support is dead, but the geometry still matters. Any modern system that accepts image uploads and then passes them to a library that reads container offsets is exposed to the same class of issue.

---

## Triple containers

A triple container is valid as three formats at once. The canonical challenge is making a file that is simultaneously:

- a valid image (e.g. PNG, JXL),
- a valid document (e.g. PDF),
- a valid executable module (e.g. WASM).

The trick is to find formats whose required structures can be nested or aliased. For example, a JXL container uses ISOBMFF boxes with four-byte type codes. A PDF uses `%PDF-1.x` at offset 0 and cross-reference tables. A WASM module starts with the magic bytes `\x00asm`. With careful layout, the same byte range can satisfy two or three parsers by letting each one skip over or reinterpret blocks that belong to the others.

See [[JXL-PDF-WASM-Container]] for a worked defanged example.

---

## Parser sees what it wants to see

The reason wrappers are dangerous in scraper/ingestion pipelines is that different tools look at the same file differently:

- The upload validator checks magic bytes at offset 0 → passes as PNG.
- The AV scanner scans the entropy-coded image region → sees nothing.
- The extraction tool notices the appended ZIP → extracts a script.
- The browser renders the image → also executes whatever the page includes.

If your pipeline assumes "this file is an image because the first bytes say so," it has already lost.

---

## Defensive notes

- Validate by more than magic bytes. Parse enough structure to confirm the claimed format.
- Re-encode uploaded images instead of trusting the original bytes.
- Run format-specific tools in isolation; do not let a file be both an image and an archive in the same context.
- Treat any file with trailing or interleaved data as suspicious until proven otherwise.

---

## One-liner mental model

A polyglot is not a lie; it is a file that tells a different true story to every listener.
