---
title: 地圖系統總覽
tags:
  - system
  - hex-map
  - singularity-world
sources:
  - docs/reference/Hex地形與移動係數表.md
  - docs/reference/Hex探索揭露與漸進生成—規格草案.md
  - docs/reference/Hex格座標與cell_id規約.md
  - docs/reference/Hex格網—生成與計算.md
  - docs/reference/map_terrain_world.md
  - docs/design/世界地圖製作管線—程序化生成方案.md
  - docs/design/地圖渲染—單一管線與雙前端共用.md
  - docs/design/野外大地圖區域與場所分級規格.md
  - docs/design/範例_兩城之間路線與可能經歷.md
  - docs/decisions/005_room_cell_view_scope.md
  - docs/workorders/工單—玩家前端Hex眼圖MVP.md
date: 2026-04-11
aliases:
  - 地圖系統
  - 六角格地圖
---

# 地圖系統

> **專案**：奇點世界（Singularity World）
> **狀態標記**：【已實作】= 程式碼已落地；【規格】= 設計定案但未實作；【草案】= 仍在討論；【已棄用】= 保留歷史參考

地圖系統以 Hex 格網為基礎，涵蓋座標系統、地形移動、探索揭露、聚落生成、渲染管線。核心設計為**探索式生成**——世界不預先建好，而是隨玩家揭露黑格而逐步生成並釘死。

## 子頁面

- [[grid-and-terrain|Hex 格網與地形]] — Axial 座標、cell_id 規約、地形移動係數、格網尺度、地形字對應、移動速度公式、地上物分層
- [[exploration|探索揭露]] — 黑格/彩格/契約層/精煉層、單調精煉、設施三態、觸發與 FallbackMin、黑海與版圖擴張
- [[settlements|聚落與野外]] — 聚落門檻與降級、野外大地圖三層分級（Region/Field Node/Site）、城際距離、節點配額
- [[rendering|渲染管線]] — 雙前端共用、三層地圖、Hex 眼圖 MVP
- [[travel-experience|城際旅行體驗]] — 兩城之間路線設計、三段互動骨架、事件槽機制

## 核心決策

### ADR 005：空間單位與視野（已決斷）

**一格 = 一間房 = 玩家視線所及的空間。**

| 用語 | 含義 |
|------|------|
| **格子** | 世界的最小空間單位 |
| **房間** | 一個 MUD 節點：有 id、名稱、描述、出口、房內實體（對應 `rooms`/`entity_room`）|
| **視線所及** | 玩家當前所在的那一個空間單位內能看到的一切 |

- 玩家在「某一格」即在「某一間房」，視線範圍即該房間；無半格或跨房視線
- 視野內實體 = 與玩家同格/同房的實體（同房 NPC、其他玩家等）
- 觀測觸發、坍縮範圍、NPC 列表皆以當前房間為範圍
- 決策 004（視野內即時模擬）：僅同房 NPC 進入即時模擬，其餘惰性 + 事件回推
- 日後擴充為 2D 俯視地圖時，一格對應一房，移動一格即沿出口進相鄰房

### 兩張連通圖（定案）

| 圖 | 邊來源 | 用途 |
|----|--------|------|
| **官方聯外子圖** | 合法鄰接步 + `link_class: official` 之 `transport_edges`；`portal` **預設不計入**，除非標 `counts_as_official_link` 且驗證通過 | 幹線計數、依附、降級、Gate |
| **探索/玩法圖** | 上列併入捷徑邊、`shortcut`、`portal` 等 | 事件、AI、玩家抄近路 |

捷徑**可通**但**不當官方聯外**；升格須明流程與資料標記。

**程式**：`HexGrid::find_path_layer(_, _, LinkLayer::Official | Exploration)`；`find_path`/`reachable_from` 預設 **Exploration**。僅雙端皆 `Cell` 之 `transport_edges` 參與格上 BFS；`settlement_id` 端點待聚落表再接。

**拓撲指紋**：
- `topology_hash_public`：僅已揭露 + 對玩家可見之邊
- `topology_hash_authoritative`：伺服器已落盤之全圖（含未開霧之幹線）
- 依附與 Gate 以 authoritative + official 為準

### B 軸 Hex 格與 Field Node 的量綱差異

- **B 軸 Hex 格**：1 格 = 1 個六角格（axial `q,r`），資料載體為 `HexCell`
- **Field Node**：野外大地圖上的可抵達點，城際 hop 距離以 Field Node 計算
- 兩者**不同量綱**——Field Node 圖上的「1 格」不等於 Hex 圖上的「1 步」
- 接軌規則見[[settlements|聚落與野外]]

## 世界地圖製作管線

### 歷史方案（已棄用 2026-04-09）

- Azgaar Fantasy Map Generator 出世界骨架（地圖 Celia，seed: 938784876 -> 已更換為 Chia）
- Watabou 全家桶往下鑽城鎮/地城內部（City/Village/Cave/Dungeon/Dwelling）
- Parser A（Azgaar Full JSON -> 世界房間，省級聚合或城鎮級直出）+ Parser B（Watabou GeoJSON -> 地點內部，幾何推算拓撲）
- 舊管線基於 Celia 9MB Full JSON：7691 Voronoi cell、100 國、607 省、623 城鎮、504 路線、71 地標、383 河流、4659 歷史筆記
- **棄用原因**：探索式生成取代預建世界；Watabou 城市生成器被 Hex 格聚落佈局取代

### 現行路線

- **探索式生成**（觸發式預展開）：玩家揭露黑格 -> 後台生成地形、聚落、資源
- **聚落佈局**：道路驅動 + 功能格池 + 三層放置公式
- **世界骨架**：Azgaar（地圖 Chia，901 burgs），錨點整合方案待定
- 舊 Parser A 產出已清空、舊浮生城 639 房封存於 `data/rooms/archive/`

## 相關頁面

- [[worldview|世界觀]] — Token 降維與生命演化
- [[economy/index|經濟系統]] — 鎂與詞元、經濟引擎
- [[economy/resources-and-society|資源與社會]] — 資源點與 Token 密度
- [[npc-system/index|NPC 系統]] — NPC 行為與決策引擎
