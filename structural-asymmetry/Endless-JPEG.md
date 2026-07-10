# Endless JPEG

**Created:** Monday 18 May 2026

JPEG is the perfect example of a format that looks simple at first and then keeps going. The byte stream is just markers and segments: SOI, APP0, DQT, SOF, DHT, SOS, EOI. But the simplicity of the grammar hides a lot of room for parser traps.

---

## The grammar is small; the state machine is not

A baseline JPEG parser expects:

- `FF D8` — start of image.
- Markers in the form `FF <non-00>`.
- Length-prefixed segments (except stand-alone markers like SOI, EOI, RSTn).
- `FF D9` — end of image.

Everything else is "entropy-coded data" until the next marker. That means a malformed scan can swallow enormous byte runs. Parsers that scan for markers naively will mis-handle `FF 00` stuffing; parsers that do not check lengths will over-read; parsers that trust the SOF dimensions will allocate huge buffers for tiny files.

---

## Dimension / memory bombs

The SOF0 segment (`FF C0`) declares frame width, height, and component count. A parser that pre-allocates based on these values can be tricked into allocating gigabytes for a file of a few hundred bytes:

```
# Conceptual SOF0 structure (defanged, values illustrative):
FF C0          # SOF0 marker
00 11          # segment length
08             # precision (bits/sample)
00 10 00 10    # height, width (e.g. 16x16)
03             # number of components
# ... component specifiers
```

A hostile file might claim 0xFFFF x 0xFFFF pixels with three components. A naive decoder will multiply before checking whether the entropy data could possibly fill that frame. JPEG 2000 and JXL carry the same risk in different boxes (SIZ dimensions, jxlc size fields).

---

## Restart interval mischief

Restart markers (`RST0`–`RST7`) are supposed to divide the scan into independently decodable chunks. Some decoders use them for parallel decoding or error recovery. If you craft a file with:

- a very small restart interval (MCU count),
- an enormous declared image size,
- or inconsistent restart marker sequences,

the decoder can spend most of its time bookkeeping rather than decoding. This is useful as a CPU asymmetry weapon: small file, large decoder work.

---

## JPEG 2000 and JXL

JPEG 2000 brings tile-part streams, progression orders, and codestream markers like `FF 51` (SIZ) and `FF 93` (SOD). A single JP2 file can declare many tiles, multiple quality layers, and arbitrary precinct sizes. Decoders that follow every progression layer will decode the same image region again and again.

JXL (JPEG XL) adds container boxes (ISOBMFF-style) on top of a codestream. Boxes can be chained, duplicated, or padded. JXL also supports:

- **Noise synthesis**: the file can describe noise to be procedurally generated at decode time, shifting work from encoder to decoder.
- **Progressive groups**: a tiny file can request many refinement passes.
- **Animation**: multiple frames with timing, essentially a video loop inside an image.
- **JPEG1 transcodability**: a JXL file can be a losslessly transcoded JPEG, which means a decoder must handle both legacy and native semantics.

---

## Parser traps for scrapers

Headless browsers and image ingestion pipelines often call into system libraries (libjpeg-turbo, OpenJPEG, libjxl). The traps that matter most at scale are:

1. **OOM via declared dimensions** before entropy validation.
2. **Infinite or very long progression passes** in JPEG 2000 / JXL.
3. **Abnormal numbers of scans / components** that exercise slow code paths.
4. **Truncated EOI** causing parsers to read past the buffer or retry.
5. **APP segments with huge lengths** (e.g. APP1 Exif, APP13 Photoshop) that push parsing into metadata-only toolchains.

---

## Defensive notes

- Cap allocation from dimensions before reading entropy data.
- Limit total decode time and total output pixels.
- Do not trust the EOI marker for end-of-file; bounds-check every read.
- For JXL/JP2, validate container box sizes and reject overlapping or nonsensical boxes.
- Run image decoding in sandboxed workers with memory and CPU quotas.

---

## One-liner conceptual hex pattern

```
FF D8 FF E0 00 10 4A 46 49 46 00 01 ...  # normal JFIF start
FF C0 00 0B 08 00 10 00 10 01 01 11 00   # SOF0, 16x16, 1 component
FF DA ...                                  # SOS
# A malformed scan could continue for megabytes without another marker
FF D9                                      # EOI
```

If the scan contains no entropy data but the parser keeps looking for the next marker, you have bought a lot of CPU for a few bytes.
