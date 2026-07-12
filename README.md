# compact-comments

> A [Claude Code](https://code.claude.com) plugin that compresses code comments into **single-line, lossless, agent-optimized** notes — dense for AI agents, out of the way for humans.

Verbose comments and long doc blocks are great for people but cost an AI agent tokens and cost you vertical space. `compact-comments` rewrites them into the tersest form that still preserves **every fact** an agent needs: it collapses multi-line blocks onto one line, drops human-oriented scaffolding (articles, filler, prose rhythm), and keeps identifiers, numbers, URLs, error codes, and the crucial *why* verbatim. Your code gets cleaner; the meaning survives.

It is deliberately conservative about what it touches — comments that a compiler, linter, or type checker actually reads are left **byte-for-byte identical**.

## Install

In Claude Code:

```text
/plugin marketplace add sifat004bk/compact-comments
/plugin install compact-comments@sifat-skills
```

Then run it on a file or a directory:

```text
/compact-comments src/api/handlers.py
/compact-comments src/            # whole tree; skips vendored/build/minified
```

## How it works

1. **Asks scope** — comments only, or comments **+** docstrings/JSDoc (and, for docstrings, whether to keep doc markup).
2. **Previews the first file** as a before/after diff, then offers a menu: *Apply to the rest · Comments only · Less aggressive · Revert & stop · Other*.
3. **Applies** to the rest and **parse-checks** each file (`py_compile`, `node --check`, …) to prove no code was corrupted.

Edits are made in place, so it works best in a git repo — every change is a reviewable `git diff` you can revert.

## Use cases

- **Prep a codebase for AI agents** — denser comments mean more signal per token when Claude reads your code, without bloating what humans see.
- **Reclaim vertical space** — collapse 20–30 line module/function header blocks to a single line so the actual code fits on screen.
- **Clean up AI-generated code** — LLM output often over-comments; compact it in one pass before committing.
- **Keep the *why*, cut the *what*** — preserves rationale, gotchas, ticket refs, and incident links; trims comments that merely restate the code.
- **Whole-directory sweeps** — point it at a package; it recurses source files and skips `node_modules`, `dist`, `.venv`, `*.min.js`, and friends.

## Examples

**Collapse a multi-line block** — Python

```python
# before
# This helper takes a list of user records and filters out any that are marked
# as inactive, then sorts the remaining records by their signup date in
# descending order so that the newest users appear first.

# after
# filter out inactive users; sort by signup date desc (newest first)
```

**Keep every hard fact and the reasoning** — Python

```python
# before
# We need to retry here because the upstream S3 service occasionally returns a
# 503 during their maintenance windows, and boto3's default retry policy doesn't
# cover this particular case. See INFRA-2891 for the incident writeup.

# after
# retry: upstream S3 intermittently 503s in maint windows; boto3 default retry misses this; see INFRA-2891
```

**Compress JSDoc prose, preserve the types verbatim** — TypeScript

```ts
// before
/**
 * Debounce a function so it only runs after calls stop.
 * @param {Function} fn - the function to debounce
 * @param {number} ms - milliseconds to wait after the last call
 * @returns {Function} the debounced function
 */

// after
/** debounce fn: runs only after calls stop. @param {Function} fn fn to debounce @param {number} ms ms to wait after last call @returns {Function} debounced fn */
```

**Leaves functional & legal comments untouched**

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import os                       # noqa: F401
x = compute()                   # type: ignore[no-any-return]
# Copyright 2026 Acme Corp. Licensed under Apache-2.0.
```

Every line above is left exactly as-is.

## What it never rewrites

To keep changes safe, these are preserved byte-for-byte:

- **Shebangs** and **encoding lines** (`#!/usr/bin/env python3`, `# -*- coding: utf-8 -*-`)
- **Linter / type / tool directives** — `# noqa`, `# type: ignore`, `# pylint:`, `# mypy:`, `# shellcheck disable=`, `// eslint-disable`, `// @ts-ignore`, `// prettier-ignore`, build pragmas, source-map comments, JSX pragmas, `# region`, …
- **JSDoc *type* annotations** (`@param {T}`, `@type {T}`) — in JS they *are* the types
- **Doctests** (`>>>` lines) — they get executed
- **Commented-out code** — that's disabled code, not prose
- **License / copyright headers**
- Inside every comment it *does* rewrite: **TODO/FIXME markers**, ticket refs, URLs, file paths, error codes, versions, and units stay verbatim

When in doubt, it skips and tells you.

## Requirements

[Claude Code](https://code.claude.com). No other dependencies — parse checks use whatever is already on your PATH (`python`, `node`, …).

## License

[MIT](LICENSE) © Sifat Ullah Chowdhury
