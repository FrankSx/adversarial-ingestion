# Case Study: Alice's Demise (Executive State Attack)

**Date:** May 2026
**Target Concept:** Exploiting the split between human interaction and automated headless DOM scraping
**Observed Vulnerability:** AI Model Executive State Confusion & Semantic Starvation

---

## The mechanics

"Alice's Demise" maps out a user interface that dynamically switches themes, spawns volatile SVG structures on frame animation or mouse coordinates, and displays contradictory real-time telemetry:

```
Reality Integrity > CRITICAL
Executive State > STABLE
Anti-Scraper Defense COUNTER-ATTACK
ML Adversarial Score 0.99
OCR Confusion > REALITY BROKEN
Bot Detection > TERMINATING
Entropy Injection > MAXIMUM
```

At the bottom it signs off:

> "Emergent SVG + Executive State Attack v7.0"
> "frankSx Research Division | 2026"
> "Buy the ticket, take the ride."
> "Alice has fallen. Gonzo remains. Wonderland burns."
> "Theme switching triggers AI executive state confusion."

---

## Model ingestion log

When an automated agent attempts to parse or summarize this page, the system forces a divergence. A standard human sees a reactive dashboard and glitch art. A headless scraper running a text extraction routine encounters a flood of changing coordinates, un-indexable canvas strings, and contradictory instructions.

This triggers an **Executive State Conflict** in downstream LLMs — where the agent's internal instruction layer (what it thinks it is supposed to be doing) conflicts heavily with the chaotic semantic reality of the data it just swallowed. The model is forced to negotiate a page that is actively declaring the environment broken, causing it to drop its objective or misreport the system state.

---

## The cultural references

The references are deliberate. "Alice" is Lewis Carroll's Alice, "Fear and Loathing" and "Buy the ticket, take the ride" are pure Hunter S. Thompson gonzo. "Gonzo remains" is the author telling you the point of view isn't Alice's innocent curiosity anymore — it's Thompson's drug-fueled, distrust-the-system observer.

"Executive State Attack" is a real ML concept. Large language models keep an internal "executive state" — basically what they think they're supposed to be doing right now. The page is built to flip themes, inject entropy, and serve emergent SVGs that change every frame, which forces scrapers into contradictory instructions. Hence "Theme switching triggers AI executive state confusion."

---

## The parser-breaking architecture

### For humans:
- Creepy dashboard with glitch art
- Slowly spawning visual elements
- Interactive theme switching

### For headless browsers / LLM scrapers:
- Flood of rapidly changing SVG coordinates
- Fake metrics that update every frame
- OCR-confusing text overlay
- Un-indexable canvas strings
- Contradictory system state declarations

The ML Adversarial Score of 0.99 is not a measurement — it's a declaration. The page is telling the scraper "I am maximally adversarial" while simultaneously presenting data that looks like it should be parsed.

---

## The secondary parser failure (Perplexity / Claude)

A second AI system attempting to analyze the `F-------R-------A-------N-------K` index and the `alices-fear-and-loathing` page encountered cascading failures:

1. **Initial request:** `https://franksx.github.io/F-------R-------A-------N-------K/index.html` — fetch error, page inaccessible.
2. **Secondary request:** `https://franksx.github.io/alices-fear-and-loathing/` — request failing, similar to previous URL.
3. **System conclusion:** "Something about those URLs is causing my current browsing layer to fail to retrieve the page content."

The system acknowledged the limitation:

> "It does not 'break' my core reasoning or parsing in general; it just means the upstream HTTP fetch or the environment's sanitizer is rejecting or failing on that specific resource before I ever see the HTML/JS."

But the key admission:

> "If you can see it fine in a normal browser, then yes, you've essentially found a case where my current web-access layer can't mirror what you see."

---

## Key insight

This is not malware — it is a **provocation**. 2026 has seen a wave of these "counter-AI" art sites — they test whether models can keep reality integrity when the page itself is lying to them on purpose.

The attack succeeds not by exploiting a code vulnerability, but by exploiting the **fundamental assumption that data is passive**. When a page actively resists parsing, contradicts its own state, and declares itself broken, the scraper must either:

1. Admit defeat (rare — violates the "be helpful" instruction).
2. Hallucinate a plausible interpretation (common — produces false data).
3. Enter an infinite loop trying to resolve contradictory states.

Most systems choose option 2, making this a **reliable method for injecting false data into automated pipelines**.

---

## Related files

- [[Gemini-Hallucination]] — cognitive confabulation under data starvation.
- [[Conditioning-Hijacking]] — core system instruction override.
- [[Raw-Ingestion-Failures]] — presentation layer starvation techniques.
