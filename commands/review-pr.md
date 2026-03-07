---
description: Review current PR changes for type safety, correctness, and code quality
---

Review my changes against the main branch.

Run `git diff main...HEAD` to get the full diff, then analyze all changed files. Prioritize type safety, viem pattern correctness, error handling, bundle impact, and security (no leaked secrets, XSS vectors, or unsafe `any` casts).

Provide feedback as plain prose with specific file:line references. No emoji headers, no bold-label patterns like "**Problem:**", no markdown headings in the output. Write like a senior engineer leaving terse, direct review comments.

Skip files that are auto-generated (ABIs in `src/abis/`), formatted (handled by linter), or have only trivial changes.

Follow the comment style in `.claude/PR_REVIEW_GUIDE.md`.
