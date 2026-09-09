# `campaign_blog` — Content type field specification

**Status:** Draft for developer handoff
**Owner:** Brand & Marketing (content)
**Audience:** Drupal developer building the refreshed Stories template
**Source:** *Stories Content Type – Fields Needed* (internal doc), plus an audit of the current live template

---

## 1. Why we're doing this

The current Stories template is effectively hardcoded. Layout, quote blocks, image sizing and interview formatting all live as raw markup inside a single WYSIWYG body field. That means:

- Every new story format needs a developer, or an author pasting fragile HTML.
- Markup drifts between stories, so nothing is consistently responsive or accessible.
- We can't restyle a component site-wide — we'd have to edit every node.
- We can't query or reuse content (e.g. "show all Health stories tagged Student experience") because it isn't structured.

The refresh moves the story body from **one blob of HTML** to a **stack of typed components (Drupal Paragraphs)**. Authors choose components from a list and fill in fields; the theme owns all markup and CSS.

**This document is the contract.** It defines every field, its machine name, type, cardinality and behaviour. If something isn't in here, it isn't in scope — raise it rather than improvising.

---

## 2. Architecture decision

| Decision | Choice |
|---|---|
| Body structure | **Paragraphs** (`entity_reference_revisions`), unlimited cardinality |
| Markup ownership | Theme templates only. No layout markup in author-entered content. |
| Text format | New restricted **"Story body"** CKEditor 5 format (see §6) |
| Taxonomy vs list fields | **Taxonomy** for Degree / Cohort / Tag, so terms can change without a deploy |
| Media | All images and video via **Media** entities, never direct file upload |

**Non-negotiable:** authors must not be able to enter `<div class="row">`, inline `style=` attributes, or `<style>` blocks. Every layout decision is a field value, not markup. That is the whole point of the refresh.

---

## 3. Node-level fields

Machine names are proposals — keep them unless they clash with existing fields on the site. Drupal caps field machine names at 32 characters including the `field_` prefix; all of the below fit.

| # | Label | Machine name | Type | Cardinality | Required | Notes |
|---|---|---|---|---|---|---|
| 1 | Title | `title` | Node title (string) | 1 | Yes | Existing. Max 120 chars enforced in the form; longer titles break the card layout. |
| 2 | Summary | `field_summary` | Text (plain, long) | 1 | Yes | Doubles as the meta description **and** the teaser on the Stories landing page and related-story cards. Help text: "150–160 characters. Written for search results, not as an intro paragraph." Character counter on the widget. |
| 3 | Display date | `field_display_date` | Date (date only) | 1 | No | Defaults to node published date; override only when backdating a republished story. Cards show `MMM DD YYYY` (e.g. `Jun 26 2025`). |
| 4 | Story format | `field_story_format` | List (text) | 1 | Yes | `feature`, `listicle`, `interview`, `guide`. Drives template variations (e.g. listicles get an auto table of contents). See §5.1. |
| 5 | Degree | `field_degree` | Taxonomy term ref → `story_degree` | Unlimited | No | Values in §4.1. Autocomplete or checkboxes. Leave empty rather than selecting "None" — do **not** create a "None" term. |
| 6 | Cohort | `field_cohort` | Taxonomy term ref → `story_cohort` | Unlimited | No | Values in §4.2. **OPEN — see §7.2.** |
| 7 | Tag | `field_story_tag` | Taxonomy term ref → `story_tag` | 1 | Yes | Values in §4.3. This is the landing-page filter, so cardinality is deliberately 1 — one story, one filter bucket. |
| 8 | Thumbnail image | `field_thumbnail` | Media ref (Image) | 1 | Yes | Used on cards and social share. Alt text lives on the media entity, not here. Target ratio 16:9. |
| 9 | Banner image | `field_banner` | Media ref (Image) | 1 | Yes | Full-width hero. Minimum 1920px wide. Alt text on the media entity. |
| 10 | Banner display | `field_banner_display` | List (text) | 1 | Yes | `static` (default), `parallax`. Parallax must be disabled automatically when the browser reports `prefers-reduced-motion: reduce`. |
| 11 | Introduction | `field_introduction` | Text (formatted, long) | 1 | Yes | Standfirst / intro paragraph. Restricted format: `<p> <em> <strong> <a>` only. **The divider line under the intro is CSS in the template — it is not authored content.** |
| 12 | Story content | `field_story_content` | Entity reference revisions → Paragraph | Unlimited | Yes | The body. Allowed Paragraph types in §5. Drag-to-reorder, collapsed previews. |
| 13 | Author name | `field_author_name` | String | 1 | No | **NEW.** See §7.5. |
| 14 | Author role | `field_author_role` | String | 1 | No | **NEW.** e.g. "Bachelor of Science student". |
| 15 | Author image | `field_author_image` | Media ref (Image) | 1 | No | **NEW.** Square crop. |
| 16 | Author bio | `field_author_bio` | Text (plain, long) | 1 | No | **NEW.** Max ~300 chars. |
| 17 | Author placement | `field_author_placement` | List (text) | 1 | No | **NEW.** `top`, `bottom`, `both`, `none` (default `bottom`). |
| 18 | Related stories | `field_related_stories` | Entity ref → node: `campaign_blog` | Max 3 | No | Manual curation. When empty, the template falls back to a View: 3 most recent published stories sharing this story's Tag, excluding the current node. See §5.9. |
| 19 | Reading time | *computed* | — | — | — | **Not an author field.** See §7.1. |
| 20 | Feature on landing page | `field_featured` | Boolean | 1 | No | **OPEN — see §7.3.** |
| 21 | Share buttons | — | — | — | — | **OPEN — see §7.4.** |
| 22 | Related events | — | — | — | — | **OPEN — recommend dropping. See §7.6.** |

