---
name: sync-upstream
description: Sync fork with upstream and auto-resolve conflicts
argument-hint: "[optional: upstream branch, default 'main']"
disable-model-invocation: true
---

# Sync Fork with Upstream

Sync this fork with upstream (`EveryInc/compound-engineering-plugin`) using merge, auto-resolve conflicts based on our local customization rules, and open a PR.

## Step 1: Setup & Fetch

Add the upstream remote if missing, then fetch:

```bash
git remote get-url upstream 2>/dev/null || git remote add upstream https://github.com/EveryInc/compound-engineering-plugin.git
git fetch upstream
```

## Step 2: Check for New Commits

Determine the upstream branch (use `$ARGUMENTS` if provided, otherwise `main`).

```bash
git rev-parse main
git rev-parse upstream/main
```

If identical: report **"Already up to date with upstream."** and stop.

Otherwise, show new commits:

```bash
git log --oneline main..upstream/main
```

## Step 3: Create Branch & Merge

```bash
BRANCH="sync/upstream-$(date +%Y-%m-%d)"
git checkout -b "$BRANCH" main
git merge upstream/main --no-edit
```

If the branch name exists, append a time suffix: `sync/upstream-YYYY-MM-DD-HHMMSS`.

**If the merge is clean**, skip to Step 5.

## Step 4: Auto-Resolve Conflicts

For each conflicted file, read the file and resolve using these rules:

### Resolution Rules

| Conflict | Resolution |
|----------|------------|
| `mcpServers` block added by upstream | **Remove it** — this fork does not use mcpServers in plugin.json |
| Ruby/Rails keywords, agents, or references | **Use our version** — we've generalized to framework-agnostic |
| `"rails"`, `"ruby"` in keywords arrays | **Remove them** — keep TypeScript and generic keywords only |
| Version numbers (in plugin.json, marketplace.json) | **Take upstream's version** |
| New upstream content (new agents, commands, skills, features) | **Accept upstream's additions** |
| Description strings with component counts | **Take upstream's counts**, then verify against actual files |
| Whitespace or formatting differences | **Take upstream's formatting** |

### Resolution Process

For each conflicted file:

1. Read the file to see conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
2. Apply the rules above to determine the correct resolution
3. Edit the file to remove conflict markers with the resolved content
4. `git add <file>`

After all conflicts are resolved:

```bash
git merge --continue
```

**If a conflict is genuinely ambiguous** (not covered by the rules above), show the conflict to the user and ask how to resolve it. This should be rare.

## Step 5: Push & Create PR

```bash
git push -u origin "$BRANCH"
```

Create PR with a summary of what was synced:

```bash
gh pr create --title "Sync upstream (YYYY-MM-DD)" --body "$(cat <<'EOF'
## Upstream Sync

Merges N commit(s) from [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin).

### New commits

```
<commit list from step 2>
```

### Conflict resolutions (if any)

<list each file that had conflicts and what resolution was applied>
EOF
)"
```

## Step 6: Clean Up

```bash
git checkout main
```

Report the PR URL to the user.
