---
title: Hex 格網與地形
tags:
  - system
  - hex-map
  - terrain
  - singularity-world
sources:
  - docs/reference/Hex地形與移動係數表.md
  - docs/reference/Hex格座標與cell_id規約.md
  - docs/reference/Hex格網—生成與計算.md
  - docs/reference/map_terrain_world.md
date: 2026-04-11
aliases:
  - Hex格網
  - Axial座標
  - 地形移動係數
---

# Hex 格網與地形

> Axial 座標系統、cell_id 規約、格網尺度、資料結構、地形總表（完整 34 種）、移動速度公式、地上物分層、地形字對應。

## 座標系統

本專案採用 **Axial 座標 (q, r)**，**平頂（flat-top）** 六角格。`q`、`r` 為 **32 位元帶符號整數**（JSON 以 number 傳輸時不得丟精度）。

- 六向鄰接方向：東、東北、西北、西、西南、東南（對應 `src/hex/coord.rs` 之 `HexDir`）
- 螢幕投影與 even-q 偏移由 `editor-leptos` 的 `hexToPixel` 處理
- 外部資料（如 Watabou）須在**匯入管線單點換算**為本專案 axial 後再產生 `cell_id`
- 不在 cell_id 規約中另定義投影或 even-q/odd-r 偏移

### 螢幕方位表述慣例

平頂六角沒有「純上/純下」一格一邊的鄰居。協作時建議：

- **最精準**：直接報 `cell_id`（`q,r`）或「從 `(q,r)` 往 **東/東北/西北/西/西南/東南** 一格」
- **粗略**：可說「畫面上偏左/偏右/偏上/偏下那一塊」，但同一句話請盡量補上座標或截圖

**狀態**：【已實作】`src/hex/coord.rs`

## cell_id 規約（v0.1）

**正規格式**：`{q},{r}` — 無空白、無前後綴、負數直接寫負號。

| `q` | `r` | `cell_id` |
|-----|-----|-----------|
| 0 | 0 | `0,0` |
| 5 | -2 | `5,-2` |
| -10 | 7 | `-10,7` |

- **正則驗證**：`^(-?\d+),(-?\d+)$`
- **JSON 物件形式**（可選）：`{ "q": 5, "r": -2 }`，與字串互轉規則：`format = "{q},{r}"`
- 同一事件/同一 log 欄位建議**擇一貫通**（全字串或全物件），避免混用
- **`chunk_id` 不併入 `cell_id`**，另軌處理；拓撲與依附檢核仍以世界座標 `q,r` 為準
- 目前 `data/hex/grid.json` 內每格 `HexCell.coord` 與 `cell_id` 一一對應，執行期由 `coord` 依規約序列化即可
- 若未來變更座標系，應 Bump 規約版本並提供遷移腳本；舊 `cell_id` 不得靜默 reinterpret

**狀態**：【規格】v0.1（2026-04）

## 格網尺度

### 六角格幾何（正六角，邊長 s）

| 朝向 | 寬 | 高 |
|------|-----|-----|
| 尖頂 pointy-top | s*sqrt(3)（≈173.2 @s=100）| 2s（= 200 @s=100）|
| 平頂 flat-top | 2s（= 200 @s=100）| s*sqrt(3)（≈173.2 @s=100）|

### 實際尺寸參數（2026-04-10 定案）

**N=6 的設計用途**：一格內要容納約 6 個角色印記時，反推邊長、相機與字級。

| 參數 | 螢幕值 | 真實值 | 備註 |
|------|--------|--------|------|
| HEX_R（外接圓半徑）| 114 px | 5.3 m | 世界座標；正六角邊長 = 外接圓半徑，故 s=100 等同 HEX_R=100 |
| 格寬（flat-to-flat）| 197 px | 9.2 m | 約 14 步跨距 |
| 字圓直徑 | 28 px | 1.3 m | 語義 = 成人一步跨距（~0.65 m）|
| 字圓半徑 | 14 px | 0.65 m | 一步跨距 |
| 地形字 | 48 px | — | Chiron GoRound TC 字型 |
| 比例尺 | 21.5 px/m | — | |
| 手機可見格數 | ≈ 2 橫排 | — | 390px 寬基準，拖曳查看其餘 |

玩家端渲染用 **HEX_R = 57 px**（格寬 ~98 px）。

**狀態**：【已實作】`editor-leptos` 與 `map_terrain_world.md` 定案

## 資料結構