---

## 4. Taxonomy vocabularies

Create as vocabularies, not list fields, so content can add or rename terms without a code deploy. Seed with exactly these terms.

### 4.1 `story_degree` — Degree
- Arts & social sciences
- Business & economics
- Health, medicine & psychology
- Law, governance & policy
- Security, international affairs & Asia-Pacific studies
- STEM (Science, technology, engineering & mathematics)

*(No "None" term — an unselected field means "not degree-specific".)*

### 4.2 `story_cohort` — Cohort
- Domestic undergraduate
- Domestic postgraduate
- International undergraduate
- International postgraduate
- Higher degree by research
- Short courses

### 4.3 `story_tag` — Tag (landing page filter)
- Hello Canberra
- Our people
- Student experience
- Why choose ANU

---

## 5. Paragraph types (story components)

Each is its own Paragraph bundle. Only the types listed here are permitted in `field_story_content`.

### 5.1 Rich text — `p_rich_text`
The default workhorse. Prose, subheadings, links, lists.

| Field | Machine name | Type | Card. | Req. | Notes |
|---|---|---|---|---|---|
| Text | `field_text` | Text (formatted, long) | 1 | Yes | Uses the restricted "Story body" format (§6). |

Notes for the dev:
- Headings inside this field start at `<h2>`. The node title is the only `<h1>` on the page. `<h3>` allowed, `<h4>` and below are not.
- For `field_story_format = listicle`, the template auto-numbers `<h2>`s and builds a jump-link table of contents from them. Authors must not type "1." into the heading text.

### 5.2 Image — `p_image`

| Field | Machine name | Type | Card. | Req. | Notes |
|---|---|---|---|---|---|
| Image | `field_media` | Media ref (Image) | 1 | Yes | Alt text on the media entity. |
| Caption | `field_caption` | Text (formatted, long) | 1 | No | Visible below the image. `<em> <strong> <a>` only. |
| Width | `field_width` | List (text) | 1 | Yes | `standard` (default, content column), `wide` (breaks out of the column), `full` (full-bleed edge to edge). |

