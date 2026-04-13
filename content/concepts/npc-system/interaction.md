---
title: NPC 間交互行為
tags: [npc, interaction, social, npc-npc, dialogue]
sources:
  - docs/reference/NPC之間交互行為.md
  - docs/discussions/003_NPC交互對話系統.md
  - docs/discussions/004_Web_LLM_調度與複數玩家.md
date: 2026-04-11
---

# NPC 間交互行為

NPC 間的互動分為三個層次：**微互動**（模板句）、**AI 對話**（LLM 生成）、**主題劇本**（結構化話題）。

## 微互動（Micro-interaction）

- **觸發條件**：同房 ≥2 NPC 時，每次 tick 有 **15% 機率**觸發
- **模板庫**：`db/npc_social`，8 條基礎句
- **效果**：對話者心境 +1

## AI 對話（CallAITalkNPCToNPC）

### 觸發方式（三層）

| 權重 | 觸發條件 | 說明 |
|------|---------|------|
| **重** | 閒置 tick 到期 | 有玩家在的房間 |
| **中** | 排班事件 | 換班/交接 |
| **輕** | 隨機 | 補足世界背景感 |

### 配對分數公式

```
score(A, B) =
  + 100  若 npc_thread 活躍
  + familiarity / 20
  + 3    若同職場
  - 20   若近 120 秒內剛說過
  + rand(0, 3)
```

### 玩家優先規則

**60 秒內有 Talk 的房不觸發 NPC 間對話。**

### 玩家餘音（Echo）

同房玩家 Talk 後，節錄對白暫存。NPC 閒聊觸發時（Talk 後逾 60 秒、4 分鐘內），「餘音」注入 NPC↔NPC prompt。

## 主題劇本（npc_to_npc_topics）

- `data/npc_to_npc_topics.json`
- 結構：topic_id / participants_filter / opening_hint / follow_up_hints

## NpcNpcSummaries

NPC 間對話的摘要記憶，儲存於 `npc_npc_summaries` 表。

## 後台 NPC-NPC 對話現況

**架構保留，觸發時機待重新設計。** GPU 優先給有玩家的房間。

## 相關頁面

- [[npc-system/dialogue]] — 玩家↔NPC 對話系統
- [[npc-system/memory]] — 記憶層
- [[npc-system/dialogue-forge]] — 對話鍛造爐
- [[architecture/observation-system]] — GPU 算力配置原則