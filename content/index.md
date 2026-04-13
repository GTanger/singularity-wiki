# Index — 奇點世界

> 奇點世界（Singularity World）的設計知識庫，由 LLM 從對話與文件中編譯而成。

## 導覽
- [[#核心系統]] · [[#設計與決策]] · [[#技術快照]] · [[#思維]] · [[#開放問題]]

## 核心系統

### 世界觀
- [[concepts/worldview|世界觀]] — Token 高維能量本體論、同頻相斥、富態化、聚念場、降維與星盤
    - [[concepts/worldview/token-ontology|Token 本體論]] — 高維頻率實體、同頻相斥、跨頻交互
    - [[concepts/worldview/dimensionality-reduction|降維與星盤]] — 降維通道、星盤映射、Token 濃度
    - [[concepts/worldview/rich-overgrowth|富態與拉鋸]] — 富態化、聚念場、能量飽和
    - [[concepts/worldview/stripping-and-equipment|剝名與裝備]] — 剝名粒度、裝備語義
    - [[concepts/worldview/electronics-and-machinery|電子與機械]] — 精密機械限制、文明科技邊界
    - [[concepts/worldview/cognition-layers|認知層次]] — 認知分級、對話 prompt 設計
    - [[concepts/worldview/logic-closure|邏輯閉環]] — 世界觀自洽性檢視、否決方向

### 角色系統
- [[concepts/character-system/index|角色系統]] — 外顯三維、SoulSeed、361 拓撲、迷霧機制
    - [[concepts/character-system/three-dimensions|外顯三維與四相資源]] — 體質/氣脈/靈敏、四相公式
    - [[concepts/character-system/soulseed|SoulSeed 與信號光譜]] — int64 種子、三軸光譜、性格推導
    - [[concepts/character-system/topology-361|361 拓撲系統]] — 四區分層、二十主樞、五常邏輯閘、760 條邊權
    - [[concepts/character-system/fog-and-identity|迷霧與身份]] — 迷霧三層、狀態/指派/身份分離、角色模板

### 詞盤系統
- [[concepts/wordplate-system/index|詞盤系統]] — 數學引擎、500 詞元向量、五層管線、造句、疊煉
    - [[concepts/wordplate-system/word-vectors|詞元與向量]] — 500 詞元 × 1030 維、十類語義
    - [[concepts/wordplate-system/five-layer-pipeline|五層管線]] — 採集→材料→剝名→結晶→提出
    - [[concepts/wordplate-system/sentence-crafting|造句系統]] — 向量引擎、三五七字句式、三階共振
    - [[concepts/wordplate-system/three-fork|三叉路口與疊煉]] — 入竅/附魔/鑄鎂、插座/插頭、疊煉系統
    - [[concepts/wordplate-system/skills-and-equipment|技能與裝備]] — 觸發權重公式、20 槽位、背包負重

### NPC 系統
- [[concepts/npc-system/index|NPC 系統]] — 需求驅動、造土壤不寫劇本、決策引擎、對話、記憶
    - [[concepts/npc-system/decision-engine|決策引擎]] — The Brain 三層架構、馬斯洛五層、觀測分級
    - [[concepts/npc-system/dialogue|對話系統]] — 玩家↔NPC、NPC↔NPC、對話品質管線
    - [[concepts/npc-system/dialogue-pool|對話池]] — 8 種延伸機制、佔位符系統、抽句規則
    - [[concepts/npc-system/dialogue-forge|對話鍛造爐]] — 三層漏斗、world_lexicon、event_seeds
    - [[concepts/npc-system/memory|記憶系統]] — L0 現場 / L1 話題 / L2 關係 / L3 傳聞
    - [[concepts/npc-system/activation|活化突破線]] — A–I 突破線詳細紀錄、實作進度
    - [[concepts/npc-system/interaction|NPC 間互動]] — 微互動、配對分數、玩家餘音
    - [[concepts/npc-system/observation|觀測系統]] — 觀測分級、腦規模、資源調度
    - [[concepts/npc-system/identity-emergence|身份湧現]] — 社交行為、微互動、品質門檻

### 戰鬥系統
- [[concepts/combat-system/index|戰鬥系統]] — 全自動文字結算、三版融合、ADR-001
    - [[concepts/combat-system/damage-chain|傷害鏈]] — 命中/傷害/減傷/相位干涉、地形共鳴、常數表
    - [[concepts/combat-system/objects-interaction|物件交互]] — 借物系統、環境物件、插頭/插座語義
    - [[concepts/combat-system/philosophy|戰鬥哲學]] — 留人/送行、戰鬥即社會行為、死亡規則

### 經濟系統
- [[concepts/economy/index|經濟系統]] — 鎂貨幣、湧現經濟、產消閉環
    - [[concepts/economy/currency-and-flow|貨幣與流通]] — 鎂面額、Faucet/Sink、鎂池、價格發現、六大物流
    - [[concepts/economy/resources-and-society|資源與社會]] — Token 密度、資源成熟度、社會組織四層次
    - [[concepts/economy/settlements-and-buildings|聚落與建築]] — 房間設計規範、建築分級
    - [[concepts/economy/trade-and-logistics|貿易與物流]] — 物流涵蓋一切、運輸成本、貿易路線

### 地圖系統
- [[concepts/hex-map/index|地圖系統]] — Hex 格網、探索揭露、聚落、渲染管線
    - [[concepts/hex-map/grid-and-terrain|格網與地形]] — Axial 座標、cell_id、地形移動係數
    - [[concepts/hex-map/exploration|探索與揭露]] — 黑格/彩格/契約層/精煉層、單調精煉、兩段式生成
    - [[concepts/hex-map/settlements|聚落系統]] — 五級門檻、降級規則、佈局生成、區域約束
    - [[concepts/hex-map/rendering|渲染與視野]] — 雙前端管線、野外三層、Hex 眼圖 MVP
    - [[concepts/hex-map/travel-experience|旅行體驗]] — 兩城路線、隨機事件、移動體驗設計

### 技術架構
- [[concepts/architecture/index|技術架構]] — 單機 MUD、Rust/PG/Ollama 技術棧、開發管線
    - [[concepts/architecture/tech-stack|技術選型]] — Rust + PostgreSQL + Ollama、Go→Rust 遷移
    - [[concepts/architecture/room-system|房間系統]] — 房間=格點=視野、描述生成、LLM 管線
    - [[concepts/architecture/observation-system|觀測分級]] — L0–L4 分級、行程約束、雲端 vs 本機
    - [[concepts/architecture/implementation-pipeline|開發管線]] — 碼農工單流程、DB 遷移、設定參數
    - [[concepts/architecture/architecture-refactor|架構整頓]] — ADR-008 god-package 拆分
    - [[concepts/architecture/collaboration|協作約定]] — 設計者/碼農/組長/審局者職責邊界

## 設計與決策
- [[concepts/design-decisions|設計決策與原則]] — 核心命題、七大設計支柱、10 份 ADR
- [[concepts/tool-strategy|器策略：裝備優先原則]] — MCP > Skills > CLAUDE.md、已落地裝備
- [[concepts/resource-pipeline|資源五層管線]] — 五層結構、11 項定案

## 技術快照
- [[concepts/rust-migration|Rust 遷移決策]] — Go→Rust 原因、技術棧約束、遷移策略
- [[concepts/hex-visual|Hex 視覺調整]] — 甲骨文字型、HEX_R 調整、Canvas 預載
- [[concepts/subcell-movement|格內連續移動]] — 像素級移動、步行/跑步、碰撞三階段
- [[concepts/deepseek-v4|DeepSeek V4 觀望]] — Benchmark 待驗、等 API + LMSYS

## 思維
- [[concepts/wenyanwen-compression|文言文壓縮框架]] — 錨點解壓、壓縮管線、與詞盤成句的關聯

## 開放問題
- Q1: 詞盤系統缺設計項（7 項待定，見[[concepts/wordplate-system/index|詞盤系統]])
- Q2: NPC 系統突破線 E–I 尚未實作（詳見 [[concepts/npc-system/activation|活化突破線]]）
- Q3: 經濟系統三階段實作順序待確認
- Q4: 架構整頓（god-package 拆分）進度追蹤，見 [[concepts/architecture/architecture-refactor|架構整頓]]
