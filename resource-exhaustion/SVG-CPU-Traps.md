# SVG CPU Traps

**Target:** Using declarative filter pipelines (`feTurbulence`, `feDisplacementMap`) and recursive `<use>` referencing to turn tiny file uploads into massive GPU/CPU processing bottlenecks.

---

## The SVG complexity asymmetry

SVG is a vector format defined by XML markup. Unlike raster images where processing cost is roughly proportional to pixel count, SVG processing cost depends on:

- The complexity of the DOM tree.
- The number of filter operations applied.
- The recursion depth of `<use>` references.
- The computational cost of procedural texture generation.

A 2KB SVG file can consume more CPU/GPU resources than a 20MB JPEG. This asymmetry is the attack surface.

---

## Filter pipeline bombs

SVG filters are declared in `<defs>` and applied via the `filter` attribute. They form a directed acyclic graph of image-processing operations. Key expensive primitives:

### `feTurbulence`

Generates Perlin or fractal noise procedurally. The `numOctaves` parameter controls complexity:

```xml
<filter id="noise">
  <feTurbulence type="fractalNoise" baseFrequency="0.01" numOctaves="5"/>
</filter>
```

At `numOctaves="5"`, the browser generates 5 octaves of fractal noise across the entire filter region. Increasing this value exponentially increases computation cost with minimal file size growth.

### `feDisplacementMap`

Uses one image to distort another. Combined with turbulence:

```xml
<filter id="displace">
  <feTurbulence type="turbulence" baseFrequency="0.05" numOctaves="3" result="turb"/>
  <feDisplacementMap in="SourceGraphic" in2="turb" scale="100" xChannelSelector="R" yChannelSelector="G"/>
</filter>
```

This forces the browser to: generate turbulence → sample the source graphic → displace every pixel based on the turbulence texture. At high resolution, this is GPU-intensive.

### Filter chaining

Filters can reference other filters via `in` and `result` attributes. A deep chain of 10+ filter primitives, each operating on the full pixel buffer, creates a GPU shader pipeline that can stall even dedicated graphics hardware.

---

## Recursive `<use>` referencing

The `<use>` element instantiates a copy of a defined element. When combined with recursive references:

```xml
<defs>
  <g id="a">
    <use href="#b" transform="scale(0.5)"/>
  </g>
  <g id="b">
    <use href="#a" transform="scale(0.5)"/>
  </g>
</defs>
<use href="#a"/>
```

Most browsers detect and break simple recursion loops. But complex multi-level references with transforms can exhaust memory before the cycle detector triggers, especially in headless environments with conservative recursion limits.

---

## Canvas feedback loops

When SVG containing filters is rendered to a `<canvas>` via `drawImage()`, and the canvas content is then fed back into the SVG as an image source, you create an iterative filter pipeline:

```
canvas → SVG filter → canvas → SVG filter → ... (each frame)
```

This is the basis of the "Endless JPEG" technique: each frame applies lossy JPEG compression via `canvas.toDataURL('image/jpeg', quality)`, then re-embeds the result, creating generation loss in real-time. The computational cost is frame-rate × filter complexity × canvas resolution.

---

## Headless browser exploitation

Automated scrapers using headless Chrome, Puppeteer, or Playwright are particularly vulnerable:

1. They render SVG at full resolution for screenshot capture.
2. They don't apply the same resource limits as user-facing browsers.
3. They often run in resource-constrained containers.
4. A single malicious SVG can stall the entire scraping pipeline.

---

## Defensive checklist

- Set explicit SVG rendering timeout in headless environments (e.g., 5 seconds max).
- Limit `numOctaves` to a reasonable value (2-3) for untrusted SVG.
- Count filter primitives and reject SVG with more than N operations.
- Render untrusted SVG at low resolution first, then scale up only if safe.
- Run SVG processing in isolated workers with CPU quotas.
- Detect and break `<use>` recursion before rendering.
- For thumbnail generation, rasterize via a safe transcoding pipeline (ImageMagick with policy limits) rather than browser rendering.

---

## Related files

- [[Endless-JPEG]] — canvas feedback loop exploitation.
- [[Entropy-Poisoning]] — context window and embedding attacks.
