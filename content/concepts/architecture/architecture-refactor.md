---
title: 架構整頓紀錄（ADR 008）
tags: [architecture, refactor, modules, postgresql]
sources:
  - decisions/008_架構整頓規劃.md
date: 2026-04-11
---

# 架構整頓紀錄（ADR 008）

**整頓完成時間：2026-02-12**

## 整頓動機

1. `db/` 成為 god-package
2. `server/handler` 為單一巨型檔案
3. SQLite 與 PostgreSQL 共存
4. Go 代碼殘留

## 整頓結果：模組結構

```
src/main.rs              — 入口
src/lib.rs               — 模組宣告 + run_server()
src/ai/                  — LLM 呼叫
src/bin/                 — 獨立工具
src/config/              — 可調參數
src/store/               — 資料層（PostgreSQL + HashMap 快取）
src/db/                  — 資料存取介面
src/entity/              — 角色實體、枚舉型別
src/model/               — Room、Exit 等共用結構
src/game/                — 遊戲時間、視野
src/economy/             — 經濟引擎
src/world/               — 地圖格點、移動
src/npc/                 — NPC 行為
src/npcnpc/              — NPC↔NPC 對話
src/combat/              — 戰鬥系統
src/event/               — 事件常數與紀錄
src/gametext/            — 文案模板
src/server/              — axum HTTP/WebSocket、session、simulation loop
```

## 關鍵拆分決策

### db/ god-package 拆分
- NPC CRUD → `src/npc/`
- AI 呼叫 → `src/ai/`
- 對話池 → `src/db/dialogue`
- 背版/記憶 → `src/db/backstory`

現行 `src/db/` 職責：**純資料存取介面**，不含業務邏輯。

### server/handler 拆分
舊單一 handler → 現行 **~25 個葉子檔案**

### SQLite 完全移除
無遺留、無雙軌並存期。

## 整頓後不得回頭的約束

1. 不得重新引入 god-package
2. 不得引入新的全域 init()
3. 不得繞過 store 直接讀寫 DB
4. JSON 不得成為執行期資料源

## 相關頁面

- [[tech-stack]] — 技術選型背景
- [[implementation-pipeline]] — 模組結構後的開發流程
- [[npc-system/decision-engine]] — npc/ 子模組詳情