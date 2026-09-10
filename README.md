# anu-stories

Revamp of the ANU Stories and refresh of Drupal content type for blogs on the ANU Study site.

## Documentation

| Document | Purpose |
|---|---|
| [`docs/campaign-blog-field-spec.md`](docs/campaign-blog-field-spec.md) | **Developer handoff spec.** Every field on the refreshed `campaign_blog` content type — machine names, types, cardinality, Paragraph components, text format rules, acceptance criteria. This is the source of truth for the build. |
| [`docs/component-ideas.md`](docs/component-ideas.md) | Backlog of story component ideas for later phases. Ideas only — nothing here is committed. |

## Current state

- Field spec drafted, pending sign-off on the open decisions in §7 of the spec (reading time, cohort, featured flag, share buttons, author byline, related events).
- **First build pass landed**, scoped to the components in `Example Template.dc.html` from the Claude Design handoff (the "Study Stories redesign" canvas): `theme/` has Drupal Twig templates + CSS for `p_rich_text` (incl. the numbered listicle), `p_image`, `p_quote`, `p_highlight` and `p_interview`, plus the node header and author byline. See [`theme/README.md`](theme/README.md) for how to wire it into an existing theme, and [`docs/implementation-notes.md`](docs/implementation-notes.md) for the three amendments this pass made to the field spec (a `p_quote` style field, a `small` image width, and the `p_highlight` style→visual mapping).
- [`preview/example-template.html`](preview/example-template.html) is a static, dependency-free rendering of the same components — open it directly in a browser to check against the original design canvas without a Drupal environment.
- Not yet built: banner/hero, related-stories, CTA band, site nav, `p_embed`, `p_divider`, and the Card / Featured card view modes — none appear in Example Template; see `theme/README.md` §5.
