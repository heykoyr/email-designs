# Asset Migration

**Status: all assets live and verified. Three things to confirm before sending.**

Logo, hero, links, unsubscribe tags and postal address are all wired. Both colour modes
have been rendered and checked.

---

## Hero — live

| Property | Value |
| --- | --- |
| URL | `https://res.cloudinary.com/xyiwqpad/image/upload/w_536,q_auto:good/v1788190896/diaspora-hero-536x469.png` |
| Delivered | 536x470, 65 KB, **transparency preserved** |
| Rendered at | 268x235 — exactly 2x, retina-sharp, aspect matches with no distortion |

Verified rendering in both colour modes: the phones sit directly on the background with
**no white block behind them in dark mode**.

> ### A correction worth recording
>
> An earlier revision of this document claimed that *every* Cloudinary transform on this
> account flattens the alpha channel. **That was wrong.** It was true of one specific
> asset, not of the account.
>
> The original `diaspora-hero-1088x952` upload loses its alpha under
> `w_536,q_auto:good` — the delivered PNG comes back as colourtype 3 with no `tRNS`
> chunk. The re-uploaded `diaspora-hero-536x469` survives the *same* transform with
> transparency intact. Both are stored as RGBA masters, and the URL form (with or
> without the `.png` extension) makes no difference. Something about how the first file
> was ingested is the cause.
>
> Practical consequences:
>
> - The transform is fine to use — 730 KB down to 65 KB, alpha intact.
> - Do not reuse the `diaspora-hero-1088x952` asset. It is the one that flattens.
> - **The generalisable rule stands:** after putting a transparent PNG through a CDN
>   transform, download the delivered file and confirm it still has an alpha channel or
>   a `tRNS` chunk. Do not assume, and do not trust a light-mode preview — a flattened
>   background is invisible against a white surface and only shows up in dark mode.
>
> Note the URL includes the version segment (`v1788190896`). For this asset the
> extensionless and versioned forms resolve, but `/diaspora-hero-536x469.png` without a
> version returns 404 — so keep the version in place.

---

## Confirm before sending

### 1. The referral landing page returns 404

`https://diaspora.mobi/referral` is the destination for **both** primary CTAs — "Open
Diaspora" and "Start Referring". It is wired in as supplied, but the page does not
currently exist:

```
https://diaspora.mobi/          ->  HTTP 200
https://diaspora.mobi/referral  ->  HTTP 404   (also 404 with a trailing slash, and on www)
```

The domain itself resolves, so this is a real 404 rather than a blocked request. Either
the landing page is not deployed yet or the path differs. **The campaign's two main
conversion points currently lead nowhere** — resolve this before any send.

### 2. The postal address may not satisfy CAN-SPAM

The footer reads `Delaware, USA`. CAN-SPAM requires a **valid physical postal address** —
a street address, or a PO box or private mailbox registered with the USPS or a
commercial mail-receiving agency. A state and country alone is unlikely to qualify, and
a vague or missing address is both a compliance exposure and a common spam-filter
trigger.

If Diaspora is a Delaware-registered entity, the registered agent's street address is
usually the right thing to put here. Worth confirming with whoever handles the
incorporation.

### 3. Three links could not be verified from the build environment

Not evidence of a problem — these hosts block requests from this environment entirely,
including their own root domains. Click each once to confirm:

| URL | Result | Why |
| --- | --- | --- |
| `https://facebook.com/diaspora.mobi` | HTTP 400 | the `facebook.com/` root also returns 400 |
| `https://www.tiktok.com/@diaspora.app` | no response | the `tiktok.com/` root is also unreachable |
| `https://apps.apple.com/us/app/diaspora-connect-thrive/id6480373791` | no response | the `apps.apple.com/` root is also unreachable |

Verified reachable: `diaspora.mobi` (200), `x.com/diaspora_mobi` (200),
`linkedin.com/company/diaspora-app` (200), `instagram.com/getdiaspora` (200),
Google Play (200).

