# jens-thoemmes.com — analysis & improvement plan

Working document. Everything here is open for discussion — edit inline, strike what you
disagree with, add answers under "Open questions", and I'll revise.

**Status:** draft 1, 2026-08-01
**Scope:** discoverability, positioning and content of https://jens-thoemmes.com
**Not yet covered:** technical audit (markup, accessibility, responsive layout,
performance) — see [Blocker](#blocker) below.

---

## Affiliation (as given, to be reflected everywhere)

- **UTOPI — UMR 5311, CNRS** — Unité de recherche Transitions, Organisations,
  Politiques, Inégalités. Toulouse, Maison de la Recherche, 5 allées Antonio Machado.
  Created 1 January 2026 from the merger of **CERTOP (UMR 5044)** and **LaSSP**.
- **Taylor's University**, Malaysia — chair in work, employment and organisation
  (exact title to confirm, see Q1).

This matters beyond a line of text: the merger is recent, so any page or profile still
reading "CERTOP / UMR 5044" is now out of date. Search engines and readers currently
have no path from the old unit name to the new one except through you.

---

## Blocker

The session that produced this document had no outbound network access — the
environment's policy denies everything except package registries, so the page itself
was never fetched (`WebFetch` → 403 for all hosts; proxy logged
`connect_rejected — policy denial` for `jens-thoemmes.com:443`).

Findings below therefore rest on **search-index data only**: the title tag, how the
homepage text is summarised, which URLs are indexed, and how the site ranks against
other profiles. Claims are marked accordingly:

- **[verified]** — directly observed in search results
- **[inferred]** — deduced from what did *not* appear; absence from an index is not
  proof of absence from the page

To lift the blocker, any one of these is enough:

- `curl -sL https://jens-thoemmes.com > page.html` locally, then commit it here
- point me at the repo holding the site source (`owner/repo`)
- allow the domain in the environment's network policy

---

## Findings

### F1 — The site is outranked on its own name  [verified]

A search for the exact string `jens-thoemmes.com` returns the site **7th**, behind
ResearchGate, SSRN, Semantic Scholar, HAL and Google Scholar. Losing a query that *is*
the domain name indicates very little inbound linking.

Consequence: the profiles you don't control are the canonical version of you. Your
publisher pages, journal masthead and lab profile all rank above the one page whose
content is entirely yours.

### F2 — Only the root URL is indexed, and only at title depth  [verified, strengthened]

Four searches restricted to `jens-thoemmes.com` — probing navigation, contact, CV,
publications, books, projects and teaching — returned exactly one URL every time:
`https://jens-thoemmes.com/`. No subpages. More telling, the index appears to hold
only the title and one or two sentences of description: queries asking directly about
the navigation menu, contact details and publication list came back with that content
not present in the indexed page.

So it is not merely that subpages are missing from the index — the homepage's own body
text is largely absent from it too. Candidate causes, in the order worth checking:

1. **content rendered client-side**, so crawlers store an empty shell — this is now the
   leading hypothesis and the first thing to test
2. body text lives inside images or a PDF rather than HTML
3. no `sitemap.xml`, plus weak or JS-only internal linking
4. the site genuinely is one short page

If (4), that's a legitimate choice — but then each section needs a stable `#anchor` so
sections can be linked and cited. If (1) or (2), every metadata improvement in P2 is
wasted effort until it is fixed.

### F3 — Generic title tag  [verified]

Current: `Jens Thoemmes - Sociologist & Research Professor`

No field, institution or location. It competes with every sociologist's homepage and
matches none of the queries people actually type.

Proposed: `Jens Thoemmes — Sociology of Work & Time | UTOPI, CNRS Toulouse`

### F4 — Homepage reads as topic prose, not as an entry point  [inferred]

What comes through the index is research-theme description. Absent from anything
indexed: a citable publication list with DOIs, a downloadable CV, an email address. If
these existed and were crawlable, at least one would likely have surfaced.

### F5 — No visible language strategy  [inferred]

Indexed content is English-only, while the work spans French, German and English — and
the two 2024 books split that way (Lexington in English, Octarès in French). The
audience is at least bilingual; the site presents as monolingual.

### F6 — The 2024 books are not the site's centre of gravity  [inferred]

*Time autonomy and work in France, Germany, and China* (Lexington, 2024) and
*La seconde autonomie* (Octarès, 2024) are the most concrete, most linkable things you
published recently, and nothing in the indexed homepage foregrounds them.

---

## Plan

Ordered by payoff per hour, not by dependency. P1 items are worth doing before the
technical audit; nothing in P1 needs the blocker lifted.

### P1 — Authority and accuracy

- [ ] **A1.** Add `jens-thoemmes.com` to the profiles that already outrank it:
      UTOPI/CNRS staff page, HAL CV, ResearchGate, Temporalités masthead, Cairn,
      Google Scholar, ORCID. One line each; effect compounds. *(~30 min, largest
      single payoff — addresses F1)*
- [ ] **A2.** Update the affiliation to **UTOPI — UMR 5311** on the site and on every
      profile in A1, keeping one sentence of continuity: "UTOPI (UMR 5311), formed in
      2026 from CERTOP and LaSSP". Readers arriving with the old name need the bridge.
      *(~30 min)*
- [ ] **A3.** Add the Taylor's University chair alongside the CNRS line — the dual
      Europe/Asia position is genuinely distinctive and currently invisible.

### P2 — Findability

- [ ] **B1.** Rewrite `<title>` and `<meta name="description">` per F3; description
      should name working time, temporalities, social regulation, and the institutions.
      *(10 min)*
- [ ] **B2.** Add `sitemap.xml`, `robots.txt`, a canonical URL, and JSON-LD `Person`
      schema with `affiliation` (UTOPI, Taylor's) and `sameAs` pointing at Scholar,
      HAL, ORCID, ResearchGate. Ties the identities together for search engines.
      *(~1 hr)*
- [ ] **B3. Do this first.** Confirm the homepage renders server-side. Evidence in F2
      now points the other way: the index holds the page at title depth only, which is
      what a client-side-rendered page looks like from outside. Nothing else in P2 has
      any effect if crawlers see an empty shell. Quick local test — `curl -sL
      https://jens-thoemmes.com | wc -c` and check whether your own prose is in the
      output, or view source in the browser and search for a sentence from the page.
      *(15 min to diagnose; scope follows from the answer)*

### P3 — Content

- [ ] **C1.** Books above the fold: cover, publisher, year, ISBN, purchase and DOI
      links. *(~1 hr — F6)*
- [ ] **C2.** Publication list as real HTML text — not a PDF, not JS-injected —
      reverse-chronological, DOI-linked, with open-access copies on HAL linked where
      they exist. *(~2 hrs — F4)*
- [ ] **C3.** Visible contact route and a dated CV download. *(~30 min)*
- [ ] **C4.** A short Temporalités section: editor-in-chief is a standing editorial
      role and a reason for people to return to the site rather than pass through.
- [ ] **C5.** Decide the language question (Q3) and implement — at minimum correct
      `lang` attributes; ideally FR/EN, with `hreflang` if both ship.

### P4 — Technical audit (needs blocker lifted)

- [ ] **D1.** Semantic structure: one `h1`, ordered heading hierarchy, landmarks.
- [ ] **D2.** Accessibility: colour contrast, alt text, keyboard navigation, focus
      states, `prefers-reduced-motion`.
- [ ] **D3.** Responsive behaviour at 360 / 768 / 1280 px; no horizontal scroll.
- [ ] **D4.** Core Web Vitals: image formats and dimensions, font loading, LCP.
- [ ] **D5.** Hygiene: HTTPS redirect, `www` canonicalisation, 404 handling, and
      whether any analytics in use is GDPR-appropriate for an EU-based academic site.

---

## Open questions

1. **Q1.** Exact title and department of the Taylor's University position?
2. **Q2.** Is the site one page or several? If several, which URLs exist?
3. **Q3.** How was it built — hand-written HTML, a static generator, WordPress, a
   site builder? This determines whether P2/P4 items are edits or a rebuild.
4. **Q4.** Languages: English only, or FR/EN — or FR/EN/DE?
5. **Q5.** Who is the page for, in priority order? Academic peers, students,
   journalists, or Malaysian/Asian institutional contacts? The ordering changes what
   belongs above the fold, and I'd rather optimise for your answer than for a guess.
6. **Q6.** Is there an ORCID to include in the `sameAs` list?

---

## Notes

- No code changes accompany this document — it is analysis only.
- Effort estimates assume you can edit the site directly; if it's a hosted builder with
  limited template access, B2 and D1 may not be reachable and the plan should shift
  weight toward P1 and P3, which don't depend on markup control.