This replaces the current practice of authors resizing images by hand — that's why sizes are inconsistent today. **Width is a field, not a pixel value.** Responsive image styles are configured per width option by the developer.

### 5.3 Embed — `p_embed`

| Field | Machine name | Type | Card. | Req. | Notes |
|---|---|---|---|---|---|
| Media | `field_media` | Media ref (Remote video / social embed) | 1 | Yes | YouTube, Vimeo, Instagram reel. TikTok — see risk below. |
| Caption | `field_caption` | Text (formatted, long) | 1 | No | Visible description below the embed. |

⚠️ **Build risk — please scope before committing to a date.** Drupal core's oEmbed handles YouTube and Vimeo out of the box. Instagram and TikTok do **not** work via core oEmbed: Instagram's oEmbed endpoint now requires a Meta app ID and access token, and TikTok needs its own provider. This means either a contrib module (e.g. Social Embed / oEmbed Providers), a custom media source, or accepting that Instagram reels are pasted as blockquote embeds. **Flag which route you're taking and what it costs before build starts.** Also confirm the embed script's impact on Core Web Vitals — third-party embeds should be lazy-loaded / click-to-load.

### 5.4 Quote — `p_quote`

| Field | Machine name | Type | Card. | Req. | Notes |
|---|---|---|---|---|---|
| Quote | `field_quote` | Text (plain, long) | 1 | Yes | Plain text. The template supplies the quotation marks and styling — authors must not type decorative quote marks. |
| Attribution name | `field_attrib_name` | String | 1 | No | |
| Attribution role | `field_attrib_role` | String | 1 | No | e.g. "Bachelor of Health Science, 2024". |
| Image | `field_media` | Media ref (Image) | 1 | No | Optional headshot beside the quote. |

Renders as `<figure><blockquote>…</blockquote><figcaption>…</figcaption></figure>`.

### 5.5 Highlight — `p_highlight` *(NEW)*
The "potential to do a highlight?" idea from the source doc. A visually distinct call-out box for a key takeaway, tip or statistic.

| Field | Machine name | Type | Card. | Req. | Notes |
|---|---|---|---|---|---|
| Heading | `field_heading` | String | 1 | No | |
| Text | `field_text` | Text (formatted, long) | 1 | Yes | `<p> <ul> <ol> <li> <strong> <em> <a>`. |
| Style | `field_style` | List (text) | 1 | Yes | `tip`, `key_takeaway`, `stat`. |

### 5.6 Interview Q&A — `p_interview` *(NEW)*
The "NEW – Interview heading" idea. Today interviews are hand-formatted as bold `Q:` lines in the body — inconsistent, and invisible to search engines as Q&A.

| Field | Machine name | Type | Card. | Req. | Notes |
|---|---|---|---|---|---|
| Speaker | `field_speaker` | String | 1 | No | Shown once above the block, e.g. "George". |
| Speaker image | `field_media` | Media ref (Image) | 1 | No | |
| Q&A pairs | `field_qa_pairs` | Entity ref revisions → `p_qa_pair` | Unlimited | Yes | Nested paragraph, below. |

**`p_qa_pair`** (nested, not selectable at the top level):

| Field | Machine name | Type | Card. | Req. |
|---|---|---|---|---|
| Question | `field_question` | Text (plain, long) | 1 | Yes |
| Answer | `field_answer` | Text (formatted, long) | 1 | Yes |

Dev note: emit `FAQPage` / `Question` schema.org JSON-LD for these — free rich-result eligibility that the current hardcoded format throws away.

### 5.7 Call to action — `p_cta`

| Field | Machine name | Type | Card. | Req. | Notes |
|---|---|---|---|---|---|
| Heading | `field_heading` | String | 1 | Yes | |
| Text | `field_text` | Text (formatted, long) | 1 | No | |
| Link | `field_link` | Link | 1 | Yes | Internal or external. Link text required. |
| Style | `field_style` | List (text) | 1 | Yes | `inline` (mid-article band), `end` (full-width, end of story). |