---

## Verify after uploading

**All images return 200** — run from the repository root:

```bash
grep -oE 'src="https://[^"]*"' diaspora-referral-campaign/Diaspora_Email.html | sed 's/src="//;s/"$//' | sort -u | while read u; do echo "$(curl -s -o /dev/null -w '%{http_code}' -L "$u")  $u"; done
```

**Send a real test.** Push through SendGrid to Gmail iOS, Gmail Android, Gmail web,
Apple Mail and Outlook, then check:

- [ ] Hero renders, is sharp, and has **no white box behind the phones in dark mode**
- [ ] Logo appears in header and footer, not squashed, in light and dark
- [ ] Feature cards 3-across desktop / 2-across mobile
- [ ] "How it works" 4-across desktop / 2x2 mobile
- [ ] Campaign-detail icons visible on the pale cards
- [ ] No sideways scrolling on any phone
- [ ] The unsubscribe link resolves to a real URL and actually unsubscribes you
- [ ] Not truncated by Gmail's "View entire message"

A browser preview is not an email client. The ASM tags render as literal `<%...%>` text
outside SendGrid — expected, not a bug.

---

## Completed

### Logo — live

| Mode | URL |
| --- | --- |
| Light | `https://res.cloudinary.com/xyiwqpad/image/upload/diaspora-logo-lockup-light-264x48.png` |
| Dark | `https://res.cloudinary.com/xyiwqpad/image/upload/diaspora-logo-lockup-dark-264x48.png` |

The files are **264x50** (ratio 5.28:1). The HTML renders 132x25 header, 106x20 footer,
111x21 / 90x17 on mobile — all matching that ratio, header at exactly 2x.

> If the lockup is ever re-cropped, **recalculate those six numbers**. An `<img>` whose
> `width`/`height` ratio disagrees with the file gets squashed — email clients honour the
> attributes and do not letterbox.

### Links — wired

| Location | Destination |
| --- | --- |
| Logo, header + footer | `https://diaspora.mobi/` |
| "Open Diaspora" | `https://diaspora.mobi/referral` (404 — see above) |
| "Start Referring" | `https://diaspora.mobi/referral` (404 — see above) |
| X | `https://x.com/diaspora_mobi` |
| LinkedIn | `https://www.linkedin.com/company/diaspora-app` |
| TikTok | `https://www.tiktok.com/@diaspora.app` |
| Facebook | `https://facebook.com/diaspora.mobi` |
| Instagram | `https://www.instagram.com/getdiaspora` |

### Unsubscribe — SendGrid ASM tags

| Footer link | Tag |
| --- | --- |
| "unsubscribe" | `<%asm_group_unsubscribe_raw_url%>` |
| "manage your email preferences" | `<%asm_preferences_raw_url%>` |
| Recipient address shown in the footer | `{{email}}` |

Confirm the tag names against SendGrid's docs for **your** sending mode — Marketing
Campaigns and dynamic templates differ, and an unrecognised tag renders as empty text
rather than erroring, so it fails silently.

`{{email}}` resolves in a v3 dynamic template when `email` is present in your
`dynamic_template_data`. Through Marketing Campaigns, swap it for that editor's
insert-field token.

**One-click unsubscribe is separate.** Gmail and Yahoo require bulk senders to support
RFC 8058 — a `List-Unsubscribe` / `List-Unsubscribe-Post` **header**, set at the send
level in SendGrid. It cannot be done from this HTML file.

### Postal address

Footer reads `Delaware, USA`. See the compliance note above.

---

# Appendix — Asset reference

Every URL below verified **HTTP 200**. PNGs were decoded to confirm dimensions and ink
colour. Last verified 2026-08-31.

**Feature card icons** — 32x32, white glyphs on coloured badges. Prefix all paths with
`https://res.cloudinary.com/xyiwqpad/image/upload/`.

