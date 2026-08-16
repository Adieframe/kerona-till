# Kerona on the go — till

Offline sales tracker for the Polokwane activation (2026-09-05, two trading days).
One device, König line only: Cappuccino · Tea · Hot Chocolate · Soup, R15 flat per cup.

No accounts, no server, no reception needed. Everything is stored in the device's
local storage.

**Status: built and functionally tested, not yet deployed.** Built 2026-08-16.
See [Next actions](#next-actions) for what is still open.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole app — markup, CSS, JS and the SVG icon set, all inline |
| `manifest.webmanifest` | PWA manifest: name, colours, icons, standalone display |
| `sw.js` | Cache-first service worker; bump `CACHE` on every change |
| `icon-180/192/512(-maskable).png` | Home-screen icons — the real Kerona pod mark, cream on near-black |

Icon source and tooling live in `../brand/`: `kerona-mark.svg` (the mark alone,
`currentColor`), the two full lockups, `pdf2svg.py` (converts the brand PDF to
SVG — no converter is installed on this machine) and `make_icons.py` (rasterises
the mark to PNG — no SVG rasteriser either, so it flattens and scanline-fills the
paths itself). Brand orange is **#D6701A**.

The icons are **square on purpose**. iOS and Android apply their own corner mask
to home-screen icons; baking rounded corners into the PNG makes the OS round an
already-rounded shape and leaves visible notches at the corners. `icon-512-maskable.png`
holds the mark at 46% so it survives Android's more aggressive crop.

## How the team uses it

**Till side**
- Tap a drink once per cup. Each drink has its own colour — milky coffee, rooibos
  red, dark brown, yellow — so the tile is found by hue before the word is read.
  A white pill on the tile shows that drink's count.
- The white strip under the drinks is the basket: total cups in this transaction,
  with the rand next to it. **Undo** sits beside it and removes the last tap.
- **CASH** or **CARD** records the sale and clears the screen for the next customer.
- After submitting, a black bar appears **above** the pay buttons offering **UNDO**
  for 4 seconds — that voids the sale just recorded (wrong payment button, wrong
  customer). Tapping any drink to start the next customer dismisses it early.

**Totals side**
- Tap the black header bar (`Totals ▸`) to flip. Tap `◂ Back to till` to return.
  Only the header flips, so the product grid can't flip by accident mid-service.
- Shows cups and rand per product, and the cash/card split, for **Today** or **Both days**.
- **Copy summary** puts a plain-text summary on the clipboard — paste into WhatsApp.
- **Download CSV** saves every transaction (one row per sale) to the device.

- **Reset local data** — small underlined text at the very bottom. It is *not* a
  tappable button: press and **hold it for 3 seconds**, at which point it turns
  red and reads "Keep holding…", then the phone buzzes and a confirm dialog
  appears. Letting go early cancels and nothing happens. Export the CSV first —
  this cannot be undone.

## Deploying it

The app needs to be served over HTTPS **once** so the device can install it.
After that it runs with no connection at all.

1. Put these files in the root of a **new, separate public repo** — not the personal
   context repo, which would expose everything else in it.
2. Settings → Pages → deploy from `main` / root.
3. On the device, open the Pages URL **while there is reception**, then
   Share → *Add to Home Screen* (iOS) or ⋮ → *Install app* (Android).
4. Open it from the home-screen icon and put the phone in airplane mode to confirm
   it still loads before the day.

## Before the day — worth doing

- Run 20 practice sales, check the totals, download the CSV, then wipe them with
  **Reset local data** at the bottom of the totals page so the team starts the
  day on zero.
- Turn off auto-updates and low-power mode on the device; the app requests a screen
  wake lock but the OS can still override it.
- The data lives in browser local storage for this origin. Do not clear browsing
  data, and do not use private/incognito mode.

## The artwork

The four drink illustrations are inline SVG line art, defined once as `<symbol>`s
at the top of `index.html` and referenced with `<use>`. They inherit the tile's
text colour, so the same drawing works on a light tile and a dark one.

**Cappuccino** coffee bean · **Tea** rooibos sprig · **Hot Chocolate** cacao pod ·
**Soup** hen.

The tea icon is a **rooibos** sprig — needles on a stem, the way the plant
actually grows — not the broad leaf used for black tea. The button still reads
"Tea" because that is what is on the menu, but the drawing says rooibos.

There is no logo in the header — just the wordmark.

## Changing things

- Price: `PRICE` at the top of the `<script>` in `index.html`.
- Products: the `PRODUCTS` array, same place. Four fits the 2×2 grid; more needs a
  grid change.
- After editing any file, bump `CACHE` in `sw.js` or installed devices keep serving
  the old version.

## What has been tested

Driven through a real browser at 375×812. Verified: empty basket on load; exactly
+1 per tap; per-drink counts; **Undo** removing only the last tap; both payment
paths writing correct records; screen clearing after submit; the undo strip
appearing, voiding the right sale, self-clearing at 4s and dismissing on the next
drink tap; totals maths per product and per payment method; the Today/Both-days
scope switch; the summary text; the CSV contents; the 3-second footer press
(short press does nothing, long press + confirm erases); persistence across
reload; and no broken SVG references.

**Not tested: offline.** Service-worker registration is blocked in the local
preview harness, so cache-first behaviour has never actually run. `sw.js` serves
correctly (200, right MIME) but the offline path needs one real check on the
HTTPS deploy — step 4 of *Deploying it*. This is the single highest-value thing
left to verify, because it is the whole premise of the tool.

## Build log

| Date | Change |
|---|---|
| 2026-08-16 | First build: 4 drink buttons, cash/card submit, flip-to-totals, CSV + clipboard export, local-storage persistence, wake lock |
| 2026-08-16 | Basket counter moved **below** the drinks with **Undo** beside it; title set to "Kerona on the go"; per-drink tile colours |
| 2026-08-16 | Erase-all removed from the totals screen (too easy to hit); replaced with a hidden 3-second press on the footer + confirm |
| 2026-08-16 | Line-art illustrations added to each drink; Kerona pod mark added to the header |
| 2026-08-16 | Header mark removed again (didn't look right); its paths stripped from the file |
| 2026-08-16 | Tea icon changed from a broad tea leaf to a rooibos needle sprig |
| 2026-08-16 | Undo strip cut 6s → 4s, moved above the pay buttons, slimmed, and set to dismiss on the next drink tap |
| 2026-08-16 | Home-screen icons replaced with the real Kerona pod mark, converted from the brand PDF (cream on near-black) |
| 2026-08-16 | Reset surfaced as visible small print — "Reset local data" at the foot of the totals page, still a 3-second hold + confirm |

## Next actions

1. **Change the drink button colours** to what Yana asked for. Current values are
   the four `--cappuccino / --tea / --hotchoc / --soup` pairs at the top of the
   `<style>` block (each has a matching `-edge` border colour). _(to confirm:
   Yana's actual colours.)_
2. **Confirm the price: R20 rather than R15.** ⚠️ Two things to settle here, and
   they are different problems:
   - If the price is simply R20 for everything, it is a one-line change (`PRICE`).
   - If R20 applies **only to cash** and card stays R15 (or any split by payment
     method), the app cannot do it as written — it has a single flat `PRICE` and
     computes the basket total before the payment method is known. That needs a
     per-method price and a rework of the basket display, which shows a rand
     figure before CASH or CARD is pressed.

   Note this also moves the numbers in [[sa-coffee]], where the activation
   economics are all modelled at R15/cup.
3. **Ndivhu's decision on a cash calculator** — whether the till should also work
   out change owed to cash customers. If yes it needs a "cash tendered" entry and
   a change-due display, which is a real addition to the submit flow, not a tweak.
4. **A second version for the Kerona product** (Dunefoods superfood line, the
   other activation point). One product instead of four, otherwise the same app.
   Cleanest route is a separate copy with its own `PRODUCTS` array and its own
   storage key, so the two devices never share or overwrite data.

Also still open, from the build rather than from the team:

- **Deploy and do the airplane-mode check** (see *Deploying it*).
- **Home-screen icon** is still the generic cup drawn at the start, not a Kerona
  mark. Cosmetic, but it is what the team taps on the day.
