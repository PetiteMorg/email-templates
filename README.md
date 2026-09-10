# Petite Mort — Transactional Email Templates

The post-purchase notification set, in the approved Direction A (Warm Editorial) language.

**This repository no longer describes a Shopify project.** It did until September 2026, and the
Shopify framing is what made this set hard to follow. The Shopify store is gone. Three of these
seven emails are now rendered and sent by our own Medusa store; the other four are the parcel
lifecycle. The `.liquid` files can no longer run anywhere.

To review the set, open **[`preview.html`](preview.html)**, which renders all seven inline. That
is the link to share. [`index.html`](index.html) is the older card gallery and still works.

---

## Who sends what

| Email | Sent by | State |
|---|---|---|
| Order confirmation | **The Medusa store** | Code is on `main`. Not sending, see below. |
| Order cancelled | **The Medusa store** | Same. |
| Refund notification | **The Medusa store** | Same. |
| Shipping confirmation | QLS | **Not verified.** See the open question below. |
| Shipping update | QLS | Not verified. |
| Out for delivery | QLS | Not verified. |
| Delivered | QLS | Not verified. |

The three the store sends are wired to its own events:

```
order.placed      ->  src/subscribers/order-confirmation-email.ts
order.canceled    ->  src/subscribers/order-cancelled-email.ts
payment.refunded  ->  src/subscribers/order-refund-email.ts
```

in `PetiteMorg/pm-medusa-store`, rendering through `src/lib/order-emails/`.

### The open question, and it is not small

**Nothing in the store's code or documentation says QLS emails the customer.** The only record of
that arrangement is a line in Notion. If QLS is not actually configured to send them, then nobody
sends shipping, delivery or out-for-delivery notifications at all, and the customer hears nothing
between paying and the parcel arriving. That needs confirming in the QLS dashboard, not assumed
from this table.

---

## Can these send today? No.

Read from the store's own admin on 10 September 2026, Settings → Notifications:

```
Provider   none beyond local
Sender     info@petitemort.co
```

> Only the local provider is registered, which logs instead of sending.

Two separate things both have to change before a customer receives anything:

1. **A provider has to exist.** `RESEND_API_KEY` is not set in production, so the local provider
   is the only one registered and it writes to the log. Resend is the chosen provider, not
   Klaviyo, because these are transactional: a receipt has to arrive whether or not someone
   accepted marketing, and Klaviyo would want the copy to live in Klaviyo.
   **`RESEND_API_KEY` and `REVIEW_EMAIL_FROM` must be set together** or the notification module
   throws at boot and the whole backend fails to start.
2. **The safety switch has to be turned up.** `ORDER_EMAIL_MODE` defaults to `off`, which renders
   and logs but transmits nothing. `allowlist` sends only to named addresses; `live` sends to
   customers. It already stopped a real customer address on the first test, which is the reason
   it exists.

---

## Where the design actually lives now

The seven `direction-a-warm-editorial.html` files are the design source and the thing to review.
They carry sample data.

For the three the store sends, the design is re-implemented in code in
`src/lib/order-emails/design.ts` and `render.ts`, because an email has to be built from a live
order rather than filled into a static file. **The two were compared on 10 September 2026 and
match**: the same twelve colours and the identical font stack
(`Inter, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif`).

Nothing enforces that. `design.ts` names this repository as its source of truth but does not read
from it, so the two can drift with no test failing. If a change is made here, it has to be made
there in the same breath.

## Archive, kept for reference, cannot run

- **The seven `shopify.liquid` files.** These were the production templates when the store was on
  Shopify. Medusa does not read Shopify Liquid, and the Shopify store no longer exists. They are
  kept because they hold the approved copy and the guarded edge cases, which is worth having when
  the code version is extended. They are not a deployment target.
- **[`INSTALL-RUNBOOK.md`](INSTALL-RUNBOOK.md)** is the instructions for pasting those templates
  into the Shopify admin. There is no Shopify admin. Read it as history.

The logo address in all fourteen files was corrected on 10 September 2026. Every one of them
pointed at `petitemort.co/cdn/shop/files/...`, which stopped serving images when the Shopify store
closed and now answers with the maintenance page as HTML, so the logo was arriving blank. They
now use `cdn.petitemort.co`.

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

Buttons use a 25px radius and cards 16px. Typeface is Inter, as on the site.

Negative states, cancellation and refund, swap the blush panel for a muted `#f6f2f0` with a grey
rule, keeping terracotta only on the money. Bad news in a warm celebratory tint reads as
tone-deaf.

---

## What every template covers

Beyond the standard order summary:

- **A delivery estimate**, computed per market from our own lead times, because the store has no
  carrier accounts. Weekend endpoints are pushed to the Monday, since no market here delivers at
  weekends.
- **A progress strip**, Confirmed → Packed → Shipped → Delivered, advancing with the email and
  built from table cells so it survives image blocking.
- **FAQs chosen to remove the tickets we actually get**, including why an order shows more line
  items than were bought, since sets are expanded into individual bottles.
- **The thank-you card and €10 gift card named explicitly**, so the customer expects them before
  opening the box.
- **No separate VAT line.** VAT is disabled customer-facing on this store, confirmed by Amed on
  21 August: it is priced in and settled in the accounting back end. The Liquid version guards
  the VAT row behind `{% if tax_price > 0 %}` so it appears automatically if a registration is
  ever added. The code version reads the order's own tax total; whether it behaves the same way
  at zero has not been checked against a real order.

## Verification, and what it was verification of

Every Liquid template was rendered offline across 11 data permutations, including partial
shipment, unpaid cancellation, partial refund, no-discount and paid shipping, with all computed
dates checked by hand. All passed, and the pass caught a real bug:
`fulfillment.updated_at` does not exist, and the Delivered email had been using it for the
delivery time.

That work tested **the Liquid files**, which no longer run. It is not evidence about the code
version that now sends. The code version has been rendered against real orders through
`src/scripts/preview-order-email.ts` and looked at, which is a weaker guarantee than 11
permutations and worth strengthening.

---

## Open

- **Confirm QLS actually emails the customer** for the four parcel emails. Everything else here is
  moot if it does not.
- **Set `RESEND_API_KEY` and `REVIEW_EMAIL_FROM`** in production, together, then verify the
  backend still boots.
- **Send one real test** with `ORDER_EMAIL_MODE=allowlist` before going anywhere near `live`.
- **Dutch copy** for the three the store sends. The per-language question that blocked this on
  Shopify does not apply any more: the code carries EN and NL side by side in
  `src/lib/order-emails/copy.ts`.
- **A drift test** so this repository and `design.ts` cannot disagree silently.