- **`HexCell`**：含 `coord`、`terrain`、`zone`、`tags`
- **`HexGrid`**：含 `cells` 陣列、`barriers`（`(coord, dir)` 單向擋）、`portals`（非相鄰連通）
- 鄰接由幾何決定；屏障以 `(coord, dir)` 單向擋；portal 做非相鄰連通
- **非聚落核**常指 `Urban` 聚落核以外的格（實務上多以 `zone`/`tags`/世界分區區分）

**狀態**：【已實作】`src/hex/grid.rs`、`src/hex/cell.rs`

## 地形係數完整總表

| Terrain 枚舉 | 中文名稱 | 可步行 | move_cost |
|---|---|:---:|:---:|
| `Road` | 道路 | 是 | 0.5 |
| `Bridge` | 橋樑 | 是 | 0.5 |
| `Grassland` | 草原 | 是 | 1.0 |
| `Farmhouse` | 農舍 | 是 | 1.0 |
| `Inn` | 旅店 | 是 | 1.0 |
| `Tavern` | 酒館 | 是 | 1.0 |
| `Blacksmith` | 鐵匠鋪 | 是 | 1.0 |
| `GeneralStore` | 雜貨店 | 是 | 1.0 |
| `Clinic` | 醫館 | 是 | 1.0 |
| `Workshop` | 工坊 | 是 | 1.0 |
| `Market` | 市集 | 是 | 1.0 |
| `GuildHall` | 公會大廳 | 是 | 1.0 |
| `Temple` | 神殿 | 是 | 1.0 |
| `Academy` | 學院 | 是 | 1.0 |
| `Library` | 圖書館 | 是 | 1.0 |
| `Barracks` | 兵營 | 是 | 1.0 |
| `GuardPost` | 衛所 | 是 | 1.0 |
| `Warehouse` | 倉庫 | 是 | 1.0 |
| `Granary` | 糧倉 | 是 | 1.0 |
| `Dock` | 碼頭 | 是 | 1.0 |
| `Bathhouse` | 浴場 | 是 | 1.0 |
| `Courthouse` | 法院 | 是 | 1.0 |
| `Jail` | 監所 | 是 | 1.0 |
| `TownHall` | 市政廳 | 是 | 1.0 |
| `Bank` | 銀行 | 是 | 1.0 |
| `Mint` | 鑄幣所 | 是 | 1.0 |
| `Stables` | 馬廄 | 是 | 1.0 |
| `Caravanserai` | 商旅驛站 | 是 | 1.0 |
| `Theater` | 劇院 | 是 | 1.0 |
| `Arena` | 競技場 | 是 | 1.0 |
| `Observatory` | 觀測台 | 是 | 1.0 |
| `Alchemist` | 鍊金工房 | 是 | 1.0 |
| `MageTower` | 法師塔 | 是 | 1.0 |
| `Embassy` | 使館 | 是 | 1.0 |
| `PrisonYard` | 囚院 | 是 | 1.0 |
| `Forest` | 森林 | 是 | 1.5 |
| `Hills` | 丘陵 | 是 | 1.5 |
| `Desert` | 沙漠 | 是 | 1.5 |
| `Tundra` | 凍原 | 是 | 1.5 |
| `FarmField` | 農田 | 是 | 1.5 |
| `Jungle` | 叢林 | 是 | 2.0 |
| `Swamp` | 沼澤 | 是 | 2.0 |
| `Water` | 水域 | 否 | Infinity |
| `Mountain` | 山地 | 否 | Infinity |
| `Wall` | 牆體 | 否 | Infinity |

### 規則重點

- `walkable = false` 的地形（Water、Mountain、Wall）不可進入
- `move_cost` 越高代表越慢，1.0 為基準，0.5 代表更快
- Infinity 表示不可通行（設計語義上等同阻擋）
- **move_cost 是地形（地理）阻力，與地上物無關**——樹、礦等地上物是格內實體，佔位阻擋路線，玩家須繞行

**狀態**：【已實作】`src/hex/cell.rs` 之 `Terrain::walkable()` 與 `Terrain::move_cost()`

## 移動速度公式（2026-04-10 定案）

```
base_speed = 60 px/s（真實步行 x2 倍率）
actual_speed = base_speed / move_cost
```

| 地形 | move_cost | 實際速度 | 穿越一格耗時 |
|------|-----------|---------|-------------|
| 道路 Road | 0.5 | 120 px/s | ~0.8 秒 |
| 草原 Grassland | 1.0 | 60 px/s | ~1.6 秒 |
| 森林 Forest | 1.5 | 40 px/s | ~2.5 秒 |
| 沼澤 Swamp | 2.0 | 30 px/s | ~3.3 秒 |

