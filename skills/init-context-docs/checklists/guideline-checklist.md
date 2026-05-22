# Domain Guideline Quality Checklist

Used by both generation agents (to know what good looks like before writing) and verification agents (to confirm or correct the output).

## Hard Constraints

- [ ] Under 200 lines — if over, cut explanations and redundant examples first
- [ ] Located at `docs/<domain>-guidelines.md`
- [ ] No secrets, API keys, passwords, or tokens
- [ ] Every rule is strictly necessary — unnecessary requirements reduce agent task success rates; remove anything an agent could infer from reading the source code or that wouldn't cause a mistake if absent

## Content Quality

- [ ] Every rule is repo-specific — describes how THIS codebase does things, not general best practices (e.g., "this repo validates input in `middleware/validator.ts`", not "always validate user input")
- [ ] Rules are actionable imperatives, not explanations or tutorials
- [ ] Only covers conventions an agent couldn't infer from reading the source code — structural overviews and architectural maps don't belong here
- [ ] No absolute language ("Never", "Always", "Must") unless verified against the codebase; use "Prefer" / "Avoid" with known exceptions listed
- [ ] Code examples included only where the pattern is non-obvious from the rule itself
- [ ] Most critical rules appear first; verification commands appear last — content in the middle receives the least agent attention (lost-in-the-middle effect)

## Reference Accuracy (verification agents)

- [ ] Every file path, class name, function name, and library reference exists in the codebase — verify with Grep/Glob
- [ ] Every factual claim about library or framework behavior is accurate — use WebSearch when uncertain
- [ ] Every rule using absolute language has no counter-examples in the codebase — grep for violations

## Cross-Document Consistency (verification agents)

- [ ] No rules contradict rules in other `docs/*-guidelines.md` files — if conflict exists, the more specific rule wins
- [ ] No rules duplicate rules already in another guideline file

## Verification Section

- [ ] Ends with a Verification section listing commands agents can run to check compliance (e.g., `npm test`, `ruff check src/`) — this is the highest-leverage item in an agent context file; do not skip it
- [ ] Every command listed actually exists in the repo (script, Makefile target, or installed tool)

## Verification Commands

```bash
wc -l docs/<domain>-guidelines.md                                                    # must be < 200
grep -iE "(api_key|password|secret|token|credential)\s*[:=]\s*(\"[^\"]{4,}\"|'[^']{4,}'|[A-Za-z0-9+/_-]{20,})" docs/<domain>-guidelines.md  # must be empty
grep -E "(Always|Never|Must|All) " docs/<domain>-guidelines.md                      # review for absolute language
grep -oE "[a-zA-Z0-9_/-]+\.(ts|js|py|java|go|md)" docs/<domain>-guidelines.md \
  | xargs -I{} sh -c '[ -f "{}" ] || echo "Missing: {}"'                            # verify file references exist
```
