# Diaspora — Referral Campaign Email

A production HTML email for the Diaspora referral campaign, built from four approved
Figma frames (Desktop/Mobile × Light/Dark) as **one** HTML document.

| File | Purpose |
| --- | --- |
| `Diaspora_Email.html` | The complete email. Paste into SendGrid as-is. |
| `ASSET_MIGRATION.md` | Asset inventory, verification status, and the outstanding to-do list. |
| `README.md` | This file. |

**Status:** structurally complete and asset-verified. Not yet shippable — the logo
asset and every link destination are still placeholders. See
[Before you send](#before-you-send).

---

## Architecture: one email, four states

The four Figma frames are four *rendering states of one email*, not four emails.
Every section, string, and image exists exactly **once** in the markup. There is no
desktop copy beside a mobile copy, and no light copy beside a dark copy.

```
A. Header          logo + social row
B. Hero            headline, body, Open Diaspora CTA, three-device mockup
C. Features        6 cards (Matchmaking → Business Discovery)
D. Referral        prize cards ($300 / $500), Start Referring CTA
E. How it works    4 numbered steps
F. Campaign details  period, winner selection, what counts
G. Final CTA       App Store + Google Play badges
H. Footer          unsubscribe, address, logo, social row
```

Layout is table-based throughout. No flexbox, CSS grid, JavaScript, absolute
positioning, or CSS transforms carry any critical layout. The three-device hero
mockup is a single flat raster asset — never reconstructed from layered HTML.

### How the responsive columns work

The feature and step grids are `<div>`s styled `display:inline-block`, **not** a
`<td>`-based grid. That choice is deliberate:

- The common `<tr>{display:block}` + `<td>{display:inline-block}` technique is
  unreliable in the Gmail apps — once the parent row is switched to `display:block`,
  the cells do not consistently re-flow.
- These columns are **50% wide by default**, so a client with *zero* media-query
  support still gets a clean 2-across grid. Desktop widens them to 33.3% (features)
  and 25% (steps) via `@media (min-width:601px)`.
- The failure mode is therefore two columns everywhere — never one stuck column, and
  never a horizontal scrollbar.
- Outlook desktop (Word engine) ignores `inline-block` entirely, so each grid is
  additionally wrapped in an **MSO-only ghost table** with explicit pixel widths
  (181px × 3, 136px × 4). Those widths are derived from the real 544px content box
  (640 − 24×2 outer padding − 24×2 body padding) and were verified against the
  rendered DOM.

Two sections use `stack-row` / `stack-cell` (`display:block` under 600px) to stack
from two columns to one: the hero and the campaign-details pair. Source order is
already the intended mobile order, so nothing needs reordering.

### How light and dark work

Light mode is carried entirely by **inline attributes and `bgcolor`**, so it needs no
CSS support at all and is correct everywhere by default. Dark mode is layered on top
as colour-only overrides — these rules never touch layout:

- `@media (prefers-color-scheme: dark)` — Apple Mail, iOS/iPadOS Mail, Outlook.com,
  Yahoo, and other honouring clients.
- `[data-ogsc]` / `[data-ogsb]` — the attributes Outlook (Windows and mobile) stamps
  onto elements whose foreground/background it rewrote.

> **The Gmail caveat.** The Gmail apps on iOS and Android support **neither**
> `prefers-color-scheme` nor `[data-ogsc]`. In Gmail dark mode you get Gmail's own
> automatic inversion, and there is no way to opt out. This is why light mode is the
> state that must be correct unconditionally and dark mode is treated as an
> enhancement — not the other way round.

### Gmail clipping

Gmail truncates a message body past roughly **102 KB** and hides the remainder behind
"View entire message". This file is **~57 KB — about 57% of that budget**, with the
App Store badges and unsubscribe link comfortably inside it.

Headroom comes from engineering, not from cutting campaign content: no Base64, no
duplicated desktop/mobile or light/dark markup, no inline SVG blobs, no Figma Dev Mode
output, and a single shared social-icon set rather than per-mode copies.

### Accessibility

- Real heading semantics: one `<h1>`, five `<h2>`, thirteen `<h3>` — verified in the
  rendered DOM, not just the source.
- Every layout table carries `role="presentation"`; the wrapper is
  `role="article"` with `lang="en"`.
- Decorative icons use `alt=""`. Meaningful images carry descriptive alt text, so the
  logo still reads as "Diaspora" and the hero describes the three app screens.
- No campaign message depends on images alone — every headline, prize amount, date and
  rule is live text.
- Text contrast meets WCAG AA in both palettes, including the muted footer
  (`#667085` on white; `#CFC7EE` on `#2C0A84`).

---

## Before you send

`Diaspora_Email.html` will not render correctly until these are replaced. Every one is
marked with the reserved host `TODO-REPLACE.invalid` (RFC 2606 — it can never resolve,
so a missed placeholder fails loudly instead of silently pointing somewhere real).

```bash
grep -n "TODO-REPLACE" Diaspora_Email.html
```

1. **Logo lockup** — two hosted images (light + dark), 264×48 for a 132×24 render.
2. **Link destinations** — 5 social profiles (×2 locations), Open Diaspora,
   Start Referring, unsubscribe, manage preferences.
3. **Postal address** — required by CAN-SPAM and its international equivalents.

Full detail and export specs are in `ASSET_MIGRATION.md`.

---

## SendGrid

Paste the complete file into a SendGrid HTML/Code editor. It contains no JavaScript,
no external stylesheet, and no invented template syntax.

**Recipient address.** The footer uses `{{email}}`, which resolves in a v3 **dynamic
template** when `email` is present in `dynamic_template_data`. If you send through
**Marketing Campaigns** instead, swap it for that editor's own insert-field token.
Confirm against SendGrid's current documentation for whichever sending mode you use —
an unresolved token renders as empty text, not as an error.

**Unsubscribe.** Do not hand-roll this. Wire the two footer links to SendGrid's
subscription-management tags so suppression is handled for you:

| Purpose | Tag |
| --- | --- |
| Unsubscribe from this group | `<%asm_group_unsubscribe_raw_url%>` |
| Manage preferences | `<%asm_preferences_raw_url%>` |
| Global unsubscribe | `<%asm_global_unsubscribe_raw_url%>` |

Verify these against SendGrid's docs for your account's sending mode before launch.

**One-click unsubscribe.** Gmail and Yahoo require bulk senders to support RFC 8058
one-click unsubscribe. That is a `List-Unsubscribe` / `List-Unsubscribe-Post` **header**,
configured at the send level — it cannot be set from this HTML file.

---

## Known deviations from Figma

These are deliberate, and each is a cross-client reliability trade:

1. **Top gradient fade.** The frames fade the outer canvas tone into the content
   surface over roughly the top 12% of the body. Reproduced as a flat surface colour —
   Outlook has no gradient support without VML, and the fade is near-invisible in light
   mode.
2. **Testimonials omitted.** "Real people. Real connections. Real impact." appears in
   both *desktop* frames but in **neither mobile frame**. Since this is one email with
   one set of content, it is omitted rather than hidden per-breakpoint — hidden content
   is a mild spam signal and the Gmail apps hide it unreliably anyway. To reinstate it
   for all four states, add it as a normal `col-feature`-style 3-up grid.
3. **Social icons are one shared grey set** (`#97A1B2`) rather than a dark set for
   light mode and a white set for dark. The grey reads acceptably on both surfaces, and
   collapsing the pair removed ten redundant `<img>` tags. See `ASSET_MIGRATION.md` to
   restore the per-mode swap.
4. **Typography.** Aeonik is not loaded as a webfont — an email should not depend on
   one. The stack falls back to Arial, so metrics differ slightly from the frames.
5. **Square corners in Outlook desktop.** The Word engine ignores `border-radius`.
   Cards and buttons render square there. Accepted.

---

## Testing status

Verified in this repository:

- HTML parses with no unclosed, stray, or mismatched tags; no duplicate IDs; MSO
  conditional comments balanced 15/15.
- All 20 image URLs return **HTTP 200**.
- No Base64, no `<script>`, no external stylesheet, no inline SVG, no Tailwind/React
  remnants, no `href="#"`.
- Rendered-DOM measurements at 376px and 1000px viewports: **no horizontal overflow**,
  container 640px, features 181px × 3 / 156px × 2, steps 136px × 4 / 156px × 2.

**Not verified — a browser preview is not an email client.** Gmail iOS, Gmail Android,
Gmail web, Apple Mail and Outlook rendering has **not** been executed. Run this through
Litmus or Email on Acid, and send a live seed test, before launch. Gmail iOS is the
primary target for this campaign.

The Apple App Store link could not be reached from the build environment (`apple.com`
is unreachable there entirely, including its root domain) — it is unverified, not
known-broken. The Google Play link returns HTTP 200.

---

## Deliverability

HTML optimisation can improve technical quality and reduce potential deliverability
issues, but Inbox vs Spam placement also depends on SendGrid/domain authentication,
SPF, DKIM, DMARC, sender reputation, sending IP reputation, recipient engagement,
complaint rate, list quality and other factors outside this file.
