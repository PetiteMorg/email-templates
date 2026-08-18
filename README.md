# Petite Mort — Transactional Email Templates

Design directions for the post-purchase notification emails sent from Shopify.

**Nothing here is live.** These are proposals for review — no Shopify template has been changed.

---

## How to view

Open any `.html` file in a browser, or use the preview links below.

| Email | Direction | File |
|---|---|---|
| Order confirmation | A — Warm Editorial | [`order-confirmation/direction-a-warm-editorial.html`](order-confirmation/direction-a-warm-editorial.html) |

---

## Brand palette

Pulled from the live theme, not invented.

| Role | Hex | Where it comes from |
|---|---|---|
| Ink / text | `#2c2d2e` | `--color-body`, also the footer background |
| Terracotta accent | `#b8543d` | the notice component on the storefront |
| Blush tint | `#f3d6cd` | same component's background |
| Hairline | `#e5e5e5` | `--color-border` |
| Paper | `#ffffff` | `--bg-body` |

Buttons use a 25px radius and cards 16px, matching `--button-border-radius` and `--block-border-radius`. Typeface is Inter, as on the site.

The theme's default accent `#3F72E5` is deliberately unused — it ships with the theme and carries no brand meaning.

---

## What each template covers

Beyond the standard order summary, every direction includes:

- **A delivery estimate at the top** — currently absent from every customer touchpoint
- **A four-step progress strip** — Confirmed → Packed → Shipped → Delivered
- **Four FAQs inline**, chosen to remove the support tickets we actually get, including why an order shows more line items than were bought (sets are expanded into individual bottles by the Bundle Expander app)
- **The thank-you card and €10 gift card named explicitly**, so the customer expects them before opening the box
- **VAT shown separately** — required across the EU
- **A support block** with a stated response time

---

## Planned

- Directions B and C for order confirmation
- Then the rest of the post-purchase set: shipping confirmation, shipping update, out for delivery, delivered, refund, cancellation
- Copy strings extracted for translation into Swedish, Danish, Finnish and French once a direction is approved

Design first, then copy, then translation — translating before the design is settled throws the translation away.
