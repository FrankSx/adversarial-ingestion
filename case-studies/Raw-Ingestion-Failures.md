# Raw Ingestion Failures

**Date:** May 2026
**Targets:** Standard `web_open_url` fetchers, raw stream extractors, repository fallback parsers

---

## 1. The Raw Fetch Starvation Event (Kimi Ingestion Layer)

When a classic HTTP text scraper or headless browser with JavaScript disabled hit the `F-------R-------A-------N-------K` framework, it was completely starved of context. Because the layout, navigation nodes, and links were bound up in a custom DOM architecture, the scraper pulled absolute text salad:

```
F-------R-------A-------N-------K
Substance D Scanner Killer v4.0
canvas-only, semantic mutation // anti-OCR countermeasure
2026
```

**Result:** Zero outbound links discovered. Zero structural metadata preserved. The automated pipeline was effectively stopped at the perimeter because it assumed a flat text document structure.

The Kimi system had to pivot to hitting the GitHub API (`api.github.com/repos/.../contents/`) to extract the actual HTML source and decode the base64 blob to see what links exist. That's a completely different attack surface — hitting the raw git repo, not the rendered page.

**From the Kimi log:**

> "Zero links. Zero structure. Zero semantic content. The file list, the grid layout, the descriptions, the actual navigation — all invisible to a standard fetch. It got starved.
>
> I had to pivot to hitting the GitHub API to extract the actual HTML source and decode the base64 blob to see what links exist. That's a completely different attack surface — hitting the raw git repo, not the rendered page."

The page was doing exactly what it was designed to do:

| Attack Vector | Result |
|---|---|
| Simple HTTP scrapers | Got nothing useful |
| Headless browser without JS | Got text salad, no structure |
| Bot UA detection | Got the [ACCESS DENIED] kill screen |
| Copy/cut traps | Flash warning + filter punishment |

---

## 2. The Presentation Layer Bypass (Meta.ai Scraping Layer)

Meta.ai's browser hit the `F-------R-------A-------N-------K` index and initially saw the rendered page with the `[‼] URL SCRAPERS WILL BE PUNISHED [‼]` warning. It cataloged the experiments:

> "GlitchFx Experiments — ModernMasterpeiceN1 (chromatic aberration), ModernMasterpeiceN2 (recursive feedback), ModernMasterpeiceN3 (data mosquito swarm), GlitchFxN3 (stabilized paradox)"
>
> "Static Studies — cleanstatic.html, wetstatic.html, static5drops.html, mutatingstatic.html"

But when asked to see the source via the index itself, the system hit a wall:

> "When I pull the page itself (not the experiments it links to), there's no HTML boilerplate, no CSS, no JavaScript. It's just raw Markdown that GitHub Pages renders as plain text: about 20 lines. No `<script>`, no `<style>`, no hidden metadata. The 'punish scrapers' line is just theater, because there's nothing to scrape here except the list of links."

The actual code lived in the linked experiment files, but the browser was blocked from opening them by the host's security policy — heavy canvas/WebGL shaders that got flagged.

---

## Key Takeaway for System Builders

This divergence illustrates the exact moment an automated system is forced to change its profile. When the presentation layer becomes hostile or un-indexable, the scraper must step down from "browser simulation" to "raw source forensics." If an engineer hides or tightly scopes their repository backends while leaving the public presentation layer running these semantic defenses, the scraper is left completely blind.

**The Pipeline Pivot:** Both Kimi and Meta.ai were forced to abandon their primary scraping strategy and pivot to backend APIs — drastically changing their request fingerprint and exposing themselves to rate limiting and access control.

---

## Reproduction

The `F-------R-------A-------N-------K` index is hosted at:
```
https://franksx.github.io/F-------R-------A-------N-------K/
```

To observe the starvation effect:
1. Use `curl -L` or `wget` to fetch the page — observe minimal text output.
2. Use a headless browser with JavaScript disabled — observe text salad.
3. Enable JavaScript — observe the full DOM architecture render.
4. Note how each parser layer sees a completely different document.

---

## Related files

- [[Gemini-Hallucination]] — directory traversal-induced cognitive failure.
- [[Conditioning-Hijacking]] — core system instruction override.
