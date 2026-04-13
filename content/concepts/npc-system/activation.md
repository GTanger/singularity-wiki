---
title: 活化系統——突破線 A–I
tags: [npc, activation, implementation, milestones]
sources:
  - docs/implementation/NPC生成流程調整—依討論001.md
  - docs/implementation/NPC對話記憶與背版—設計.md
  - docs/implementation/NPC對話記憶與背版—實作步驟與檔案流程.md
  - docs/reference/記憶系統對照NPC活化系統.md
date: 2026-04-11
---

# 活化系統——突破線 A–I

NPC 活化系統以「突破線」為里程碑，每道突破線對應一次可觀測的行為躍升。

## 突破線一覽

| 線 | 名稱 | 核心效果 | 狀態 |
|---|------|---------|:----:|
| **A** | 鎂消耗 | NPC 每遊戲日 -8 鎂，馬斯洛鏈激活 | ✅ |
| **B** | Brain 停留 | 到達目的地後停留，不亂飛 | ✅ |
| **C** | 性格偏移決策 | SoulSeed 三軸影響意圖選擇 | ✅ |
| **D** | NPC 事件日誌 | 每次行為寫入事件 | ✅ |
| **E** | Disposition 心境 | 情緒值影響閒置文本口吻 | ✅ |
| **F** | NPC 間互動 | 微互動 + AI 對話 + 主題劇本 + 記憶四層 | ✅ |
| **G** | 背版組裝 | Talk 前 BuildIdentity | ✅ |
| **H** | Archival 記憶 | NPC 記住跟玩家的過去 | ✅ |
| **I** | CallAITalk 接入 | LLM 優先 + 模板 fallback | ✅ |

> A-C = 齒輪（決策引擎運轉）→ D-F = 模板突破線（NPC 有故事）→ G-I = 記憶層（故事在 Talk 時說出來）

## 突破線詳述

### A — 鎂消耗（`npc_expense.rs`）
- `deduct_daily_expense()` 每遊戲日扣 8 鎂
- 低於閾值（50 鎂）觸發生存緊迫度

### B — Brain 停留（`brain_arrival.rs`）
- 到達目的地後依行動類型套用到達效果
- 停留時間由計時器控制

### C — 性格偏移（`decision.rs` V2）
- `decide_with_inertia()` — 當前意圖 1.8× 加權優勢
- `fallback_intents()` — 逐級降級（SeekJob → Beg → Wander）
- 三軸人格影響意圖權重

### D — 事件日誌（`event/`）
- 每次行為變化寫入事件記錄

### E — Disposition 心境（`disposition.rs`）
- 事件觸發加減值；時段微調：黎明 +1、深夜 -1

### F — NPC 間互動
- 微互動 15% 機率、AI 對話、主題劇本、記憶四層

### G — 背版組裝（`db/backstory`）
`BuildIdentity(npc_id)` 組裝：identity / summary / relationship 三個 Block

### H — Archival 記憶
- 對話結束後壓成 1-3 條記憶條目
- 每 NPC 100 條上限
- 零命中時不 fallback 到無關最新條

### I — CallAITalk 接入
```
玩家 Talk → use_ai_for_talk → 是：CallAITalk → 否/失敗：PickFromDialogue
```

## 實作狀態總覽

| 類別 | 已實作 | 待實作 |
|------|--------|--------|
| 移動系統 | ✅ 四種模式、BFS | ⬜ 聚念場影響移動 |
| 排班系統 | ✅ GetScheduleTarget | ⬜ 離職/解僱/流動 |
| 決策引擎 | ✅ V1+V2 | ⬜ V3 目標堆疊 |
| 對話系統 | ✅ LLM + 模板 + NPC-NPC | ⬜ 多輪 NPC-NPC |
| 記憶系統 | ✅ 背版 + archival + 四層 | ⬜ 語意搜索（embedding） |
| 語意檢索 | ⬜ 目前為多關鍵字評分 | → pgvector + bge-m3 |

## 相關頁面

- [[npc-system/decision-engine]] — 決策引擎詳細設計
- [[npc-system/memory]] — 記憶系統四層架構
- [[npc-system/dialogue]] — 對話系統
- [[npc-system/interaction]] — NPC 間交互行為