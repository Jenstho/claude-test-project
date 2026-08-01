# jens-thoemmes.com — analysis & improvement plan

Working document. Everything here is open for discussion — edit inline, strike what you
disagree with, answer the open questions, and I'll revise.

**Status:** draft 2, 2026-08-01 — first draft based on search data only; this one is
based on the source.
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
- **"1995-2026"** in the publications subtitle; the data ends at **2025**.
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

## Plan

Ordered by payoff. P0 is the one that changes the site's trajectory; everything else is
polish by comparison.

### P0 — Make the content crawlable

- [ ] **A1.** Pre-render the 76 publications into `#publicationsList` as static HTML;
      convert the JS from *builder* to *filter*. Keep `publicationsData` as the source of
      truth and generate the markup from it. *(F1 — the single highest-value change)*
- [ ] **A2.** Give each publication a stable `id` anchor so individual works can be
      linked and cited. *(cheap once A1 lands)*

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
- [ ] **C2.** Compute the stat-card counts from the data; fix "1995-2026" → 2025. *(F10)*
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
- [ ] **E2.** Consider fetching from HAL at build time rather than maintaining the list
      by hand — there's already a half-built "HAL sync" affordance in the code
      (`halSyncBtn`). Would eliminate the drift risk in C2 permanently.

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

---

## Notes

- No changes have been made to the site repo — this is analysis only. The portfolio repo
  was cloned read-only; say the word and I'll open a branch there with the P0/P1 fixes.
- Contrast figures are computed from the CSS custom properties, not sampled from a
  rendered page, so they hold for the default tokens only.
- F8 and all visual-design questions are the honest limits of a source-only review.
