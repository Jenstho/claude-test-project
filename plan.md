# jens-thoemmes.com — analysis & improvement plan

Working document. Everything here is open for discussion — edit inline, strike what you
disagree with, answer the open questions, and I'll revise.

**Status:** draft 3, 2026-08-01 — adds the agreed direction (HAL as source of truth,
annual maintenance, coverage through 2026, remove the 2026+ button). Draft 1 was based on
search data only; draft 2 onward is based on the source.
**Source analysed:** `Jenstho/jens-thoemmes-portfolio` @ `b4507cf`
("SEO enhancements: meta tags, structured data, social sharing", 2025-12-06)
**Site:** single-page static site on GitHub Pages (`CNAME` → jens-thoemmes.com)

```
index.html          124 KB / 3,233 lines   (markup + CSS + JS + all publication data)
assets/social-card.svg
robots.txt
sitemap.xml
README.md
```

---

## Corrections to draft 1

Draft 1 was written without network access and inferred several things from search
results. Three were wrong, and the record should say so:

| Draft 1 claimed | Actually |
|---|---|
| No `sitemap.xml`, `robots.txt`, canonical or JSON-LD | All present, plus Open Graph, Twitter cards, Dublin Core, Google Scholar `citation_*` tags and ORCID |
| No language strategy | Full EN/FR/DE translation layer exists, `data-en/fr/de` on content and a `translations` object |
| Publication list probably missing | 76 entries with authors, publisher, ISBN, pages, tags, themes — well structured |

The SEO groundwork is done. What holds the site back is narrower and more specific than
draft 1 supposed: **the good content is invisible to crawlers, and the good i18n work is
invisible to search engines.** Details below.

---

## Findings

Every finding below is verified against the source unless marked otherwise.

### F1 — Publications are client-side only: the site's substance is uncrawlable ⚠️ top priority

`index.html:1019` ships an empty container:

```html
<div id="publicationsList" class="publications-container">
  <!-- Publications will be loaded here -->
</div>
```

All 76 entries live in a JS object (`publicationsData`, line 1185) and are injected at
line 2473 via `container.innerHTML = publications.map(...)`. Crawlers that don't execute
JS — and most academic, social and AI crawlers don't — see none of it.

This confirms the hypothesis from draft 1 and explains why search knows the site only at
title depth, and why it ranks 7th on its own domain name. The static HTML contains the
hero bio, seven theme cards and a footer. That's the entire indexable corpus of a page
whose actual value is 76 publications spanning 1995–2025.

**Fix:** emit the 76 entries into the HTML at build time and let JS filter what's already
there, rather than create it. Progressive enhancement, not rendering. Concretely: keep
`publicationsData` as the single source of truth, add a small script that renders it into
`#publicationsList` and commits the result; the existing filter code then operates on
real DOM nodes. This is the highest-value change on the list by a wide margin.

### F2 — Trilingual content is invisible to search engines

The translation layer is genuinely good — EN/FR/DE across hero, themes, filters and
footer. But:

- `<html lang="en">` is hard-coded and **never updated** when the language changes
  (no `documentElement.lang` assignment anywhere in the file)
- **no `hreflang` links** (0 occurrences)
- language changes don't alter the URL, so FR and DE have no indexable address
- `og:locale:alternate` advertises `fr_FR` and `de_DE` that no crawler can reach

Net effect: three languages of content compete as one English page. For a researcher
publishing across France and Germany, this is a real loss of reach.

**Fix:** minimum — set `document.documentElement.lang` on switch. Proper — give each
language a URL (`/fr/`, `/de/`, or `?lang=fr` with canonical + `hreflang`), which pairs
naturally with the F1 fix since both need a small build step.

### F3 — Social preview image is an SVG, so previews break

`og:image` and `twitter:image` point at `assets/social-card.svg`, declared
`1200×630`. Facebook, LinkedIn, X and Slack do **not** render SVG previews — they will
show no image at all. For a page shared mainly on LinkedIn and X, this is a silent
failure on every share.

**Fix:** export the SVG to PNG at 1200×630, point both tags at it, keep the SVG as the
editable source.

### F4 — Author tooling ships to visitors

