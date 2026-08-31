# Asset Migration

Status of every external asset and link in `Diaspora_Email.html`.

Verification method: each URL was requested over HTTPS and its status code recorded;
PNGs were decoded to confirm dimensions and actual ink colour. "Verified" below means
the URL returned **HTTP 200** and the file was inspected — not that it was assumed.

Last verified: 2026-08-31.

---

## 1. Outstanding — blocks send

### 1.1 Logo lockup (2 assets)

The header and footer logo were previously a CSS box containing the letter **"D"** plus
the word "Diaspora" as live text. In the footer both were set to `#FFFFFF` on a white
background, so the entire footer logo was **invisible in light mode**. That has been
replaced with a proper image lockup, which still needs the artwork.

| Slot | Placeholder in HTML | Export at | Renders at |
| --- | --- | --- | --- |
| Light mode | `https://TODO-REPLACE.invalid/diaspora-logo-lockup-light-264x48.png` | 264×48 | 132×24 header, 110×20 footer |
| Dark mode | `https://TODO-REPLACE.invalid/diaspora-logo-lockup-dark-264x48.png` | 264×48 | 132×24 header, 110×20 footer |

Export notes:

- **Aspect ratio must be 5.5:1.** The `width`/`height` attributes are fixed for
  Outlook, so a different ratio will distort. Mobile scales the same asset down via CSS.
- Export at **2×** (264×48) for the 132×24 render. PNG-24 with transparency.
- Include the full lockup — the `|D)` mark *and* the "Diaspora" wordmark with its
  orange `o`. That orange character is why this must be an image rather than text.
- **Light** = the dark-purple lockup. **Dark** = the white lockup.
- Both go through the existing `.icon-swap-light` / `.icon-swap-dark` pair, so once the
  URLs are in, the mode swap works with no further edits.

### 1.2 Link destinations

15 dead `href="#"` links were replaced with named, greppable placeholders. Find them all:

```bash
grep -n "TODO-REPLACE" Diaspora_Email.html
```

| Placeholder | Appears | Needs |
| --- | --- | --- |
| `TODO-REPLACE.invalid/home` | header + footer logo | Diaspora homepage |
| `TODO-REPLACE.invalid/open-app` | hero CTA | "Open Diaspora" destination |
| `TODO-REPLACE.invalid/refer` | referral CTA | "Start Referring" destination |
| `TODO-REPLACE.invalid/social/x` | header + footer | X profile |
| `TODO-REPLACE.invalid/social/linkedin` | header + footer | LinkedIn profile |
| `TODO-REPLACE.invalid/social/tiktok` | header + footer | TikTok profile |
| `TODO-REPLACE.invalid/social/facebook` | header + footer | Facebook profile |
| `TODO-REPLACE.invalid/social/instagram` | header + footer | Instagram profile |
| `TODO-REPLACE.invalid/unsubscribe` | footer | SendGrid ASM tag — see README |
| `TODO-REPLACE.invalid/preferences` | footer (×2) | SendGrid ASM tag — see README |

### 1.3 Postal address

The footer carries a literal `TODO-REPLACE:` line where Diaspora's physical mailing
address must go. A postal address is required by CAN-SPAM (US) and equivalent rules
elsewhere. SendGrid Marketing Campaigns can inject the sender address for you; a v3
dynamic template cannot, so it must be in the HTML.

### 1.4 Recipient token

The footer uses `{{email}}`. Confirm it matches your sending mode before launch — see
the SendGrid section of the README. An unresolved token renders as empty text.

---

## 2. Recommended — quality, not blocking

### 2.1 Hero mockup is under-resolution

`diaspora-email-hero.png` is **272×297** — this is the low-resolution source the
original brief explicitly warned against. It renders at 268 CSS px, so it is
effectively **1× and will look soft on every retina display**.

- Current: `https://res.cloudinary.com/xyiwqpad/image/upload/v1787093903/diaspora-email-hero.png` (verified, HTTP 200, 76 KB)
- Needed: re-export the approved three-device composition at **536×586** (2×) or larger,
  preserving the 272:297 aspect ratio and the approved composition. Do not recompose it.
- Upscaling the existing file cannot recover detail — this needs a fresh Figma export.

### 2.2 Campaign-detail icons are 1×

`CalendarDots`, `Trophy` and `UserCheck` are 16×16 native and render at 16px, so they
are 1×. Re-export at 32×32 for crispness on retina. Not urgent — they are small glyphs.

### 2.3 Restoring the per-mode social icon swap

The hosted social set is a single mid-grey (`#97A1B2`, confirmed by pixel inspection)
that reads acceptably on both the light and dark surfaces. The previous markup carried a
light/dark `<img>` pair per icon whose two `src` values were **byte-identical**
(confirmed by md5) — ten tags doing nothing. They were collapsed to one `<img>` each.

To restore true per-mode icons, host a white set, then for each of the ten icons replace:

```html
<img class="social-icon" src="LIGHT_URL" width="20" height="20" alt="Diaspora on X" style="width:20px;height:20px;display:inline-block;">
```

with the swap pair:

```html
<img class="social-icon icon-swap-light" src="LIGHT_URL" width="20" height="20" alt="Diaspora on X" style="width:20px;height:20px;display:inline-block;">
<img class="social-icon icon-swap-dark"  src="DARK_URL"  width="20" height="20" alt=""               style="width:20px;height:20px;display:none;">
```

