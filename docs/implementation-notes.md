# Implementation notes — Example Template build

**Status:** First build pass, scoped to `Example Template.dc.html`.
**Source:** Claude Design handoff bundle, "Study Stories redesign" (`chats/chat1.md`, `chats/chat2.md`), reviewed against `docs/campaign-blog-field-spec.md`.

This documents what got built, the three places the field spec needed
amending to match what the design canvas actually shows, and what's out of
scope for this pass. Per the field spec's own §12 ("changes go in as a pull
request to this file"), the amendments below are applied directly to
`campaign-blog-field-spec.md` in this same change — this file is the
rationale, that file is the updated contract.

## Scope

Built: everything `Example Template.dc.html` demonstrates — section
heading, highlight (3 styles), quote (2 styles, with/without author),
interview heading + Q&A, listicle, image (2 widths), author byline.

Not built: banner/hero, related-stories, CTA band, site nav, the embed
paragraph (`p_embed`), the divider paragraph (`p_divider`), and the Card /
Featured card view modes — none of these appear in Example Template. They're
specced already; this is a sequencing choice, not a scope cut. See
`theme/README.md` §5.

## Amendment 1 — `p_quote` gets a style field

The spec (§5.4) has one `p_quote` template with an optional attribution.
The prototype has two visually distinct quote treatments in active use —
"Quote B" (big gold quotation mark, no border, author omitted) and "Quote H"
(copper left-bar, author shown) — and the content team explicitly asked
in `chats/chat2.md` for *every* quote style to support both "with author"
and "no author", i.e. attribution presence and visual style are independent
choices.

**Change:** added `field_style`, list (text), required, `gold_mark`
(default) | `copper_bar`, to `p_quote`. Attribution stays controlled by
whether `field_attrib_name` is filled in — no separate style value needed
for that axis.

`copper_bar` uses colours outside ANU's core black/white/gold palette
(`#8B4A1F` / `#C9803F`) — flagged as a deliberate "ANU Reporter" exploration
in `chats/chat2.md`, not yet signed off by Brand & Marketing for site-wide
use. Tokens for it live in `theme/css/tokens/colors.css` as
`--story-copper*`, separated from the core `--anu-*` tokens so they're easy
to find and remove if Brand doesn't approve them.

## Amendment 2 — `p_image` gets a `small` width

The spec (§5.2) lists `standard` | `wide` | `full`. Example Template also
uses a narrower (340px) inline image — the "campus-bikes" photo — that
doesn't fit any of those three (all are ≥ the reading column width).

**Change:** added `small` to `field_width`'s allowed values.

## Amendment 3 — `p_highlight` style mapping is fixed, not free-form

The spec's three `field_style` values (`tip`, `key_takeaway`, `stat`) are
content-meaning categories with no visual spec attached. The prototype has
eight lettered visual options (A–H) explored in
`Study Stories - Component Library.dc.html`, of which Example Template uses
three: A (gold-tint panel + icon), B (bordered card, gold left-bar), F
(copper left-bar on warm grey).

**Decision:** rather than exposing all eight as a style choice (out of
scope per the build brief — see below), each of the three spec'd style
values now has exactly one fixed rendering: `tip` → A, `key_takeaway` → B,
`stat` → F. This means `stat` renders with the copper (off-palette)
treatment, which is a naming mismatch worth a second look — "stat" reads as
"big number callout" but nothing here numbers anything. If a numeric stat
callout is wanted later (it's in `docs/component-ideas.md`'s "high value /
low effort" list), it likely needs its own Paragraph type rather than
reusing `p_highlight`'s `stat` value for a differently-shaped component.
Flagging for the content team rather than deciding unilaterally.

Same off-palette caveat as Amendment 1 applies to the `stat` treatment.

## Why not build all 8 quote / 6 highlight variants

`Study Stories - Component Library.dc.html` documents more options than
Example Template uses. Built only what Example Template shows, on the
assumption that a smaller, precisely-scoped first pass is easier to review
and sign off than a larger speculative one — the full variant set is
straightforward to add on top of this structure (new `field_style` values +
matching CSS blocks) once Brand & Marketing confirms which ones are wanted
site-wide, particularly given the off-palette question above.

## Other adaptations from prototype to production markup

- The prototype's page-level eyebrow text ("Example template — full
  article with spacing & media") is a design-canvas label, not real
  content. The production header eyebrow renders `field_story_tag`
  instead — the closest real field to a small caps label above the title.
- Inline `style="..."` attributes throughout the prototype (by design — see
  `README.md` "About the design files") became real CSS classes in
  `theme/css/components.css`; no inline styles remain, which is also a
  non-negotiable in the field spec itself (§2) for anything an author could
  type, and good practice for anything a developer writes too.
- The listicle in Example Template is hand-built `<div>`/`<span>` markup in
  the prototype; in production it's a plain `<ol>` inside `p_rich_text`'s
  "Story body" format (ol/li are allowed tags — §6), styled via CSS counters
  in `.story-rich-text ol`. This keeps `p_rich_text` as the one component
  authors reach for, rather than adding a dedicated listicle Paragraph type
  the spec doesn't call for.
