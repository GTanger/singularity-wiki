---
title: NPC 系統
tags:
  - npc
  - core-system
  - singularity-world
  - game-design
sources:
  - docs/NPC活化系統.md
  - docs/reference/NPC相關設定—已實做與未實做.md
  - docs/reference/奇點決策引擎架構.md
  - docs/reference/奇點馬斯洛需求系統.md
  - docs/discussions/002_NPC需求驅動與求職機制.md
  - docs/implementation/NPC活化系統—實作清單與實作計畫.md
  - docs/implementation/NPC生成流程調整—依討論001.md
  - docs/implementation/NPC對話記憶與背版—設計.md
  - docs/implementation/NPC對話記憶與背版—實作步驟與檔案流程.md
  - docs/reference/記憶系統對照NPC活化系統.md
  - docs/discussions/003_NPC交互對話系統.md
  - docs/discussions/004_Web_LLM_調度與複數玩家.md
  - docs/reference/NPC之間交互行為.md
  - docs/dialogue_forge_plan.md
  - docs/dialogue_forge_walkthrough.md
date: 2026-04-11
---

# NPC 系統

**奇點世界**最大的單一系統。NPC 從需求出發產生動機、從動機產生行動、從行動產生軌跡與相遇，最終在對話中展現人格與記憶。

> 核心哲學：**造土壤，不是寫劇本**——設計者不替 NPC 做決定，只提供環境壓力，讓行為自然湧現。

---

## 設計哲學

### 需求驅動生活，不是腳本驅動表演

NPC 的一切行為從需求出發：餓了、窮了、累了、孤單了、不甘心了。需求產生動機，動機產生行動，行動產生軌跡，軌跡產生相遇，相遇產生對話。**人之所以像人，不是因為每步都走對，是因為不管怎樣都會動。**

### 造土壤而非寫劇本

設計者的角色是觀測者——偶爾介入，但不替 NPC 做決定。NPC 的行為多樣性來自：

- **簡單的需求規則**在**豐富的環境**中交互作用
- **不同的性格參數**面對**相同的困境**產生不同的反應
- **行為的連貫性**讓觀測者感受到「這個 NPC 有自己的人生」

螞蟻只有幾條規則，但蟻巢的行為看起來像有智慧。NPC 腦的設計同理。

### 同構、同驅、同感

NPC 與玩家共用同一套規則：同一個 `Character` 結構（[[soulseed|SoulSeed]]、三軸屬性、裝備、背包、鎂）、同一套戰鬥公式、同一套房間移動機制、同一套插頭插座互動。差異僅在驅動方式：玩家由人類操作，NPC 由模板 + 決策引擎驅動。

**三層「像人」**（討論 002 共識）：

| 層級 | 內涵 |
|------|------|
| **同構** | 同一套狀態、插座、世界規則；玩家能做的結構上 NPC 也能做 |
| **同驅** | NPC 也有需求（生存、賺錢、安定），需求驅動行為，不是腳本 |
| **同感** | 決策有偏好、有脈絡、有一致性；soul_seed 推性格、目標堆疊、情境選擇 |

**決策引擎**才是「能有多像」的瓶頸：最簡版＝固定優先序，進階＝soul_seed 權重與目標堆疊，完整＝目標規劃、記憶、情境推理。

### 世界是堆疊出來的

單一玩家不足以定義整個世界。**大量 NPC 在共享規則下反覆互動，才可能自然生成世界的樣貌**：移動與排班讓人同框，需求與經濟讓人競合，對話與記憶讓關係與話題可累積——世界因此長出可辨識的社會紋理。

> **一句話**：世界不是劇本寫死的，是鎮上許多人過日子堆出來的；玩家走進其中一條時間線，既是觀測者也是擾動者。

### 活化＝持續演進（無最終版）

本專案沒有「NPC 活化的封閉最終版」，只有不斷進化的版本。資料在變、邏輯在變、Prompt 在變、本地 LLM 模型在變。技術棧以 Rust 為準；Ollama 模型為可替換預設。

---

## 子頁面

