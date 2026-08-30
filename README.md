# Diaspora Email

## Architecture
One HTML document contains one copy of each campaign section. Responsive table-based layout is used to move the same content from desktop to the mobile proportions shown by the supplied Figma states.

## Responsive states
Desktop is based on the 640px Figma composition. At <=600px, the hero becomes vertically oriented, feature cards remain in a 2-column grid, and the How It Works / Campaign Details sections follow the supplied mobile arrangement.

## Light/dark
Light-mode colors are inline. A `prefers-color-scheme: dark` block provides the approved dark palette for clients that support the media query. Gmail may still apply its own dark-mode transformations in some contexts, so final client QA is required.

## Gmail clipping
The HTML avoids duplicated desktop/mobile or light/dark markup, Base64 images, JavaScript, and large inline SVG blobs. The document is intended to remain comfortably below Gmail's practical clipping threshold, but exact clipping behavior still depends on final template wrapping and transport.

## Assets
Raster images are referenced by HTTPS URLs. Replace any unresolved Diaspora-hosted URLs in `ASSET_MIGRATION.md` before production.

## SendGrid
Paste the complete HTML into a SendGrid HTML editor/template. No SendGrid-specific personalization variables are invented. Existing unsubscribe/manage-preference links are placeholders unless you map them to your verified SendGrid links.

## Known limitations
Actual Gmail iOS, Gmail Android, Apple Mail, and Outlook client rendering was not executed in this environment. Browser preview is not equivalent to device-client testing.

HTML optimization can improve technical quality and reduce potential deliverability issues, but Inbox vs Spam placement also depends on SendGrid/domain authentication, SPF, DKIM, DMARC, sender reputation, sending IP reputation, recipient engagement, complaint rate, list quality and other factors.
