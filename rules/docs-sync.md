# Docs Sync Rules

## Deduplication During Sync

When running `/docs-sync` or reviewing/updating files in the `.claude/` directory:

1. **Scan for duplicate content** across all files in `.claude/` and `.claude/rules/`. If two files describe the same concept, consolidate into one location and remove the duplicate.

2. **Within a single file**, check for repeated sections that say the same thing in different words. Keep the most precise version.

3. **Cross-file overlap**: `CLAUDE.md` should be a concise index and quick reference. Detailed rules belong in `.claude/rules/*.md`. If `CLAUDE.md` duplicates detailed content from a rules file, keep the detail in the rules file and replace the `CLAUDE.md` section with a brief summary and reference.

## What Counts as a Duplicate

- Same rule stated in two different files
- Same architectural description with equal detail in multiple places
- Same table or command reference copy-pasted across files

## What is NOT a Duplicate

- A brief summary in `CLAUDE.md` that references a detailed rule in `rules/*.md`
- The same concept applied to different contexts
- Cross-references between files pointing to the same source of truth
