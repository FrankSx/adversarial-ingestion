# Adoption Agency DOM Harness

**Target:** HTML5 tree-builder mutations where formatting tags closed out of order cause elements to migrate outside their assumed sanitization boundaries.

---

## The HTML5 adoption agency algorithm

When the HTML5 parser encounters a formatting element (like `<b>`, `<i>`, `<a>`) that is closed out of order relative to other open formatting elements, it runs the **Adoption Agency Algorithm** (AAA). This algorithm reconstructs the DOM tree by "adopting" the mis-nested element into a different parent — often completely outside its original container.

The algorithm is deterministic. It runs the same way in every HTML5-compliant browser. But it is also one of the most complex parts of the HTML5 parsing spec, and it behaves in ways that surprise both developers and security engineers.

---

## How the AAA works (simplified)

1. If a formatting element is closed but other formatting elements were opened after it, the parser finds the "furthest block" — the nearest block-level ancestor.
2. It removes the formatting element from its current position.
3. It re-inserts the formatting element (and any children) into the furthest block, reconstructing the tree.
4. The result: content that was logically inside one container now appears inside another.

---

## The security implication

A server-side sanitizer (like DOMPurify, Bleach, or a custom regex filter) makes decisions about what content is allowed based on where it appears in the DOM. If the sanitizer's parser builds a different tree than the browser's parser, the sanitizer may allow content that the browser will later execute in an unexpected context.

Example scenario:

```html
<div class="user-content">
  <b><div>user text</b> more text</div>
</div>
```

A naive sanitizer might see:
- `<b>` inside `.user-content` → allowed.
- `<div>` inside `<b>` → invalid but harmless.
- Closing `</b>` before `</div>` → malformed, but text content is safe.

The browser's HTML5 parser runs the AAA and produces:

```html
<div class="user-content">
  <b></b>
  <div><b>user text</b> more text</div>
</div>
```

The `<div>` has migrated OUT of the `<b>` and is now a direct child of `.user-content`. If the sanitizer's CSS injection filter was only watching for `<div>` inside `<b>`, it missed the reconstructed tree.

---

## Mutation XSS (mXSS) via AAA

The classic mXSS attack exploits the gap between sanitizer output and browser parsing:

1. Attacker submits: `<p style="/*\">*/\nx:expression(alert(1))//">`
2. Sanitizer parses, sees a harmless style attribute, allows it.
3. Browser re-parses the sanitized string, the AAA runs, the comment boundary shifts.
4. Result: `expression(alert(1))` executes.

The adoption agency is one of several HTML5 mechanisms that cause this divergence. Others include:

- **Foster parenting**: elements that should be inside `<table>` get hoisted out.
- **Implied tags**: `<tbody>`, `<head>`, `<body>` inserted automatically.
- **Parse errors**: recovery behaviors that differ between parsers.

---

## Testing the divergence

To identify if a sanitizer is vulnerable:

1. Feed it deeply nested, mis-ordered formatting elements.
2. Compare its output DOM tree against Chrome/Firefox/Safari.
3. Look for elements that changed parents during browser parsing.
4. Check if event handlers, style attributes, or script URLs moved into allowed contexts.

---

## Defensive checklist

- Use a sanitizer that parses with the SAME engine as the target browser (e.g., DOMPurify uses the browser's own DOMParser).
- Never sanitize once and assume the result is stable. The browser may re-parse.
- Test with mis-nested `<b>`, `<i>`, `<a>`, `<font>`, `<u>` inside block elements.
- Watch for attribute values that contain comment boundaries (`/*`, `*/`) or escape sequences.
- Run sanitizer output through a second parse-and-serialize cycle before deployment.

---

## Related files

- [[Invisible-Gutters]] — trailing data exploitation.
- [[Traversal-String-Confusion]] — URL/path parser divergence.
