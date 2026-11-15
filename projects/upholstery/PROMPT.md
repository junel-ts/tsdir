# Prompt for the agent (copy everything in the box)

```text
Work ONLY inside projects/upholstery/ in this repo. Treat that folder
as the project root. Read projects/upholstery/README.md in full before
writing code. Section 1 (Project Config) is authoritative; do not
guess anything marked TBD.

Do not overwrite existing files, restart supplier research, add
suppliers that are not in the workbook, or fabricate any supplier
information.

Start with Phase 0: inspect the workbook in data/raw/, write
docs/source-inventory.md, and show me the proposed column mappings.
If anything is ambiguous, stop and ask me.

Then complete Phase 1, including the verification pass and website
lookup approved in Section 1, using only the tools in Section 7.3 and
staying under credit_cap. Continue through Phases 2-5 in order,
meeting each exit gate and writing docs/handoffs/phase-N.md.

BUILD ONLY: complete Phases 0-5 and stop. Do not deploy, connect
Cloudflare, buy a domain or change repo settings.

Git: work on a branch named build/upholstery. Commit after each phase,
never commit .env files, secrets or node_modules, and push the branch.
When Phase 5 passes, open a pull request into main. Never push to main.

Finish with the status report (expected status: LOCAL_BUILD_ONLY).
```
