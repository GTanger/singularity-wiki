---
title: 觀測分級與行程約束
tags: [architecture, observation, npc, simulation, performance]
sources:
  - decisions/009_觀測分級與行程約束.md
  - discussions/005_觀測分級與行程約束_雲端與程式模擬.md
  - docs/implementation/觀測驅動與腦規模.md
date: 2026-04-11
---

# 觀測分級與行程約束

## 核心原則（ADR 009）

「**黑暗中，世界按規則活動；鏡頭抵達時，才開口說話。**」

| 層級 | 定義 | 允許的算力 |
|---|---|---|
| **0 — 未觀測** | 無玩家在附近 | 僅抽樣輕量效果（最多 50 NPC） |
| **1 — 同畫面** | 玩家在同一房間 | 移動 + 狀態更新 |
| **2 — 凝視** | 玩家主動 Look NPC | 行為描述、完整狀態 |
| **3 — 對話** | 玩家與 NPC 互動 | LLM 生成（AI Talk）|

GPU 只用於**第 3 層（對話）**。其餘層級不得呼叫 LLM。

## 單一世界時鐘

所有 NPC 共用同一遊戲時間軸。無個人時間線。

- Tick 錯開：NPC 初始化時隨機 0～3 小時延遲
- 行程約束（assignment）是**硬規則**，不是骰子修正
- 確定性比隨機性更重要

## 觀測圓

**觀測圓 = 玩家所在房間 ∪ 相鄰房間**

### 未觀測輕量模擬（`unobserved.rs`）

- `RunUnobservedWorldTick` 最多抽樣 50 NPC
- 有行程的 NPC：走一步朝目標前進
- 無行程的 NPC：隨機套用抽象效果
- 不跑敘事、不跑 LLM

## 行程約束規則

「**從外部觀察他人 = 機械性節律，無內在生命。這正好符合遊戲主題。**」

### 約束優先順序
1. Assignment（指派）> 意圖決策
2. Schedule（時段）> 隨機行為
3. 骰子只在無硬約束時才生效

## 算力配置策略

| 觸發條件 | 執行 |
|---|---|
| 玩家進入房間 | room_view 生成、NPC 列表 |
| 玩家 Look NPC | 完整狀態讀取 |
| 玩家發起 Talk | LLM 呼叫，concurrent |
| NPC-NPC 對話 | LLM 呼叫，**排隊** |

## 腦規模與觀測一致性

- 90,000 NPC 不主動模擬
- 觀測圓外的 NPC 靠事件日誌回溯重建狀態

## 相關頁面

- [[npc-system/decision-engine]] — 決策引擎詳細設計
- [[npc-system/observation]] — NPC 觀測層級與行為
- [[tech-stack]] — 算力資源約束背景