Keep `alt=""` on the dark copy so screen readers do not announce each network twice.
The `.icon-swap-*` CSS already exists and needs no change. Note the swap only takes
effect in clients that honour `prefers-color-scheme` or `[data-ogsc]` — **not the Gmail
apps**.

---

## 3. Verified — no action needed

All 20 image URLs below returned **HTTP 200**.

### Hero
| Asset | URL | Notes |
| --- | --- | --- |
| Three-device mockup | `.../v1787093903/diaspora-email-hero.png` | 272×297 — see §2.1 |

### Feature card icons — 32×32, white glyphs on coloured badges (correct)
| Feature | URL |
| --- | --- |
| Matchmaking | `.../v1787094187/diaspora-email-svg-15.png` |
| Networking | `.../v1787094171/diaspora-email-svg-16.png` |
| News | `.../v1787094172/diaspora-email-svg-17.png` |
| Community | `.../v1787094173/diaspora-email-svg-18.png` |
| Messaging | `.../v1787094174/diaspora-email-svg-19.png` |
| Business Discovery | `.../v1787094175/diaspora-email-svg-20.png` |

### Social icons — 40×40, rendered 20×20 (2×), mid-grey `#97A1B2`
| Network | URL |
| --- | --- |
| X | `.../v1787094166/diaspora-email-svg-10.png` |
| LinkedIn | `.../v1787094166/diaspora-email-svg-11.png` |
| TikTok | `.../v1787094168/diaspora-email-svg-12.png` |
| Facebook | `.../v1787094169/diaspora-email-svg-13.png` |
| Instagram | `.../v1787094170/diaspora-email-svg-14.png` |

> The header previously referenced `svg-5` and `svg-6` for TikTok and Facebook while the
> footer used `svg-12` and `svg-13`. Those pairs are **byte-identical** (md5 match) —
> the same files under two names. Standardised on `svg-12` / `svg-13` in both places.

### Campaign-detail icons — the light/dark pair

The hosted source glyphs are **white**, so they only ever worked as the dark-mode
variant. The light-mode variant is the *same asset* recoloured to brand purple by
Cloudinary's `e_colorize` transform — no second upload was required. Both variants
verified HTTP 200.

| Icon | Light (`e_colorize` → `#2C0A84`) | Dark (source, white) |
| --- | --- | --- |
| Calendar | `.../e_colorize,co_rgb:2C0A84/v1787365567/CalendarDots-1.png` | `.../v1787365567/CalendarDots-1.png` |
| Trophy | `.../e_colorize,co_rgb:2C0A84/v1787365567/Trophy-1.png` | `.../v1787365567/Trophy-1.png` |
| User check | `.../e_colorize,co_rgb:2C0A84/v1787365568/UserCheck-1.png` | `.../v1787365568/UserCheck-1.png` |

To change the light tint, edit the hex in `co_rgb:` — no re-upload needed.

### App store badges
| Badge | URL |
| --- | --- |
| App Store | `.../v1787365843/Mobile_app_store_badge.png` (120×40) |
| Google Play | `.../v1787365844/Mobile_app_store_badge-1.png` (135×40) |

### Store links
| Store | URL | Status |
| --- | --- | --- |
| Apple | `https://apps.apple.com/us/app/diaspora-connect-thrive/id6480373791` | **Unverified** — `apple.com` is unreachable from the build environment entirely, including its root domain. Not known-broken; confirm manually. |
| Google Play | `https://play.google.com/store/apps/details?id=mobi.diaspora.app` | Verified, HTTP 200 |

---

## 4. Fixed in this pass

Broken references found and resolved:

| Was | Problem | Now |
| --- | --- | --- |
| `.../diaspora-email-svg-14.pngg` | Double-`g` typo → **HTTP 404**, header Instagram icon broken | Corrected |
| `.../UserCheck-1.pngg` | Double-`g` typo → **HTTP 404** | Replaced with the tinted light variant |
| `https://YOUR-CDN-HOST.example.com/.../detail-calendar-dots.png` | Placeholder host, never resolved | Replaced with tinted light variant |
| `https://YOUR-CDN-HOST.example.com/.../detail-trophy.png` | Placeholder host, never resolved | Replaced with tinted light variant |
| Footer body text `color:#FFFFFF5` | Invalid 7-digit hex — ignored by the parser, then invisible in dark mode | `#667085` + `.txt-footer` (dark: `#CFC7EE`) |
| Footer logo + wordmark `color:#FFFFFF` | White on a white background — **invisible in light mode** | Replaced with the image lockup (§1.1) |
| `.footer-copy` | No dark-mode override — `#667085` on `#2C0A84` failed contrast | `.txt-footer` added |
| Outer page background | No dark override — light lavender gutter framed the dark email | `.bg-outer` added (dark `#1A0550`) |
| Hero column widths | Declared `280 + 312 = 592px` inside a real **544px** content box | `260 + 284 = 544` |
| 15 × `href="#"` | Dead links | Named `TODO-REPLACE.invalid` placeholders |
| No postal address | CAN-SPAM exposure | Placeholder line added to footer |
| `{{email}}` | A real personal address hardcoded in the footer of the **currently published** repo version | Replaced with the `{{email}}` token |
