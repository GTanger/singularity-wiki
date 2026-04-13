---
title: 對話鍛造爐
tags: [npc, dialogue, llm, forge, world-lexicon]
sources:
  - docs/dialogue_forge_plan.md
  - docs/dialogue_forge_walkthrough.md
date: 2026-04-11
---

# 對話鍛造爐

**對話鍛造爐**是讓 NPC 用自然行為產生遊戲文字資產的三層漏斗架構。

## 設計哲學

「**世界是有人說話說出來的，不是設計師填出來的。**」

NPC 聊天 → 產出對白 → 好的詞彙沉澱為世界語言 → 世界語言反哺後續對話。正回饋迴圈。

## 三層漏斗架構

```
第一層（0.8B 話癆）：大量、廉價、快速
   ↓ 品質篩選
第二層（4B 日報蒸餾器）：精煉、壓縮、每日
   ↓ 人工提名
第三層（雲端萃取）：定案、進入設定集
```

### 第一層：0.8B 話癆（✅ 已實作）
- 模型：`sorc/qwen3.5-instruct:0.8b`
- 品質門檻：降至 25 分
- 產出大量未篩選對白

### 第二層：4B 日報蒸餾器（⬜ 待實作）
- 每遊戲日產出 `daily_digest`
- pgvector embedding 語意存放

### 第三層：雲端萃取（⬜ 待實作）
- `world_lexicon` 累積 ≥20 個被提名詞彙時觸發
- 正式收入設定集

## Phase 1 實作

| 檔案 | 改動 |
|------|------|
| `server_defaults.json` | model 改為 0.8b；threshold 改為 25 |
| `simulation.json` | tick 加速 |
| `data/event_seeds.json` | 30 個事件種子 |
| `src/npcnpc/lexicon.rs` | 世界詞典寫入 |
| `src/npcnpc/trigger.rs` | 對話觸發器 |
| PostgreSQL | `world_lexicon` 表 |

### world_lexicon 表

```sql
CREATE TABLE world_lexicon (
  id SERIAL PRIMARY KEY,
  term TEXT NOT NULL,
  context TEXT,
  source_npc_id TEXT,
  source_room_id TEXT,
  frequency INT DEFAULT 1,
  nominated BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## 品質門檻對照

| 用途 | 門檻 |
|------|------|
| 玩家↔NPC Talk | ≥35 分 |
| 對話鍛造爐 Phase 1 | ≥25 分 |
| ScoreNpcDialogue 滿分 | 80 分 |

## 相關頁面

- [[npc-system/dialogue]] — 對話系統總覽
- [[npc-system/interaction]] — NPC 間交互行為
- [[npc-system/memory]] — 記憶四層模型