# Skill Description Audit

技能描述稽核 SOP — 提升 Hermes 技能描述的觸發準確率。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Hermes Agent](https://img.shields.io/badge/Hermes-1.0+-blue)

## 功能說明

Hermes 使用技能描述作為**唯一**的信號來決定載入哪個技能。
描述太弱 = 載錯技能 = 工作流斷裂。本技能提供完整的 SOP，用於稽核、修復、維護整個 Hermes 技能庫的描述。

## 快速開始

```bash
# 從這個 repo 安裝
hermes skills install "https://raw.githubusercontent.com/lunkerchen/skill-description-audit-skill/main/SKILL.md" --yes

# 或在本機載入
skill_view(name='skill-description-audit')
```

## 工作流程

SOP 涵蓋五個步驟：

1. **盤點** — 掃描所有技能，依字元數標記空值/弱描述
2. **修復空值** — 使用標準格式重寫遺漏的描述
3. **擴展弱描述** — 新增觸發詞、範圍邊界、關聯技能
4. **超短指令映射** — 將 1-3 字快捷指令映射到技能（透過 `ask-laban`）
5. **驗證** — 檢查長度限制、衝突、雙語覆蓋

完整 SOP 見 [`SKILL.md`](./SKILL.md)。

## 使用時機

| 情境 | 動作 |
|---|---|
| 新增技能 | 先寫描述，再寫內容 |
| 技能不觸發 | 稽核並擴展觸發詞 |
| 季度維護 | 完整盤點 + 衝突檢查 |
| 合併外部技能前 | 移除個人引用，泛化內容 |
| 使用者修正錯誤匹配後 | 將修正記錄為觸發詞 |

## 核心概念

### 描述格式

```
<是什麼> — <核心功能>。<如何運作>。適用情境：<觸發場景>。
```

### 品質閾值

| 評分 | 標準 | 動作 |
|---|---|---|
| ✅ 良好 | 150-400 字元，有觸發詞，雙語 | 可發布 |
| ⚠️ 需改善 | 50-150 字元，缺少場景 | 擴展 |
| ❌ 破損 | < 50 字元或空白 | 立即重寫 |

### 邊緣情況處理

- **衝突技能** — 在較窄的技能描述中加入消歧詞
- **重疊領域** — 明確標示階段邊界（例如：設計文件 vs 實作文件）
- **視角技能** — 短描述可接受（不是路由器）

## 自動化

### 批次掃描

```bash
find ~/.hermes/skills -name 'SKILL.md' -not -path '*/node_modules/*' | while read f; do
  name=$(grep '^name:' "$f" | head -1 | sed 's/name.*: *//;s/"//g;s/ //')
  desc=$(sed -n '/^description:/,/^[^ ]/p' "$f" | head -1 | sed 's/description.*: *//;s/^"//;s/"$//')
  echo "${name}: ${#desc} chars"
done | sort -t: -k2 -n
```

### 維護時程

- **每次新增技能** — 先寫描述再寫內容
- **每月** — 掃描新的空值/弱描述
- **每季** — 完整盤點 + 衝突檢查
- **使用者修正後** — 記錄修正的觸發詞
- **發布外部前** — 移除個人引用，泛化內容

## 相關技能

- [`ask-laban`](https://github.com/lunkerchen/hermes-agent) — 路由器技能。超短指令解碼器在此。
- [`skill-management`](https://github.com/lunkerchen/hermes-agent) — 匯入/匯出技能、發布到 GitHub、鎖定檔管理。
- [`hermes-agent`](https://github.com/lunkerchen/hermes-agent) — Hermes Agent 完整參考。

## 授權

MIT
