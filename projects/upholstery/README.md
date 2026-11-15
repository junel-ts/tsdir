# Upholstery Supplier Directory — Master README and Agent Handoff

> **What this is:** The handoff for a coding agent (Claude Code, Codex, Cursor or similar) that turns the owner's upholstery supplier list into a verified, crawlable, static supplier directory website.
> **Stack:** Astro + TypeScript (strict) + Tailwind CSS, deployed as a static site on Cloudflare Pages, from a private GitHub repo.
> **Backend:** None for launch. Search runs on a static JSON index in the browser.
> **How to use it:** Fill in the `[TBD]` values in Section 1 if you have them, save this file as `README.md` in the project root, put the workbook in `data/raw/`, then give the agent the prompt in Section 17.

> **Scope for this repo: BUILD ONLY.** The agent completes Phases 0-5 and delivers a verified local static build of the directory. It does **not** deploy, connect Cloudflare, buy domains or create repos. Phase 6 is done later by the owner.

---

## 0. Read this before taking any action

This project is a directory of **upholstery materials suppliers**. Everything project-specific lives in **Section 1 (Project Config)** and in the taxonomy files the agent builds from the real data. Do not assume any category list or geography that Section 1 does not state.

The owner **already has supplier lists**. Do not restart supplier discovery, replace the lists with examples, or imply that a dataset has been verified when it has not. Finding the official website of a company that is already on the list is allowed only when Section 1 approves `website_lookup`.

### Non-negotiable principles

1. **Use the owner's real data only.** Never fabricate suppliers, products, prices, stock, hours, addresses, certifications, brands, reviews or service areas.
2. **Preserve originals.** Source files are read-only. Write normalized outputs separately.
3. **Provenance over polish.** Every published fact traces to a source row and, where verified, to evidence with a real observation date.
4. **Fail closed.** Unknown stays blank. Unreviewed rows never silently become published facts.
5. **A search tag or a memory guess is not a fact.** Columns that record *which search found a company*, or values an AI model recalled without checking (country, category, description, even the URL), are provenance. They are not the company's location, capability or confirmed website.
6. **Semantic HTML first.** Core directory content must be in the built HTML, readable with JavaScript off.
7. **No empty or thin pages.** Never generate a country, region or category page with zero eligible suppliers.
8. **No live-data fiction.** A verified website is not a verified inventory, price list or delivery promise.
9. **Safe by default.** No secrets, personal contact data or private notes in the repo's public output or the built site.
10. **Ship in gates, report with evidence.** Finish each phase's exit gate before the next. Report the commands you ran, the real counts and the real URL. Never claim "shipped" without a live check.

---

## 1. Project Config

The agent must read this section first and treat every value here as authoritative. Anything left as `[TBD]` is unknown: the agent asks about it at the point it matters, or works without it, and never guesses.

