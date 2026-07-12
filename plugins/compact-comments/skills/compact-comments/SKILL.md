---
name: compact-comments
description: >-
  Compress code comments into single-line, lossless, agent-optimized notes —
  collapsing verbose or multi-line comments (and, when asked, docstrings/JSDoc)
  to the tersest form that still preserves every bit of meaning an AI agent
  needs, while keeping the surrounding code clean for humans. Use whenever the
  user wants to shorten, compact, minify, condense, densify, or "agent-ify"
  comments in a file or directory; make comments terse/dense for Claude;
  collapse multi-line comment blocks onto one line; or cut comment verbosity
  without losing information. Trigger on phrasings like "make these comments
  shorter for AI", "summarize the comments in this folder losslessly", "flatten
  comment blocks", or "compress the docstrings". Do NOT use for writing
  brand-new comments, deleting all comments, or generating human documentation.
---

# Compact Comments

Rewrite the comments in a file or directory into **single-line, lossless,
agent-optimized notes**. The goal is a codebase where the *code* stays clean and
readable for humans, and the *comments* become dense shorthand carrying full
meaning for an AI agent (especially Claude) in the fewest characters and the
fewest lines possible.

"Agent-optimized" means you may throw away everything a comment does purely for
human comfort — articles, framing, politeness, prose rhythm — because the reader
is a model that parses telegraphic text effortlessly. "Lossless" means you may
**not** throw away meaning: every fact, reason, caveat, identifier, and number a
future agent would need must survive.

## Inputs

The user picks a **file or a directory**.

- If a path is given in the request, use it.
- If none is given, ask which file or directory to process.
- For a directory, recurse over source files. **Skip** vendored/generated/build
  trees (`node_modules`, `.git`, `dist`, `build`, `out`, `target`, `.venv`,
  `venv`, `__pycache__`, `vendor`, `.next`, `coverage`), minified files
  (`*.min.js`, `*.min.css`), lockfiles, and anything binary or non-source.
  Respect `.gitignore` if present.
- Before doing work on a directory, report the plan: how many files, grouped by
  language. This plus the preview step (below) keeps a bulk edit safe.

## The prime directive

**Only comment *text* may change. Every other byte of the file stays identical.**

You are not reformatting code, renaming things, or fixing bugs — you are
rewriting comment prose in place. Non-comment tokens, string contents,
whitespace in code, and blank lines between statements must come out byte-for-byte
unchanged. The only structural change you may make is collapsing a multi-line
comment *block* into a single comment line (see below) — which removes comment
lines, never code lines.

"Lossless" = preserve all **meaning**; discard only **redundancy** (facts fully
recoverable from the code or another comment) and **prose scaffolding** (words
present only for human readability).

## Never touch these — functional and legal comments

Some comments are read by a compiler, interpreter, linter, type checker, doc/build
tool, or the law. Their exact bytes matter. Rewriting them causes **silent
breakage** — the code still looks fine but behaves differently or stops type-checking.
Leave every one of these **byte-for-byte identical**:

- **Shebangs**: `#!/usr/bin/env python3`, etc. (must stay line 1).
- **Encoding / mode lines**: `# -*- coding: utf-8 -*-`, `// -*- mode: ... -*-`.
- **Linter / type / formatter directives**: `# noqa`, `# noqa: E501`,
  `# type: ignore`, `# pragma: no cover`, `# pylint: disable=...`, `# mypy: ...`,
  `# flake8: noqa`, `# shellcheck disable=SC1091`, `# shellcheck source=...`,
  `// eslint-disable`, `// eslint-disable-next-line`, `/* eslint-disable */`,
  `// @ts-ignore`, `// @ts-expect-error`, `// prettier-ignore`,
  `// istanbul ignore next`, `// swiftlint:disable ...`, `// deno-lint-ignore`.
  (General rule: any `# <tool>: <directive>` / `// <tool>-<directive>` a linter
  or checker parses — if in doubt it's one, skip it.)
- **Build / codegen / tool pragmas**: `//go:build`, `// +build`, `//go:generate`,
  `//nolint`, `#pragma ...`, `//# sourceMappingURL=...`, webpack/vite magic
  comments (`/* webpackChunkName: "x" */`, `/* @vite-ignore */`), JSX pragmas
  (`/** @jsx h */`, `/** @jsxImportSource preact */`), `// SPDX-License-Identifier: ...`.
- **IDE region/fold markers**: `// #region` / `// #endregion`, `#region` /
  `#endregion`, `// MARK: -`.
- **License / copyright headers**: never compress or reword legal text.
- **Commented-out code**: this is disabled *code*, not prose. You cannot shrink
  code and stay lossless. Leave it verbatim. (If it looks like dead code the user
  might want gone, mention it — don't act on it.)
- **JSDoc *type* information in JS/TS**: in JS projects, JSDoc **is** the type
  system. Preserve `@param {T}`, `@returns {T}`, `@type {T}`, `@typedef`,
  `@template`, `@callback`, `@enum`, `@satisfies` and their type expressions
  exactly. You may compress the surrounding *prose*, not the types.
- **Doctests**: any `>>> ` / `... ` example lines inside a docstring are executed
  by `doctest`. Keep them verbatim.
- **Preserve verbatim inside any comment you do rewrite**: marker tokens
  (`TODO`, `FIXME`, `HACK`, `XXX`, `BUG`, `NOTE`, `WARNING`, `@deprecated`) and
  their assignees, issue/ticket refs (`JIRA-123`, `#4567`), URLs, file paths,
  error codes, flags, version numbers, and units. These are grep anchors and hard
  facts — never paraphrase them.

When unsure whether a comment is functional, **leave it alone** and note it. A
skipped comment is free; a corrupted directive is a silent bug.

## What to compress

Scope is **only comments worth shrinking** — don't churn comments that are
already terse, because needless edits create diff noise and hide the real
changes. Target:

- Multi-line comment **blocks** → collapse to one line.
- Long single-line prose comments.
- Docstrings and doc-comments (Python `"""..."""`, `/** ... */`) — **only if the
  user opted to include them** (see Workflow step 1); prose parts only, keeping
  doctests and JSDoc type tags verbatim as above. If they chose comments-only,
  leave every docstring/doc-comment byte-for-byte untouched.

Leave short, already-dense comments as they are.

## How to compress — the craft

Two kinds of information live in comments, and they deserve opposite treatment:

- **WHY** — rationale, gotchas, non-obvious constraints, links to incidents,
  historical context, "don't do X because Y". This is the crown jewel: the agent
  **cannot** recover it from the code. Preserve every bit of it, just tighter.
- **WHAT** — a restatement of what the code plainly does. The agent **can**
  recover this by reading the code, so compress it hard (but, being lossless,
  keep a trace rather than deleting the comment outright).

Techniques:

- **Cut prose scaffolding**: articles (`a`/`an`/`the`), fillers (`basically`,
  `essentially`, `note that`, `please`, `we`, `here`, `this function`,
  `is used to`, `is responsible for`, `in order to` → `to`).
- **Use symbols and shorthand** an agent reads instantly: `→` (returns/leads to),
  `&` / `;` (and / clause break), `|` (or), `w/`, `w/o`, `b/c`, `e.g.`, `i.e.`,
  `==` `!=` `>=` `<=`, `+`. Join what were separate lines/sentences with `;`.
- **Keep verbatim**: identifiers, numbers, URLs, error codes, units, negations,
  and conditions. Terseness must never flip a meaning — `not` and edge-case notes
  stay.
- **One line, always**. Let the line grow as wide as it needs; never wrap a
  comment onto a second line. Turning one comment into two lines is forbidden.
  Collapsing a block *comment* into one line is the whole point.
- **Keep delimiters valid and position stable**: a `/* ... */` block becomes a
  one-line `/* ... */`; a run of `//` / `#` lines becomes a single `//` / `#`
  line in the same place; a trailing `x = 1  # ...` stays trailing on its line;
  a leading block stays directly above the same code. Keep the comment's
  indentation matching the code.
- **Strip decoration, keep content**: `# ===== SECTION: Parsing =====` → `# SECTION: Parsing`.

## Workflow

1. **Pick comment scope — ask the user first.** Whether to rewrite docstrings is
   a real safety/aggressiveness dial, and it's the user's to own — so ask before
   touching anything, unless the request already states it. Offer:
   - **Comments only** — rewrite `#`, `//`, `/* */` etc.; leave every docstring
     and doc-comment (`"""..."""`, `/** */`) completely alone. Safest: nothing a
     doc generator, `help()`, a framework, or the JSDoc type system reads is
     changed. *This is the default if you cannot ask (e.g. a non-interactive
     run).*
   - **Comments + docstrings** — also compress docstring/doc-comment prose. When
     they pick this, ask one quick follow-up: **keep doc markup** (RST `` `` /
     `:param:` / JSDoc `@tag` decoration preserved verbatim — safe if Sphinx /
     pdoc / JSDoc renders these) or **drop markup** for maximum terseness.
     Default to *keep markup* when unsure.

   Honor an explicit scope in the request without re-asking ("just the comments,
   not docstrings" → comments only). Doctests and JSDoc **types** are preserved
   regardless — they're functional, not prose.
2. **Resolve target.** Get the file(s)/dir. For a dir, enumerate source files
   (skipping the trees/patterns above) and report the count by language.
3. **Inventory & classify.** For each file, find every comment. Classify each as
   *skip* (functional/legal/commented-out/short, or a docstring when scope is
   comments-only) or *compress*. Beware strings that merely contain `//`, `#`, or
   `/* */` — those are not comments; never edit inside a string literal.
4. **Rewrite compress-targets** per the craft above, keeping all other bytes
   identical.
5. **Preview, then offer explicit choices — never just ask "looks good?".** After
   compacting the first file, show its before/after diff, then present a short
   menu of concrete next actions (use AskUserQuestion when available; the tool's
   "Other" stays open for free-form direction). Tailor the menu to what actually
   changed this run, drawing from:
   - **Approve** — keep this file's changes and, on a directory run, continue
     applying the same way to the remaining files. Label it "Apply to the rest"
     for a directory, "Keep it" for a single file.
   - **Comments only from here** — undo the docstring / doc-comment changes (in
     this file too) and continue compacting just real comments. Offer this only
     if you actually touched docstrings.
   - **Less aggressive** — keep more wording and leave borderline-short comments
     alone; redo this file safer, then re-preview.
   - **Revert & stop** — restore this file to its original bytes and abort the run.
   - **Other** — let the user name a specific tweak (e.g. "keep JSDoc markup",
     "leave the module header alone", "go even terser").
   Act on their pick before touching anything else. This menu is the safety gate.
6. **Apply to the rest** once they approve. Track a short per-file tally: comments
   compressed, comments skipped-as-functional, characters/lines saved.
7. **Sanity-check.** Where a cheap parse check exists, run it to prove no code
   was corrupted — e.g. `python -m py_compile <f>`, `node --check <f>`,
   `ruff check <f>`, `tsc --noEmit` (if already configured). Report anything you
   skipped for safety so the user knows what wasn't touched and why.

Note if the target isn't under version control (no `.git`): recommend a commit or
backup first, since edits are in place. The preview step mitigates this, but a
clean diff to review/revert is the best safety net.

## Examples

**1 — Multi-line prose block → one line (Python).**
Before:
```python
# This helper takes a list of user records and filters out any that are marked
# as inactive, then sorts the remaining records by their signup date in
# descending order so that the newest users appear first.
def active_sorted(users): ...
```
After:
```python
# filter out inactive users; sort by signup date desc (newest first)
def active_sorted(users): ...
```

**2 — Preserve the WHY and every hard fact, cut the scaffolding (Python).**
Before:
```python
# We need to retry here because the upstream S3 service occasionally returns a
# 503 during their maintenance windows, and boto3's default retry policy doesn't
# cover this particular case. See INFRA-2891 for the incident writeup.
```
After:
```python
# retry: upstream S3 intermittently 503s in maint windows; boto3 default retry misses this; see INFRA-2891
```

**3 — JSDoc: compress prose, keep types verbatim (JS/TS).**
Before:
```js
/**
 * Debounce a function so it only runs after calls stop.
 * @param {Function} fn - the function to debounce
 * @param {number} ms - milliseconds to wait after the last call
 * @returns {Function} the debounced function
 */
```
After:
```js
/** debounce fn: runs only after calls stop. @param {Function} fn fn to debounce @param {number} ms ms to wait after last call @returns {Function} debounced fn */
```

**4 — Functional/legal comments: leave untouched.**
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import os  # noqa: F401
x = compute()  # type: ignore[no-any-return]
# Copyright 2026 Acme Corp. Licensed under the Apache License, Version 2.0.
```
All five stay exactly as-is.

**5 — Docstring with a doctest: compress prose, keep the doctest verbatim (Python).**
Before:
```python
def add(a, b):
    """Add two numbers together and return the resulting sum.

    >>> add(2, 3)
    5
    """
```
After:
```python
def add(a, b):
    """add two numbers → sum.
    >>> add(2, 3)
    5
    """
```
(The `>>>`/result lines are untouched; only the prose collapsed. A docstring may
stay multi-line *only* to keep a doctest runnable — that's the one exception to
one-line, since flattening it would break `doctest`.)

**6 — Commented-out code: not prose, leave verbatim.**
```python
# result = legacy_path(payload)   # <- left as-is, not "summarized"
```

## Reminders

- Skipping is always safe; corrupting a directive is a silent bug. When in doubt,
  skip and say so.
- Don't touch already-terse comments — no diff noise.
- Never let compression flip a meaning; keep negations and conditions exact.
- One line per comment (doctests excepted); code stays clean, comments stay dense.
