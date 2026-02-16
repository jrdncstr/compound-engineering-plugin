---
name: sync-upstream
description: Rebase local commits onto upstream and force-push
argument-hint: "[optional: upstream branch, default 'main']"
disable-model-invocation: true
---

# Sync Fork with Upstream (Rebase)

Rebase this fork's local commits onto upstream (`EveryInc/compound-engineering-plugin`), resolve conflicts using local customization rules, and force-push.

## Step 1: Setup & Fetch

Add the upstream remote if missing, then fetch:

```bash
git remote get-url upstream 2>/dev/null || git remote add upstream https://github.com/EveryInc/compound-engineering-plugin.git
git fetch upstream
```

## Step 2: Check for New Commits

Determine the upstream branch (use `$ARGUMENTS` if provided, otherwise `main`).

```bash
git log --oneline main..upstream/main
```

If empty: report **"Already up to date with upstream."** and stop.

Otherwise, show the new upstream commits and continue.

## Step 3: Rebase onto Upstream

```bash
git rebase upstream/main
```

**If the rebase is clean**, skip to Step 5.

## Step 4: Auto-Resolve Conflicts

For each conflicted file during rebase, read the file and resolve using these rules:

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
5. `git rebase --continue`

Repeat until the rebase completes.

**If a conflict is genuinely ambiguous** (not covered by the rules above), show the conflict to the user and ask how to resolve it.

## Step 5: Verify Counts

After rebase, verify component counts match actual files:

```bash
echo "Agents: $(find plugins/compound-engineering/agents -name '*.md' | wc -l | tr -d ' ')"
echo "Commands: $(find plugins/compound-engineering/commands -name '*.md' | wc -l | tr -d ' ')"
echo "Skills: $(ls -d plugins/compound-engineering/skills/*/ | wc -l | tr -d ' ')"
```

If counts in description strings don't match, fix them in:
- `plugins/compound-engineering/.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`
- `plugins/compound-engineering/README.md`

Amend the last commit with count fixes if needed.

## Step 6: Force Push

```bash
git push origin main --force-with-lease
```

Report the result: how many upstream commits were incorporated and whether any conflicts were resolved.