```yaml
project_name:        "Topstitch Digital Upholstery Supplier Directory"
industry:            "upholstery materials and supplies"
one_line_purpose:    "Help upholstery shops and trimmers find suppliers of leather, vinyl, fabric, notions and adhesives"
target_users:        "upholstery shops, auto / marine / furniture trimmers, furniture makers, trade buyers"

launch_scope:        # countries published in this release, in order
  - "CA"
  - "US"
  - "GB"
later_scope:         # kept in the data, not published yet
  - "every other country in the Supplier Master tab"

source_files:        # every file the agent may import suppliers from
  - path:  "data/raw/Upholstery_Suppliers_Canada_USA_UK_Directories.xlsx"
    tab:   "Supplier Master"
    header_row: 4
    notes: "Rows 1-2 are a title banner and row 3 is blank. 609 names, one row per supplier or brand."
ignore_tabs_note:    "The other 18 tabs are copies, indexes, queues or summaries of Supplier Master: provenance only. The Canada, USA and UK Directory tabs hold the same 400 names with empty checklist columns."

provenance_only_columns:   # columns that describe how a row was found or guessed, NOT facts
  - "Region"
  - "Country / Base"                  # an AI model's memory estimate; use ONLY to order verification work
  - "Working Categories"              # preliminary
  - "Original Categories"
  - "Product Focus / Applicability"   # model-written text; never publish
  - "Entity Type"                     # preliminary
  - "Location Basis"
  - "List Origin"
  - "Verification Notes"

candidate_service_types:   # starting list; the agent reconciles with real data
  - "manufacturer"
  - "distributor"
  - "retailer"
  - "tannery"
  - "mill"
  - "equipment_maker"
  - "wholesale_trade_accounts"        # only if the site says it sells wholesale or to the trade
candidate_categories:      # starting list; only used when evidence supports it
  - "leather"
  - "vinyl"                           # includes faux leather and coated fabrics
  - "outdoor_fabric"
  - "indoor_residential_fabric"
  - "notions"                         # broad: foam, thread, zippers, tools, machinery
  - "adhesives"
# The agent may PROPOSE sub-categories of notions from evidence, and asks the owner before adding any.

verification_pass:   "approved"       # may the agent fetch each supplier's own site?
website_lookup:      "approved"       # may the agent search for the official site of rows with no URL?
verification_scope:  "Rows where Country / Base is Canada (27), USA (303), United Kingdom (70) or Not determined (18). All other rows are imported as unreviewed and left alone this release."
verification_tools:  "Firecrawl scrape, Firecrawl search, Alexandria (second check only, see 7.3)"
credit_cap:          "[TBD: max Firecrawl credits for this run]"
production_domain:   "https://www.topstitchdigital.com"
base_path:           "/directory"     # the directory lives at https://www.topstitchdigital.com/directory/
github_repo:         "[TBD until Phase 6; must be PRIVATE]"
reuse_rights_status: "unchecked for all rows"
```

### 1.1 Part of topstitchdigital.com

This directory is a section of the existing Topstitch Digital website, not a standalone site.

