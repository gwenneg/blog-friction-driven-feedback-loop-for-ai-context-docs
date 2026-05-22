---
name: setup-friction-capture
description: Install the friction capture hook that records AI context doc gaps at the end of each session
allowedTools:
  - Bash(bash *)
  - Bash(cat *)
  - Bash(chmod *)
  - Bash(curl *)
  - Bash(echo *)
  - Bash(jq *)
  - Bash(mkdir *)
  - Edit
  - Read
  - Write
---

# Instructions

## Phase 1: Install or update the toolkit files

If `.claude/scripts/install-friction-capture.sh` already exists, run it directly:

```bash
bash .claude/scripts/install-friction-capture.sh
```

Otherwise, bootstrap by downloading the update script first, then run it:

```bash
mkdir -p .claude/scripts .claude/skills/update-context-docs

TAG=$(curl -fsSL https://api.github.com/repos/gwenneg/blog-friction-driven-feedback-loop-for-ai-context-docs/releases/latest | jq -r '.tag_name')
if [[ -z "$TAG" || "$TAG" == "null" ]]; then
  echo "Error: could not fetch latest release tag." >&2
  exit 1
fi

curl -fsSL "https://raw.githubusercontent.com/gwenneg/blog-friction-driven-feedback-loop-for-ai-context-docs/${TAG}/skills/setup-friction-capture/scripts/install-friction-capture.sh" \
  -o .claude/scripts/install-friction-capture.sh
chmod +x .claude/scripts/install-friction-capture.sh

bash .claude/scripts/install-friction-capture.sh
```

In both cases: exit codes 0 and 2 mean success — continue to Phase 2. Any other non-zero exit means failure — stop and report the error.

## Phase 2: Configure the SessionEnd hook

Read `.claude/settings.json` if it exists. Check whether a SessionEnd hook already calls `friction-hook.sh`. If not, add it.

The hook entry to add:
```json
{
  "matcher": "",
  "hooks": [
    {
      "type": "command",
      "command": "bash .claude/scripts/friction-hook.sh",
      "timeout": 5000
    }
  ]
}
```

Use `jq` to merge this into the existing `hooks.SessionEnd` array, preserving all other content. If `.claude/settings.json` does not exist, create it with just the hooks block.

## Phase 3: Optionally install the pre-push git hook

Present this explanation to the user:

> **Pre-push blocking hook (optional)**
>
> The toolkit includes a `pre-push` git hook that blocks the push once `FRICTION_SESSION_THRESHOLD` distinct session directories have accumulated under `.claude/friction/` (default: 3). It prints:
> `Push blocked: N session(s) with unprocessed friction in .claude/friction/. Run /update-context-docs to improve the repo docs, or use --no-verify to skip.`
>
> Blocking rather than just printing ensures the message always reaches the user — including when an AI agent is doing the push, since a failed push forces the agent to surface the error. The push can always be bypassed with `--no-verify`. Teams can tune the threshold by setting `FRICTION_SESSION_THRESHOLD` in their environment.
>
> Adding it to the repo means everyone on the team gets this behaviour automatically, as long as they run `git config core.hooksPath .githooks` once in their clone.

Then use `AskUserQuestion` to ask whether to add it to the repo:
- **Yes** — install the hook and include it in the PR
- **No** — skip it, the rest of the setup continues unchanged

If the user selects **Yes**:

```bash
mkdir -p .githooks
TAG=$(cat .claude/.friction-capture-version)
curl -fsSL "https://raw.githubusercontent.com/gwenneg/blog-friction-driven-feedback-loop-for-ai-context-docs/${TAG}/skills/setup-friction-capture/.githooks/pre-push" \
  -o .githooks/pre-push
chmod +x .githooks/pre-push
```

Add `.githooks/pre-push` to the files staged in Phase 5, and add this to the PR body:

> **For each team member:** run `git config core.hooksPath .githooks` once in your clone to activate the pre-push hook.

Then use a second `AskUserQuestion` to ask whether to activate the hook locally right now:
- **Yes** — run `git config core.hooksPath .githooks`
- **No** — skip, the hook file is still committed for the team

If the user selected **No** at the first question, skip the rest of this phase entirely.

## Phase 4: Update .gitignore

Fetch the `.gitignore.example` file from this repository:

```bash
TAG=$(cat .claude/.friction-capture-version)
curl -fsSL "https://raw.githubusercontent.com/gwenneg/blog-friction-driven-feedback-loop-for-ai-context-docs/${TAG}/.gitignore.example"
```

For each line in the fetched content, check whether it already exists in the project's `.gitignore`. Add any missing lines. Never duplicate existing entries. If `.gitignore` does not exist, create it.

## Phase 5: Commit and open a PR

Stage these files:
- `.claude/scripts/friction-hook.sh`
- `.claude/scripts/install-friction-capture.sh`
- `.claude/skills/update-context-docs/SKILL.md`
- `.claude/.friction-capture-version`
- `.claude/settings.json`
- `.gitignore`
- `.githooks/pre-push` (only if the user opted in at Phase 3)

Create a branch named `chore/setup-friction-capture` (add `-2`, `-3`, etc. if it already exists).

Commit with message: `chore: set up friction capture ($(cat .claude/.friction-capture-version))`

Push the branch and open a PR with a body that describes what was added and includes this note for reviewers:

  > **For each team member:** after this PR is merged, add `FRICTION_CAPTURE=1` to your `.claude/settings.local.json` to opt in to friction capture. That file is not committed and stays local.

Include the standard `Generated with Claude Code` footer.

## Phase 6: Offer personal opt-in

After the PR is created, ask:

> Friction capture is now set up for the team. To opt in yourself, I can add `FRICTION_CAPTURE=1` to your `.claude/settings.local.json` (this file is never committed). Would you like me to do that?

If yes: ensure `.claude/settings.local.json` has `FRICTION_CAPTURE` set to `"1"` in its `env` block. Create the file if it doesn't exist; merge the key if it does.
