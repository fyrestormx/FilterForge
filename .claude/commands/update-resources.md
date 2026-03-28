---
name: update-resources
description: Check all external data sources for updates and sync local data. Covers community filter authors, staffmod data, item/skill databases, and upstream skills.
user_invocable: true
argument-hint: "[--dry-run] [--source=name]"
---

# Update Resources

Check all external data sources that FilterForge depends on and update local data where changes are detected.

## Data Sources

### 1. PD2 Official Filter Authors (PRIMARY)

**Source:** `https://raw.githubusercontent.com/Project-Diablo-2/LootFilters/main/filters.json`
**Local:** `data/filters.json`
**What to check:** New filter authors added to the official PD2 LootFilters list that aren't in our local copy.
**How to update:** Fetch the upstream JSON, compare author list, add any new entries to `data/filters.json`. For new authors, also scan their repo for `.filter` files and add entries to `data/author-filters.json`.

### 2. Community Filter Author Repos

**Source:** Each author's GitHub repo (listed in `data/author-filters.json`)
**Local:** `data/author-filters.json` and hardcoded URLs in `js/editor.js` (template loader ~line 3750+)
**What to check:** New/renamed/removed `.filter` files in each author's repo.
**How to update:**
- For each author in `data/author-filters.json`, use `gh api repos/{owner}/{repo}/contents` to list current `.filter` files
- Compare against stored file list
- Add new files, remove deleted files, update sizes
- Also update the hardcoded template URLs in `js/editor.js` if files changed

### 3. HiimFilter Staffmod Data

**Source:** `https://raw.githubusercontent.com/Maaaaaarrk/HiimFilter-PD2-Filter/main/builderfilter/08-nmag/26-Nmag_Staffmod%5BALL-EXCEPT%3DOnlyFilter%5D.filter`
**Local:** Staffmod skill arrays in `js/editor.js` (inside `buildFilter()`, search for `var aznSkills`, `var sorSkills`, etc.)
**What to check:** New skills added, skill IDs changed, tier changes.
**How to update:** Download the upstream file, parse the ItemDisplay rules, compare skill IDs and names against the local arrays. Flag any differences.

### 4. PD2 Wiki — Item Filtering Reference

**Source:** `https://wiki.projectdiablo2.com/wiki/Item_Filtering`
**Local:** Various constants in `js/editor.js`:
  - `KNOWN_FLAGS` array (condition flags like UNI, SET, RARE...)
  - `KNOWN_VALUE_CODES` array (SOCKETS, ILVL, RUNE...)
  - `KNOWN_OUTPUT_TOKENS` array (NAME, RUNENAME, colors...)
  - `D2_COLORS` object (color token to hex mapping)
  - `SKILL_DATA` object (class skill codes and names)
**What to check:** New condition flags, new output tokens, new color codes, renamed skills.
**How to update:** Fetch the wiki page, parse the token tables, diff against local arrays. This is best done manually with agent guidance since wiki HTML parsing is fragile.

### 5. UI/UX Pro Max Skill (upstream)

**Source:** `https://github.com/nextlevelbuilder/ui-ux-pro-max-skill`
**Local:** `.claude/commands/uiuxreviewthis.md`
**What to check:** New checklist items, updated design rules, new anti-patterns.
**How to update:** Fetch the upstream SKILL.md, diff against local skill. Present changes to user before applying.

## Workflow

### Step 1: Check each source

For each data source, fetch the upstream version and compare against local. Use `gh api` for GitHub repos and `curl` for raw files. Collect all differences.

### Step 2: Report findings

Present a summary table:

```
## Resource Update Report

| Source | Status | Changes |
|--------|--------|---------|
| PD2 Filter Authors | 2 new authors | NewAuthor1, NewAuthor2 |
| Wolfie's repo | 1 new file | hardcore.filter |
| HiimFilter staffmod | No changes | — |
| PD2 Wiki tokens | 3 new flags | YOURFLAG1, YOURFLAG2 |
| UI/UX Skill | Updated | 5 new checklist items |
```

### Step 3: Ask before updating

Present options:
1. Update all sources
2. Update specific source(s)
3. Dry run only (just show the report)

If `--dry-run` was passed as argument, stop after the report.
If `--source=name` was passed, only check that source.

### Step 4: Apply updates

For each approved update:
1. Modify the local file(s)
2. Commit with message: `Update resources: [source name]`

Push only after all updates are committed.

## Source Name Reference (for --source flag)

- `authors` — PD2 Official Filter Authors (#1)
- `repos` — Community Filter Author Repos (#2)
- `staffmods` — HiimFilter Staffmod Data (#3)
- `wiki` — PD2 Wiki tokens/flags (#4)
- `uiux` — UI/UX Pro Max Skill (#5)

## Key File Locations

- `data/filters.json` — Author list with GitHub API URLs
- `data/author-filters.json` — Author files with raw download URLs and sizes
- `js/editor.js` — Template loader URLs (~3750), staffmod arrays (~2700), token arrays (~770), skill data (~510), D2 colors (~358)
- `.claude/commands/uiuxreviewthis.md` — UI/UX review skill