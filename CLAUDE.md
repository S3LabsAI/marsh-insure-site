# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The demo build of **Marsh.insure**, the local agency site for Greg Marsh's Allstate agency in Westminster, MD. It is the sister site of IHateMyRates.com (repo `S3LabsAI/Ihate-my-rates`). Both sites feed one agency pipeline. This one is the agency's trusted local home and carries the local SEO work. IHateMyRates.com is the campaign site and links back here.

Right now the whole site is one file, `index.html` (~5 MB). It is a self-unpacking bundle exported from a design tool, not hand-written HTML. There is no package.json, build step, linter or test suite. It deploys as a static site on Vercel (one Pro account shared with IHateMyRates.com) from the GitHub repo `S3LabsAI/marsh-insure-site`; `vercel.json` holds clean URLs and security headers, and `README.md` lists the Vercel project settings.

The governing document is the Notion page "Greg Marsh Insurance: Scope of Work" (https://app.notion.com/p/4201bd8759cc81c086aac4003c3eacd0). Where this file and the SOW disagree, the SOW wins. Tell Tom about the mismatch. Keep `PROGRESS.md` current (done, in progress, open placeholders, decisions) and read it at the start of each session. It does not exist yet; create it with the first real change.

**Preview:** open the HTML file in a current Chrome, Safari or Firefox. It needs JavaScript and `DecompressionStream`. Nothing renders with JS disabled.

## How the bundle works

Same exporter and format as the IHateMyRates.com repo. The outer document is only a loader. The real page lives in `<script>` data islands:

- `__bundler/template`: the page HTML as a **JSON-encoded string**. Asset references are bare UUIDs (`src="18a697f8-..."`).
- `__bundler/manifest`: JSON map `uuid -> {mime, data (base64), compressed (gzip bool)}`:
  - `18a697f8-…` PNG: header logo
  - `6d806275-…` WebP: Greg's hero photo
  - `c76a3421-…` PNG: office photo (~3.3 MB decoded; the main reason the file is 5 MB)
  - `1a6eb29e-…` JS (gzip): the x-dc support runtime, loaded by the template's first `<script src>`
  - `41c854f1-…` / `2e82744b-…` JS (gzip): React / ReactDOM 18.3.1 UMD
  - 11 woff2 files: Geist and Geist Mono subsets
- `__bundler/ext_resources`: maps the original unpkg React 18.3.1 URLs to manifest UUIDs.
- `__bundler/page_order`: nested iframe pages. Empty here.

At load the bootstrap decodes the manifest into blob/data URLs, swaps each UUID in the template, and replaces `document.documentElement` with the template. The runtime then renders the `<x-dc>` component with React.

## Editing

There is no unpack/pack tool in this repo yet. The IHateMyRates repo has `tools/bundle.py` (`unpack` writes `build/template.html`, `pack` writes it back); copy it here, point it at this file name, and git-ignore `build/`. Until then:

```bash
# unpack the template to edit it
python3 -c "import json,re;s=open('index.html').read();m=re.search(r'<script type=\"__bundler/template\">(.*?)</script>',s,re.S);open('template.html','w').write(json.loads(m.group(1)))"
```

Rules for writing it back:

- Never do blind text replacements on the outer file. Parse the `__bundler/template` island with `json.loads`, edit the decoded HTML, write it back with `json.dumps`.
- The result must contain no literal `</script>`. Escape it as `<\/script>`.
- Assets: base64-decode, gunzip if `compressed`, change, re-encode. Keep UUIDs stable.
- Geist is licensed under the SIL OFL. Unlike the IHateMyRates bundle, this file has no license comment yet; add the OFL notice at the top before the file is redistributed, and keep it there.

The SOW calls for several real pages (see below), which this single-page bundle can't easily become. Expect to move to a real stack at some point. When that happens, update this file and the project instructions' Stack line.

## Template language (`<x-dc>` component format)

- `<helmet>…</helmet>`: goes into `<head>` (the Geist `@font-face` rules plus a small global `<style>` for body, links, box-sizing).
- The body markup uses **inline styles everywhere**. No class-based CSS.
- `<script type="text/x-dc" data-dc-script data-props='…'>`: the component logic, a `class Component extends DCLogic` with React-style `state` / `setState`. `renderVals()` returns every value and handler the template uses.
- `{{ expr }}` interpolates. `<sc-if value="{{ bool }}">` and `<sc-for list="{{ arr }}" as="item">` handle conditionals and loops. Conditionals take booleans only, so `renderVals` precomputes flags (`sent` / `notSent`, `showFacts`).
- `sc-camel-on-click`, `sc-camel-on-submit` and similar map to React camelCase props.
- `style-hover` / `style-focus` set inline overrides for those states.
- `hint-placeholder-val` / `hint-placeholder-count` only affect the design editor preview. Keep them in sync when you add `sc-if` / `sc-for` blocks.
- `data-props` declares editor props. Currently only `showFacts` (boolean, default true), which toggles the four-fact strip under the hero.
- The runtime drops anything in `<body>` outside `<x-dc>` and drops unknown attributes such as `data-*`. Scripts that must survive go in `<head>` (see how IHateMyRates handles `src/shred.html`).

Data lives on the component class: `plans` (the coverage cards, also the source of the quote form's chips; an entry with `adds` such as Bundle selects several chips) and `steps` (the four claim steps).

## Page structure and behavior

One page, top to bottom:

1. Dark top bar: address, hours, email
2. Sticky header: logo, anchor nav (Coverage, Claims, Quote, Office), phone, "Get a quote"
3. Hero (`#top`): H1, two CTAs (quote + call), Greg's photo, optional facts strip
4. `#cover`: coverage cards from `plans`. Clicking a card pre-selects that line in the quote form
5. `#claims`: four claim steps from `steps`
6. `#quote`: quote form with coverage chips, name, phone, email, ZIP
7. `#office`: address, phone, email, hours, office photo, Google Maps link
8. Blue CTA band, then footer

**The quote form has no backend.** Only HTML `required` on name and phone. "Submitting" just flips `sent` and shows a thank-you with the picked lines. No source tag, no consent text. Footer Privacy / Terms / Accessibility links point to `#top`.

## Design tokens (inline, keep consistent)

- **Colors:** page `#eef2f7`, ink / dark sections `#0b1a2e`, accent blue `#1d5bc4`, light panel `#f4f7fb`, white cards `#fff`, rules `#d5dce6` / `#d6dde7` / `#b9c3d0`, body text `#46566c`, labels `#6b7a8f`, nav text `#2a3a50`, footer text `#c9d3e1` / `#8e9db2`, footer rule `#24364f`.
- **Type:** Geist (400–600) for everything; headings weight 600 with negative letter-spacing (-0.02 to -0.035em) and `clamp()` sizes. Geist Mono for small uppercase labels, section numbers (`01 / Coverage`), phone numbers and the top bar.
- **Layout:** max width 1240px, side padding 24px, sections 72px vertical padding. 4px border-radius on buttons and image frames. 1px rules, with a `#0b1a2e` top rule on grids and lists. Grids use `repeat(auto-fit,minmax(min(100%,Npx),1fr))` so they collapse on mobile without media queries.

## Project requirements this build doesn't meet yet

These come from the project instructions and the SOW. Treat them as the backlog, and don't stall on missing client inputs: use clearly marked placeholders and list each one in `PROGRESS.md`.

- **Pages.** SOW pages: Homepage, Coverage accordion (Auto, Home, Renters, Life, Umbrella; opening one closes the others), Get a Quote form, Meet Greg, Meet the Team (standard headshots, name, title, short bio), Reviews (Google reviews via API where allowed, otherwise the agency's existing reviews), Terms, Privacy. The demo has only the homepage sections and no reviews.
- **Coverage lines from one config.** The demo lists Auto, Home, Bundle, Renters, Life, Business in `plans`, the H2 says "Five lines", the hero copy and facts strip say "auto, home, life, business", and the footer lists four. The SOW list is Auto, Home, Renters, Life, Umbrella. Final lines get confirmed at kickoff, so drive every mention from a single list.
- **Lead routing.** Every submission carries source tag `Marsh.insure`. Destination is undecided, tried in this order: (1) Allstate Lead Manager, (2) agency CRM such as AgencyZoom, (3) our own secure store plus email forward with the tag. Put submission behind one adapter so the destination swaps without touching the form. Add real validation (ZIP, phone, email).
- **Compliance copy.** Terms, Privacy and form consent text come from the agency. Until they arrive use a visible `[COMPLIANCE COPY PENDING]` placeholder. Don't write legal or consent language. Don't use Allstate logos or brand assets unless Tom confirms approval (check what the header logo PNG contains before launch).
- **Calls.** The number `(410) 876-7660` is hard-coded in seven places, in three formats. The call button must dial the agency-supplied tracking number, read from one config value, and be labeled at the agency level ("Call Marsh Insurance"), not with a person's name.
- **Agency name.** Undecided: "Greg Marsh Insurance" (SOW) or "Marsh & Associates Insurance" (client portal). The demo uses "Greg Marsh Agency". Keep it in one place. The demo also shows `gregorymarsh@allstate.com` in six places; confirm which address the agency wants public.
- **Copy.** Natural, warm, professional, built around Greg and his team. No invented bio facts, reviews or credentials. InsurePages writes bios from the agency's notes.
- **Local SEO (this site only).** Page `<title>` is currently "Bundled Page" and there's no meta description, sitemap, robots.txt or schema. Needed: local-search titles, descriptions, headings and internal links, plus LocalBusiness / InsuranceAgency JSON-LD matching the Google Business Profile: 1812 Baltimore Blvd, Suite B, Westminster, MD 21157; licensed in MD, PA, DE, VA, WV. Search Console ready.
- **Analytics.** Basic traffic plus form-submission tracking, reported separately from IHateMyRates.com.
- **Responsive.** Check desktop, tablet and mobile. The header nav and the 120px `dl` label column are the likeliest to crowd on small screens.
- **Performance.** Recompress the office photo (WebP/AVIF, sensible width) before launch.

## Out of scope

Don't build: email hosting, ongoing SEO or blogging, paid ads, CRM selection or migration, live rate quoting, customer portal or payments, interactive calculators, new photography. Log any that come up in `PHASE-2.md` with one line each. New pages, features or integrations need a written change request first.

## How to work

- Make changes directly. Local, reversible work (editing, previews, builds, Vercel preview deploys) is fine.
- Ask Tom first before anything public or hard to undo: production deploys, domains or DNS, test leads into Lead Manager or a live CRM, deleting files or branches.
- Before calling a page done: build (once there is one), check mobile, tablet and desktop widths, and submit the form locally to confirm the payload carries `source: "Marsh.insure"`.
- Everything is handed to the agency at the end. Keep accounts, env vars and config documented and free of anything tied to Tom's personal setup.
