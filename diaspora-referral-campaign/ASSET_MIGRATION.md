# Asset Migration — Step-by-Step

Everything still needed to make `Diaspora_Email.html` sendable, in the order to do it.

**Roughly 20 minutes.** Three files to upload, then a series of find-and-replace edits.
Nothing here requires design work — the artwork has already been exported from Figma.

Every unfinished value in the HTML uses the host `TODO-REPLACE.invalid`. That domain is
reserved by RFC 2606 and can never resolve, so anything you miss breaks visibly instead
of quietly pointing at nothing. The last step is a single command that proves none are
left.

> **Do the steps in order.** Step 7 verifies steps 1–6, so it only means something once
> the earlier ones are done.

---

## Step 1 — Upload three images to Cloudinary

You were given three files. Upload all three to the **same Cloudinary account already
serving this email** (cloud name `xyiwqpad`), into the root of your Media Library.

| File | What it is |
| --- | --- |
| `diaspora-logo-lockup-light-568x133.png` | Logo lockup, dark-purple — for light mode |
| `diaspora-logo-lockup-dark-568x133.png` | Logo lockup, white — for dark mode |
| `diaspora-hero-1088x952.png` | Three-device mockup, sharp — replaces the blurry 272px one |

All three came straight out of the Figma file, at 4× with transparent backgrounds.

**Keep the filenames exactly as they are.** Cloudinary turns the filename into the
public ID, and every URL below assumes these names. If Cloudinary appends a random
suffix (it does this when "Unique filename" is on), turn that setting off or rename the
asset afterwards.

> **Why no version number in the URLs below?** Cloudinary's `/v1234567890/` segment is
> optional. Leaving it out makes the URLs predictable, so you can paste them without
> copying anything out of the dashboard. The one trade-off: if you ever re-upload a file
> under the same name, add the version segment to bust the CDN cache.

---

## Step 2 — Point the logo at your uploads

Open `Diaspora_Email.html` and run two find-and-replace-**all** operations.

**Replace 1** — light logo (2 occurrences: header and footer)

Find:
```
https://TODO-REPLACE.invalid/diaspora-logo-lockup-light-568x133.png
```
Replace with:
```
https://res.cloudinary.com/xyiwqpad/image/upload/diaspora-logo-lockup-light-568x133.png
```

**Replace 2** — dark logo (2 occurrences)

Find:
```
https://TODO-REPLACE.invalid/diaspora-logo-lockup-dark-568x133.png
```
Replace with:
```
https://res.cloudinary.com/xyiwqpad/image/upload/diaspora-logo-lockup-dark-568x133.png
```

That is the whole logo fix. Do **not** change the `width`/`height` attributes — they are
already set to the lockup's true 4.27:1 ratio (141×33 header, 107×25 footer, and 120×28
/ 90×21 on mobile). Changing one without the other will squash the logo.

---

## Step 3 — Swap in the sharp hero

The hero currently points at a **272×297** source rendered at 268px — effectively 1×, so
it looks soft on every modern phone. The replacement is 1088×952 and also matches the
Figma composition more closely (the old file carried about 60px of empty vertical
padding, which shrank the phones).

On **line 298**, make two changes to the `<img class="hero-img" ...>` tag:

Find:
```
https://res.cloudinary.com/xyiwqpad/image/upload/v1787093903/diaspora-email-hero.png
```
Replace with:
```
https://res.cloudinary.com/xyiwqpad/image/upload/w_536,q_auto:good/diaspora-hero-1088x952.png
```

Then, **in that same tag only**, change the height:

```
height="293"   →   height="234"
```

Leave `width="268"` and the inline `style` alone.

> **What `w_536,q_auto:good` does.** Cloudinary resizes the 1088px master down to 536px
> (2× the 268px render, so it stays crisp on retina) and compresses it on the fly. The
> 742KB master never reaches the inbox — you get roughly 100–200KB instead. Nothing to
> install; it is just part of the URL.
>
> Deliberately **not** using `f_auto` here: automatic format switching depends on the
> client's `Accept` header, and email clients and image proxies are inconsistent about
> it. Forcing PNG keeps the transparency working everywhere.

---

## Step 4 — Fill in the links

Ten find-and-replace-all operations. Every "Find" string below is unique, so
replace-all is safe.

| # | Find | Replace with |
| --- | --- | --- |
| 1 | `https://TODO-REPLACE.invalid/home` | Diaspora homepage (logo links, header + footer) |
| 2 | `https://TODO-REPLACE.invalid/open-app` | "Open Diaspora" hero button destination |
| 3 | `https://TODO-REPLACE.invalid/refer` | "Start Referring" button destination |
| 4 | `https://TODO-REPLACE.invalid/social/x` | X profile URL |
| 5 | `https://TODO-REPLACE.invalid/social/linkedin` | LinkedIn profile URL |
| 6 | `https://TODO-REPLACE.invalid/social/tiktok` | TikTok profile URL |
| 7 | `https://TODO-REPLACE.invalid/social/facebook` | Facebook profile URL |
| 8 | `https://TODO-REPLACE.invalid/social/instagram` | Instagram profile URL |
| 9 | `https://TODO-REPLACE.invalid/unsubscribe` | SendGrid unsubscribe tag — see Step 5 |
| 10 | `https://TODO-REPLACE.invalid/preferences` | SendGrid preferences tag — see Step 5 |

