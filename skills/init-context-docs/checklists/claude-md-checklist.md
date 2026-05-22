# CLAUDE.md Quality Checklist

Used by both generation agents (to know what good looks like before writing) and verification agents (to confirm or correct the output).

## Hard Constraints

- [ ] Present at repo root as `CLAUDE.md`
- [ ] Contains `@AGENTS.md` import — without this, Claude Code does not load the agent guidance
- [ ] No secrets, API keys, passwords, or tokens

## Content Quality

Apply this test to every line: "Would removing this cause Claude to make mistakes?" Remove any line that fails.

- [ ] `@AGENTS.md` import is present at or near the top
- [ ] Does not @import guideline files directly — guidelines are loaded on demand via the AGENTS.md index or `.claude/rules/` path-scoped loaders; importing them here makes them always-loaded, consuming context on every session regardless of relevance
- [ ] Only contains Claude Code-exclusive content: build/test/lint commands, pre-commit hook behavior, Claude-specific behavioral preferences
- [ ] Every build/test command listed actually works in this repo
- [ ] Under 100 lines — most content should be in AGENTS.md, not here

## No Duplication

- [ ] Does not repeat coding conventions, naming patterns, or code style (belongs in AGENTS.md)
- [ ] Does not repeat architectural context or project structure (belongs in AGENTS.md)
- [ ] Does not repeat domain-specific rules (belongs in `docs/*-guidelines.md`)
- [ ] Does not repeat any content already imported via `@AGENTS.md`

## Verification Commands

```bash
wc -l CLAUDE.md                                                        # must be < 100
grep "@AGENTS.md" CLAUDE.md                                            # must match
grep -iE "(api_key|password|secret|token|credential)\s*[:=]\s*(\"[^\"]{4,}\"|'[^']{4,}'|[A-Za-z0-9+/_-]{20,})" CLAUDE.md  # must be empty
grep -iE "(naming|code style|architecture|always use)" CLAUDE.md       # review — likely belongs in AGENTS.md
```
