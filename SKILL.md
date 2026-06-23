---
name: skill-description-audit
description: >
  Audit and improve Hermes skill descriptions to maximize trigger accuracy.
  Use when adding new skills, noticing a skill isn't firing reliably, or doing
  periodic maintenance. Fixes empty descriptions, expands trigger words,
  ensures bilingual coverage (繁體中文 + English).
---

# Skill Description Audit SOP

## Purpose

Skill descriptions are the **only** signal the agent uses to decide which skill to load.
A weak description = wrong skill = wrong workflow. This SOP ensures every skill
has a description that fires reliably on Laban's actual language patterns.

## When to Run

- Adding a new skill (always)
- A skill isn't triggering when expected (reactive)
- Periodic maintenance (quarterly or after major skill additions)
- Before merging external skills (e.g. from reverse-skill, other agents)

## Step 1: Inventory

```bash
find ~/.hermes/skills -name 'SKILL.md' -not -path '*/node_modules/*' | while read f; do
  name=$(grep '^name:' "$f" | head -1 | sed 's/name.*: *//;s/"//g;s/ //')
  desc=$(sed -n '/^description:/,/^[^ ]/p' "$f" | head -1 | sed 's/description.*: *//;s/^"//;s/"$//')
  echo "${name}: ${#desc} chars"
done | sort -t: -k2 -n
```

Flag anything under 100 chars as **weak**. Anything under 20 chars as **empty**.

## Step 2: Fix Empty Descriptions

For each empty/missing description:

1. Read the SKILL.md body to understand what the skill does
2. Write a description in this format:

```
<What it is> — <Core capability>. <How it works>. Use when <trigger scenarios>.
```

Examples:
- `deep-research` → "Iterative web research with structured synthesis and source verification. Multi-round investigation, parallel workflows, STORM methodology. Use when user says 深入調查、深度研究、research this thoroughly, or needs to go beyond surface-level search results."
- `photography-quote-assistant` → "專業攝影報價助手。將拍攝時間、後期修圖、短片剪輯、設備費用轉化為結構化 HTML 報價單。Use when generating photography quotes or pricing estimates."

Rules:
- Start with the core function (what it does)
- Include 1-2 sentences on how it works
- End with explicit trigger scenarios (both 中文 and English)
- Target 150-300 chars

## Step 3: Expand Weak Descriptions (< 150 chars)

For skills with short descriptions, add:

1. **Trigger words** — What does Laban actually say? Include both Chinese and English variants
2. **Scope boundaries** — What it does NOT cover (prevents wrong matches)
3. **Related skills** — Which other skills to load alongside it

Format:
```
<existing description>. Use when [scenario 1], [scenario 2], [scenario 3]. Also triggers on: [synonyms].
```

## Step 4: Ultra-Short Command Mapping

Add to `ask-laban` skill's "Ultra-short command decoder" table when:

- Laban uses 1-3 word shortcuts that should map directly to a skill
- The shortcut is used 2+ times per month
- There's no ambiguity (one clear target skill)

Common patterns to watch for:
- 「報價」→ `photography-quote-assistant`
- 「PO 文」→ `social-post`
- 「做個網頁」→ `baoyu-design`
- 「查股票」→ `stock-technical-analysis`
- 「翻譯」→ `baoyu-translate`
- 「做支片」→ `hyperframes`

## Step 5: Validate

After changes, verify:

1. No description exceeds 1024 chars (YAML frontmatter limit)
2. No two competing skills have overlapping trigger words without disambiguation
3. Perspective skills (e.g. `steve-jobs-perspective`) can keep short descriptions — they're content generators, not routers
4. Empty descriptions in perspective/unused skills are acceptable — they rarely trigger

## Quick Reference: Description Quality Checklist

| Score | Criteria |
|---|---|
| ✅ Good | 150-400 chars, has trigger words, covers 中文+English |
| ⚠️ Needs work | 50-150 chars, missing trigger scenarios |
| ❌ Broken | < 50 chars or empty |

## Automation

For bulk fixes, use the Python helper:

```bash
# Run from ~/.hermes/skills/
python3 fix-weak-descriptions.py
```

This scans all SKILL.md files, finds empty/weak descriptions, and patches them.
