---
name: maintain-fork
description: Generate GSD phase plan to sync fork with upstream
argument-hint: ""
allowed-tools:
  - Write
  - Edit
  - Bash
  - AskUserQuestion
  - SlashCommand
---

<objective>
Generate a GSD phase plan for syncing the local get-shit-done fork with upstream changes while preserving marketplace compatibility and willist-specific modifications.

This uses `/gsd:plan-phase` to create a proper executable phase plan following GSD patterns.
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/plan-phase.md
@~/.claude/get-shit-done/templates/phase-prompt.md
</execution_context>

<context>
**Known fork-specific modifications to preserve:**
- `.claude-plugin/` directory (removed in upstream, required for marketplace)
- `marketplace.json` with willist references
- All `${CLAUDE_PLUGIN_ROOT}` path replacements (not `@~/.claude/get-shit-done/`)
- `README.md` marketplace installation instructions
- `package.json` repository URL pointing to willist fork

**Known conflict patterns:**
1. Upstream removes `.claude-plugin/` files → always keep your versions
2. Upstream adds new commands with hardcoded paths → must convert to `${CLAUDE_PLUGIN_ROOT}`
3. Upstream modifies shared files → preserve willist references

**Pre-sync requirements:**
- No uncommitted changes
- Upstream remote configured: `git remote -v | grep upstream`
- Current commit noted for rollback if needed
</context>

<process>

<step name="verify_prerequisites">
**Check prerequisites:**

```bash
# Verify we're in get-shit-done repo
git remote -v | grep -E "(origin|upstream).*get-shit-done"

# Check for uncommitted changes
git status --short

# Verify upstream remote
git remote -v | grep upstream
```

If uncommitted changes exist, ask user to commit or stash first.

If upstream remote missing, provide setup:
```bash
git remote add upstream https://github.com/glittercowboy/get-shit-done.git
```
</step>

<step name="fetch_upstream">
**Fetch latest upstream changes:**

```bash
git fetch upstream

# Count commits to integrate
git log --oneline main..upstream/main | wc -l

# Show what's coming
git log --oneline main..upstream/main | head -10
```

Report: "Fetched N commits from upstream. Ready to create sync plan."
</step>

<step name="check_existing_phases">
**Check if sync phase already exists:**

```bash
ls -d .planning/phases/*sync* 2>/dev/null
```

If sync phase exists, ask user:
- Reuse existing phase plan?
- Create new phase (archive old)?

If new phase needed, determine phase number (e.g., 09.2-sync-upstream or 10.1-sync-upstream).
</step>

<step name="determine_phase_number">
**Determine appropriate phase number:**

```bash
# List existing phases
ls -d .planning/phases/*/ 2>/dev/null | sort
```

Use decimal phase numbering for urgent sync work:
- If last phase is 09, use 09.1 or 09.2
- If last phase is 10, use 10.1
- Format: `XX.Y-sync-upstream`
</step>

<step name="create_phase_directory">
**Create phase directory:**

```bash
PHASE_DIR=".planning/phases/XX.Y-sync-upstream"
mkdir -p "$PHASE_DIR"
```

Example: `.planning/phases/09.2-sync-upstream/`
</step>

<step name="invoke_plan_phase">
**Use /gsd:plan-phase to create executable plan:**

Invoke slash command to create proper GSD phase plan:

```
/gsd:plan-phase XX.Y-sync-upstream
```

This will:
- Prompt for phase goal and description
- Guide through task breakdown
- Create proper XML-structured PLAN.md
- Follow GSD planning conventions

**Phase description to use:**
```
Sync local get-shit-done fork with glittercowboy/get-shit-done upstream while preserving marketplace compatibility.

Will integrate N upstream commits, resolve conflicts using preservation rules, and force push updated fork.
```

**Expected tasks (2-3 plans, each 2-3 tasks):**

Plan 1: Rebase execution
- Fetch upstream and start rebase
- Resolve conflicts (plugin files, commands, templates)
- Complete rebase

