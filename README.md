# Kerona on the go — till

Offline sales tracker for the Polokwane activation (2026-09-05, two trading days).
**Two tills in one app**, chosen per device on first launch:

| Till | Products | Tile colour |
|---|---|---|
| **König** | Cappuccino · Tea · Hot Chocolate · Soup | tan / red / deep blue / yellow |
| **Kerona** | Kerona (single product) | navy `#170833` |

**R20 flat per cup**, same price for cash and card, both tills.

No accounts, no server, no reception needed. Everything is stored in the device's
local storage.

**Status:** deployed at **https://adieframe.github.io/kerona-till/** (repo
`Adieframe/kerona-till`). The single-till version was installed to an iPhone and
**confirmed working offline on 2026-08-16**. The two-till version built
**2026-08-17** is tested in the browser but **not yet uploaded or re-checked on
device** — see [Next actions](#next-actions).

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
- On first launch the device asks **which counter it is on**. That choice is
  remembered; the chooser does not appear again unless someone uses *Switch till*.
- Tap a product once per cup. Each has its own colour, so the tile is found by hue
  before the word is read. A white pill on the tile shows that product's count.
  On the Kerona till there is a single full-screen button.
- The white strip under the drinks is the basket: total cups in this transaction,
  with the rand next to it. **Undo** sits beside it and removes the last tap.
- **CASH** or **CARD** records the sale and clears the screen for the next customer.
- After submitting, a black confirmation bar slides up across the bottom offering
  **UNDO** for 4 seconds — that voids the sale just recorded (wrong payment button,
  wrong customer). It sits over the pay buttons, which is fine: tapping any product
  to start the next customer dismisses it immediately.

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
  this cannot be undone. It only clears **this** till — the other till's
  totals are untouched.

## Deploying it

The app needs to be served over HTTPS **once** so the device can install it.
After that it runs with no connection at all.

1. Put these files in the root of a **new, separate public repo** — not the personal
   context repo, which would expose everything else in it.
2. Settings → Pages → deploy from `main` / root.
3. On the device, open the Pages URL **while there is reception**, then
   Share → *Add to Home Screen* (iOS, Safari only) or ⋮ → *Install app* (Android).
   iOS then asks how to open it — choose **web app**, not "open in Safari".
4. Open it from the home-screen icon and put the phone in airplane mode to confirm
   it still loads before the day.

### ⚠️ Always open it from the home-screen icon

On iOS the installed web app gets its **own storage, separate from Safari**. The
same URL opened in a Safari tab is a different, empty till, and sales recorded
there will not appear in the app's totals. Anyone opening it "just to check" in a
browser tab splits the day's numbers across two stores with no warning.

Deleting the home-screen icon deletes its data. Export the CSV first.

## Before the day — worth doing

- Run 20 practice sales, check the totals, download the CSV, then wipe them with
  **Reset local data** at the bottom of the totals page so the team starts the
  day on zero.
- Turn off auto-updates and low-power mode on the device; the app requests a screen
  wake lock but the OS can still override it.
- The data lives in browser local storage for this origin. Do not clear browsing
  data, and do not use private/incognito mode.

## The artwork

The five product illustrations are inline SVG line art, defined once as `<symbol>`s
at the top of `index.html` and referenced with `<use>`. They inherit the tile's
text colour, so the same drawing works on a light tile and a dark one.

**Cappuccino** coffee bean · **Tea** rooibos sprig · **Hot Chocolate** cacao pod ·
**Soup** hen · **Kerona** the brand pod mark, converted from the vector PDF.

The tea red and the hot-chocolate blue are sampled from the packaging Yana sent
(Skimmelberg Buchu box, noa & co Deep Sleep bag), white-balanced against the white
print on each pack. Kerona's navy is sampled from the "Coffee here" banner.

The tea icon is a **rooibos** sprig — needles on a stem, the way the plant
actually grows — not the broad leaf used for black tea. The button still reads
"Tea" because that is what is on the menu, but the drawing says rooibos.

There is no logo in the header — just the wordmark.

## Changing things

- Price: `PRICE` at the top of the `<script>` in `index.html`. One constant, used
  by both tills and both payment methods.
- Tills and their products: the `TILLS` object, same place. Each till has a `name`,
  a `slug` (used in the CSV filename), a `store` key and a `products` list. Four
  products fit the 2×2 grid and one gets a full-screen button; anything else needs
  a grid change.
- Colours: the `--<product>` / `--<product>-edge` pairs at the top of the `<style>`.
- After editing any file, bump `CACHE` in `sw.js` or installed devices keep serving
  the old version.

## What has been tested

Driven through a real browser at 375×812. Verified: empty basket on load; exactly
+1 per tap; per-drink counts; **Undo** removing only the last tap; both payment
paths writing correct records; screen clearing after submit; the undo strip
appearing, voiding the right sale, self-clearing at 4s and dismissing on the next
drink tap; totals maths per product and per payment method; the Today/Both-days
scope switch; the summary text; the CSV contents; the 3-second press on *Reset
local data* (short press does nothing, long press + confirm erases); persistence
across reload; and no broken SVG references.

Re-verified for the two-till build (2026-08-17): the chooser appears on a fresh
device and not again afterwards; the choice survives a reload; each till shows
only its own products; **the two stores stay completely independent** (recording
on one leaves the other's totals untouched); *Switch till* returns to the chooser
without deleting anything; R20 pricing on both tills and both payment methods;
CSV filenames come out as `kerona-konig-<date>.csv` and `kerona-kerona-<date>.csv`;
the summary text names the till; and data from the old single-till build is
migrated into the König store on first run.

**Offline: confirmed on device, 2026-08-16.** Installed to an iPhone home screen
from the Pages URL, then opened in airplane mode — it loaded and ran. That was
the last unverified assumption and it is the whole premise of the tool, so it is
worth re-checking after any deploy that changes `sw.js`.

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
| 2026-08-16 | Published to GitHub Pages, installed on an iPhone home screen, **offline confirmed in airplane mode** |
| 2026-08-17 | Price R15 → **R20**, flat across products and payment methods |
| 2026-08-17 | Tea red and hot-chocolate blue resampled from Yana's packaging references; edge padding increased; header extended under the status bar; `dvh` ordering fixed so the pay buttons stop clipping |
| 2026-08-17 | Confirmation strip moved back to the bottom of the screen; "Tap a drink to start" → "Tap a product" |
| 2026-08-17 | **Second till added**: a chooser on first launch, a single-product Kerona till in brand navy with the pod mark, separate storage per till, and *Switch till* on the totals page |

## Next actions

1. **Upload the two-till build and re-check it on the phone.** The device is still
   running the single-till version. After uploading, open it **with reception** so
   the new service worker installs, then confirm the chooser appears, pick the
   right till, and re-test airplane mode. Until then nothing on this page is live.
2. **Ndivhu's decision on a cash calculator** — whether the till should also work
   out change owed to cash customers. If yes it needs a "cash tendered" entry and
   a change-due display, which is a real addition to the submit flow, not a tweak.
   Still open.
3. **Check the R20 price against the rest of the plan.** The till is now R20 flat,
   but [[sa-coffee]] models the whole activation at R15/cup — the R150,000 revenue
   figure, the per-cup margins and the R4,725 Yoco fee all move. That is a
   spreadsheet job, not an app job, but it follows from this change.
4. **Confirm the Kerona product name and price on the menu.** The till shows it as
   "Kerona" at R20, same as the König line. _(to confirm: whether the superfood
   drink is actually sold at R20.)_

Done since the last round: Yana's colours applied, R20 pricing, edge padding and
the status-bar gap, confirmation strip moved to the bottom, and the second till.

### How updates get deployed

Files are currently on GitHub by **manual web upload** — there is no GitHub
authentication on this Mac (no SSH key, no credential helper), so pushes are not
possible from here yet. To change that, generate an SSH key and add the public
half to the GitHub account; after that, deploys are one command.

Either way, for every deploy:

1. Bump `CACHE` in `sw.js`, or installed devices keep serving the old version.
2. Upload the changed files.
3. On the device, open the app **with reception** so the new service worker
   installs, then re-check airplane mode if `sw.js` changed.
