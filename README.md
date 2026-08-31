# Email Designs

Production HTML email templates — hand-built, table-based, and tested for
cross-client rendering. Each template lives in its own folder with its own README
and asset inventory.

## Templates

| Template | Description | Status |
| --- | --- | --- |
| [`diaspora-referral-campaign/`](diaspora-referral-campaign/) | Diaspora referral campaign. One document, four rendering states (Desktop/Mobile × Light/Dark). Built for SendGrid. | Assets pending — see its [ASSET_MIGRATION.md](diaspora-referral-campaign/ASSET_MIGRATION.md) |

## House rules

Every template in this repository follows the same engineering constraints:

- **One document per email.** Content exists once. Breakpoints and colour modes are
  produced with CSS, never with duplicated markup.
- **Tables for all critical layout.** No flexbox, CSS grid, JavaScript, absolute
  positioning, or CSS transforms in the load-bearing structure.
- **Light mode inline, dark mode layered.** Light colours live in inline attributes so
  they need zero CSS support; dark mode is added via `prefers-color-scheme` and
  `[data-ogsc]`/`[data-ogsb]`, and only ever overrides colour.
- **No Base64.** All images are hosted over HTTPS with explicit `width`/`height`.
- **Under Gmail's clip threshold.** Gmail truncates past roughly 102 KB; templates aim
  to stay well beneath it.
- **Unresolved values fail loudly.** Placeholders use the reserved host
  `TODO-REPLACE.invalid` (RFC 2606), so a missed one breaks visibly instead of
  silently pointing somewhere real. Find them with
  `grep -rn "TODO-REPLACE" .`

## Using a template

1. Open the template folder's `README.md` for its architecture and client notes.
2. Work through its `ASSET_MIGRATION.md` to replace every placeholder.
3. Verify no placeholders remain: `grep -rn "TODO-REPLACE" .`
4. Test in a real email client — Litmus, Email on Acid, or a live seed send. A browser
   preview is not an email client.