Plan 2: Post-sync verification and fixes
- Register new upstream commands in plugin.json
- Replace hardcoded paths with ${CLAUDE_PLUGIN_ROOT}
- Verify fork-specific files preserved
- Commit fixes and force push
</step>

<step name="verify_plan_created">
**Verify plan was created:**

```bash
ls -la "$PHASE_DIR"/*-PLAN.md
```

Should show: `XX.Y-01-PLAN.md` (and potentially `XX.Y-02-PLAN.md` if split)

Plan should follow GSD XML structure with:
- Frontmatter (phase, type, domain)
- Objective with purpose and output
- Execution context with workflow references
- Context with project/planning references
- Tasks with type/name/files/action/verify/done
- Verification and success criteria
- Output specification
</step>

</process>

<success_criteria>
- [ ] /gsd:plan-phase invoked successfully
- [ ] Phase directory created (XX.Y-sync-upstream/)
- [ ] PLAN.md file(s) created with XML structure
- [ ] Plan uses proper GSD workflows and templates
- [ ] Tasks follow 2-3 per plan guidance
- [ ] Each task has files/action/verify/done fields
- [ ] Success criteria measurable
- [ ] User knows next steps (/gsd:execute-plan)
</success_criteria>

<output>
After /gsd:plan-phase completes, the phase plan is ready for execution at:

```
.planning/phases/XX.Y-sync-upstream/XX.Y-01-PLAN.md
```

**Next steps:**

1. Review the generated plan(s)
2. Execute with: `/gsd:execute-plan .planning/phases/XX.Y-sync-upstream/XX.Y-01-PLAN.md`
3. Or execute entire phase: `/gsd:execute-phase XX.Y-sync-upstream`

**Example sync phase structure:**

Phase: 09.2-sync-upstream
- Plan 1: Rebase execution (3 tasks)
  - Task 1: Fetch and start rebase
  - Task 2: Resolve conflicts systematically
  - Task 3: Complete rebase and verify

- Plan 2: Post-sync fixes (3 tasks)
  - Task 1: Update plugin.json registry
  - Task 2: Fix hardcoded paths
  - Task 3: Verify and force push

Each plan creates atomic commits, followed by summary documentation.
</output>

<usage>
**Generate sync plan:**
```
/maintain-fork
```

This creates a GSD phase plan following proper patterns. Review the generated plan, then execute it.
</usage>