- Set Astro `site: "https://www.topstitchdigital.com"` and `base: "/directory"`. Every page, asset, canonical URL, sitemap entry and internal link must sit under `/directory/` (for example `/directory/suppliers/ca/british-columbia/`). Use Astro's base-aware URL helpers; never hard-code root-relative links.
- The directory's own home page is `/directory/`. Do not create pages, a `robots.txt` or a sitemap at the site root, because they would clash with the main site. Write the directory sitemap to `/directory/sitemap-index.xml` and list in the Phase 5 handoff the one line the owner should add to the main site's `robots.txt`.
- Keep styling self-contained so it can't break the main site, and link the header logo back to `https://www.topstitchdigital.com/`.
- How `/directory/` gets routed on the live domain (Cloudflare route, reverse proxy or the main site's host) is decided by the owner at deployment and is out of scope here.

### 1.2 What the agent should know about this list

These figures were measured from the workbook on 2026-11-15. Re-measure them in Phase 0 and report any difference.

- **609 names** in `Supplier Master`, with no duplicate names.
- **Only 67 came from the owner's original research** (`List Origin` = "Original 67"). The other 542 were recalled by an AI model and never checked. Per rule 14 in Section 6.3, they get no extra trust: some may not exist, may have closed, or may not sell upholstery materials.
- **314 rows have a URL and 295 have none.** Among the 400 launch-scope names, 220 have a URL and 180 need a website lookup. The URLs on model-recalled rows were recalled too, so each one must be confirmed to belong to the named company.
- **Every country value is a memory estimate** (591 rows) or "Not determined" (18 rows). None is a verified location.
- **The 18 "Not determined" rows all have URLs, and 13 are from the original research.** Verify them: any that prove to be in Canada, the USA or the UK are published.
- **8 cells hold two URLs** separated by a line break, and **63 URLs point to product pages** rather than a homepage (rules 11 and 12).
- **Many entries are brands, not companies** (for example a fabric brand owned by a larger group). Set `entity_type` and `parent_company` from the supplier's own site only.
- The directory tabs' "Market Presence", "Website Verification", "Wholesale / Trade" and "Contact / Email" columns are empty or "Not checked" for every row. They contain nothing to import.

---

## 2. Product definition

The product is a structured, searchable **supplier and service directory** for the industry in Section 1. It answers questions like:

- "Who supplies X near me / in my country or region?"
- "Which companies repair, sell, rent or manufacture X?"
- "Can I get an auditable record of this company: its website, what it does, where it operates and how we know?"

**Included at launch:** master directory, supplier profiles, country/region/category hubs where the data supports them, static search, honest verification labels, metadata, sitemap, robots and a Cloudflare deployment.

**Not included unless separately built and verified:** live inventory, pricing, compatibility claims, delivery promises, lead forwarding or RFQ forms, accounts, paid ranking, reviews, AI-written product claims, public write APIs.

---

## 3. Architecture decision (ADR-001)

**Astro** for static HTML by default, data-driven routes and minimal client JavaScript. **TypeScript strict** for the data pipeline. **Tailwind** for styling. **Cloudflare Pages** for static hosting with preview deploys from Git.

Do not add SSR, a server adapter, a database or a separate backend for launch. Those need a documented requirement and a new decision record.

```text
OWNER'S SOURCE FILES (read-only)
      |
      v
IMPORT + NORMALIZE + DEDUPE (build-time, TypeScript)
      |-- needs-review / rejected reports (never public)
      v
VERIFICATION PASS (optional, approved per Section 1)
      |-- evidence rows with real observation dates
      v
VALIDATION + PUBLICATION GATE
      |
      +--> Astro static pages, JSON-LD, sitemap --> Cloudflare Pages
      +--> public search-index.json (safe fields only)
```

Raw files are the input. Canonical records are the source of truth. Generated JSON and pages are disposable build artifacts.

---

## 4. Repository layout

Work inside the owner's **existing project folder**. Put Astro in `apps/web`, never over the root. If something already exists, inspect and reuse it.

```text
<project root>/
├── README.md                  # this file
├── .gitignore
├── .env.example               # templates only
├── apps/web/                  # Astro app
│   ├── astro.config.mjs
│   ├── package.json
│   ├── public/robots.txt
│   └── src/
│       ├── components/  (SupplierTable, SupplierCard, Breadcrumbs, Search)
│       ├── layouts/BaseLayout.astro
│       ├── lib/        (suppliers.ts, routes.ts, taxonomy.ts, seo.ts)
│       ├── generated/  # JSON from the data pipeline
│       └── pages/
│           ├── index.astro
│           ├── search.astro
│           ├── suppliers/index.astro
│           ├── suppliers/[country]/index.astro
│           ├── suppliers/[country]/[region]/index.astro
│           ├── suppliers/[country]/[region]/[category]/index.astro
│           └── vendors/[id].astro
├── data/
│   ├── raw/                   # owner's originals; never modified
│   ├── canonical/             # suppliers, locations, capabilities, evidence
│   ├── schemas/               # JSON Schema (draft 2020-12)
│   ├── taxonomy/              # countries, regions, service types, categories
│   ├── mappings/              # one explicit column mapping per source file
│   └── reports/               # import summary, needs-review, validation errors
├── scripts/
│   ├── import-suppliers.mjs
│   ├── verify-suppliers.mjs   # only if verification is approved
│   ├── validate-data.mjs
│   ├── build-public-data.mjs
│   └── audit-site.mjs
├── docs/
│   ├── source-inventory.md
│   ├── data-dictionary.md
│   ├── decisions.md
│   └── handoffs/phase-N.md
└── tests/
    └── fixtures/              # synthetic, clearly marked TEST ONLY
```

These are target deliverables, not claims that they exist.

---

## 5. Local setup

1. Confirm Node.js LTS and npm are installed (`node --version`, `npm --version`) at a version the current Astro release supports. Check the official guide: https://docs.astro.build/en/install-and-setup/
2. Check Git: `git status`. If the folder is not a repo yet, the owner should run `git init` and make a first commit before the agent changes anything.
3. Create Astro only inside `apps/web`, with a minimal starter and strict TypeScript. Stop if `apps/web` already holds a project.
4. Add Tailwind using the current official Astro integration for the installed version.
5. Record the exact Node, Astro and Tailwind versions in `docs/decisions.md`.

---

## 6. Canonical data model — first engineering priority

Schema, import and validation come **before** any frontend work.

### 6.1 Entities

```text
Supplier 1---N Location
Supplier 1---N Capability
Supplier 1---N Evidence
Country  1---N Region
Capability references the service-type and category taxonomies
```

One company is one supplier record. Never create a new supplier per region or category.

**`suppliers.csv`**

| Field | Rule |
|---|---|
| `supplier_id` | Stable opaque ID, not derived from a mutable name |
| `company_name` | As presented by the source |
| `website_url` | Only if in the source; never guessed |
| `company_domain` | Derived from the URL; not proof of identity |
| `entity_type` | `company`, `brand` or `unknown`; set from evidence, not from the source's guess |
| `parent_company` | Only if the supplier's own site states it; brands of one parent are never merged automatically |
| `verification_status` | `unreviewed`, `website_confirmed`, `services_confirmed`, `rejected` |
| `rejection_reason` | Required when rejected (e.g. "site shows no [industry] business") |
| `last_verified_at` | Real check date only; never backfilled |
| `source_record_ids` | All source rows this supplier came from |
| `active` | Controls publication candidacy |
| `notes_internal` | Never published |

**`locations.csv`**: `location_id`, `supplier_id`, `location_type` (`physical`, `headquarters`, `service_area`, `unknown`), `country_code` (ISO 3166-1 alpha-2), `region_code` (ISO 3166-2 where one exists), `city`, `evidence_id`.
A physical location is not a service area. A service area needs its own evidence.

**`capabilities.csv`**: `capability_id`, `supplier_id`, `service_type`, `category`, `brand_name` (only if stated by the source), `evidence_id`, `confidence` (`confirmed` or `unconfirmed`, never a made-up number).
Being one service type never implies another.

**`evidence.csv`**: `evidence_id`, `supplier_id`, `source_record_id`, `source_url`, `source_type` (`official_site`, `registry`, `directory`, `supplier_file`, `other`), `claim_type` (`identity`, `location`, `service`, `category`, `brand`, `other`), `claim_summary` (our own words, narrowly worded), `observed_at`, `review_status` (`unreviewed`, `confirmed`, `contradicted`, `expired`).
A supplier file alone supports only an `unreviewed` candidate.

### 6.2 Taxonomy

- Stable IDs, display labels, URL slugs and search aliases are separate fields.
- Start from `candidate_service_types` and `candidate_categories` in Section 1, then reconcile against the real data and evidence. Document every addition or removal.
- An unknown category stays unknown. Never default it to a catch-all such as "general".
- Use ISO country and subdivision codes. Handle countries without useful subdivisions without inventing regions.

### 6.3 Source-to-canonical mapping rules

1. Inventory every source file: tabs, header rows, row counts, encoding, overlaps between files.
2. Import suppliers **only** from the files and tabs listed in Section 1. Other tabs are provenance references, never extra supplier rows.
3. Write an explicit mapping config per source file in `data/mappings/`. No fuzzy AI column guessing.
4. Columns listed in `provenance_only_columns`, and any column whose header or contents say it records a search, query or "not verified" value, are stored as provenance only. **Never create location or capability rows from them.**
5. Preliminary category or type columns are stored raw and never published as capabilities without evidence.
6. Record `source_file`, `source_tab`, `source_row` and a raw row hash in an import ledger.
7. Normalize URLs and whitespace cautiously and keep raw values for audit. Never infer country from a domain ending (`.ca`, `.co.uk`).
8. Deduplicate on canonical domain plus name. **Do not auto-merge** when one domain covers several names, branches or subsidiaries (large companies often have many pages on one domain). Send those to needs-review.
9. Generate stable IDs with a crosswalk so re-imports don't renumber suppliers.
10. Output is deterministic: same input, same records. Report rows processed, unique suppliers, merged, rejected, pending and publication-eligible.
11. A cell holding several URLs is split. All are kept as source URLs; none counts as the confirmed website until verification.
12. A URL pointing to a product or category page is kept as the source URL. The supplier's website is recorded as the site root only after verification confirms the site.
13. Rows with no URL are imported as `unreviewed` with a blank website. Never construct a URL from a company name.
14. Rows that the source marks as recalled by an AI model, rather than found by research, get no extra trust. Their URL, country, category and description are unverified until the supplier's own site confirms them. Model-written descriptions are never published.

### 6.4 Validation

JSON Schemas (draft 2020-12) in `data/schemas/`, validated with `ajv` + `ajv-formats`, **plus** business-rule checks:

- Unique IDs; all foreign keys resolve.
- Taxonomy codes are valid; URLs are well-formed.
- No contradictions (a rejected supplier published, a confirmed capability with no evidence).
- No slug collisions; no private fields in public JSON.
- Text is escaped; no control characters; no unsupported numeric claims (stock, lead times, prices).

**Hard errors** (bad schema, broken FKs, duplicate keys, invalid taxonomy on an approved record) exit nonzero and block the build. **Warnings** (missing website, unknown location, possible duplicate, stale evidence) go to reports and keep the claim unpublished. Every message names the file, row, field and a suggested fix.

---

## 7. Verification pass (only where Section 1 approves it)

A list of names and URLs on its own gives almost nothing publishable. Two permissions in Section 1 control this step.

### 7.1 Website check (`verification_pass: approved`)

- For each supplier with a URL, fetch **only that supplier's own website**: a few pages at most (home, about, contact, products or services). Respect `robots.txt` and rate-limit politely.
- First confirm the site belongs to the named company and shows a business in the industry from Section 1. If the URL is dead, parked or belongs to a different company, record that as `contradicted` evidence and treat the supplier as having no URL.
- Then record evidence for what the site itself states: physical location(s), service types, categories, whether it is a company or a brand, its parent company if stated, and brands only if the site lists them.
- Write `claim_summary` in our own words. Copy no descriptions, images or logos.
- Use the real observation timestamp.
- If the site shows no business in the industry, mark the supplier `rejected` with the reason.
- If the site is unreachable or unclear, leave the supplier `unreviewed` and list it in `needs-review.csv`.
- If the verified location is outside `launch_scope`, keep the supplier in the data but do not publish it this release.
- Never guess from the domain or add suppliers that are not in the source files. The only third-party source allowed is the second check in 7.3.

### 7.2 Website lookup (`website_lookup: approved`)

For suppliers with no usable URL:

- Search for the company's official website using its name plus the industry and the country the source suggests.
- Accept a site only if it clearly shows the same company name and the industry. Record the search query, the chosen result and the date as evidence, then run the 7.1 check on it.
- If there is no clear match, leave the website blank, keep the supplier `unreviewed` and list it in `needs-review.csv` as "no official site found". Never pick the closest-looking result.
- Search results are used only to find the official site. Published facts come from the official site, plus the limited second check in 7.3.
- This is not supplier discovery. Never add a company that is not in the source files.

If lookup is **not** approved, rows without a URL stay `unreviewed` and unpublished.

### 7.3 Tools for this step

Use only the tools named in `verification_tools` in Section 1. Do not use any other search engine, scraper or data source.

- **Firecrawl scrape** reads the supplier's own website (7.1).
- **Firecrawl search** finds the official website of a row with no URL (7.2).
- **Alexandria** (Firecrawl's library of structured data providers) is a **second check, used only when needed**: when the supplier's own site is unreachable, unclear, or does not state the company's identity or location.

Rules for Alexandria:

- It may confirm only that the company exists, its official website and its location. Service types, categories and brands still come from the supplier's own site alone.
- Record each result as its own evidence row: `source_type` `registry` or `directory`, the provider's name, what it stated and the real date.
- If Alexandria and the supplier's site disagree, publish neither claim and list the supplier in `needs-review.csv`.
- A supplier confirmed only through Alexandria is labelled that way on its profile (for example "Listed in [provider], checked 2026-11-15"), never "Website confirmed".
- Alexandria and Firecrawl calls cost credits. Respect `credit_cap` in Section 1: stop and report when it is reached, and never retry a failed call in a loop.

### 7.4 Order of work

1. Only the rows named in `verification_scope`.
2. Countries in `launch_scope` order.
3. Within a country, suppliers that already have a URL before suppliers that need a lookup.

A partial pass must still leave a launchable site. If verification is **not** approved at all, the agent builds everything else and reports that hubs will be empty or minimal until evidence exists.

---

## 8. Publication rules

- **Master directory and profile:** identity is supported (at minimum `website_confirmed`), not rejected or inactive, and the location is in `launch_scope`.
- **Country / region hub:** the supplier has a verified physical location or evidenced service area there.
- **Category hub:** the capability is supported by evidence.
- **Profiles show only established facts.** Omit unknowns rather than showing "N/A" or "0".
- **Labels say exactly what was checked**, e.g. "Website confirmed 2026-11-15". Never "Verified Seller", "Trusted" or "Top rated" unless that was actually assessed.
- **Reuse rights:** while `reuse_rights_status` is unchecked, publish only the company name, website link and facts we wrote ourselves from evidence.

---

## 9. URL architecture and pages

| Page | URL | Content |
|---|---|---|
| Global directory | `/suppliers/` | Countries with eligible suppliers, counts, search |
| Country hub | `/suppliers/{country}/` | Supplier count, links to regions and categories |
| Region hub | `/suppliers/{country}/{region}/` | Suppliers located or serving there |
| Region + category | `/suppliers/{country}/{region}/{category}/` | Evidence-backed matches |
| Supplier profile | `/vendors/{supplier_id}/` | One canonical page per supplier |
| Search | `/search/` | Static index with keyword, country, region and category filters |

`getStaticPaths` rules: generate only routes with at least one eligible supplier; compute every count from data; count companies, not capability rows; breadcrumbs point to real parents; one canonical URL per supplier.

Listing pages are titled "Suppliers in [place]". No "Top 10" or "Best" rankings without a documented, real ranking method.

---

## 10. Search

Phase one: a sanitized `search-index.json` with public fields only (ID, name, profile URL, domain, country/region codes, evidenced service types and categories, aliases). Filter in the browser with accessible controls and put query state in the URL. Measure payload size and shard by country if it grows large. Never advertise stock or price search.

A backend search API is a later, separate project with its own decision record, versioned contract and tests.

---

## 11. Design, accessibility and performance

Readable tables, cards on narrow screens, clear labels and provenance indicators, semantic headings and landmarks, breadcrumb `<nav>`, keyboard-operable filters, visible focus, good contrast, minimal JavaScript and no third-party trackers by default. Use only owned or licensed images.

---

## 12. SEO and AI-discoverability

- Unique titles and meta descriptions; absolute canonical URLs built from `production_domain`.
- XML sitemap of public canonical pages only; `robots.txt` pointing to it; no accidental `Disallow: /`.
- JSON-LD where it matches visible content: `Organization` or `LocalBusiness` on profiles, `BreadcrumbList`, `ItemList` on listings.
- No invented phones, ratings, coordinates, hours, prices or reviews in structured data. No mass-generated FAQs.
- Optional `/llms.txt` as a navigation aid only.
- Check the built HTML with JavaScript off. This aims at accurate discoverability, not guaranteed rankings.

---

## 13. Git, GitHub and deployment

### 13.1 Git workflow
- The owner makes the first local commit before the agent starts.
- The agent commits after every phase with a clear message.
- Before Phase 2, the agent checks `.gitignore` (`node_modules/`, `.astro/`, `dist/`, `.env*` except `.env.example`) and tells the owner exactly which `data/` files will be committed.
- Raw source files may contain private research. The agent asks before committing them.

### 13.2 GitHub
This project already lives in the owner's **private** GitHub repo. The agent works on a branch named `build/<project-folder-name>`, commits after every phase, pushes that branch, and opens a pull request when Phase 5 passes. It never pushes to `main` and never changes repo settings.

### 13.3 Cloudflare Pages
- Connect the private repo. Set the root directory to `apps/web` (or the repo root, if the build script lives there; pick one and document it).
- Build command: the tested single script that runs validate → build public data → `astro build`. CI does not re-import or mutate reviewed canonical data.
- Output directory: `dist` (relative to `apps/web`).
- Environment: the public site URL and any non-secret build settings only. Set `NODE_VERSION` to match local.
- Static output only; no SSR adapter.
- If `production_domain` is TBD, deploy to the `*.pages.dev` preview, and note that canonicals and the sitemap must be rebuilt once the domain is set.

### 13.4 Release smoke test (against the live URL)
Homepage loads; `/suppliers/` shows real counts; at least one real country, region, category and profile page loads where eligible records exist; supplier links resolve; search works with correct empty states; robots and sitemap are correct; no internal links 404; no private data in `dist/`; commit and rollback path recorded.

If GitHub or Cloudflare access is missing, stop at a verified local build and report the blocker.

---

## 14. Phases and exit gates

| Phase | Work | Exit gate |
|---|---|---|
| **0. Audit** | Inspect folder, Git state, source files; write `docs/source-inventory.md` with tabs, headers, counts, overlaps and proposed mappings | Nothing overwritten; real sources identified; **ambiguous mappings raised with the owner before Phase 1** |
| **1. Data** | Schemas, taxonomy, mappings, importer, dedupe, ledger, validation, reports, tests; run the verification pass and website lookup where approved; generate public data | Reproducible import with zero unexplained hard errors on **real** data; every public record traces to source rows; real eligible counts by country/region/category |
| **2. Scaffold** | Astro in `apps/web`, Tailwind, layout, header/footer, lint/type checks | Dev and production builds pass; no fabricated pages |
| **3. Routes** | Four directory levels and profiles from canonical data | Every eligible supplier appears where supported; no empty hubs; no slug collisions |
| **4. Search** | Static index and accessible filters | Known names, regions and categories found; blank, typo and no-result cases handled; no private fields |
| **5. SEO** | Metadata, canonicals, JSON-LD, sitemap, robots, 404, link audit, methodology page | Structured data matches visible content; sitemap holds canonical routes only; tests green |
| 6. Launch (**not in scope for this agent**) | Owner connects Cloudflare Pages later | Not run by this agent |
| 7+ (later) | RFQ/lead forms, inventory feeds, backend search, more countries | Separate decision records, consent and privacy handling, tests |

Each phase writes `docs/handoffs/phase-N.md`: what changed, commands and output, input/output counts, limitations, next steps. A new agent must be able to resume from the docs alone.

**If the source files are missing or unreadable, stop after Phase 0 and report it. Never continue on test fixtures.**

---

## 15. Test matrix

| Area | Must prove |
|---|---|
| Schema | Valid records pass; required fields, enums and formats enforced |
| Referential | Broken FKs fail |
| Import | Mappings stable; raw files unchanged; row totals reconcile |
| Dedupe | Exact duplicates caught; shared-domain branches not auto-merged |
| Grounding | Provenance-only columns never become locations or capabilities |
| Routing | Deterministic slugs; no empty hubs; no duplicates |
| HTML | Supplier data present with JS off; proper table headers |
| Search | Correct results and empty states; no private fields |
| SEO | Unique titles and canonicals; valid JSON-LD; sitemap and link audit |
| Security | Escaped text; no secrets or private data in `dist/` |

Report tests that exist and passed separately from tests that are only planned.

---

## 16. Final status report (required at the end)

```text
Status: SHIPPED / LOCAL_BUILD_ONLY / BLOCKED
Repository + commit: <actual>
Deployed URL: <actual or NOT DEPLOYED>
Source files imported: <names, tabs, row counts>
Unique suppliers: <count>
Rejected: <count, with top reasons>
Needs review: <count>
No official website found: <count>
Eligible public suppliers: <count by country>
Country / region / category hubs: <counts>
Supplier profiles: <count>
Validation and tests: <actual results and warnings>
Build command and result: <exact>
Live smoke test: <exact or not run>
Known limitations: <specific>
Next action: <specific>
```

Never substitute guessed counts or a hypothetical URL.

---

## 17. Prompt to give the agent

Use the prompt in `PROMPT.md` in this project folder.

---

## 18. References

- Astro: https://docs.astro.build/
- Astro routing: https://docs.astro.build/en/guides/routing/
- Astro deploy: https://docs.astro.build/en/guides/deploy/
- Cloudflare Pages: https://developers.cloudflare.com/pages/
- JSON Schema 2020-12: https://json-schema.org/draft/2020-12
- Schema.org: https://schema.org/

Install commands and integrations change between versions. Confirm them against current official docs on the machine doing the build.
