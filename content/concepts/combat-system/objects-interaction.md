---
title: 物件互動 — 房間非人物件系統
tags:
  - system
  - combat
  - interaction
  - ui
  - singularity-world
sources:
  - docs/房間非人物件互動.md
date: 2026-04-11
---

# 物件互動 — 房間非人物件系統

> 讓房間中的非人物件成為可互動的一等公民，與角色共享同一套插頭插座語義。

## 核心原則

- **描述即物件**：房間描述中 `〔物件名〕` 以琥珀色可點擊標記
- **插頭插座統一**：非人物件與角色使用相同 `do_action` 流程
- **歸屬權真實**：有主之物受保護，偷竊有社會後果

## 標記語法

| 標記 | 用途 | 樣式 |
|------|------|------|
| `【名字】` | 人物名（已有） | 綠色 `.desc-highlight` |
| `〔物件名〕` | 可互動物件 | 琥珀色 `.desc-object` |

`〔〕` 內文字須與該房 `objects[].name` 一致。

## 互動流程

點擊 `〔物件〕` → 前端送 `Look` → Log 顯示觀看敘事 → 下一行 append 其他可用動作（【閱讀】【嗅聞】等）→ 點擊再送對應 `do_action`。

### 導航分流

| 物件類型 | 點擊行為 |
|----------|----------|
| 巷道/路段（〔大街三段〕） | 直接 Move |
| 建築/商鋪（〔焦黑木門〕） | 先 Look，Log 再顯示【進入】 |

## 物件定義格式

物件寫在各房間 JSON 的 `objects` 欄位（`data/rooms/`）：

```json
{
  "id": "life_hall_plaque",
  "name": "匾額",
  "owner": "inn",
  "sockets": ["Look", "Read"],
  "responses": {
    "Look": "匾額掛在櫃台正上方...",
    "Read": "匾上以行草寫著【浮生】二字..."
  }
}
```

### 動詞（插座）列表

| 動詞 | 說明 | 典型目標 |
|------|------|---------|
| Look | 端詳外觀 | 所有物件 |
| Move | 移動/進入 | 門、通道、出口 |
| Read | 閱讀文字 | 匾額、佈告欄、書籍 |
| Use | 操作 | 灶台、井、織布機 |
| Open | 開啟 | 箱子、門、抽屜 |
| Unlock | 解鎖 | 上鎖的門/物件 |
| BreakLock | 破壞鎖 | 上鎖物件（可能有聲響） |
| Sit | 坐下 | 椅子、蒲團 |
| Smell | 嗅聞 | 香爐、酒甕 |
| Taste | 品嚐 | 酒甕、食物 |
| Take | 拾取 | 野花、草藥、掉落物 |
| Chop | 砍伐 | 樹木、柴堆 |
| Operate | 操作機關 | 暗門、絞盤 |

## 歸屬權模型

```
歸屬權
├── 無主（owner: ""）→ 任何人自由互動
├── 有主（owner: "inn" / "npc_id"）
│   ├── Look/Read/Smell → 不受限
│   ├── 操作性動詞（Use/Open/Take）→ 需檢查權限
│   │   ├── 看守者在場 → 被阻止
│   │   ├── 無看守、有人在場 → 被目擊
│   │   └── 四下無人 → 偷用成功
│   └── 可租借（owner_rentable）
└── 玩家所有 → 未來擴充
```

### 偷竊後果

| 情境 | 後果 |
|------|------|
| 四下無人，偷用成功 | 事件記入日誌；物主定期巡查 |
| 有人目擊 | 日誌 + 目擊者記憶；NPC 可能當場反應 |
| 看守者阻止 | 無實質後果；頻繁嘗試降好感 |

## 有狀態物件

部分物件有內部狀態（如門：locked → closed → open），不同狀態開放不同插座。

```
locked → [Unlock/BreakLock] → closed → [Open] → open → [Operate] → closed
```

## 實作狀態

| 階段 | 狀態 |
|------|------|
| 純觀看（Look/Read/Smell） | ✅ 已完成 |
| 歸屬權 + 偷竊 | 🔲 設計完成，未實作 |
| 有狀態物件 | 🔲 設計完成，未實作 |
| 物件環境敘事 | 🔲 設計完成，未實作 |
| 互動產出（對接經濟） | 🔲 規劃中 |
| 導航出口融合 | 🔲 UI 排程 |

## 靜態檢查

`cargo run --bin checkrooms -- -brackets -strict`：驗證 Move/Look 契約 + `〔〕` 必對應 `objects[].name`，失敗則中止建置。

## 相關頁面

- [[concepts/combat-system/index|戰鬥系統]] — 插頭插座語義
- [[concepts/combat-system/philosophy|戰鬥哲學]] — 借物系統
- [[concepts/npc-system/index|NPC 系統]] — NPC 看守、反應
- [[concepts/economy/index|經濟系統]] — 物件互動產出
