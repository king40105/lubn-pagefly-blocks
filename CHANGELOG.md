# Changelog

Human-readable summary of the fixes already applied, grouped by block.
Each block's own file has the same information as inline comments closer
to the actual code; this is the scannable version.

## blocks/pricing/pricing-block.html

- **Sticky PMS plan table header**: plan names and prices now stay pinned
  together while scrolling the feature list (previously only the names
  stuck; prices scrolled away).
- **Sticky broke at tablet widths (769–1024px)**: a horizontal-scroll
  container in that range silently disabled `position:sticky`. Removed the
  scroll band; that width range now gets the compact mobile-style layout
  instead, so sticky works at every width.
- **Per-plan description collapses once actually stuck**: the sticky bar
  stays compact while scrolling, full description shows normally at rest.
  Detected via direct `getBoundingClientRect()` comparison on scroll, not
  `IntersectionObserver` (which proved unreliable once PageFly's own
  wrapper markup was in the mix).
- **Sticky offset auto-detects the site nav's real height**, recalculated
  on every scroll frame — not just once on load/resize — because the
  nav's rendered height can change *while scrolling* (e.g. an announcement
  bar collapsing, or a "shrink on scroll" nav), and a stale offset left a
  visible gap with a table row showing through it.
- **`body{overflow-x:clip}` override removed entirely.** It was added to
  fix the sticky table (the theme's own `overflow-x:hidden` on `<body>`
  breaks `position:sticky`), but touching `<body>` site-wide broke
  full-bleed sections on *other* PageFly pages that rely on content
  overflowing past the body's boundary. Trade-off accepted: sticky may not
  work on this page until the theme's own CSS is fixed once, site-wide,
  instead of worked around per-block.
- **Accessory lightbox: added Prev/Next navigation.** Arrow buttons
  (desktop) and swipe left/right (touch), plus arrow-key support. Wraps
  around at either end. Navigates one continuous sequence in DOM order:
  hero photo → 2 thumbnails → unboxing video → Key Storage → Wall-mount
  bracket. Deduplicated by content (`data-zoom`/`data-video` value, not
  just element identity) since each accessory row has two independent
  click targets — the thumbnail and its name text — for the same photo;
  without dedup, Next would appear to do nothing on the first click after
  landing on an accessory.
- **Nav buttons hidden below 640px** — not enough width for a photo plus
  two 44px circular buttons without overlapping it. Swipe covers mobile
  navigation instead.
- **Colors, heading type scale, and tablet breakpoint aligned with
  why-lubn-lockbox/save-more-than-70.** Ink/mut/line/soft tokens updated
  to the shared palette (`#001C39` / `#5A7189` / `#e7e5df` / `#f7f6f2`)
  across all three wrapper scopes (`.lbn-rentbuy`, `.lbnp`,
  `.lbn-specs-wrap`), including two hardcoded colors that weren't using
  variables. "PMS Data & Showing Suite" now uses the same 50/36/24/24
  desktop-to-mobile heading scale as the other two blocks. Tablet
  breakpoint renamed `1023px` → `1024px` to match exactly (was 1px off).
- **Lightbox: swipe now works anywhere on the screen, not just on the
  photo**, and switching photos slides (fade + translateX) instead of
  swapping instantly. Backdrop changed from a dark scrim to a near-white
  tone so the whole screen reads as one continuous surface — previously
  the dark backdrop looked inert, so people didn't realize they could
  swipe there too. Close/Prev/Next buttons restyled for contrast against
  the lighter backdrop. Added a `justSwiped` guard so a swipe that ends
  over the backdrop doesn't also register as a tap-to-close.
- **Lightbox: main gallery and accessories now navigate as two separate
  groups** instead of one continuous sequence. Opening Key Storage or
  Wall-mount bracket and hitting Next only cycles between those two —
  it no longer continues into the hero photo/thumbnails/video. Grouped by
  DOM position (`.closest('.lbn-acc')`) rather than a new data attribute,
  so no markup changes were needed.

## blocks/why-lubn-lockbox/why-lubn-lockbox.html

- Rebuilt as a real `<table>` per tab (Connectivity / Access Control /
  Business Intelligence) instead of images, so the comparison content is
  crawlable/indexable.
- Mobile: card-stacked layout instead of horizontal scroll — each feature
  becomes its own card, the three products' answers stack inside it, each
  with its own small icon (SVG `<symbol>`/`<use>` sprite, defined once,
  referenced everywhere — keeps file size down from what 48 duplicated
  inline SVGs would cost).
- Multiple PageFly-cascade collisions found and fixed: a blanket
  `td span{display:block}` rule kept winning over more specific
  mobile-only rules (icon visibility, decimal-price sizing, price-row
  product-name visibility) because of higher selector specificity —
  fixed by adding `!important` and/or more specific selectors throughout.
- Last-row rounded-corner bug: a desktop-only "only round the bottom
  corners of the last row" rule was winning over the mobile "round all
  four corners" rule due to higher specificity, cutting the last card's
  corners square on mobile. Fixed with a matching-specificity mobile
  override.
- Font sizes matched to the equivalent heading/body tier used elsewhere on
  the live page (measured directly via DevTools, not guessed): 50/36/24/24
  desktop→mobile for the H2, 18/18/16/16 for body copy.

## blocks/save-more-than-70/save-more-than-70.html

- Same card-stacked mobile pattern as `why-lubn-lockbox`, adapted to 2
  columns instead of 3.
- Icon sprite collision: 4 Illustrator-exported SVGs each used the same
  generic class names (`.st0`, `.st1`) for fill color. Combined into one
  sprite, a later icon's `.st0{fill:none}` silently overrode an earlier
  icon's `.st0{fill:#001c39}` globally, making that icon invisible. Fixed
  by inlining each icon's fill color directly onto its paths instead of
  relying on shared CSS classes.
- "Lockbox Retail Price" row has a distinct mobile layout (price → product
  name → full illustrated icon, centered) instead of the icon-beside-text
  layout every other row uses — scoped to that one row via
  `tr.lbns-price-row` so it can't affect any other card.