### 5.8 Divider — `p_divider`
No fields. A styled section break. Included so authors stop faking one with an empty paragraph or a horizontal rule pasted as HTML.

### 5.9 What is *not* a Paragraph
Related stories, the author card and the banner are **node fields rendered by the template in fixed positions**, not components an author places. Keeping them out of the body stops authors from accidentally putting "You may also like" in the middle of a story.

---

## 6. Text format: "Story body"

Create one new CKEditor 5 text format, used by every formatted-text field above. This is the single most important guardrail in the refresh.

**Allowed tags:** `<p> <h2> <h3> <ul> <ol> <li> <strong> <em> <a href hreflang> <br>`

**Explicitly disallowed:** `<div> <span> <style> <script> <table> <img> <iframe> <h1> <h4> <h5> <h6>`, the `style` attribute, and the `class` attribute.

- Images and embeds go through the Image / Embed paragraphs, never inline in text.
- "Source" / view-source button: **off** for content authors. This is what allowed the hardcoded markup in the first place.
- Enable Drupal's "Limit allowed HTML tags and correct faulty HTML" and "Convert line breaks" filters.
- Editor toolbar: bold, italic, link, bulleted list, numbered list, heading (H2/H3 only), remove format. Nothing else.
- Configure paste-as-plain-text so pasting from Word doesn't reintroduce inline styles.

---

## 7. Open decisions — content team to sign off before build

These are the items flagged "needed?" in the source doc. Each has a recommendation; please confirm or overrule, and the developer builds to the confirmed answer.

### 7.1 Reading time — **Recommend: keep, but compute it**
Don't make this an author-entered field; it will go stale the moment a story is edited. Compute it in a preprocess hook: total word count across all `p_rich_text` and `p_interview` text, divided by 200 words/minute, rounded up, minimum 1. Display as "4 min read" beside the display date.

### 7.2 Cohort — **Recommend: keep, but hide it from the filter UI**
The heatmap argument for dropping it is about the *filter*, not the *data*. Cohort is genuinely useful for targeting stories into audience-specific journeys (e.g. surfacing international postgraduate stories on international course pages) even if nobody filters by it on the landing page. Keep the field, remove it from the public filter bar, and revisit in six months. If nothing has consumed it by then, drop it.

### 7.3 Feature on landing page — **Recommend: replace the boolean with curated references**
A per-node "Feature this" checkbox always ends the same way: everything is featured and the flag means nothing, and nobody knows who ticked what. Instead, put a `field_featured_stories` entity-reference field (max 3, ordered) **on the Stories landing page** itself. One place to look, explicit ordering, and one person owning it. If you'd rather keep the boolean, it needs an editorial rule about how many can be on at once — and someone to enforce it.

### 7.4 Share buttons — **Recommend: replace with a single "Copy link" button**
The heatmap says the current button row isn't used, which matches behaviour everywhere: on mobile people share through the OS share sheet, not a widget. Third-party share buttons also carry a tracking/privacy cost and slow the page. Suggest one lightweight "Copy link" button using the native Web Share API where available. Low cost, no third-party scripts. If share is dropped entirely, that's also a defensible call — just make it deliberately.

### 7.5 Author / byline — **Recommend: add it, as node fields (as specced above)**
Currently missing, and it's the single biggest credibility gap: prospective students trust a named student far more than an anonymous institutional voice. Four plain fields on the node plus a placement selector is the cheapest version.
*Consider later, not now:* if the same students write repeatedly, this should become a proper `story_author` content type referenced from the story, so a bio is edited once and updates everywhere. Don't build that in phase 1 — but pick machine names that don't block it.

### 7.6 Related events — **Recommend: drop**
Never used, and the only genuine candidates (Open Day, Discovery Day) are seasonal and better served by the CTA component pointing at the events site. Adding a field we won't populate is maintenance debt. If it's genuinely wanted later, `p_cta` already covers 90% of it.

