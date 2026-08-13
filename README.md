# Lubn PageFly Blocks

Source of truth for the custom HTML/Liquid blocks pasted into PageFly on
[lubn.com](https://lubn.com). Each block is a single self-contained file
(HTML + inline `<style>` + inline `<script>`, no external dependencies)
because that's what PageFly's "HTML/Liquid" element accepts — PageFly does
not support linking to external files, so everything for a block has to
live in that one file.

**This repo is not connected to PageFly or Shopify.** There is no
auto-deploy. Publishing a change means: edit the file here, test it
locally (see below), then copy the whole file's contents and paste it over
the existing block in the PageFly editor, then hit Publish there.

## Why this repo exists

Before this, every change lived only in chat history with no diff, no
history, and no way to tell which version was actually live. This repo
fixes that: every change is a commit with a real message, `git log` shows
the history, and `git blame` shows why a given line exists.

## Structure

```
blocks/
  pricing/                 → /pages/pricing  (id="Plans" PMS plan table,
                              Buy/Rent panel, hardware specs, accessory
                              lightbox with prev/next, swipe, and slide
                              transitions — main gallery and accessories
                              navigate as two separate groups)
  why-lubn-lockbox/         → /pages/lockbox  ("Why Lubn Lockbox?" 3-tab
                              comparison table — Connectivity / Access
                              Control / Business Intelligence)
  save-more-than-70/        → /pages/lockbox  ("Save more than 70% with
                              Lubn" 2-column comparison table)
docs/
  design-system-audit.md    → shared colors, type scale, RWD breakpoints,
                              and known PageFly compatibility gotchas —
                              read this before touching any block's CSS
```

Each block file's own top comment has a per-file change log of fixes
already applied and *why* — read it before changing that file, several
fixes exist specifically to counteract a PageFly default that isn't
obvious from the CSS alone.

## Before you touch any block's CSS — read this

PageFly injects its own default styling into the page (table borders,
`<body>` overflow, span display, etc.), and those defaults have repeatedly
collided with rules in these blocks in ways that only show up on the
*live* PageFly page, never in a plain local preview. The pattern that keeps
recurring:

1. A rule in one of these blocks works fine in isolation.
2. PageFly's own CSS, elsewhere on the real page, happens to target the
   same element with a competing rule of equal or higher specificity.
3. Whichever rule is declared later in the cascade wins — which may not be
   ours.

The fix has consistently been: give the intended rule `!important` *and*
enough selector specificity to beat anything PageFly might already be
doing, and validate on the actual PageFly page, not just a local file.
`docs/design-system-audit.md` and each block's top comment have the
specific instances already found — check there first before assuming a new
CSS bug is something else.

## Local preview

Each block is a standalone file — open it directly in a browser to see it
render (no build step). Product images/icons that live on Shopify's CDN
won't load from a plain local file (cross-origin), so treat those as
broken-image placeholders when testing locally; everything else (layout,
interactions, sticky behavior, lightbox nav) will behave the same as on
the live page.

For anything sticky/scroll-position-dependent (the pricing table header,
the mobile card "stuck" state), test at a real width — some of these
behaviors only trigger past specific breakpoints (768px, 1024px, 1200px).

## Making a change

1. Edit the relevant file under `blocks/`.
2. Test locally in a browser at both desktop and mobile widths.
3. Commit with a message that says what changed and why (see
   `CHANGELOG.md` for the tone/format already established — short,
   specific, states the actual root cause when it's a bug fix).
4. Copy the full file contents into PageFly's HTML/Liquid element for that
   block, replacing what's there.
5. Publish in PageFly, then verify on the live page — not just the PageFly
   preview, which doesn't always reflect the same CSS cascade as the
   published page.
