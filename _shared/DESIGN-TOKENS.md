# Direction A — Warm Editorial · design tokens

Locked 2026-08-20 when Direction A was approved. Every template in this repo uses
these values and nothing else. If a value is not on this list, it does not belong
in a Petite Mort transactional email.

## Colour

| Token | Hex | Use |
|---|---|---|
| Ink | `#2c2d2e` | Headings, primary text, CTA fill, support card fill, footer band |
| Ink 70 | `#57585a` | Body copy |
| Ink 45 | `#8a8a8a` | Eyebrow labels, meta, muted copy |
| Ink 30 | `#b4b4b4` | Legal footnote |
| Terracotta | `#b8543d` | Accent — active state, links, discount, panel border |
| Terracotta deep | `#7a5348` | Copy *inside* a blush panel (contrast on tint) |
| Blush | `#f3d6cd` | Positive/neutral highlight panel, chips, links on dark |
| Paper | `#ffffff` | Card |
| Shell | `#efeae6` | Page background outside the card |
| Sand | `#faf7f5` | Secondary card (gifts, notes) |
| Sand deep | `#f6f2f0` | Image placeholder, negative-state panel |
| Hairline | `#ebe6e3` | Dividers |
| Rail idle | `#e0d5d0` | Inactive progress dot |

**Negative states (cancelled, refund-pending) do not use terracotta or blush as the
lead panel.** Bad news in a warm celebratory tint reads as tone-deaf. Those use Sand
deep `#f6f2f0` with a `#b4b4b4` left border, and terracotta returns only for the
money line, which is the one thing the customer wants to find.

## Type

Inter, falling back to `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`.
No webfont link — Inter is not reliably available in email clients, and the fallback
stack is what most recipients will actually see. Do not add `@import`; Outlook strips
it and Gmail ignores it.

| Role | Size | Weight | Tracking |
|---|---|---|---|
| H1 | 30px / 1.2 | 600 | −.02em |
| H2 (in-body) | 22px | 600 | −.01em |
| Body | 15px / 1.65 | 400 | — |
| Body small | 13.5px / 1.6 | 400 | — |
| Eyebrow | 11px | 600 | .14em, uppercase |
| Meta | 12.5px | 400 | — |

## Geometry

- Card width `600px`, `max-width:100%`, radius `16px`
- Horizontal padding `40px` (mobile clients collapse gracefully; no media queries needed at 600px)
- Buttons radius `25px`, padding `15px 40px`
- Panels radius `14px`, secondary cards `12px`, thumbnails `10px`
- Accent panel left border `4px solid`

Radii match the live theme's `--button-border-radius: 25px` and `--block-border-radius: 16px`.

## Build rules

1. **Tables, not divs, for layout.** `role="presentation"` on every layout table.
2. **Inline styles only.** No `<style>` block, no classes — Gmail strips head styles
   on forwarded mail and Outlook.com rewrites them.
3. **No background images, no flexbox, no grid, no `position`.**
4. Every `<img>` gets explicit `width`, `alt`, and `style="display:block;border:0"`.
5. Progress rail is built from table cells and coloured dots, never an image, so it
   survives image-blocking (the default in Outlook and many corporate clients).
6. A preheader `<div>` sits first in `<body>`, hidden, carrying the one line that
   matters — it is the second thing after the subject that a recipient reads.
