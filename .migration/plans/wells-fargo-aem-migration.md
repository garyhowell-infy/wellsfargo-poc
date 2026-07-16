# Wells Fargo → AEM Edge Delivery Migration Plan

## Goal
Migrate three pages from `www.wellsfargo.com` into this AEM Edge Delivery Services project — scraping each source page, analyzing its structure, building content-driven import infrastructure (parsers + transformers + import script), importing the content, and verifying rendering against the originals.

## Target URLs
1. `https://www.wellsfargo.com/` (homepage / marketing)
2. `https://www.wellsfargo.com/about/` (about / corporate)
3. `https://www.wellsfargo.com/biz/` (small business hub)

## Context & Important Notes
- The existing `tools/importer/` infrastructure is purpose-built for **WKND Adventures** — its blocks (`hero`, `cards`, `featured-article`, etc.) and `BLOCK_REGISTRY` are keyed to WKND's DOM and will **not** correctly parse Wells Fargo pages. This migration needs its **own** analysis, block variants, and import scripts; we will not overwrite WKND infrastructure blindly.
- These three pages are likely **three distinct templates** (marketing homepage, corporate/about, business hub) with little block overlap — expect several new block variants.
- Wells Fargo is a large, script-heavy site with personalization, cookie/consent banners, and possibly login widgets. Scraping captures the public marketing DOM; interactive/auth widgets will be dropped or need placeholder treatment.
- Detection must stay **purely DOM-driven** (no URL matching or positional assumptions), per the project's content-driven import rule.
- **No git operations** — the user manages those via the Console UI.

## Approach
1. **Scrape & scope** — scrape all three URLs (content, metadata, images, cleaned HTML). Group into templates and produce a short scope report (templates, blocks needed, effort, risks).
2. **Per-page analysis** — for each page, identify sections, section styles, and content sequences; decide default-content vs blocks; map to existing blocks where they genuinely fit, or define new block variants.
3. **Build new block variants** (JS + CSS + content model) only where existing blocks don't fit.
4. **Build import infrastructure** — parsers (one per block variant), cleanup + section transformers, and a Wells-Fargo `BLOCK_REGISTRY` in the import script.
5. **Import content** — bundle (`--format=iife --global-name=CustomImportScript`) and run the bulk importer against the three URLs.
6. **Verify** — preview each imported page locally and compare visually to the original; iterate on parsers/CSS until it matches.
7. **Lint & finalize** — run `npm run lint`, fix issues, and summarize (flag anything that couldn't be migrated).

## Checklist
- [ ] Scrape the 3 URLs (homepage, /about/, /biz/) — capture DOM, metadata, and images
- [ ] Group pages into templates and produce a migration scope report
- [ ] Analyze the homepage: sections, section styles, content sequences, block mapping
- [ ] Analyze /about/: sections, section styles, content sequences, block mapping
- [ ] Analyze /biz/: sections, section styles, content sequences, block mapping
- [ ] Decide block reuse vs new variants across the three templates
- [ ] Build/verify any new block variants (JS + CSS + content model)
- [ ] Write parsers for each block variant (`tools/importer/parsers/`)
- [ ] Write cleanup + section-boundary transformers for the Wells Fargo DOM
- [ ] Assemble import script with a Wells-Fargo `BLOCK_REGISTRY` (DOM-detection only)
- [ ] Bundle the import script with esbuild (iife + `CustomImportScript` global)
- [ ] Run the bulk importer to generate content HTML for all 3 pages
- [ ] Preview each page and visually compare to the original; iterate to match
- [ ] Run `npm run lint` and fix any issues
- [ ] Summarize results and flag any content that couldn't be migrated (auth widgets, personalization, etc.)

---
*Ready to start on the three confirmed URLs. Working assumptions: (1) this is a **full migration** of all three pages (not just a scope report), and (2) imported content lands in this project's content directory. Execution requires Execute mode — approve and I'll begin with the scrape + scope step.*
