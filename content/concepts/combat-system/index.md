---
tags:
  - system
  - combat
  - singularity-world
aliases:
  - 戰鬥系統
sources:
  - docs/reference/戰鬥系統概念—三版融合.md
  - docs/reference/戰鬥系統概念公式—提案版.md
  - docs/reference/戰鬥系統概念設計—提取自gemini概念檔.md
  - docs/reference/借物與戰鬥意圖—用語與系統草案.md
  - docs/reference/死亡與昏迷—規格.md
  - docs/reference/機構賦權—討論稿.md
  - docs/decisions/001_combat_unified_rules.md
  - docs/implementation/011_戰鬥接線三件套—碼農工單.md
  - docs/房間非人物件互動.md
date: 2026-04-11
---

# 戰鬥系統

> **專案**：奇點世界（Singularity World）

戰鬥系統採**全自動文字結算**——後端一次性計算並產出文字戰報，前端僅顯示。設計核心為 ADR-001「同一套規則」：玩家 vs NPC、NPC vs NPC、玩家 vs 玩家皆走同一組公式與邏輯，不因實體身份區分流程。

戰鬥不只是傷害計算，更是**社會行為**。每次攻擊都是社會決策，玩家必須選擇意圖：「留人」（制伏）或「送行」（致死），這個選擇決定戰鬥結束條件、後果、以及 NPC 態度變化。

## 子頁面

- [[damage-chain|傷害鏈]] — 核心公式、命中/傷害/減傷/相位干涉、地形環境共鳴、元素/狀態異常
- [[philosophy|戰鬥哲學]] — 留人/送行、戰鬥即社會行為、借物系統、死亡/昏迷規則
- [[objects-interaction|物件互動]] — 房間非人物件、可攻擊判定、戰鬥觸發

## 關鍵設計決策（ADR-001）

- **玩家 = NPC**：不因「是否為玩家」區分規則，無 PvE/PvP 兩套
- **後端唯一真理**：可執行性、結算、戰報皆在後端；前端為顯示窗口，不得預測勝負
- **插頭/插座語義**：Attack 為插頭，目標需具備 `[Agent]` 插座
- **田忌賽馬**：勝負靠裝備、技能、屬性配置，不靠操作速度
- **文字戰報**：後端產出結構化 log（`[Initiative]`、`[Action:Skill]`、`[Action:Miss]`、`[Result]` 等標籤），前端依標籤渲染

## 三版融合

戰鬥公式歷經三份文件的設計迭代，最終融合為單一參考：

| 版本 | 主要貢獻 |
|------|----------|
| Gemini 概念檔（提取版） | 戰鬥時序模型、環境共鳴、SkillObject 定義 |
| 提案版 | 將所有概念統一為單式鏈；未實裝項以預設值代入 |
| Gemma3 版 | 綜合傷害公式、CT 閾值觸發狀態異常、元素/環境乘數 |

**融合原則**：所有已知概念元素皆納入公式，未實裝時以 `1`（乘性中性）或 `0`（加性中性）代入，公式仍成立。日後逐步啟用各概念時，只需替換預設值，不改公式結構。

## 詞盤與技能對接

戰鬥與[[concepts/wordplate-system/index|詞盤系統]]透過 `SkillObject` 銜接：

| 欄位 | 說明 |
|------|------|
| `base_dmg` | 基礎威力（詞元能階加總 × α） |
| `ap_threshold` | 行動槽門檻 |
| `qi_cost` | 氣脈消耗基礎值 |
| `resonance_tier` | 共鳴階級（一階 1.0 / 二階 1.5 / 三階 2.5） |
| `attribute_tags` | 屬性標籤 |
| `effects` | 附加效果 |

**執行流**：蓄能（AP）→ 檢核（Qi ≥ Qi_Drain）→ 結算（Formula）→ 回饋（Log）

## 實作狀態

### 已實作

| 項目 | 實作位置 |
|------|----------|
| `ResolveV2` 核心結算 | `src/combat/mod.rs` |
| 先手判定（Dex） | `resolve_v2` |
| 命中/閃避公式 | `hit_check` |
| 地形修正代入 | `terrain_from_room` |
| 留人/送行分流 | `CombatOpt.Subdue` |
| SoulSeed → 戰鬥係數 | `expand_soul_seed_to_combat_axes` |
| 裝備屬性欄位 | `effective_stats` |
| 死亡排除（房間/排班/腦驅動） | store + SQL 路徑 |
| NPC 死亡除名與區域事件 | handler + narrate |
| 借物/留人/送行插座 | `do_action` handler |
| 戰報標籤 | `[Initiative]` `[Action:Skill]` `[Action:Miss]` `[Result]` |

### 未實裝

行動槽時序（AP/Tick）、詞盤技能結算（SkillObject）、共鳴階級倍率、元素傷害、狀態異常 CT、環境影響、姿態/僵直系統、氣脈消耗與虛弱、破甲、反彈、連擊門檻、玩家重生/復活、借物喝破後自動開戰、獨立昏迷時間戳。

## 相關頁面

- [[concepts/character-system/index|角色系統]] — 外顯三維（Vit/Qi/Dex）、SoulSeed、角色實體結構
- [[concepts/npc-system/index|NPC 系統]] — NPC 決策引擎、腦驅動、好感度、死亡排除邏輯
- [[concepts/wordplate-system/index|詞盤系統]] — 詞元、造句共鳴、SkillObject 編譯、技能對接戰鬥
- [[concepts/economy/index|經濟系統]] — 機構賦權、借物歸還與經濟流轉