For every visitor, the page injects a green **"Extract Data"** button into the nav
(line 2936), binds `Ctrl+Shift+E`, offers a "Copy Your Publications Data" modal with a
raw-JSON textarea (line 2910), a BibTeX generator, and logs tips to the console:

```
🔧 Quick access: Type "publicationsData.copyToClipboard()" in console to copy all data
💡 Tip: Click the green "Extract Data" button or press Ctrl+Shift+E …
```

Nothing here is a security issue — the data is public by design. It's a
credibility and clarity issue: a visitor sees a maintenance control they have no use
for, in a colour that competes with the site's own accent.

**Fix:** decide who it's for (Q4). A per-publication BibTeX/RIS export is a genuine
service to academic readers and worth keeping. The bulk JSON dump, the keyboard
shortcut and the console tips belong behind a `?dev=1` flag or in a separate script.

### F5 — `sitemap.xml` is mostly non-functional

Five URLs, four of them fragments (`/#research`, `/#publications`, `/#about`,
`/#contact`). Search engines ignore fragment URLs — they are not separate resources. And
of those four anchors, **only `#publications` exists in the document**; `#research`,
`#about` and `#contact` match no `id` (verified against every `id` in the file).

**Fix:** reduce the sitemap to the single real URL — or, once F1/F2 land, list the real
language URLs. Add the missing `id`s to the research, about and contact regions so the
anchors at least resolve.

### F6 — Accessibility: one clear failure, otherwise decent

Credit first: skip-nav link, `aria-label` on every control, `role="button"` +
`tabindex="0"` on the cards, **and** a real `keydown` handler for Enter/Space
(line 2596). That's better than most academic sites.

The failures:

- **`--text-tertiary: #9ca3af` on white = 2.54:1** — fails WCAG AA for normal text
  (4.5:1) and even the 3:1 large-text threshold. `--text-secondary: #6b7280` is fine at
  4.84:1. Darkening the tertiary token to roughly `#6b7280` or below fixes it.
- **No `<main>` landmark** — screen-reader users get no "jump to main content" target
  (the skip-nav points at `#publications`, a `<section>`).
- **Heading hierarchy** — "Research Themes" is an `<h2>` *inside* `<section
  id="publications">`, so themes read as a subsection of publications. It should be its
  own `<section>`.
- **Modal has no Escape key and no focus trap** — keyboard users can't dismiss it.
- **No `prefers-reduced-motion`** despite 10 `transition` rules and smooth scrolling.

### F7 — Theme and language reset on every page load

Zero `localStorage` in the file. A visitor who picks Deutsch and dark mode gets English
and light mode back on reload, and on every subsequent visit. Two lines to fix, and it
removes a small recurring irritation.

### F8 — Only one breakpoint

A single `@media (max-width: 768px)`. Nothing between tablet and desktop, and no upper
bound on line length for large screens — the 7-card theme grid and the publication list
are the parts most likely to suffer. Untested claim, since I can't render the page (Q5).

### F9 — Two unused `preconnect` hints

`fonts.googleapis.com` and `fonts.gstatic.com` are preconnected (lines 36–37), but no
Google Fonts stylesheet is ever loaded — the site uses a system font stack
(`-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto`). So both hints open TLS
connections to Google that are never used. Delete them: two fewer third-party
handshakes, and one less GDPR question for an EU-hosted academic site.

### F10 — Metadata inconsistencies and drift risks

- **`<title>` is the weakest string on the page**: `Jens Thoemmes - Sociologist &
  Research Professor` — no field, institution or location, while `og:title` is already
  much stronger (`CNRS Sociologist | Working Time & Industrial Relations`). Align the
  title with the og:title.
- **Publication counts are hardcoded** in the stat cards (7 / 23 / 40 / 3 / 2 / 1 / 76).
  I verified they currently match the data exactly — but they'll drift on the next
  publication. Compute them from `publicationsData`.
