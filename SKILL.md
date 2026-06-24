---
name: skill-description-audit
description: >
  Audit and improve Hermes skill descriptions to maximize trigger accuracy.
  Use when adding new skills, noticing a skill isn't firing reliably, or doing
  periodic maintenance. Fixes empty descriptions, expands trigger words,
  ensures bilingual coverage (繁體中文 + English).
---

# Skill Description Audit SOP

Audit and improve Hermes skill descriptions so the agent reliably loads the right skill
on every request. Weak descriptions cause wrong-skill matches, wasted tokens, and broken
workflows.

## Why This Matters

Skill descriptions are the **only** signal the agent uses to decide which skill to load.
There is no fallback mechanism — if the description doesn't match the user's words,
the skill never fires.

A good description:
- Matches how the user **actually speaks** (not how you wish they spoke)
- Includes both 中文 and English trigger words
- Defines scope boundaries (what it does NOT do)
- Stays under 1024 chars (YAML frontmatter limit)

## When to Run

| Scenario | Action |
|---|---|
| Adding a new skill | Always — write description first, body second |
| Skill not triggering | Audit and expand trigger words |
| Quarterly maintenance | Full inventory scan |
| Before merging external skills | Strip personal refs, generalize |
| After user corrects a wrong skill match | Note the correction as a trigger word |

## Step 1: Inventory

Scan all skills and flag weak/empty descriptions:

```bash
find ~/.hermes/skills -name 'SKILL.md' -not -path '*/node_modules/*' | while read f; do
  name=$(grep '^name:' "$f" | head -1 | sed 's/name.*: *//;s/"//g;s/ //')
  desc=$(sed -n '/^description:/,/^[^ ]/p' "$f" | head -1 | sed 's/description.*: *//;s/^"//;s/"$//')
  echo "${name}: ${#desc} chars"
done | sort -t: -k2 -n
```

Classification:
- **Empty**: < 20 chars — needs immediate fix
- **Weak**: 20-100 chars — expand trigger words
- **Good**: 100-400 chars — review for completeness
- **Heavy**: > 400 chars — ok, but watch token cost when loaded

## Step 2: Fix Empty Descriptions

For each empty/missing description:

1. Read the SKILL.md body to understand what the skill does
2. Write a description in this format:

```
<What it is> — <Core capability>. <How it works>. Use when <trigger scenarios>.
```

**Examples:**

| Skill | Description |
|---|---|
| `deep-research` | "Iterative web research with structured synthesis and source verification. Multi-round investigation, parallel workflows, STORM methodology. Use when user says 深入調查、深度研究、research this thoroughly, or needs to go beyond surface-level search results." |
| `photography-quote-assistant` | "專業攝影報價助手。將拍攝時間、後期修圖、短片剪輯、設備費用轉化為結構化 HTML 報價單。Use when generating photography quotes or pricing estimates." |
| `stop-slop` | "去除 AI 寫作味 — 砍掉模板化廢話、套路句式、虛假強調，讓文字像人寫的。Scans for AI clichés and robotic cadence. Rewrites to sound naturally human. Use before ANY published content: Threads posts, articles, social media, emails, proposals." |

**Rules:**
- Start with the core function (what it does in one phrase)
- Include 1-2 sentences on how it works
- End with explicit trigger scenarios (both 中文 and English)
- Target 150-300 chars
- Never exceed 1024 chars (YAML frontmatter hard limit)

## Step 3: Expand Weak Descriptions (< 150 chars)

For skills with short descriptions, add three elements:

1. **Trigger words** — What does the user actually say? Include both Chinese and English variants, formal and colloquial
2. **Scope boundaries** — What it does NOT cover (prevents wrong matches)
3. **Related skills** — Which other skills to load alongside it

**Template:**
```
<existing description>. Use when [scenario 1], [scenario 2], [scenario 3]. 
Also triggers on: [synonyms]. Not for: [scope exclusion].
```

**Example transformation:**

Before: `"Generate project ideas via creative constraints."` (48 chars)

After: `"Generate project ideas via creative constraints. Covers brainstorming frameworks, constraint-based ideation, and idea evaluation matrices. Use when brainstorming new projects, generating creative concepts, or exploring product ideas with structured constraints."` (261 chars)

