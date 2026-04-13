---
title: 戰鬥哲學 — 留人/送行、借物、死亡規則
tags:
  - system
  - combat
  - philosophy
  - singularity-world
sources:
  - docs/reference/借物與戰鬥意圖—用語與系統草案.md
  - docs/reference/死亡與昏迷—規格.md
  - docs/reference/機構賦權—討論稿.md
  - docs/decisions/001_combat_unified_rules.md
date: 2026-04-11
---

# 戰鬥哲學 — 留人/送行、借物、死亡規則

> 戰鬥不只是傷害計算，更是**社會行為**。每次攻擊都是社會決策。

## 三位一體：借物 / 留人 / 送行

**設計語言**：婉而直接。不直稱竊、制伏、殺害，但一聽即明。

| 意圖 | 顯示名 | 插座 id | 意涵 |
|------|--------|---------|------|
| 非合意取物 | **借物** | `Borrow` | 不告而取為「借」 |
| 制伏 | **留人** | `Subdue` | 放倒但不送上路 |
| 致死 | **送行** | `Slay` | 送其最後一程 |

**對仗**：借物＝不告而取；留人＝先留下不送；送行＝送上路。規格內文仍可用「制伏/致死」指稱。

### 借物系統

- 目標：同房實體背包中 `qty > 0` 的物品（選品邏輯同 Trade）
- 三分支：成功 / 失手 / 被喝破（`NpcBehaviorReactionLine`）
- 事件型別：`borrow` / `borrow_fail`（與 Trade 分開）
- 與 Trade 對比：Trade 為雙方合意、鎂與物之原子交割；借物為單方嘗試移轉占有
- **未實裝**：容器/場所物、歸屬/看守（11.8）；喝破後自動開戰

### 留人（制伏）

- `ResolveV2` + `CombatOpt.Subdue = true`
- 敗方 HP 保底為 **1**（不觸發死亡流程）
- 敘事前綴「意在留人」
- 好感分離：`FavSubdue`（獨立於 `FavSlay`）
- **未實裝**：獨立 `unconscious_until` 欄位、甦醒 tick

### 送行（致死）

- 敗方 HP 可至 **0** → 觸發 NPC 死亡除名
- 敘事前綴「意在送行」
- 好感：`FavSlay`

### 舊客戶端遷移

過渡期：舊 `Attack` 請求由後端自動映射為 `Slay`（送行）。

---

## 死亡與昏迷規格

### 定義

| 術語 | 定義 | 系統判定 |
|------|------|---------|
| **死亡** | 氣血歸零，不再參與世界 | `Vit ≤ 0`（僅送行允許） |
| **留人（制伏）** | 放倒、暫失戰力 | `Vit = 1`（保底） |
| **昏迷（敘事）** | Vit=1 的文案表述 | 與留人一致 |

### 死亡排除處（已實作）

| 處所 | 規則 | 實作 |
|------|------|------|
| 房內可互動名單 | Vit ≤ 0 不出現 | `GetEntitiesInRoom` 過濾 |
| 排班 | 死亡 NPC 不排班 | `ApplySchedules` 跳過 |
| 腦驅動/尋路 | 死亡 NPC 不決策 | `GetNPCIDsWithRoom` 過濾 |

### NPC 死亡：區域事件與除名（已實作）

- **區域事件**：NPC 死亡 → `LogNPCEvent(EvtDeath)` → 向死亡房＋鄰房＋該 NPC 所指派場所之全部房間廣播「傳來消息：【{名}】倒下了。」
- **職業除名**：`RemoveAssignmentsForEntity` + `RemoveScheduleForEntity`（持久化）
- **entity_room**：死亡實體保留所在房間紀錄，僅排除可互動/可驅動名單

### 玩家死亡

- Vit ≤ 0 時戰報「你敗下陣來」（僅敘事）
- **重生/復活：未實作**
- 被留人擊敗：Vit 保底 1，不觸發死亡敘事

### 不變更處

- entity_room 不主動清空
- 重生/復活待日後補
- 獨立昏迷時間戳待日後補

---

## 機構賦權與戰鬥的交集

機構賦權（詳見[[concepts/economy/index|經濟系統]]）與戰鬥的交集：
- **借物歸還**：借物成功後，物品的歸屬權與經濟流轉由機構記帳
- **送行的經濟後果**：NPC 死亡 → 池 -1 錠（10,000 鎂）、全部職業除名
- **留人無經濟後果**：制伏不觸發死亡回收

---

## 備選用語（曾討論）

| 制伏備選 | 致死備選 | 語感 |
|----------|----------|------|
| 切磋 | 了斷 | 江湖套語 |
| 留情 | 奪命 | 一硬一婉 |
| 點到 | 見血 | 武術向 |
| 放倒 | 了結 | 口語直白 |

已定案為**留人/送行**。

## 相關頁面

- [[concepts/combat-system/index|戰鬥系統]] — 總覽、ADR-001
- [[concepts/combat-system/damage-chain|傷害鏈]] — 核心公式
- [[concepts/combat-system/objects-interaction|物件互動]] — 房間物件、歸屬權
- [[concepts/npc-system/index|NPC 系統]] — 死亡排除、好感度
- [[concepts/economy/index|經濟系統]] — 機構賦權、鎂池