玩家點擊目標位置後字圓以 `actual_speed` 直線移動，撞到不可通行實體即停止。**無自動尋路**——系統不替玩家規劃路線，玩家自行判斷繞行。

路快野慢，move_cost 本身就是「有路不走自找苦吃」的機制。不需要額外的快速旅行系統。

### NPC 尋路注意

目前 `HexGrid::find_path()` 是 BFS（按步數最短），**尚未使用 `move_cost` 作加權最短路**。此函式用於 NPC 尋路，非玩家移動（玩家為直線移動、無尋路）。

**單步能否走**：`HexGrid::can_walk(from, to)` — 目標格存在、相鄰、目標地形 `walkable`、該向無 barrier。

## move_cost 與地上物分層（2026-04-10 定案）

| 層 | 定義 | 性質 | 範例 |
|---|---|---|---|
| **地形（move_cost）** | 地面的物理性質 | 不可改變（[[worldview\|Token 輻射]]下萬物富態化瘋長，砍了也長回來）| 泥沼黏腳、碎石崎嶇、草皮平坦 |
| **地上物** | 格內實體，各有座標、佔位 | 阻擋移動路線，玩家須繞行 | 樹、礦脈、灌木、巨石 |

**設計依據**：
- Token 輻射下萬物富態化瘋長，伐木清障無意義——人走樹長，地上物視為**永久地景**
- move_cost 是踏上這塊地的基底代價（腳下的泥還是石頭），地上物是路上的阻擋（繞不繞得過去）
- 同樣是森林格（move_cost=1.5），樹少的格穿越容易，樹多的格可能擠到走不過去——**密度決定實際可通行性**，而非地形類型一刀切
- 資源點（礦、特殊植物）就是地上物本身，玩家走到旁邊才能互動採集

**格內移動規則**：
- 玩家在格內以 `actual_speed`（受 move_cost 影響）連續移動
- 碰到地上物實體即停止，須點擊繞開
- 地上物之間的間隙即為可通行路徑——密林中窄縫可穿，但速度疊加地形減速
- 資源採集 = 走到地上物旁邊 + 互動，不是站在格上就能採

## 地形字對應

大地圖只存各地形**類型一字**，顯示時從候選字亂序取字。實作見 `src/world/terrain_display.rs`。

| 鍵（存檔用） | 名稱 | 候選字 |
|---|---|---|
| 木 | 樹木 | 木/林/森 |
| 山 | 山脈 | 山/岳/巒 |
| 石 | 碎石 | 石/磊/岩 |
| 沼 | 沼澤 | 沼/澤/泥 |
| 川 | 河流 | 蜿/巜/巛 |
| 水 | 水域 | 水/沝/淼 |
| 草 | 草原 | 艸/芔/茻 |
| 荒 | 荒地 | 荒/旱/焦 |
| 道 | 道路 | 道/路/徑 |
| 巷 | 巷 | 巷 |
| 火 | 岩漿 | 炎/焱/燚 |
| 冰 | 寒冰 | 冰/凍/冽 |
| 田 | 農地 | 田/畓/畕 |
| 谷 | 深谷 | 谷/豀/豁 |
| 霧 | 迷霧 | 霧/靄/霙 |
| 牆/門/關 | 邊界與門狀態 | 單字 |
| 地 | 房屋內部地板 | 地 |

共 17 類地形字對應。

**狀態**：【已實作】`src/world/terrain_display.rs`

## 新角色出生

**唯一規則**：新角色出生在**草原**。實作固定世界座標 **(q,r)=(0,0)**：該格契約為 `Terrain::Grassland`（`hex_editor::ensure_player_spawn_grassland_coord`）；`entities.hex_q`/`hex_r` 於創角成功後寫入 PostgreSQL。**創角不再寫入 `entity_rooms`**。舊版 MUD 房間視野仍於首次登入時由 `ensure_entity_in_room` 填預設房，待 WebSocket 改為純六角視野後可移除。

## 相關頁面

- [[hex-map/index|地圖系統總覽]]
- [[exploration|探索揭露]] — 黑格揭露與生成機制
- [[settlements|聚落與野外]] — 聚落門檻與佈局
- [[rendering|渲染管線]] — 前端顯示與 Hex 眼圖
