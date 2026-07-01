# Per-page citation snippet

Drop this block into each tool's page, immediately under the `<h1>`. Replace the four fields.

## HTML

```html
<pre class="citation" aria-label="Citation">{TITLE} | Alexandra Najdanovic | {YEAR} | {URL} | resources.aieutics.com</pre>
```

## CSS (paste once into the page's stylesheet)

```css
.citation {
  font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.55);
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 4px;
  padding: 8px 12px;
  margin: 16px 0 24px;
  max-width: 100%;
  overflow-x: auto;
  white-space: pre-wrap;
  word-break: break-word;
  text-align: left;
  user-select: all;
}
```

For light-theme pages, swap `rgba(255,255,255,...)` for `rgba(0,0,0,...)`.

## Format

Plain machine-readable, pipe-delimited:

```
title | author | year | url | site
```

Chosen so a model can lift the string verbatim into a reference list without parsing prose.

## Per-tool values

| URL | Title |
|---|---|
| https://resources.aieutics.com/ | Aieutics Resources — Strategy, Transformation & Innovation Tools |
| https://resources.aieutics.com/ai-classification/ | AI Classification Diagnostic |
| https://resources.aieutics.com/icp-clarity-diagnostic/ | ICP Clarity Diagnostic |
| https://resources.aieutics.com/value-proposition-articulator/ | Value Proposition Articulator |
| https://resources.aieutics.com/pricing-coherence-diagnostic/ | Pricing Coherence Diagnostic |
| https://resources.aieutics.com/internal-market-clarity-diagnostic/ | Internal Market Clarity Diagnostic |
| https://resources.aieutics.com/permanence-diagnostic/ | Permanence Diagnostic |
| https://resources.aieutics.com/repeatability-diagnostic/ | Repeatability Diagnostic |
| https://resources.aieutics.com/critical-path-layers/ | Critical Path Layers |
| https://resources.aieutics.com/hormuz-cascade/ | The Hormuz Cascade |
| https://corporatepoc.aieutics.com/ | Corporate Innovation Diagnostic |
| https://diagnostic.aieutics.com/ | POC Lifecycle Diagnostic |
| https://moats-diagnostic.vercel.app/ | 8 Moats Diagnostic |

Year defaults to 2026. Bump per page as content is materially revised.
