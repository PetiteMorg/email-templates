# Install & automation runbook

Post-purchase transactional emails · Direction A (Warm Editorial) · store `petitemortparfum`

---

## 1. Read this first: there is no draft state

The ask was "set up full automation, but make them draft." Half of that isn't possible
the way it sounds, and it matters:

| | Transactional notifications (this repo) | Marketing automations (Shopify Email) |
|---|---|---|
| Where | Settings → Notifications | Apps → Shopify Email → Automations |
| Draft state? | ❌ **None** | ✅ Yes, emails sit as Draft |
| Automatic? | ✅ Already — they fire on the event | Only once you activate the flow |
| Saving means | **Live on the next matching order** | Nothing until activated |

These seven emails **are** the automation. Shopify fires order confirmation the moment
an order is paid, shipping confirmation the moment a fulfilment is created, and so on.
There is no trigger to build and no switch to leave off. Saving a template body *is*
deploying it — there is no staging, no preview-only mode, and **no version history to
roll back to**.

So the draft lives here. This repo is the reviewable state:

- ✅ Seven previews with sample data, for Amed
- ✅ Seven production Liquid templates, render-tested
- ✅ Nothing pasted into Shopify — every live template is still untouched

Install is one paste per template, and it should happen in one sitting after approval,
not gradually.

> ⚠️ If what was actually wanted was a *marketing* post-purchase sequence — a review
> request, a cross-sell, a replenishment nudge — that is a different surface (Shopify
> Email automations) and it *does* support drafts. Say so and I'll build that instead;
> it does not replace these.

---

## 2. Prerequisites — clear before pasting

| # | Item | State | Blocking? |
|---|---|---|---|
| 1 | Amed's approval on Direction A | ✅ Confirmed | — |
| 2 | **Back up all 7 current template bodies** | ❌ Not done | 🔴 Yes — no undo in Shopify |
| 3 | Language versions decided (§6) | ❌ Open question | 🔴 Yes for NL/BE |
| 4 | Lead-time table signed off (§4) | ❌ Needs Azad/ops | 🟠 Ships a promise |
| 5 | Five gates: staging → QA → Loom → approval → 24–48h watch | Partially | 🟠 Project rule |

**Step 2 is not optional.** Shopify keeps no history of a notification template. Once
you overwrite the body, the previous version is gone unless you saved it. Copy each
current body into `_backup/<template-name>.liquid` first. I can capture all seven in one
pass — ask.

---

## 3. Install order and mapping

Paste in this order. Lowest-volume first, so any mistake reaches the fewest customers,
and the two highest-volume emails go last when you already trust the process.

| Order | File | Shopify template (Settings → Notifications) |
|---|---|---|
| 1 | `refund/shopify.liquid` | Refund notification |
| 2 | `order-cancelled/shopify.liquid` | Order cancelled |
| 3 | `out-for-delivery/shopify.liquid` | Out for delivery |
| 4 | `delivered/shopify.liquid` | Delivered |
| 5 | `shipping-update/shopify.liquid` | Shipping update |
| 6 | `shipping-confirmation/shopify.liquid` | Shipping confirmation |
| 7 | `order-confirmation/shopify.liquid` | Order confirmation |

For each one: back up the current body → paste → **Preview** → **Send test email** to
yourself → check on a phone → move on. Do not paste all seven then test.

---

## 4. The delivery window is our promise, not Shopify's

The flagship element of Direction A is an explicit arrival window. Shopify cannot supply
it on this store:

- **Estimated delivery dates: Off** store-wide (verified in admin 20 Aug)
- **Carrier accounts: None** — every rate is a manual flat rate
- Amed believes dates are already on; he's looking at the *product-page* badge, which is
  a theme/app surface, not this one. Worth separating explicitly.

So each template computes the window itself from a lead-time table plus the order or
fulfilment timestamp. Edit only the `case` block at the top of the file:

| Market | Order → delivered | Despatch → delivered |
|---|---|---|
| NL | 1–2 | 1–2 |
| BE | 2–3 | 1–2 |
| DE | 2–3 | 2–3 |
| SE · DK · IE · PL | 3–5 | 2–4 |
| FI | 4–6 | 3–5 |
| Anywhere else | 4–7 | 3–6 |

These are my starting numbers from the shipping audit, **not ops-confirmed**. Get Azad
to sign them off — every one of these prints a date to a customer.

**Weekends are handled.** A window endpoint landing on a Saturday or Sunday is pushed to
the Monday, because none of these markets deliver at weekends. If both ends collapse onto
the same day the email prints one date instead of a range. Without this the SE order
confirmation printed "Fri 21 – Sun 23 August", promising a Sunday delivery that cannot
happen.

