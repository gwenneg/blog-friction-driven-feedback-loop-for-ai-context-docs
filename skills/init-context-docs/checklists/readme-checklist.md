# README.md Quality Checklist

Used by both generation agents (to know what good looks like before writing) and verification agents (to confirm or correct the output).

## Hard Constraints

- [ ] Present at repo root as `README.md`
- [ ] No secrets, API keys, passwords, or tokens

## Content Quality

- [ ] Project purpose and description are clear in the first few lines — front-load the most critical information
- [ ] Tech stack and key dependencies are listed
- [ ] Project structure overview is present
- [ ] Build and run instructions are accurate and tested
- [ ] Links to AGENTS.md and `docs/` for human navigation of the full documentation system (agents discover AGENTS.md via CLAUDE.md import; this link is for humans)

## No Duplication

- [ ] Does not repeat cross-cutting conventions from AGENTS.md — link to it instead
- [ ] Does not repeat domain-specific rules from `docs/*-guidelines.md` — link to them instead
- [ ] Stays high-level: overview, getting started, structure — detailed conventions belong elsewhere

## Verification Commands

```bash
grep -iE "(api_key|password|secret|token|credential)\s*[:=]\s*(\"[^\"]{4,}\"|'[^']{4,}'|[A-Za-z0-9+/_-]{20,})" README.md  # must be empty
grep -iE "^##? .*(install|build|getting started|usage|quick start)" README.md  # must match
grep -E "\[.*\]\(AGENTS\.md\)|\[.*\]\(docs/" README.md                # must have links to docs
```
