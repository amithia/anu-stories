# Story components — integration guide

Templates and CSS implementing `Example Template.dc.html` from the Claude
Design handoff ("Study Stories redesign") against the Paragraph types
defined in [`../docs/campaign-blog-field-spec.md`](../docs/campaign-blog-field-spec.md).
Read [`../docs/implementation-notes.md`](../docs/implementation-notes.md) first —
it covers the three places this build amends that spec.

This is a **drop-in component set**, not a full theme — it assumes an
existing ANU Drupal theme with the content model in §3–§6 of the field spec
already built, and copies in as follows.

## 1. Content model

Build the fields, vocabularies and Paragraph types in the field spec first
(§3–§6), with these two amendments already folded in — see
`docs/implementation-notes.md` for the reasoning:

- `p_quote` gets a new **Style** field, `field_style`, list (text),
  required, values `gold_mark` (default) | `copper_bar`.
- `p_image`'s **Width** field (`field_width`) gets a fourth value, `small`,
  alongside the spec's `standard` | `wide` | `full`.

## 2. Templates

Copy into your theme's `templates/` directory:

```
templates/paragraph/paragraph--p-rich-text.html.twig
templates/paragraph/paragraph--p-image.html.twig
templates/paragraph/paragraph--p-quote.html.twig
templates/paragraph/paragraph--p-highlight.html.twig
templates/paragraph/paragraph--p-interview.html.twig
templates/paragraph/paragraph--p-qa-pair.html.twig
templates/node/node--campaign-blog--full.html.twig
templates/node/story-byline.html.twig
```

`node--campaign-blog--full.html.twig` includes `story-byline.html.twig` via
a namespaced path, `@anu_stories/partials/story-byline.html.twig` — either
register that namespace in your theme's `.info.yml`:

```yaml
libraries:
  - anu_stories/story-components
```
```yaml
# <yourtheme>.info.yml
components:
  namespaces:
    anu_stories/partials: theme/templates/node
```

or, simpler, replace the `@anu_stories/partials/...` path in the two
`{% include %}` calls with your theme's own machine name /
`templates/node/story-byline.html.twig`.

## 3. CSS

Copy `css/tokens/*.css` and `css/components.css` in and attach as a library
in your theme's `<yourtheme>.libraries.yml`:

```yaml
story-components:
  css:
    theme:
      css/tokens/colors.css: {}
      css/tokens/typography.css: {}
      css/tokens/spacing.css: {}
      css/tokens/base.css: {}
      css/components.css: {}
```

`tokens/fonts.css` pulls Public Sans from Google Fonts — **replace with your
theme's existing webfont pipeline** before this ships (see the note at the
top of that file); it's only wired up as-is so `/preview/example-template.html`
resolves standalone without a build step.

If your theme already defines any of these CSS custom properties (colours,
type scale, spacing) under different names, skip `tokens/*.css` and repoint
`components.css`'s `var(--anu-*)` / `var(--fs-*)` / `var(--space-*)` calls
at your existing tokens instead of introducing a second token layer.

## 4. PHP preprocessing

Copy the two functions in `anu_stories.theme` into your theme's `.theme`
file, renaming the `anu_stories_` prefix to your theme's machine name:

- `anu_stories_preprocess_node__campaign_blog()` — computes reading time.
- `anu_stories_preprocess_paragraph__p_interview()` — builds the FAQPage
  JSON-LD block.

## 5. What's deliberately not here

Banner/hero, the related-stories card row, the CTA band, and site
navigation are not part of `Example Template.dc.html` and aren't built in
this pass — they live in the other prototype files under `project/` in the
original design handoff. See `docs/implementation-notes.md` §"Out of
scope" before starting that work, since the banner in particular
(`field_banner_display: parallax`) needs a `prefers-reduced-motion` check
the field spec already flags.

## 6. Verifying against the prototype

`preview/example-template.html` is a static, dependency-free rendering of
this exact component set with real ANU campus photography — open it
directly in a browser to compare against `Example Template.dc.html` without
a Drupal environment. It is markup + CSS classes, not inline styles, so it
doubles as a living reference for what each Paragraph type should produce.
