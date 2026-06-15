# CTI Mastermind Portal — Build Report

**Built:** 2026-06-15
**Output:** `index.html` (single self-contained file, replaces previous dashboard)
**Deployed target:** https://robert-upchurch.github.io/cti-mastermind-portal/ (parent agent handles deploy)

## File stats
- **File size:** 2.16 MB (well under the 10 MB cap)
- **CDN dependencies:** 0 — fully self-contained single-file HTML/CSS/JS
- **Embedded agents:** 59 (verified: 59 `"name":` and 59 `"tier_num":` occurrences; JSON block parses to 59 records)
- **HTML validity:** valid `<!DOCTYPE html>`, exactly one `<script>`/`</script>` pair, ends with `</html>`

## Agent counts
| Tier | Count |
|------|-------|
| Operational (tier 1) | 34 |
| Inner Mastermind (tier 2) | 9 |
| Extended Mastermind (tier 3) | 16 |
| **Total** | **59** |

## Department / council counts
| Department | Count |
|------------|-------|
| IT & Poseidon | 8 |
| Marketing & Media | 7 |
| Compliance | 5 |
| Finance | 3 |
| Recruiting | 4 |
| Operations | 4 |
| Leadership | 3 |
| Classic Wisdom Council | 10 |
| Future & AI Council | 15 |

(9 departments total. Operational tier spans the first 7; the two councils hold the Inner + Extended mastermind tiers.)

## Design
- Matches existing Poseidon aesthetic: dark navy default (`--bg-primary:#0a0e1a`), gold accent (`#d4a03c`), same fonts, radii, shadows, card style, trident logo.
- **Tier accent colors:** Operational = emerald green (`#34d399`), Inner Mastermind = gold, Extended Mastermind = blue (`#60a5fa`). Stat values and card top-stripes are color-coded per tier.
- Light/dark toggle persists in `localStorage` (`cti-portal-theme`), default dark.
- Mobile responsive (3 → 2 → 1 column grid). Print-optimized CSS for PDF.

## Features implemented
- **Tier tabs:** All 59 / Inner Mastermind 9 / Extended 16 / Operational 34.
- **Contextual subfilter:** Operational tab shows the 7 departments; mastermind tabs show All / Classic Wisdom / Future & AI. "All" tab hides the subfilter.
- **Cards:** tier color stripe, avatar with initials, name, function/council-role subtitle, `Archetypes: X + Y` line (op agents), essence quote, department badge, click-to-open.
- **Modal:** name + tier badge + word count, `Download PDF` / `Download .md` toolbar, full rendered markdown body, sticky footer with Previous/Next pager and position counter. Keyboard: Esc closes, ←/→ navigates.
- **Markdown renderer:** hand-rolled (~120 lines), handles H1–H4, bold, italic, code spans, links, bulleted + numbered lists, tables, blockquotes, horizontal rules. YAML frontmatter is stripped/hidden. No external library.
- **Wiki-links:** `[[Name]]` and `[[Name|alias]]` render as clickable pills that switch the modal to that agent in-place; links to unknown agents render greyed-out (dead).
- **Search:** full-text across name, function, council_role, essence, department, voice_anchor, archetypes, and tags; live-updates the grid; multi-token AND matching.
- **PDF export:** `window.print()` with a print stylesheet that hides chrome, prints only the modal body, uses serif body / sans-serif headings, page-breaks before each H2, running header `[Agent Name] · CTI Mastermind Vault`, footer with page number and date. Verified via Playwright PDF render (16 pages for Marshall, headers/footers/page-breaks correct).
- **Markdown export:** Blob + `URL.createObjectURL` downloads raw `md_content` as `{Name}.md` (Obsidian-ready, frontmatter intact). Verified `Marshall.md` downloads with full content.

## QA performed (Playwright, headless Chromium)
- All 59 cards render on load, zero JS/console errors.
- Tier filtering: Operational→34, Inner→9; subfilters correct (Finance→3).
- Search "munger" → 1 result (Charlie Munger); cleared → 59.
- Modal opens with correct title, tier badge, word count, toolbar, rendered headings/blockquote/lists; frontmatter not leaked.
- Wiki-link click navigates (David → Marshall) without closing modal.
- `.md` download produces `Marshall.md` (28.5 KB, starts with frontmatter).
- Light mode toggle verified visually.
- Print PDF render verified: running header with agent name, page numbers, H2 page breaks, serif body.

## Tradeoffs / notes
- Dropped unused fields (`codename`, `reports_to`, `path`) from the embedded JSON to trim size; `md_content` (the bulk) is fully retained for every agent.
- No agent dossier actually uses Markdown tables, but the renderer supports them for completeness.
- Single compact JSON line means `grep -c '"name":'` returns 1 line; the per-occurrence count (`grep -o`) is 59.
- Helper files `_template.html` and `_build.py` are left in the folder for reproducibility; only `index.html` is the deliverable. They can be removed before deploy if desired.