Add UTM parameters now if you want click attribution — for example
`https://diaspora.mobi/refer?utm_source=sendgrid&utm_medium=email&utm_campaign=referral_2026`.

---

## Step 5 — Wire unsubscribe to SendGrid

Do not hand-roll these. Use SendGrid's own subscription-management tags so it records
the opt-out and suppresses future sends automatically. A hand-built unsubscribe link
does not suppress anything.

| Footer link | Use this as the URL |
| --- | --- |
| "unsubscribe" | `<%asm_group_unsubscribe_raw_url%>` |
| "manage your email preferences" | `<%asm_preferences_raw_url%>` |

Paste the tag itself where the URL goes, exactly as written:

```html
<a href="<%asm_group_unsubscribe_raw_url%>" class="txt-link" style="color:#5925DC;text-decoration:none;">unsubscribe</a>
```

Confirm the tag names against SendGrid's current documentation for **your** sending
mode before launch — Marketing Campaigns and dynamic templates differ, and an
unrecognised tag renders as empty text rather than throwing an error.

**Also check the recipient token.** The footer uses `{{email}}`, which resolves in a v3
dynamic template when `email` is present in your `dynamic_template_data`. If you send
through Marketing Campaigns instead, swap it for that editor's insert-field token.

**One-click unsubscribe is separate.** Gmail and Yahoo require bulk senders to support
RFC 8058. That is a `List-Unsubscribe` / `List-Unsubscribe-Post` **header**, set at the
send level in SendGrid — it cannot be done from this HTML file. Enable it there too.

---

## Step 6 — Add the postal address

Find this line in the footer:

```
TODO-REPLACE: Diaspora postal address, city, region, postcode, country
```

Replace it with Diaspora's real mailing address. A physical postal address is legally
required by CAN-SPAM (US) and its equivalents elsewhere, and its absence is a common
spam-filter trigger.

The approved Figma design has an address line here too — the source template's
placeholder reads "100 Smith Street, Melbourne VIC 3000" — so this matches the design,
it is not an addition.

> If you send via **Marketing Campaigns**, SendGrid can inject your verified sender
> address automatically. In that case use its address tag here instead of hardcoding.

---

## Step 7 — Verify

**7a. No placeholders left.** From the repository root:

```bash
grep -rn "TODO-REPLACE" diaspora-referral-campaign/Diaspora_Email.html
```

This must print **nothing**. Any output is something you missed.

**7b. Every image loads.** This prints the HTTP status of each image URL — all should be `200`:

```bash
grep -oE 'src="https://[^"]*"' diaspora-referral-campaign/Diaspora_Email.html | sed 's/src="//;s/"$//' | sort -u | while read u; do echo "$(curl -s -o /dev/null -w '%{http_code}' -L "$u")  $u"; done
```

**7c. Send yourself a real test.** Push it through SendGrid to accounts on Gmail iOS,
Gmail Android, Gmail web, Apple Mail and Outlook. Check specifically:

- [ ] Logo appears in header and footer, not squashed, in light **and** dark mode
- [ ] Hero mockup is sharp, not soft
- [ ] Feature cards: 3-across on desktop, 2-across on mobile
- [ ] "How it works": 4-across on desktop, 2×2 on mobile
- [ ] Campaign-detail icons visible on the pale cards (they are purple in light mode)
- [ ] No sideways scrolling on any phone
- [ ] Unsubscribe link actually unsubscribes you
- [ ] Not truncated by Gmail's "View entire message"

A browser preview is **not** an email client. Use Litmus or Email on Acid, or real
devices. This step is the one that catches what static analysis cannot.

---

## Optional — exact dark-mode social icons

Skip unless you want pixel-exact Figma fidelity; the email is correct without it.

The hosted social icons are a single mid-grey set (`#97A1B2`) used in both modes. Figma
specifies a dark set for light mode and a white set for dark. Grey was kept because it
reads acceptably on both surfaces, and because collapsing the pair removed ten redundant
`<img>` tags.

To restore the true swap: export the five dark-frame social icons from Figma (nodes
`1:253`, `1:255`, `1:258`, `1:260`, `1:262`), upload them, then for each of the ten
icons replace:

```html
<img class="social-icon" src="LIGHT_URL" width="20" height="20" alt="Diaspora on X" style="width:20px;height:20px;display:inline-block;">
```

with:

```html
<img class="social-icon icon-swap-light" src="LIGHT_URL" width="20" height="20" alt="Diaspora on X" style="width:20px;height:20px;display:inline-block;">
<img class="social-icon icon-swap-dark"  src="DARK_URL"  width="20" height="20" alt=""               style="width:20px;height:20px;display:none;">
```