<notes>
- This command uses /gsd:plan-phase (GSD's built-in planning workflow)
- Creates executable phase plans with XML task structure
- Plans can be executed with /gsd:execute-plan or /gsd:execute-phase
- Follows GSD's 2-3 tasks per plan guidance for quality
- Each task commits atomically for clean git history
- Always review generated plan before execution
- Backup your work before starting rebase (git reflog provides safety)
</notes>

<context>
**Known fork-specific modifications to preserve:**
- `.claude-plugin/` directory (removed in upstream, required for marketplace)
- `marketplace.json` with willist references
- All `${CLAUDE_PLUGIN_ROOT}` path replacements (not `@~/.claude/get-shit-done/`)
- `README.md` marketplace installation instructions
- `package.json` repository URL pointing to willist fork

**Known conflict patterns:**
1. Upstream removes `.claude-plugin/` files - always keep your versions
2. Upstream adds new commands with hardcoded paths - must convert to `${CLAUDE_PLUGIN_ROOT}`
3. Upstream modifies shared files - preserve willist references
</context>

<output>
Create PLAN.md with the following structure:

# Sync get-shit-done with Upstream

## Overview
Sync local fork with glittercowboy/get-shit-done upstream while preserving marketplace compatibility and willist-specific modifications.

## Pre-sync Checklist
- [ ] Commit any uncommitted changes
- [ ] Verify upstream remote is configured: `git remote -v | grep upstream`
- [ ] Note current commit: `git log -1 --oneline`

## Phase 1: Fetch and Analyze

### 1.1 Fetch upstream changes
```bash
git fetch upstream
```

### 1.2 Review upstream commits
```bash
git log --oneline main..upstream/main | head -20
```

### 1.3 Identify local commits
```bash
git log --oneline upstream/main..main
```

### 1.4 Check for potential conflicts
```bash
git diff --name-only main upstream/main
```

## Phase 2: Rebase

### 2.1 Start rebase
```bash
git rebase upstream/main
```

### 2.2 Handle conflicts (if any)

**If .claude-plugin files are deleted:**
```bash
# Keep your versions
git add .claude-plugin/plugin.json
git add .claude-plugin/marketplace.json
git rebase --continue
```

**If other files have conflicts:**
- Review both versions carefully
- Preserve willist-specific modifications
- Keep `${CLAUDE_PLUGIN_ROOT}` path patterns
- Test the resolution before continuing

## Phase 3: Post-Rebase Verification

### 3.1 Verify rebase completed
```bash
git status
git log --oneline -10
```

### 3.2 Check for hardcoded paths
```bash
# Find any that need fixing
grep -r "@~/.claude/get-shit-done/" commands/
```

**If found, replace with:**
```bash
# In each file, replace:
@~/.claude/get-shit-done/
# with:
@${CLAUDE_PLUGIN_ROOT}/get-shit-done/
```

### 3.3 Check plugin registry is complete
```bash
# List all command files
ls -1 commands/gsd/*.md | sort

# Compare with plugin.json
grep '"commands"' -A 100 .claude-plugin/plugin.json
```

**If commands are missing from plugin.json, add them:**
- New commands from upstream need to be registered
- Maintain alphabetical ordering within logical groups

### 3.4 Verify fork-specific files
```bash
# Check marketplace.json has willist
grep "willist" .claude-plugin/marketplace.json

# Check README has marketplace install section
grep -A5 "Marketplace Installation" README.md

# Check package.json has willist repo
grep "willist" package.json
```

## Phase 4: Test and Commit

### 4.1 Test the plugin
```bash
# Verify command structure
ls -la .claude-plugin/
cat .claude-plugin/plugin.json | jq .commands.length

# Spot check a few command files
head -20 commands/gsd/new-project.md
```

### 4.2 Commit any fixes
```bash
git add .
git commit -m "fix: preserve fork-specific modifications after upstream sync"
```

### 4.3 Force push (if confident)
```bash
git push origin main --force-with-lease
```

## Known Issues and Solutions

### Issue: .claude-plugin/ conflicts
**Cause:** Upstream removed this directory, fork requires it for marketplace
**Solution:** Always `git add` the conflicting files to keep your versions

### Issue: Hardcoded paths in new commands
**Cause:** Upstream adds commands using `@~/.claude/get-shit-done/`
**Solution:** Search and replace with `@${CLAUDE_PLUGIN_ROOT}/get-shit-done/`

### Issue: Missing commands in plugin.json
**Cause:** Upstream adds new commands without updating plugin registry
**Solution:** Add new commands to plugin.json in appropriate location

### Issue: README.md installation sections
**Cause:** Upstream may not have marketplace install instructions
**Solution:** Preserve the marketplace installation section pointing to willist/get-shit-done

## Rollback Plan

If rebase fails critically:
```bash
git rebase --abort
```

If you've already pushed and need to rollback:
```bash
git reflog expire --expire=now --all
git gc --prune=now --aggressive
# Then restore from backup or re-clone
```

## Success Criteria
- [ ] All upstream commits incorporated
- [ ] .claude-plugin/ directory intact
- [ ] All paths use `${CLAUDE_PLUGIN_ROOT}`
- [ ] plugin.json registers all commands
- [ ] README.md has marketplace install instructions
- [ ] marketplace.json references willist
- [ ] No hardcoded `@~/.claude/get-shit-done/` paths remain
- [ ] Force push completed successfully
</output>

<usage>
**Generate sync plan:**
```
/sync-upstream-plan
```

This creates PLAN.md in the current directory. Review it, then follow each step in sequence.
</usage>

<notes>
- This command creates a plan but does NOT execute it
- Always review the generated plan before proceeding
- Backup your work before starting a rebase
- Use `--force-with-lease` not `--force` when pushing
- Test the plugin after syncing before considering it complete
</notes>
