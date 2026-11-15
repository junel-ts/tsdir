# Topstitch Digital — Supplier Directories

Static supplier directory websites, one per industry, built by a coding agent from the owner's own supplier research.

**Keep this repo private.** The workbooks in `data/raw/` are unverified private research.

## What's here

| Folder | What it is |
|---|---|
| `projects/upholstery/` | Topstitch Digital upholstery supplier directory (Canada, USA, UK), served at www.topstitchdigital.com/directory/. README, prompt and workbook ready. |
| `template/` | Blank any-industry README and prompt for a new directory. |

Each project folder is self-contained: the agent works only inside it and builds its site in `apps/web/` there.

## What the agent does

Build only. For one project folder, the agent:

1. Inventories the workbook and proposes column mappings (it stops to ask if anything is unclear).
2. Builds the supplier schema, importer and validation.
3. Checks each supplier's own website with Firecrawl (Alexandria as a second check).
4. Builds the Astro directory site: country, region and category pages, supplier profiles, search, SEO.
5. Commits on a `build/<project>` branch and opens a pull request.

It does **not** deploy, connect Cloudflare or buy domains. That is done later by the owner.

## Run a project

1. Open the repo in Claude Code (or another coding agent).
2. Open `projects/<name>/PROMPT.md` and paste its prompt.
3. Answer the agent when it stops after Phase 0 to confirm the column mappings.
4. Review the pull request when it arrives.

Run one project at a time.

## Add a new directory

1. Copy `template/` to `projects/<new-name>/`.
2. Put the supplier spreadsheets in `projects/<new-name>/data/raw/`.
3. Fill in Section 1 of `projects/<new-name>/README.md`.
4. Use `template/PROMPT.md`, replacing `<project>` with the folder name.
