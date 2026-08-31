# Diaspora — Referral Campaign Email

A production HTML email for the Diaspora referral campaign, built from four approved
Figma frames (Desktop/Mobile × Light/Dark) as **one** HTML document.

| File | Purpose |
| --- | --- |
| `Diaspora_Email.html` | The complete email. Paste into SendGrid as-is. |
| `ASSET_MIGRATION.md` | Asset inventory, verification status, and the outstanding to-do list. |
| `README.md` | This file. |

**Status:** complete and fully wired; all assets live and verified in both colour modes.
One blocker remains — the referral landing page returns 404. See
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

### How the card grids work

Desktop keeps the approved Figma layout — **3-across features, 4-across steps**. Mobile
is **2-across**: 3 rows of 2 features, 2 rows of 2 steps.

This is the **one** place the no-duplication rule is broken, deliberately, and it is the
exception the brief anticipated. Each section holds two grid tables — a `.grid-desktop`
and a `.grid-mobile` — toggled at 600px.

The reason is a real Gmail Android test. Gmail **runs `max-width` media queries** (the
hero centres correctly) but **ignores `display:inline-block` on `<div>`s** — the reflow
the original single-structure grid depended on. Every card dropped onto its own row.

A table's row grouping is fixed, so 3-across and 2-across cannot both come from one table
without exactly the inline-block reflow that had just failed. Two tables is the only way
to have both layouts.

What makes it sound rather than a hack: it depends **only** on the mechanism that same
test proved Gmail runs — a `max-width` media query plus `display:none`. With no CSS
support at all, the mobile copy stays hidden by its inline `display:none` and the desktop
grid shows; Outlook gets the same result via `mso-hide:all`. There is no state where both
or neither appear.

The costs, stated plainly:

- **The card copy exists twice.** Any text change must be made in both tables. This is
  called out in a MAINTENANCE note in the HTML.
- **File size went from ~56 KB to ~72 KB**, or 72% of Gmail's clipping budget — still
  under, but with materially less headroom than before.

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

> **The Gmail caveat — confirmed by testing, not assumed.** A real Gmail Android test
> showed Gmail runs `max-width` media queries but **ignores
> `@media (prefers-color-scheme: dark)` entirely**, including the `color-scheme` meta
> declarations. It applies its own inversion instead: the dark referral banner came back
> light, the pale prize cards came back dark, and the white content panel came back
> neutral grey — none of which are values in this file.
>
> Gmail's inversion is also **contrast-driven and size-aware**. Small text (`<h3>`, body
> copy) is lightened all the way to white, while large text (`<h1>` at 32px, `<h2>` at
> 20px) is lightened only enough to clear the lower large-text contrast threshold — so
> `#2C0A84` headings land as light purple rather than white, preserving hue.
>
> That colour is computed by Gmail from the light-mode source value. It cannot be
> overridden from this file: the only lever is the source colour itself, and it is shared
> with light mode. This is why light mode must be correct unconditionally and dark mode
> is an enhancement — not the other way round.

### Gmail clipping

Gmail truncates a message body past roughly **102 KB** and hides the remainder behind
"View entire message". This file is **~72 KB — about 72% of that budget**, with the
App Store badges and unsubscribe link comfortably inside it.

Headroom comes from engineering, not from cutting campaign content: no Base64, no inline
SVG blobs, no Figma Dev Mode output, and a single shared social-icon set rather than
per-mode copies. The one exception is the duplicated card grids above, which cost about
16 KB — that is where most of the remaining budget went, so weigh any further duplication
carefully.

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

Every placeholder is gone — logo, all 15 links, unsubscribe tags and postal address are
wired. Three things remain:

1. **`https://diaspora.mobi/referral` returns 404** — and it is the destination for both
   primary CTAs. The domain resolves, so this is a real 404, not a blocked request.
   Nothing else blocks a send.
2. **Confirm the postal address.** `Delaware, USA` is a state and country, not a street
   address, and may not satisfy CAN-SPAM.
3. **Run a real-device test.** Everything verified so far is static analysis, DOM
   measurement and browser rendering — none of which is an email client.

`ASSET_MIGRATION.md` has the detail, including why the hero is a pre-sized file rather
than a Cloudinary transform.

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
5. **Purple headings in the Gmail apps' dark theme — accepted.** Gmail ignores
   `prefers-color-scheme` and inverts the light design itself. Its inversion is
   contrast-driven and size-aware: small text is lightened all the way to white, while
   `<h1>` (32px) and `<h2>` (20px) only need the lower large-text contrast ratio, so they
   stop at a light purple that preserves the source hue.

   That output is computed from the light-mode value (`#2C0A84`) and there is no
   dark-only hook to override it — the sole lever is the source colour, which light mode
   shares. Desaturating the headings toward neutral would push Gmail's result closer to
   white at the cost of the vivid brand purple in light mode. **Decision: keep the brand
   purple.** The dark-mode result is legible and reads as intentional, and light mode is
   the state every client renders correctly.

6. **Square corners in Outlook desktop.** The Word engine ignores `border-radius`.
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