- **"1995-2026"** in the publications subtitle (three languages), but the data ends at
  **2025**. Per your decision this is fixed by *adding* the 2026 works, not by editing
  the label down — see [Direction](#direction-hal-as-the-source-of-truth).
- **Only 2 DOIs across 76 entries.** 68 link to HAL, which is good practice — but DOIs
  are what citation managers and Scholar consume. Worth backfilling where they exist.
- `sitemap.xml` `lastmod` is a static `2025-12-06`.

### F11 — No email address anywhere

The footer "Contact" section lists three institutional affiliations and no way to make
contact — no `mailto:`, no form, no address (verified: zero `mailto:` in the file). The
Academic Profiles column is good, but a reader who wants to invite you to a conference
has to go via ResearchGate.

Also no CV file in the repo.

---

## Affiliation update — CERTOP → UTOPI

Per your correction: **UTOPI — UMR 5311, CNRS** (Unité de recherche Transitions,
Organisations, Politiques, Inégalités), formed 1 January 2026 from the merger of CERTOP
(UMR 5044) and LaSSP; plus **Taylor's University**, Malaysia.

The site predates the merger, so CERTOP appears in five places that all need changing:

| Location | Current |
|---|---|
| `index.html:868` | `<span class="position-badge">CERTOP - Université Toulouse II</span>` |
| `index.html:1030` | footer `<li>CERTOP-CNRS</li>` |
| `index.html:31` | `citation_author_institution` = `CNRS, CERTOP, Université Toulouse II` |
| JSON-LD `sameAs` | `https://certop.cnrs.fr/thoemmes-jens/` → `https://utopi.cnrs.fr/…` |
| JSON-LD `affiliation` | CNRS only; should name UTOPI and Taylor's |

Worth keeping one bridging sentence — "UTOPI (UMR 5311), formed in 2026 from CERTOP and
LaSSP" — since readers and citations will carry the old name for years. Note the
`citation_author_institution` tag feeds Google Scholar, so it matters more than its
obscurity suggests.

---

## Direction: HAL as the source of truth

**Decided (2026-08-01):** the publication list is imported from HAL rather than
maintained by hand; the "Check for New Publications (2026+)" button is removed; the site
is refreshed **at least once a year**; coverage runs **through 2026**.

### What the 2026+ button actually does — why removing it loses nothing

`index.html:2614–2778` implements a `HALSyncManager` that queries the HAL API at
`api.archives-ouvertes.fr` for `authIdHal_s:"jens-thoemmes"` restricted to
`producedDateY_i:[2026 TO currentYear+1]`. Its own comment explains the restriction:

```js
// IMPORTANT: Only checks for publications from 2026 onwards
// This is to avoid importing all 100+ existing HAL publications
```

The decisive detail: results are pushed into the in-memory `allPublications` array and
re-rendered — **and never persisted anywhere.** No commit, no storage, no file write. The
visitor who clicks it sees new entries; the next page load shows the original 76 again,
for them and for everyone else. Two further consequences:

- it does the fetch **in the visitor's browser**, so every visitor pays for it and the
  page depends on HAL's uptime and CORS policy at read time
- because it runs after load, nothing it finds is ever crawlable — the same problem as F1

So the button presents itself as a maintenance mechanism while being incapable of
maintaining anything. Removing it is correct, and the annual build-time import below
replaces it properly. That also removes the awkward "100+ publications" note — a
build-time import *wants* the full set.

### The design: import at build time, not at read time

```
HAL API ──(annual, run locally or by GitHub Action)──> publications.json
                                                            │
                                             build script ──┤
                                                            ▼
                                        index.html with 76+ entries as static HTML
```

This kills two birds: it satisfies the HAL-import requirement **and** fixes F1, because
the generated HTML contains every publication as real text. It's the same build step Q3
asks about — so answering Q3 "yes" unlocks A1, B2 and this together.

Runtime fetching is the alternative and is worse on every axis that matters here: still
uncrawlable, slower, and broken whenever HAL is down.

### Three things a naive full import would break

Worth designing around before writing the script — the current hand-curated data is
richer than what HAL returns:

1. **Themes would be lost.** The seven theme cards filter on a `themes` array that has no
   HAL equivalent; the existing transform sets `themes: []` and `tags: ['From HAL']`.
   Import everything as-is and the theme filter silently stops working. Needs either a
   keyword→theme mapping in the build script or a small hand-maintained overlay file
   keyed by HAL docid.
2. **Two publications are not in HAL.** Of 76 entries, 74 link to hal.science /
   shs.hal.science; the outliers are the Lexington/Rowman book
   (`rowman.com/ISBN/9781666969085`, ISBN 978-1-66696-908-5) and one SSRN preprint. A
   HAL-only list drops both — including one of the two 2024 books. The overlay file
   should carry them.
3. **Curated fields would thin out.** ISBN, publisher, page ranges and per-item
   `language` are in the current data; the existing transform hardcodes `language: 'en'`
   even though HAL exposes `language_s`. Map the real field, and keep ISBN/publisher in
   the overlay.

**Reconciliation is mandatory, not optional.** The first run must diff the HAL result set
against the existing 76 and report what's only-in-HAL and what's only-local. HAL coverage
of the 1995–2005 work is unverified (Q7) — a straight replacement could silently delete a
decade. Merge, then review the diff, then commit.

### Annual maintenance routine

Target: once a year, ~30 minutes. Suggested anchor — January, so the year just ended is
complete.

1. Run the import script; it rewrites `publications.json` and regenerates `index.html`.
2. Read the diff. New entries only? Commit. Anything disappeared? Investigate before
   committing — that's the reconciliation guard doing its job.
3. Update `sitemap.xml` `lastmod`.
4. Bump the year range in the subtitle — it appears in **all three** translation blocks
   (`index.html:878`, `1060`, `1103`, `1146`), which is exactly the kind of detail an
   annual routine forgets. Better: generate it from the data's max year so it can't drift.
5. Re-check the affiliation line if anything institutional has changed.

A GitHub Action on a yearly `schedule:` cron could open a PR with the regenerated files,
turning the routine into "review and merge". Worth doing only if Q3 is yes; the manual
script is a fine stopping point.

### Coverage through 2026

The data currently ends at 2025 while all three subtitles claim 1995–2026. The import
should pull everything through 2026, which closes the gap. If HAL has no 2026 records for
you yet, the honest fix is the reverse — label the range 1995–2025 until it does, rather
than advertise a year with nothing behind it. Q8.

---

## Plan

Ordered by payoff. P0 is the one that changes the site's trajectory; everything else is
polish by comparison.

### P0 — HAL import + crawlable content

These four are one piece of work; doing them together is much less effort than separately.

- [ ] **A1.** Write the HAL import script: query
      `authIdHal_s:"jens-thoemmes"` for the full date range, map `language_s` properly,
      emit `publications.json`. Include the reconciliation diff against the current 76 and
      **do not auto-delete** anything that's only local.
- [ ] **A2.** Add the overlay file for what HAL doesn't carry: `themes`, the Rowman book,
      the SSRN preprint, ISBNs, publishers.
- [ ] **A3.** Pre-render publications into `#publicationsList` as static HTML; convert the
      JS from *builder* to *filter*. *(F1 — the single highest-value change, and A1 makes
      it nearly free)*
- [ ] **A4.** Remove the `HALSyncManager` class, `addHALSyncButton()`, `syncWithHAL()` and
      the button markup (`index.html:2614–2778`, ~165 lines). Nothing else references
      them.
- [ ] **A5.** Extend coverage through 2026, or correct the three subtitles to the real max
      year — whichever the data supports. Generate the range from the data. *(Q8)*
- [ ] **A6.** Give each publication a stable `id` anchor so individual works can be linked
      and cited. *(cheap once A3 lands)*
- [ ] **A7.** Write the annual routine into `README.md` — the script invocation, the
      reconciliation check, and the four places the year range appears. In a year you will
      not remember any of it.

### P1 — Accuracy and reach

- [ ] **B1.** Affiliation → UTOPI (UMR 5311) in all five places above, with the bridging
      sentence, and the same update on your HAL, ORCID, ResearchGate and Scholar
      profiles.
- [ ] **B2.** Fix i18n visibility: set `documentElement.lang` on switch; add `hreflang`;
      give FR/DE real URLs. *(F2)*
- [ ] **B3.** `og:image`/`twitter:image` → PNG 1200×630. *(F3 — every share is currently
      previewing without an image)*
- [ ] **B4.** Align `<title>` with the stronger `og:title`. *(F10)*
- [ ] **B5.** Add an email address to the footer, and a dated CV PDF. *(F11)*
- [ ] **B6.** Link the domain from HAL, ORCID, ResearchGate, Scholar, the UTOPI staff
      page and the Temporalités masthead. Still the cheapest authority gain available,
      and it compounds with A1. *(~30 min)*

### P2 — Correctness and hygiene

- [ ] **C1.** Sitemap: drop the fragment URLs, or list real language URLs after B2. Add
      the missing `#research` / `#about` / `#contact` ids. *(F5)*
- [ ] **C2.** Compute the stat-card counts from the data rather than hardcoding
      7/23/40/3/2/1/76 — the annual import guarantees these drift otherwise. *(F10)*
- [ ] **C3.** Remove the two unused Google Fonts `preconnect` hints. *(F9)*
- [ ] **C4.** Persist theme + language in `localStorage`. *(F7)*
- [ ] **C5.** Move the bulk-export tooling behind a flag; keep per-publication
      BibTeX/RIS export as a reader feature. *(F4 — needs Q4)*
- [ ] **C6.** Backfill DOIs where they exist. *(F10)*

### P3 — Accessibility and layout

- [ ] **D1.** Darken `--text-tertiary` to pass WCAG AA. *(F6 — the one hard failure)*
- [ ] **D2.** Wrap content in `<main>`; point skip-nav at it; move "Research Themes" into
      its own `<section>`. *(F6)*
- [ ] **D3.** Escape key + focus trap for the modal. *(F6)*
- [ ] **D4.** `@media (prefers-reduced-motion: reduce)` block. *(F6)*
- [ ] **D5.** Add breakpoints between 768 px and desktop; cap measure on wide screens.
      Verify by rendering. *(F8)*

### P4 — Structural, discuss before doing

- [ ] **E1.** Split `index.html` (3,233 lines of markup + CSS + JS + data) into
      `index.html` / `style.css` / `app.js` / `publications.json`. Improves
      maintainability and lets the browser cache the parts that rarely change. Argues
      against: a single file is genuinely simple to deploy on GitHub Pages, and you may
      prefer that. *(Q3)*
- [ ] **E2.** ~~Consider fetching from HAL at build time~~ — **decided, promoted to P0.**
      See [Direction](#direction-hal-as-the-source-of-truth).
- [ ] **E3.** Optional: GitHub Action on a yearly cron that runs the import and opens a PR
      with the regenerated files, reducing the annual routine to review-and-merge. Only
      worthwhile if Q3 is yes.

---

## Open questions

1. **Q1.** Exact title of the Taylor's University position? The page says "CFW -
   Taylor's University Malaysia" / "Centre for Future of Work" — is that current, and
   should it read as a chair?
2. **Q2.** Email address to publish, and is there a CV PDF I should wire in?
3. **Q3.** Are you willing to add a build step (a small Node or Python script, run
   locally or via GitHub Actions)? A1, B2 and E1 are much cleaner with one, but all can
   be done by hand if you'd rather keep the zero-tooling setup.
4. **Q4.** Who is the "Extract Data" button for — you, co-authors, or readers? Determines
   whether it's removed, flagged, or replaced with per-publication export.
5. **Q5.** Can you send a screenshot at desktop and phone width? I can read the CSS but
   not render it, so F8 and any visual-design judgement are unverified.
6. **Q6.** Priority order of audiences — academic peers, students, journalists,
   Malaysian/Asian institutional contacts? Changes what belongs above the fold. Right
   now a visitor meets seven theme cards and a stats grid before a single publication
   title; if peers are the priority, the two 2024 books deserve that space.
7. **Q7.** Is `jens-thoemmes` the right HAL `authIdHal_s`, and does HAL hold your
   1995–2005 work? This decides whether the import can lead or must merge — the
   reconciliation guard covers us either way, but knowing in advance saves a round.
8. **Q8.** Does HAL have 2026 records for you yet? If not: label the range 1995–2025 for
   now, or leave 2026 in place and fill it at the January refresh?
9. **Q9.** The 2026+ button is going regardless. The bulk "Extract Data" tooling is a
   separate decision — still open under Q4.

---

## Notes

- No changes have been made to the site repo — this is analysis only. The portfolio repo
  was cloned read-only; say the word and I'll open a branch there with the P0/P1 fixes.
- Contrast figures are computed from the CSS custom properties, not sampled from a
  rendered page, so they hold for the default tokens only.
- F8 and all visual-design questions are the honest limits of a source-only review.
