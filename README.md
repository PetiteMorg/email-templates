# Petite Mort — Transactional Email Templates

The post-purchase notification set sent from Shopify, in the approved Direction A
(Warm Editorial) language.

**Nothing here is live.** No Shopify template has been changed. Shopify has no draft
state for transactional notifications, so this repo *is* the draft — read
[`INSTALL-RUNBOOK.md`](INSTALL-RUNBOOK.md) before pasting anything.

---

## How to view

Open [`index.html`](index.html) for the whole set as a gallery — that's the link to share
for review. Or open any file below directly.

| Email | Preview | Production Liquid | Shopify template |
|---|---|---|---|
| Order confirmation | [preview](order-confirmation/direction-a-warm-editorial.html) | [liquid](order-confirmation/shopify.liquid) | Order confirmation |
| Shipping confirmation | [preview](shipping-confirmation/direction-a-warm-editorial.html) | [liquid](shipping-confirmation/shopify.liquid) | Shipping confirmation |
| Shipping update | [preview](shipping-update/direction-a-warm-editorial.html) | [liquid](shipping-update/shopify.liquid) | Shipping update |
| Out for delivery | [preview](out-for-delivery/direction-a-warm-editorial.html) | [liquid](out-for-delivery/shopify.liquid) | Out for delivery |
| Delivered | [preview](delivered/direction-a-warm-editorial.html) | [liquid](delivered/shopify.liquid) | Delivered |
| Order cancelled | [preview](order-cancelled/direction-a-warm-editorial.html) | [liquid](order-cancelled/shopify.liquid) | Order canceled |
| Refund notification | [preview](refund/direction-a-warm-editorial.html) | [liquid](refund/shopify.liquid) | Order refund |

Previews carry sample data and are the design artefact. The `.liquid` files are what goes
into Shopify.

---

## Brand palette

Pulled from the live theme, not invented. Full token list in
[`_shared/DESIGN-TOKENS.md`](_shared/DESIGN-TOKENS.md).

| Role | Hex | Where it comes from |
|---|---|---|
| Ink / text | `#2c2d2e` | `--color-body`, also the footer background |
| Terracotta accent | `#b8543d` | the notice component on the storefront |
| Blush tint | `#f3d6cd` | same component's background |
| Hairline | `#e5e5e5` | `--color-border` |
| Paper | `#ffffff` | `--bg-body` |

Buttons use a 25px radius and cards 16px, matching `--button-border-radius` and
`--block-border-radius`. Typeface is Inter, as on the site.

The theme's default accent `#3F72E5` is deliberately unused — it ships with the theme and
carries no brand meaning.

Negative states — cancellation, refund — swap the blush panel for a muted `#f6f2f0` with a
grey rule, keeping terracotta only on the money. Bad news in a warm celebratory tint reads
as tone-deaf.

---

## What every template covers

Beyond the standard order summary:

- **A delivery estimate**, computed per market from our own lead times, because the store
  has no carrier accounts and Shopify's estimated delivery dates are off. Weekend
  endpoints are pushed to the Monday — no market here delivers at weekends.
- **A progress strip** — Confirmed → Packed → Shipped → Delivered, advancing with the
  email, built from table cells so it survives image blocking.
- **FAQs chosen to remove the tickets we actually get**, including why an order shows more
  line items than were bought (sets are expanded into individual bottles by the Bundle
  Expander app).
- **The thank-you card and €10 gift card named explicitly**, so the customer expects them
  before opening the box.
- **No separate VAT line.** VAT is deliberately disabled customer-facing on this store
  (confirmed by Amed 21 Aug — it is priced in and settled in the accounting back end), and
  Shopify's tax regions collect 0% everywhere. The order confirmation still carries a
  `{% if tax_price > 0 %}` guarded VAT row, so it renders nothing today and appears
  automatically if a tax registration is ever added. Nothing to change to install.
- **A support block with a stated response time**, and on the money emails an explicit
  "come to us before your bank" — a chargeback costs six to eight weeks and a fee to reach
  the answer we'd give the same day.

## Verification

Every Liquid template was rendered offline across 11 data permutations — including partial
shipment, unpaid cancellation, partial refund, no-discount, and paid shipping — with all
computed dates checked by hand. All pass.

That harness is python-liquid, not Shopify's engine: it catches syntax, logic and
filter-chain faults, not variable-name faults. Every variable was then checked against
Shopify's notification-variable reference and Liquid object docs — which turned up one real
bug (`fulfillment.updated_at` does not exist, and the Delivered email was using it for the
delivery time) plus two wrong names. All fixed. The handful still unconfirmed are listed in
the runbook, and each is guarded so a miss renders nothing rather than breaking.

---

## Planned

- Copy adaptation into Dutch first, then SV, DA, FI — **blocked on an open question about
  whether these templates have per-language versions in the admin**, which changes the
  size of the job by about 6×. See runbook §6.
- A holiday table per market, so windows spanning Easter or Christmas stop reading
  optimistically. Before Q4.
- Shopify Flow writing an order metafield, if we want the "previously arriving…" line on
  the shipping update to be real rather than dropped.

Design first, then copy, then adaptation — adapting before the design is settled throws
the adaptation away.