| 頁面 | 說明 |
|------|------|
| [[npc-system/decision-engine\|決策引擎]] | The Brain——三層架構、馬斯洛需求層級、觀測分級與腦規模 |
| [[npc-system/dialogue\|對話系統]] | 玩家↔NPC 對話、NPC↔NPC 對話、從對話到交易 |
| [[npc-system/memory\|記憶系統]] | Letta 式四層記憶架構（L0 現場層–L3 社會事實層）、背版與 Archival |
| [[npc-system/identity-emergence\|身份湧現與社交行為]] | 身份是結果不是輸入、微互動、AI 對話、主題劇本、品質門檻 |
| [[npc-system/activation\|活化系統]] | 突破線 A–I、演進路線圖、實作清單與階段 |
| [[npc-system/observation\|觀測分級與驅動]] | 觀測四層級、觀測圈、未觀測世界 tick、行程約束 |
| [[npc-system/interaction\|NPC 間交互行為]] | 微互動、AI 對話、配對分數、玩家餘音、記憶四層模型 |
| [[npc-system/dialogue-pool\|對話池組合機制]] | 佔位符、詞表、片段組合、情境權重、微變體 |
| [[npc-system/dialogue-forge\|對話鍛造爐]] | 三層模型（0.8B 話癆→4B 日報→雲端萃取）、事件種子、世界詞典 |

---

## 實作狀態總覽

### 突破線 A–I（全部完成）

| 階段 | 名稱 | 核心效果 | 狀態 |
|------|------|---------|:----:|
| **A** | 鎂消耗 | NPC 會餓，馬斯洛鏈激活 | ✅ |
| **B** | Brain 停留 | NPC 到達後停留，不亂飛 | ✅ |
| **C** | 性格偏移決策 | 同境不同命，個體差異 | ✅ |
| **D** | NPC 事件日誌 | NPC 有「過去」，可供對話引用 | ✅ |
| **E** | disposition（心境值） | 情緒影響閒置文本口吻 | ✅ |
| **F** | NPC 間互動 | 微互動 + AI 對話 + 主題劇本 + 記憶 | ✅ |
| **G** | 背版組裝 | Talk 時帶入「我是誰」 | ✅ |
| **H** | archival 記憶 | NPC 記住跟玩家的過去 | ✅ |
| **I** | CallAITalk 接入 | LLM 優先 + 模板 fallback | ✅ |

> A-C 是「齒輪」（決策引擎運轉）→ D-F 是「模板突破線」（NPC 有故事）→ G-I 是「記憶層」（故事在 Talk 時說出來）。

### 已實作摘要

| 類別 | 項目 |
|------|------|
| **移動** | 四種移動模式（schedule/regional/route/pathfind）、BFS 尋路、TravelerManager |
| **排班** | GetScheduleTarget、ApplySchedules、觀測驅動 |
| **決策** | 腦驅動意圖（seek_job/beg/gather/trade/wander/work/idle）、降級鏈、慣性、表面行為觀測 |
| **對話** | 玩家↔NPC LLM 對話、模板 fallback、NPC↔NPC AI 對話、對話池組合句 |
| **記憶** | 背版、archival、consolidation、NPC 間摘要、thread、dyad、rumors |
| **互動** | Look/Talk/Attack/進房反應/插座列表 |
| **NPC 池** | 可設定總量 + 定時補滿 |
| **對話鍛造爐** | Phase 1 已上線（0.8B 話癆 + 事件種子 + 世界詞典表） |

### 未實作/規劃中

| 項目 | 狀態 | 備註 |
|------|:----:|------|
| Trade 完整交易流程 | ⬜ | 依世界物流規格實作 |
| 模板 NPC 生成器 | ⬜ | 讀 archetypes 批量生成 |
| 語意檢索（embedding + 向量） | ⬜ | 目前為多關鍵字評分 |
| 完整情緒狀態機 | ⬜ | 僅有 disposition 數值 |
| 離職/解僱/流動 | ⬜ | — |
| 傳聞系統完整版 | 🟡 | L3 已做，擴充中 |
| 戰鬥反應 | ⬜ | 待戰鬥討論定版 |
| 目標堆疊（V3 長效目標） | ⬜ | 無跨周期一致性 |
| 資源點建立 | ⬜ | 礦區房間、聚念場悖論 |
| 物品多樣化 | ⬜ | 目前僅 wild_herb |
| 對話鍛造爐 Phase 2/3 | ⬜ | 依 Phase 1 觀察結果決定 |
| 生命週期 / 社會關係 / 萬人規模 | ⬜ | 遠期 |

---

## 相關頁面

- [[worldview|世界觀與設定]] — Token 降維、富態與拉鋸、同頻相斥
- [[character-system|角色系統]] — SoulSeed、三軸性格、裝備、詞盤
- [[combat-system|戰鬥系統]] — 統一戰鬥規則、地形與 γ 暴擊/偏轉
