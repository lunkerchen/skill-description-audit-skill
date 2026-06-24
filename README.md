# Skill Description Audit

Audit and improve Hermes skill descriptions to maximize trigger accuracy.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Hermes Agent](https://img.shields.io/badge/Hermes-1.0+-blue)

[繁體中文說明](./README.zh-TW.md)

## What It Does

Hermes uses skill descriptions as the **only** signal to decide which skill to load.
A weak description = wrong skill = broken workflow. This skill provides a complete
SOP for auditing, fixing, and maintaining skill descriptions across the entire
Hermes skill library.

## Quick Start

```bash
# Install from this repo
hermes skills install "https://raw.githubusercontent.com/lunkerchen/skill-description-audit-skill/main/SKILL.md" --yes

# Or load locally
skill_view(name='skill-description-audit')
```

## Workflow

The SOP covers five steps:

1. **Inventory** — Scan all skills, flag empty/weak descriptions by char count
2. **Fix Empty** — Rewrite missing descriptions using the standard format
3. **Expand Weak** — Add trigger words, scope boundaries, related skills
4. **Ultra-Short Mapping** — Map 1-3 word shortcuts to skills (via `ask-laban`)
5. **Validate** — Check length limits, conflicts, bilingual coverage

See the full SOP in [`SKILL.md`](./SKILL.md).

## When to Use

| Scenario | Action |
|---|---|
| Adding a new skill | Write description first, body second |
| Skill not triggering | Audit and expand trigger words |
| Quarterly maintenance | Full inventory scan |
| Before merging external skills | Strip personal refs, generalize |
| After user corrects a match | Log the correction as a trigger word |

## Key Concepts

### Description Format

```
<What it is> — <Core capability>. <How it works>. Use when <trigger scenarios>.
```

### Quality Thresholds

| Score | Criteria | Action |
|---|---|---|
| ✅ Good | 150-400 chars, trigger words, bilingual | Ship |
| ⚠️ Needs work | 50-150 chars, missing scenarios | Expand |
| ❌ Broken | < 50 chars or empty | Rewrite |

### Edge Case Handling

- **Conflicting skills** — Add disambiguating terms to the narrower skill
- **Overlapping domains** — Make stage boundaries explicit (e.g., design doc vs implementation doc)
- **Perspective skills** — Short descriptions are acceptable (not routers)

## Automation

### Bulk Scan

```bash
find ~/.hermes/skills -name 'SKILL.md' -not -path '*/node_modules/*' | while read f; do
  name=$(grep '^name:' "$f" | head -1 | sed 's/name.*: *//;s/"//g;s/ //')
  desc=$(sed -n '/^description:/,/^[^ ]/p' "$f" | head -1 | sed 's/description.*: *//;s/^"//;s/"$//')
  echo "${name}: ${#desc} chars"
done | sort -t: -k2 -n
```

### Maintenance Schedule

- **Every new skill** — Write description before body
- **Monthly** — Scan for new empty/weak descriptions
- **Quarterly** — Full inventory + conflict check
- **After user correction** — Log corrected trigger word
- **Before external publish** — Strip personal refs, generalize

## Related Skills

- [`ask-laban`](https://github.com/lunkerchen/hermes-agent) — Router skill. Ultra-short command decoder lives here.
- [`skill-management`](https://github.com/lunkerchen/hermes-agent) — Import/export skills, publish to GitHub, lock-file management.
- [`hermes-agent`](https://github.com/lunkerchen/hermes-agent) — Complete Hermes Agent reference.

## License

MIT
