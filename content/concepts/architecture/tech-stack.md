---
title: 技術選型
tags: [architecture, rust, postgresql, frontend, ollama]
sources:
  - decisions/004_tech_stack_architecture.md
  - decisions/010_go_to_rust_migration.md
  - docs/reference/技術選型建議書.md
  - docs/技術約束規則.md
  - docs/COLLABORATION.md
date: 2026-04-11
---

# 技術選型

## 最終決策（2026 現行）

| 層 | 選擇 | 淘汰選項 |
|---|---|---|
| 後端語言 | **Rust** (axum 0.8 + tokio) | Go（已遷移）、Node.js、Python |
| 資料庫 | **PostgreSQL** | SQLite（已移除）、JSON-only |
| 前端 | **原生 HTML/CSS/JS** | React、Vue |
| AI | **Ollama 本地** (reqwest) | OpenAI API、本地 Python bridge |
| 部署 | **systemd + Cloudflare Tunnel** | Docker（未用）|

## 後端選型分析

技術選型建議書（2026-01）評估了 5 個選項：

### Rust（最終選擇）
- 優點：記憶體安全無 GC、極低延遲、強型別系統防止 NPC 狀態腐爛
- 缺點：學習曲線陡、編譯慢
- 關鍵依賴：`axum 0.8`、`tokio`、`serde`、`bcrypt`、`reqwest`、`anyhow/thiserror`、`tracing`、`rand`、`uuid`、`futures-util`、`tower-http`

### Go（前期用，已棄）
- 棄用原因：Rust 型別系統更適合複雜狀態機；遷移於 ADR 010 完成

### Node.js（否決）
- 否決原因：單線程事件迴圈在密集計算（NPC 決策 tick）時阻塞風險高

### Python（否決）
- 否決原因：GIL 問題、動態型別對遊戲狀態管理不利

### Cloudflare Workers（否決）
- 否決原因：執行時間上限 50ms，無法支援長連接 WebSocket

## 資料庫選型

### PostgreSQL（最終）
- 唯一權威持久層
- 支援 pgvector（embedding 搜索）
- JSON 僅為種子資料與靜態設定
- 新功能禁止只寫 JSON 不落庫

### SQLite（已完全移除）
- ADR 008 整頓時完全移除，無過渡期

## 前端選型

### 原生 HTML/CSS/JS（最終）
- 無框架、無編譯步驟
- PWA 支援（ServiceWorker + manifest）
- Canvas 2D 用於 Hex 地圖渲染
- WebSocket 原生 API 對接後端

### React/Vue（否決）
- 否決原因：框架 VDOM 開銷反而成為瓶頸；無框架可精確控制渲染時機

## AI / LLM 選型

### Ollama 本地（現行）
- 透過 HTTP API 呼叫，reqwest 異步
- 目前主力：2B～4B 模型（GPU 留給玩家 Talk）
- 品質閘門：`qualityGateNpcLine` + `ScoreNpcDialogue`（門檻 ≥35 分）

### OpenAI API（否決）
- 否決原因：敏感內容審查不適合遊戲文案；成本不可控；網路依賴

## 代碼風格硬規則

### 函式規範
- 每個函式必須有中文說明頭注
- 單函式最長 50 行
- 最多三層巢狀
- 禁止 `init()` 模式

### 檔案規範
- 單檔最長 300 行
- 單一職責原則

### 協作規範
- 不引入新 crate（需明確同意）
- 改完必跑 `cargo build` + `cargo clippy -- -D warnings`
- 改完必 `./start` 重啟

## 相關頁面

- [[observation-system]] — 算力配置策略
- [[implementation-pipeline]] — 品質閘門與設定參數
- [[architecture-refactor]] — 模組結構重整紀錄