Keep `alt=""` on the dark copy so screen readers do not announce each network twice. The
`.icon-swap-*` CSS already exists. Note this only takes effect in clients that honour
`prefers-color-scheme` or `[data-ogsc]` — **not the Gmail apps**.

---

# Appendix — Asset reference

Verification method: every URL was requested over HTTPS and its status recorded; PNGs
were decoded to confirm dimensions and actual ink colour. Last verified 2026-08-31.

## Already working — no action needed

All 20 URLs below returned **HTTP 200**.

**Feature card icons** — 32×32, white glyphs on coloured badges.

| Feature | URL (prefix `https://res.cloudinary.com/xyiwqpad/image/upload/`) |
| --- | --- |
| Matchmaking | `v1787094187/diaspora-email-svg-15.png` |
| Networking | `v1787094171/diaspora-email-svg-16.png` |
| News | `v1787094172/diaspora-email-svg-17.png` |
| Community | `v1787094173/diaspora-email-svg-18.png` |
| Messaging | `v1787094174/diaspora-email-svg-19.png` |
| Business Discovery | `v1787094175/diaspora-email-svg-20.png` |

**Social icons** — 40×40 rendered at 20×20 (2×), mid-grey `#97A1B2`.

| Network | URL |
| --- | --- |
| X | `v1787094166/diaspora-email-svg-10.png` |
| LinkedIn | `v1787094166/diaspora-email-svg-11.png` |
| TikTok | `v1787094168/diaspora-email-svg-12.png` |
| Facebook | `v1787094169/diaspora-email-svg-13.png` |
| Instagram | `v1787094170/diaspora-email-svg-14.png` |

> The header previously referenced `svg-5`/`svg-6` for TikTok and Facebook while the
> footer used `svg-12`/`svg-13`. Those pairs are byte-identical (md5 match) — the same
> files under two names. Standardised on `svg-12`/`svg-13`.

**Campaign-detail icons.** The hosted glyphs are **white**, so they only ever worked as
the dark-mode variant — which is why light mode showed nothing. The light variant is now
the *same asset* recoloured by Cloudinary's `e_colorize`, so no second upload was needed.

| Icon | Light (recoloured to `#2C0A84`) | Dark (source, white) |
| --- | --- | --- |
| Calendar | `e_colorize,co_rgb:2C0A84/v1787365567/CalendarDots-1.png` | `v1787365567/CalendarDots-1.png` |
| Trophy | `e_colorize,co_rgb:2C0A84/v1787365567/Trophy-1.png` | `v1787365567/Trophy-1.png` |
| User check | `e_colorize,co_rgb:2C0A84/v1787365568/UserCheck-1.png` | `v1787365568/UserCheck-1.png` |

To change the light tint, edit the hex after `co_rgb:` — no re-upload required.

**App store badges** — `v1787365843/Mobile_app_store_badge.png` (120×40) and
`v1787365844/Mobile_app_store_badge-1.png` (135×40).

**Store links**

| Store | Status |
| --- | --- |
| Apple — `apps.apple.com/us/app/diaspora-connect-thrive/id6480373791` | **Unverified.** `apple.com` is unreachable from the build environment entirely, including its root domain. Not known-broken; confirm manually. |
| Google Play — `play.google.com/store/apps/details?id=mobi.diaspora.app` | Verified, HTTP 200 |

## Fixed in this pass

| Was | Problem | Now |
| --- | --- | --- |
| `.../diaspora-email-svg-14.pngg` | Double-`g` typo → HTTP 404, header Instagram icon broken | Corrected |
| `.../UserCheck-1.pngg` | Double-`g` typo → HTTP 404 | Replaced with the recoloured light variant |
| `YOUR-CDN-HOST.example.com/...calendar-dots.png` | Placeholder host, never resolved | Replaced with recoloured light variant |
| `YOUR-CDN-HOST.example.com/...trophy.png` | Placeholder host, never resolved | Replaced with recoloured light variant |
| Footer body text `color:#FFFFFF5` | Invalid 7-digit hex — silently ignored, then invisible in dark mode | `#667085` + `.txt-footer` (dark `#CFC7EE`) |
| Footer logo + wordmark `color:#FFFFFF` | White on white — **invisible in light mode** | Replaced with the image lockup |
| `.footer-copy` | No dark override — `#667085` on `#2C0A84` failed contrast | `.txt-footer` added |
| Outer page background | No dark override — light gutter framed the dark email | `.bg-outer` added (dark `#1A0550`) |
| Hero column widths | Declared `280 + 312 = 592px` inside a real **544px** content box | `260 + 284 = 544` |
| Logo aspect ratio | Assumed 5.5:1; the Figma lockup is **4.27:1** — would have squashed the artwork | Corrected against the real export |
| 15 × `href="#"` | Dead links | Named `TODO-REPLACE.invalid` placeholders |
| No postal address | CAN-SPAM exposure | Placeholder line added to the footer |
| Hardcoded recipient address | A real email address was baked into the footer instead of a merge token, and had been published to this repo | Replaced with `{{email}}`; the address is redacted here deliberately so this document does not reintroduce it |
