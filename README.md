# sifat-skills — Claude Code marketplace

A personal [Claude Code](https://code.claude.com) plugin marketplace.

## Plugins

### `compact-comments`

Compress code comments into **single-line, lossless, agent-optimized** notes:
collapses verbose or multi-line comments (and, when asked, docstrings / JSDoc)
to the tersest form that still preserves every fact an AI agent needs, while
keeping the surrounding code clean for humans. Leaves functional comments
(shebangs, encoding lines, linter/type directives, JSDoc types, doctests,
commented-out code, license headers) byte-for-byte untouched.

## Install

In Claude Code:

```
/plugin marketplace add REPLACE_WITH_YOUR_GITHUB_USERNAME/compact-comments
/plugin install compact-comments@sifat-skills
```

Then run it on a file or directory:

```
/compact-comments path/to/file.py
```

It will ask whether to include docstrings, preview a sample diff, then apply and
run a parse check.

## License

MIT
