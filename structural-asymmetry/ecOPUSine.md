# ecOPUSine

**Created:** Monday 18 May 2026

Audio containers are parsers too. Ogg, Opus, FLAC, and WAV all have headers, length fields, channel maps, and codec-specific frames. They are usually treated as passive media, but a scraper or browser that extracts metadata, generates waveforms, or transcodes uploads can be driven into expensive or surprising behavior by a few malformed bytes.

---

## Ogg page structure

An Ogg stream is a sequence of pages:

- `OggS` capture pattern.
- Stream structure version (usually 0).
- Header type flag.
- Granule position (64-bit).
- Bitstream serial number (32-bit).
- Page sequence number (32-bit).
- CRC checksum (32-bit).
- Number of page segments.
- Segment table (length lacing values).
- Segment data.

The granule position is the timestamp / sample counter. A decoder seeking through the file uses granule positions to build a timeline. If the granule position jumps backwards, stalls, or grows without bound, the decoder can spend a lot of time in seeking / index construction.

---

## Opus and FLAC in Ogg

Opus uses an Ogg container with an OpusHead header (channel count, pre-skip, sample rate, gain, channel mapping). OpusTags follow. The actual Opus packets are decoded 20 ms at a time.

FLAC in Ogg uses FLAC metadata blocks inside the Ogg pages, beginning with the STREAMINFO block (min/max block size, min/max frame size, sample rate, channels, bits per sample, total samples).

Both formats rely on declared totals that may not match the actual data length. A file that claims 10 hours of audio but contains only a few packets can force a decoder to allocate buffers, build indexes, or wait for packets that never arrive.

---

## WAV and the RIFF problem

WAV files are RIFF containers. They begin with `RIFF....WAVE` and contain chunks like `fmt `, `data`, `fact`, `LIST`. The `data` chunk has a 32-bit length. Two common traps:

1. **Declared size larger than file**: decoders read past EOF or loop waiting for data.
2. **Nested/list chunks with huge lengths**: metadata parsers can recurse or allocate based on the chunk size field.

WAV also supports extensible formats, cue points, and sampler loops, all of which add parser state.

---

## MediaRecorder edge cases

`MediaRecorder` in browsers produces Blobs from a stream. The exact container and codec depend on the requested `mimeType` and what the browser supports. Interesting research angles:

- **Mime type negotiation**: requesting `audio/webm;codecs=opus` may fall back to another codec, changing the container structure.
- **Timestamp discontinuities**: recorded chunks may have non-monotonic timestamps if the source track is paused or replaced.
- **Duration not in header**: some generated WebM/Opus files have an unknown duration until the last cluster is written; parsers that rely on a duration header can misbehave.
- **Codec private data**: OpusHead in WebM is stored in the CodecPrivate element; if it is missing or malformed, decoders may crash or default to stereo 48 kHz.

---

## Parser traps

- **Extreme sample rates or channel counts**: a WAV `fmt ` block or OpusHead can claim 384 kHz or 255 channels. Reject declarations that exceed reasonable bounds before allocation.
- **Giant metadata blocks**: Vorbis comments, FLAC PICTURE blocks, and RIFF `LIST` chunks can be megabytes. Limit metadata parsing size.
- **Mismatched lengths**: total samples vs. granule position vs. RIFF `data` chunk length. Treat mismatches as malformed.
- **Recursive/nested containers**: WAV with `RIFF` chunks inside `LIST` chunks can surprise parsers that assume flat structure.

---

## Defensive notes

- Parse audio metadata with bounded memory and bounded time.
- Do not transcode untrusted audio in the same process as the ingestion pipeline.
- Validate declared lengths against actual file size before allocating buffers.
- For `MediaRecorder` output, probe the first few clusters before trusting the mime type.

---

## Defanged header sketch

```
# Ogg Opus start (conceptual, not a real CRC)
4F 67 67 53 00 00 00 00 00 00 00 00 00 00 00 00  # OggS + flags + granule
00 00 00 00 00 00 00 00 00 00 00 00 01             # serial, seqno, CRC, segs
1E                                                 # segment table: 30 bytes
4F 70 75 73 48 65 61 64 01 ...                     # OpusHead packet
```

If the OpusHead channel-count byte is `FF`, a naive decoder may try to allocate a channel map for 255 channels before checking the rest of the file.