## Step 4: Ultra-Short Command Mapping

For frequently used 1-3 word shortcuts, add entries to the `ask-laban` skill's
"Ultra-short command decoder" table. Criteria:

- Used 2+ times per month
- One clear target skill (no ambiguity)
- Covers both 中文 and English variants

**Common patterns:**

| Input | Maps to |
|---|---|
| 「報價」「quote」「how much」 | `photography-quote-assistant` |
| 「PO 文」「發一篇」「寫貼文」 | `social-post` |
| 「做個網頁」「畫個頁面」 | `baoyu-design` |
| 「查股票」「看盤」 | `stock-technical-analysis` |
| 「翻譯」 | `baoyu-translate` |
| 「做支片」「轉影片」 | `hyperframes` |
| 「做個簡報」「PPT」 | `guizang-ppt-skill` |
| 「做個圖」「畫張圖」 | `baoyu-article-illustrator` |
| 「整理檔案」 | `file-organizer` |
| 「去掉 AI 味」 | `stop-slop` |

## Step 5: Validate

After changes, verify:

1. **Length** — No description exceeds 1024 chars (YAML frontmatter limit)
2. **Conflict check** — No two competing skills share trigger words without disambiguation
3. **Perspective exemption** — Perspective skills (e.g. `steve-jobs-perspective`) can keep short descriptions — they're content generators, not routers
4. **Unused skills** — Empty descriptions in perspective/unused skills are acceptable — they rarely trigger
5. **Bilingual coverage** — Core skills should have both 中文 and English trigger words

## Description Quality Checklist

| Score | Criteria | Action |
|---|---|---|
| ✅ Good | 150-400 chars, has trigger words, covers 中文+English | Ship |
| ⚠️ Needs work | 50-150 chars, missing trigger scenarios | Expand |
| ❌ Broken | < 50 chars or empty | Rewrite immediately |

## Edge Cases

### Conflicting Skills

When two skills could match the same trigger:

```
❌ BAD: Both "baoyu-translate" and "vocus-article-writing-sop" match "翻譯"
✅ GOOD: "baoyu-translate" matches "翻譯"; "vocus-article-writing-sop" matches "方格子文章改寫"
```

Add disambiguating terms to the narrower skill's description.

### Overlapping Domains

Some skills share a domain but serve different stages:

```
idea-to-design-doc     → 想法 → 設計文件（不寫 code）
idea-to-implementation-doc → 設計文件 → 實作計畫
```

Make the stage boundary explicit in each description.

### Perspective Skills

Perspective skills (e.g. `steve-jobs-perspective`, `howard-marks-perspective`)
are content generators, not routers. They can keep short descriptions since
they're triggered by explicit name reference, not keyword matching.

## Automation

### Bulk Scan

```bash
# List all skills sorted by description length (shortest first)
find ~/.hermes/skills -name 'SKILL.md' -not -path '*/node_modules/*' | while read f; do
  name=$(grep '^name:' "$f" | head -1 | sed 's/name.*: *//;s/"//g;s/ //')
  desc=$(sed -n '/^description:/,/^[^ ]/p' "$f" | head -1 | sed 's/description.*: *//;s/^"//;s/\"$//')
  echo "${name}: ${#desc} chars"
done | sort -t: -k2 -n | head -30
```

### Bulk Fix Script

For mass updates, use a Python helper that patches descriptions in-place:

```python
# Pseudocode — adapt to your needs
for skill_file in find_skill_files():
    desc = read_description(skill_file)
    if len(desc) < 100:
        new_desc = improve_description(skill_file.name, desc)
        write_description(skill_file, new_desc)
```

## Maintenance Schedule

| Frequency | Task |
|---|---|
| Every new skill | Write description before body |
| Monthly | Scan for new empty/weak descriptions |
| Quarterly | Full inventory + conflict check |
| After user correction | Log corrected trigger word |
| Before external publish | Strip personal refs, generalize |

## Related Skills

- `ask-laban` — Router skill that maps vague requests to specific skills. Ultra-short command decoder lives here.
- `skill-management` — Import/export skills, publish to GitHub, lock-file management
- `hermes-agent` — Complete Hermes Agent reference for skill configuration
