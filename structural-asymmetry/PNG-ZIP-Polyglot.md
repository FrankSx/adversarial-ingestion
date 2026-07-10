# PNG ZIP Polyglot Example

**Created:** Monday 29 June 2026

This page is a defanged walkthrough of a ZIP-over-PNG polyglot. It explains the geometry, not a drop-and-run exploit file. The goal is to understand why an image validator and an archive extractor can disagree about what the same bytes mean.

---

## What the two formats need

**PNG** expects:

- 8-byte signature: `89 50 4E 47 0D 0A 1A 0A`.
- Chunks with `length(4) | type(4) | data(length) | crc(4)`.
- `IEND` chunk marks the end. Any trailing bytes are technically outside the PNG spec but most decoders ignore them.

**ZIP** expects:

- Local file headers starting with `50 4B 03 04` anywhere in the file.
- Central directory starting with `50 4B 01 02` near the end.
- End-of-central-directory record `50 4B 05 06` at the very end.
- Offsets in the central directory must point to the actual local file headers.

Because PNG decoders stop at `IEND` and ZIP decoders scan from the end for the central directory, you can put a PNG at the front and a ZIP at the back.

---

## Layout

```
[ PNG signature ]
[ IHDR chunk ]
[ IDAT chunk(s) ]
[ IEND chunk ]        <-- PNG parser stops here
[ ZIP local file header ]
[ file data (compressed or stored) ]
[ ZIP central directory ]
[ ZIP end-of-central-directory record ]
```

The ZIP central directory stores the offset of each local file header as a relative offset from the start of the file. Since the PNG part comes first, those offsets must include the PNG length.

---

## A hand-built local file header

The following is a conceptual breakdown of a ZIP local file header, not a real file. Values are illustrative.

```
50 4B 03 04     # local file header signature
14 00           # version needed to extract (2.0)
00 00           # general purpose bit flag
00 00           # compression method (stored = 0)
00 00 00 00     # last mod file time / date
00 00 00 00     # crc-32 (placeholder)
0A 00 00 00     # compressed size (10 bytes)
0A 00 00 00     # uncompressed size (10 bytes)
08 00           # file name length (8 bytes)
00 00           # extra field length
6E 6F 74 65 2E 74 78 74  # "note.txt"
48 65 6C 6C 6F 20 77 6F 72 6C 64  # "Hello world"
```

A real stored file would need a correct CRC-32. A deflated file would need the compression-method byte set to 8 and a valid compressed payload.

---

## Central directory fix-up

The central directory entry for `note.txt` must record the offset where the local file header begins. If the PNG part is 1000 bytes long, the offset field in the central directory must be `0x000003E8` (1000). Tools like `unzip -l` and `zipinfo` read this offset, seek to it, and verify the local file header signature.

If the offset is wrong, the archive still "lists" but extraction fails. If the offset is correct, the archive works as expected.

---

## Why this matters to ingestion pipelines

An upload handler that only checks the first eight bytes decides the file is a PNG. It passes the file to an image scaler, which opens it, decodes the IDATs, and writes a clean PNG — destroying the trailing ZIP in the process. But if the handler skips re-encoding and instead stores the original bytes, then a later extraction step that scans for ZIP signatures will find the archive.

This is the polyglot split: the bytes are valid for both tools, but only if both tools are allowed to look.

---

## Defensive takeaways

- Do not trust magic bytes alone. A PNG is not just the first eight bytes.
- Re-encode or transcode user uploads; do not serve the original byte-for-byte.
- Scan the full file for additional format signatures when the source is untrusted.
- Separate image handling from archive handling at the process level.

---

## References

- PNG spec: chunk structure and trailing-byte behavior.
- ZIP APPNOTE: local file header, central directory, and EOCD layout.
- Related notebook page: [[Wrapping Wrappers]]
