# HTML email QA: a practical pre-send checklist

A reusable checklist for designers and developers building responsive HTML email. It grows out of the [Diaspora referral campaign case study](../diaspora-referral-campaign/README.md), where one browser-perfect layout had to change after testing on Gmail Android.

**Evidence key:** Mark each item **Passed (client/device/date)**, **Failed (reproduction)**, **Not tested**, or **Not applicable**. A Chrome screenshot proves browser layout, **not** Gmail, Apple Mail or Outlook rendering. The observations in the linked case study apply to that campaign's tests; do not assume every client version behaves identically.

## 1. Content and destinations

- [ ] Confirm the subject, preheader, sender, audience, dates and campaign eligibility language with the owner.
- [ ] Open **every CTA and footer link**, following redirects to the final page. Test in a browser session that resembles a real recipient, not just with an HTTP HEAD request.
- [ ] Confirm app-store destinations, deep links and any campaign landing page actually exist **before** a send.
- [ ] Replace placeholder values and confirm ESP merge fields resolve in a real send; never publish real recipient addresses in sample markup.
- [ ] Verify the required physical postal address, privacy disclosure and unsubscribe options for the sender's applicable laws and sending platform. Do not treat a state/country alone as a confirmed compliant address.

From the repository root, check unresolved placeholder hosts in the sendable HTML only:

```sh
find . -type f -name '*.html' -exec grep -nH 'TODO-REPLACE' {} +
```

No printed matches means none of that exact placeholder string was found. This command does not validate other dynamic tags or URLs.

## 2. Layout, images and accessibility

- [ ] Confirm a **table-based fallback** still reads in clients with limited CSS. In this project, every layout table has `role="presentation"`.
- [ ] Check mobile widths and enlarged text for clipping, sideways scrolling, CTA tap targets and logical reading order.
- [ ] Test meaningful image alt text, empty alt text for decoration, heading structure and contrast in all supported colour modes.
- [ ] Check all image URLs and their *delivered* dimensions, not merely source metadata. Explicit HTML width/height should preserve the delivered aspect ratio.
- [ ] Inspect transparent imagery against both light and dark backgrounds **after** any CDN transformation. A white matte may be invisible in light mode.
- [ ] Decide how columns fall back if media queries or CSS display rules are ignored. Document any intentionally duplicated mobile/desktop copy and keep the two variants in sync.

In the Diaspora campaign, Gmail Android ran the tested width query but did not apply the `display:inline-block` reflow the original layout needed. The replacement uses two grid tables shown at different breakpoints. This is an observed **project-specific workaround**, not a claim that one structure works universally.

## 3. Dark mode and clipping

- [ ] Make the **inline light-mode design usable on its own**; dark-mode selectors are enhancements, not a required layout dependency.
- [ ] Check real-client dark mode separately. The campaign's Gmail Android test showed app-managed inversion rather than honouring `prefers-color-scheme`; the outcome cannot be certified from Chrome emulation alone.
- [ ] Measure final HTML byte size *after* the email service has applied its transformations. Gmail commonly clips around **102 KB**, so leave headroom.

For a source-file baseline, run:

```sh
wc -c diaspora-referral-campaign/Diaspora_Email.html
```

That count does not include potential expansion from your sending platform.

## 4. Minimum client/device matrix

| Client or environment | What to confirm | Result |
| --- | --- | --- |
| Browser render at desktop and phone widths | Structural baseline only | Not tested for your new email |
| Gmail Android, light/dark | Grid reflow, inversion, overflow | Not tested for your new email |
| Gmail iOS, light/dark | Grid, contrast, CTA and footer | Not tested |
| Gmail web | Desktop layout, clipping, links | Not tested |
| Apple Mail | Dark-mode colours, image and type fallback | Not tested |
| Outlook desktop/web | Word-engine limitations, buttons and fallback | Not tested |

Fill the result column with real evidence before saying an email is verified. Screenshots, email headers, client/app versions, date, and a minimal reproduction are more useful than a generic “works everywhere.”

## 5. Deliverability and sending controls

- [ ] Confirm the sending domain's SPF, DKIM and DMARC configuration and the expected sender identity.
- [ ] Test unsubscribe and preference links end-to-end, including the sending platform's suppression handling.
- [ ] Where required for bulk senders, configure RFC 8058 one-click unsubscribe in **message headers**, not HTML alone.
- [ ] Send a controlled seed test and check received source, image loading, clicks, accessibility, clipping and unsubscribe behaviour.
- [ ] Record **exactly which clients were tested**, what failed and what remains unverified before approving launch.

## 6. Release record template

| Item | Evidence or owner |
| --- | --- |
| HTML file and commit | |
| Sending platform and template mode | |
| Target devices/clients and versions | |
| Link and asset check date | |
| Rendering captures | |
| Accessibility and dark-mode exceptions | |
| Final-size / clipping check | |
| Unsubscribe and address verification | |
| Known blockers | |
| Decision / approver / date | |

### Worked example from this repository

The [Diaspora campaign](../diaspora-referral-campaign/README.md) includes four **browser** renders and a real Gmail Android observation; Gmail iOS, Gmail web, Apple Mail and Outlook client rendering were not confirmed in its recorded test evidence. Its [asset and release notes](../diaspora-referral-campaign/ASSET_MIGRATION.md) identify a referral landing page returning 404 and a postal-address concern at the time of testing. **Those are launch blockers until someone re-verifies them; this guide does not claim they remain broken today.**