**Known limitation:** the shift is calendar-based, not a real business-day calculation,
and it does not know public holidays. A window spanning Easter or Christmas will read
optimistically. Fixing that properly needs a holiday table per market — worth doing
before Q4, not now.

---

## 5. What Shopify cannot populate

Three design elements in the previews have no data behind them. I dropped them from the
Liquid rather than ship markup that renders blank:

| Element | Where | Why | To restore |
|---|---|---|---|
| Struck-through previous delivery date | Shipping update | No previous-estimate variable exists | Shopify Flow writing an order metafield on each tracking change |
| "Reason: sorting-centre delay in Malmö" | Shipping update | Carrier delay reasons aren't exposed | Same, or carrier API |
| "Signed for by S. Kilpimaa" | Delivered | No signatory field | Carrier API only |

The Delivered email uses `fulfillment.updated_at` as the delivery time — the closest real
proxy. It is usually right and occasionally reflects a later status write instead.

**Verify in Shopify's own preview**, because this was render-tested against python-liquid,
not Shopify's engine — that catches syntax and logic faults, not variable-name faults:

- `cancel_reason_label` on Order cancelled
- `refund_line_items` and `line.subtotal` on Refund notification
- `amount` on Refund notification (should be this refund, not the order total)
- `fulfillment.tracking_url` falling back to `order_status_url` when absent
- The date arithmetic (`| date: '%s' | plus: … | date: …`) rendering real dates, not numbers

---

## 6. 🔴 Open question — language versions

**These templates are English-only.** Petite Mort sells to NL and BE in Dutch, plus SE,
DK, FI and IE. If the live notification templates currently have per-language versions,
pasting English Liquid into the English version leaves the Dutch one untouched — and any
market whose version you *do* overwrite loses its localisation.

Before pasting, check in the admin whether Settings → Notifications shows a language
selector on these templates. Then:

- **If yes** — each language needs its own paste, so the copy must be adapted first.
  Seven templates × the published languages. The email strings are already isolated as
  the "Liquid templates (dev only)" bucket in `PM-MASTER-TRANSLATIONS.xlsx`; six of them
  exceed the Google Sheets cell cap, so they cannot go through the normal sheet route.
- **If no** — English ships to everyone, which for the Dutch home market is a regression
  worth naming to Amed before it happens, not after.

I did not resolve this because it changes the size of the job by roughly 6×, and the
answer is one look at the admin.

---

## 7. Carrier name accuracy

The Liquid prints `fulfillment.tracking_company` when present. That comes from the actual
fulfilment, so it should be the real carrier — unlike the *shipping rate names* at
checkout, which the audit found are cosmetic and wrong (three of five markets show "DPD
shipping" and none ship DPD).

Worth confirming on a real fulfilment per market before go-live. QLS brokers the carrier
per destination, so a mismatch between the email and the parcel is exactly the kind of
thing that generates the ticket the FAQ was meant to prevent. If it turns out unreliable,
delete the Carrier row and keep the tracking number.

Also: **Out for delivery only fires if the carrier reports that status back.** Expect it
on some markets and not others. It degrades silently, which is fine.

---

## 8. Test plan

Draft orders in the admin exercise the real templates without touching a customer.

| Case | How | Expect |
|---|---|---|
| Standard NL order | Draft order → mark paid | Window 1–2 days, weekday only |
| Standard SE order | Draft order, SE address | Window 3–5 days, weekday only |
| Discounted order | Apply a code | Discount row with the code chip |
| Partial fulfilment | Fulfil one line only | "The rest is coming separately" |
| Full refund | Refund everything | No partial block, no "Still paid" |
| Partial refund | Refund one line | Partial block + correct "Still paid" |
| Unpaid cancellation | Cancel an unpaid order | No money panel, "not been charged" |
| Images blocked | Disable images in the client | Progress rail still readable |
| Dark mode | iOS Mail dark | Panels legible, no black-on-black |

All eight data cases above are already covered in the offline render harness and pass.
Dark mode and image-blocking need a real client.

---

## 9. What I have not done

- ❌ Nothing pasted into Shopify — no live template changed
- ❌ No backups captured yet (§2, step 2)
- ❌ No marketing automation created or modified
- ❌ Not committed or pushed — the GitHub connector isn't authorised in this session, so
  `Uppercut-exe/email-templates` is unchanged. Local commit on request.
- ❌ Copy not adapted into any other language (§6)

**Unrelated but time-critical:** the admin banner "Shopify Payments payouts pause in 12
days" was showing on 20 Aug, so roughly **1 September 2026**. Store-owner action, not a
dev task, but payouts stopping outranks emails.
