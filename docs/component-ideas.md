# Story component ideas — backlog

**Status:** Ideas only. Nothing here is specced or committed.
**Purpose:** A parking lot for ways to make ANU Stories more engaging for prospective students, so we can prioritise deliberately instead of adding components ad hoc.

Phase 1 (see `campaign-blog-field-spec.md`) delivers the components we already use plus Highlight, Interview and CTA. Everything below is phase 2+.

---

## The problem we're actually solving

Stories are currently walls of prose with occasional images. A prospective student skimming on a phone gets no entry points, no reason to scroll past the first screen, and no obvious next step toward applying. The components worth building are the ones that either **break up the scroll**, **build trust**, or **convert**. Anything that only makes the page prettier is lower priority.

---

## High value / low effort — do these first

| Component | What it is | Why it's worth it |
|---|---|---|
| **Key takeaways box** | 3–4 bullets at the top: "What you'll get from this story" | Gives skimmers a reason to commit. Already 80% covered by the `p_highlight` component in phase 1 — mostly an editorial pattern, not a build. |
| **Auto table of contents** | Jump links generated from `<h2>`s on listicles | Free once the body is structured. Big win on long listicles, and it surfaces jump links in Google results. |
| **Reading progress bar** | Thin bar at the top of the viewport | Cheap, and measurably reduces drop-off on long reads. |
| **Author card** | Photo, name, course, one-line bio | Trust. A named student beats institutional voice every time. Already in phase 1 fields. |
| **Degree card** | Auto-pulls the degree from `field_degree` and links to the degree page | **The conversion component.** A story about studying health should link to the health degree. Data's already there — it's a template, not a new field. |
| **Stat callout** | One big number + label | Breaks up prose, very shareable. Covered by `p_highlight` style `stat`. |

---

## Medium effort — worth scoping

| Component | What it is | Notes |
|---|---|---|
| **Photo gallery / carousel** | 3–8 images, swipeable on mobile | Big win for "Hello Canberra" and campus stories. Needs care on performance and keyboard accessibility. |
| **Timeline** | Vertical stepped list with times or dates | Perfect for "a day in the life" and "your first week" stories. |
| **Numbered listicle cards** | Each item as a card with an oversized numeral | Makes listicles look like listicles rather than a document. |
| **Comparison / before-after** | Two columns, or a slider | "What I expected vs what it's actually like" — a format we don't currently have. |
| **Location card** | Map thumbnail, address, travel time from campus | The Canberra stories mention a dozen places with no way to picture where they are. |
| **Accordion FAQ** | Expandable Q&A | Different use case to `p_interview`: FAQ answers common questions, interview tells a story. Also schema-eligible. |
| **Inline course links** | A styled block listing 2–3 related degrees | Second conversion surface, mid-article rather than at the end. |
| **Cost / checklist block** | Itemised list with a running total or tick states | "What it actually costs to live in Canberra" is a question prospective students ask constantly. |

---

## Higher effort / needs a proper case

| Component | Why it's interesting | Why it's not obvious |
|---|---|---|
| **Quiz or poll** | Genuinely high engagement; "which degree suits you" is a real user need | Needs storage, moderation and a privacy review. Probably its own project, not a story component. |
| **Ambient video hero** | Short muted loop instead of a static banner | Performance and accessibility cost is real. Needs a `prefers-reduced-motion` fallback and a hard file-size budget. |
| **Student Q&A / ask-a-student CTA** | Highest-intent conversion surface there is | Depends on an existing service to hand off to. Check what already exists before designing anything. |
| **Personalised story feed** | Related stories filtered by the reader's cohort/degree interest | Needs behavioural data and a consent story. Big. |

---

## Deliberately not recommending

- **Social share button rows** — the heatmap says nobody uses them, and they cost page weight and add third-party tracking. A single "Copy link" is enough.
- **Comments** — moderation burden with no upside for this audience.
- **Infinite scroll between stories** — hurts analytics and accessibility, and buries the CTA at the end of each story.
- **Anything with an animated entrance on every element** — reads as a template showing off, and it fights `prefers-reduced-motion`.

---

## Principles for whatever we build

1. **Every component is a field set, never markup.** The moment a component needs an author to paste HTML, it has failed.
2. **Mobile is the design target, not the adaptation.** Most of this audience is on a phone.
3. **Each component must work alone.** Authors will place them in any order — nothing may depend on what sits above or below it.
4. **Accessible by construction.** Correct semantics, keyboard operable, honours reduced motion. If a component can only work as a `<div>` soup, redesign it.
5. **Every story needs at least one route to a degree page.** Engagement that goes nowhere isn't the goal.
6. **Ship few, use them well.** Twelve components nobody understands is worse than six that are consistently applied. Add a component when three real stories are blocked without it — not before.
