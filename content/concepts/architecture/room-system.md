---
title: 房間系統——格點＝視野
tags: [architecture, room, cell, view, player, login]
sources:
  - decisions/005_room_cell_view_scope.md
  - decisions/006_login_and_player_template.md
  - docs/reference/玩家視角—NPC與房間互動.md
  - docs/reference/房間與建築設計規範.md
date: 2026-04-11
---

# 房間系統——格點＝視野

## 核心定義（ADR 005）

**一個格點 = 一個房間 = 一條視野線。**

- 無半格、無跨房間視野
- 玩家能看到的就是「當前房間內的一切」
- 移動 = 進入相鄰格點 = 視野切換

不得引入「距離視野」、「地形視距」等修正機制。視野邊界就是房間邊界。

## 玩家與 NPC 的統一模型（ADR 006）

**玩家 = NPC**，共用相同的 `entities` 資料結構。以 `kind` 欄位區分。

### 登入機制
- 憑證：id + password（bcrypt 雜湊）
- 資料表：`entity_auth`

### 建立角色流程
1. 前端送 `create_character` (name, password, background)
2. 後端建立 `entities` 記錄（`kind = "player"`）
3. 建立 `entity_auth` 記錄（bcrypt hash）
4. 返回 session token

## 玩家視角 UX 規格

### room_view 訊息結構
```json
{
  "type": "room_view",
  "room": { "id": "...", "name": "...", "desc": "...", "zone": "..." },
  "entities": [{ "id": "...", "name": "...", "kind": "npc", "activity": "..." }],
  "exits": { "north": "room_id_xxx", "east": "room_id_yyy" },
  "objects": [...]
}
```

### 互動流程
1. 玩家進入房間 → 收到 `room_view`
2. 點擊 NPC → Look（顯示外觀描述）
3. Log 下一行顯示可用動作：Talk / Attack / Trade
4. 點 Talk → 多輪對話
5. Trade 觸發：直接 Trade 按鈕或 talk intent → `open_trade: true`

## 房間與建築設計規範

### 描述寫作規則
- 主體描述 **≥50 字**
- 「一句話畫一個場景」風格
- 禁止抽象主觀詞
- 必須使用感官可驗證的內容

### Zone 體系
- 每個房間屬於一個 zone
- Zone 用於 NPC 行為決策
- Zone 濾鏡：Room Editor 提供下拉選單過濾顯示

## 房間翻譯與描述生成（工單 017/019/020）

- 623 個城鎮名需翻譯為繁體中文
- 694 個房間需唯一描述
- 描述品質閘門：30–80 中文字、不含禁用詞、不含英文字母

## 相關頁面

- [[observation-system]] — 視野與觀測層級
- [[architecture-refactor]] — 模組結構
- [[npc-system/index]] — NPC 在房間內的行為