| Feature | Path |
| --- | --- |
| Matchmaking | `v1787094187/diaspora-email-svg-15.png` |
| Networking | `v1787094171/diaspora-email-svg-16.png` |
| News | `v1787094172/diaspora-email-svg-17.png` |
| Community | `v1787094173/diaspora-email-svg-18.png` |
| Messaging | `v1787094174/diaspora-email-svg-19.png` |
| Business Discovery | `v1787094175/diaspora-email-svg-20.png` |

**Social icons** — 40x40 rendered at 20x20 (2x), mid-grey `#97A1B2`, one set for both modes.

| Network | Path |
| --- | --- |
| X | `v1787094166/diaspora-email-svg-10.png` |
| LinkedIn | `v1787094166/diaspora-email-svg-11.png` |
| TikTok | `v1787094168/diaspora-email-svg-12.png` |
| Facebook | `v1787094169/diaspora-email-svg-13.png` |
| Instagram | `v1787094170/diaspora-email-svg-14.png` |

**Campaign-detail icons.** The hosted glyphs are white, so they only ever worked as the
dark-mode variant — which is why light mode showed nothing at all. The light variant is
the same asset recoloured by Cloudinary's `e_colorize`, so no second upload was needed.
Unlike resizing, `e_colorize` preserves the alpha channel here.

| Icon | Light (`#2C0A84`) | Dark (white) |
| --- | --- | --- |
| Calendar | `e_colorize,co_rgb:2C0A84/v1787365567/CalendarDots-1.png` | `v1787365567/CalendarDots-1.png` |
| Trophy | `e_colorize,co_rgb:2C0A84/v1787365567/Trophy-1.png` | `v1787365567/Trophy-1.png` |
| User check | `e_colorize,co_rgb:2C0A84/v1787365568/UserCheck-1.png` | `v1787365568/UserCheck-1.png` |

To change the light tint, edit the hex after `co_rgb:`.

**App store badges** — `v1787365843/Mobile_app_store_badge.png` (120x40) and
`v1787365844/Mobile_app_store_badge-1.png` (135x40).

## Fixed during this build

| Was | Problem | Now |
| --- | --- | --- |
| `.../diaspora-email-svg-14.pngg` | Double-`g` typo, HTTP 404 — header Instagram icon broken | Corrected |
| `.../UserCheck-1.pngg` | Double-`g` typo, HTTP 404 | Replaced with the recoloured light variant |
| `YOUR-CDN-HOST.example.com/...` x2 | Placeholder host, never resolved | Replaced with recoloured light variants |
| Footer body text `color:#FFFFFF5` | Invalid 7-digit hex — silently ignored, then invisible in dark mode | `#667085` + `.txt-footer` (dark `#CFC7EE`) |
| Footer logo + wordmark `color:#FFFFFF` | White on white — invisible in light mode | Replaced with the image lockup |
| `.footer-copy` | No dark override — `#667085` on `#2C0A84` failed contrast | `.txt-footer` added |
| Outer page background | No dark override — a light gutter framed the dark email | `.bg-outer` added (dark `#1A0550`) |
| Hero column widths | Declared `280 + 312 = 592px` inside a real 544px content box | `260 + 284 = 544` |
| Logo aspect ratio | Assumed 5.5:1 with no asset to measure — would have squashed the artwork | Measured from the real file (5.28:1); six render sizes set to match |
| Hero source | 272x297 rendered at 268px — effectively 1x, soft on every retina screen | Re-exported from Figma; delivered 536x470 at 65 KB, exactly 2x, transparency verified |
| 15 x `href="#"` | Dead links | All wired to real destinations |
| No postal address | CAN-SPAM exposure | Address line added (see compliance note) |
| Hardcoded recipient address | A real email address was baked into the footer instead of a merge token, and had been published to this repo | Replaced with `{{email}}`, and scrubbed from git history |