---

## 8. Display / view modes

| View mode | Used on | Fields shown |
|---|---|---|
| **Full** | Story page | Banner, title, display date, reading time, tag, introduction, story content, author card, related stories |
| **Card** | Stories landing page, related stories | Thumbnail, display date, title, "Read more »" link |
| **Featured card** | Landing page hero slot | Thumbnail (larger), tag, title, summary, display date |

Card markup should be one component reused in all three placements — the current site has at least two visually similar but separately built card treatments.

---

## 9. Non-functional requirements

- **Responsive:** use ANU's existing Bootstrap-flavoured grid utilities (`row-cols-*`, `gy-*`) in templates. No bespoke breakpoint CSS where a utility exists. Components must be tested at 320px, 768px, 1024px and 1440px.
- **Accessibility (WCAG 2.1 AA):** correct heading order (one `<h1>`, no skipped levels); alt text enforced as required on the media entity for every image; `<figure>`/`<figcaption>` for captioned media; visible focus states on all links and buttons; parallax disabled under `prefers-reduced-motion`; embeds have accessible titles.
- **Performance:** responsive image styles + WebP derivatives per width option; `loading="lazy"` on everything below the fold (never on the banner); third-party embeds lazy- or click-to-load.
- **SEO:** `field_summary` → meta description and `og:description`; `field_thumbnail` → `og:image`; `Article` schema on the node; `FAQPage` schema on interview paragraphs; canonical URLs unchanged from the existing path pattern.
- **Editorial:** every Paragraph type needs a clear label, help text and a meaningful collapsed summary in the admin UI — an author must be able to tell components apart without expanding them.

---

## 10. Migration

Existing stories are a single body blob and **cannot be auto-split into Paragraphs reliably** — the components are only distinguishable by inconsistent inline markup.

Suggested approach:
1. Content team counts live stories and ranks them by pageviews.
2. Top ~20 by traffic: manually re-authored into the new components (highest quality, gets the good stories looking right).
3. The remainder: bulk-migrate the whole body into a single `p_rich_text` paragraph. Renders correctly, just doesn't use the new components until someone edits it.
4. Retire anything with negligible traffic rather than migrating it.

Decide 2 vs 3 vs 4 per story *before* the developer writes the migration.

---

## 11. Acceptance criteria

The build is done when:

- [ ] All node fields in §3 exist with the specified machine names, types and cardinality.
- [ ] The three vocabularies in §4 exist and are seeded with exactly the listed terms.
- [ ] All Paragraph types in §5 exist and are the only types selectable in `field_story_content`.
- [ ] The "Story body" format in §6 exists, is the only format available to content authors on story fields, and strips disallowed tags on save.
- [ ] An author can build a complete story — banner, intro, text, image at all three widths, embed, quote, highlight, interview, CTA — **without typing any HTML.**
- [ ] Every component passes automated accessibility checks and manual keyboard navigation at all four breakpoints.
- [ ] Related stories falls back to tag-matched recent stories when the manual field is empty.
- [ ] Reading time computes correctly and updates when a story is edited.
- [ ] A test story renders correctly in Full, Card and Featured card view modes.

---

## 12. How this gets handed over

1. **Sign off §7 first.** The open decisions change field counts — don't start the build with them unresolved.
2. **Raise one ticket per section** (node fields, vocabularies, paragraph types, text format, templates, migration), all linking back to this document.
3. **Config-first review:** the developer builds the fields and content model in a dev environment and demos the *author experience* before any theming. Cheapest point to catch a wrong field type.
4. **Author test:** one content team member builds a real story end to end in dev and reports anything that needed a workaround. A workaround now is hardcoded HTML in six months.
5. **Then theme.** Design and templating on top of a confirmed content model, never in parallel with it.
6. **This document is the source of truth.** Changes go in as a pull request to this file, not as Slack messages — otherwise we end up back where we started.
