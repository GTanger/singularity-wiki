---
title: 技術架構總覽
tags: [architecture, tech-stack, rust, postgresql]
sources:
  - decisions/004_tech_stack_architecture.md
  - decisions/008_架構整頓規劃.md
  - docs/技術約束規則.md
  - docs/COLLABORATION.md
date: 2026-04-11
---

# 技術架構總覽

## 系統定位

Singularity World 是單機部署的 MUD 文字遊戲伺服器，透過 Cloudflare Tunnel 對外，支援多玩家同時連線。核心設計原則：**PostgreSQL 為唯一持久層，記憶體為快取，JSON 僅為種子與靜態設定。**

## 子頁索引

- [[tech-stack]] — 技術選型決策（Rust/PostgreSQL/前端/Ollama）
- [[room-system]] — 房間＝格點＝視野範圍設計
- [[observation-system]] — 觀測分級與算力配置
- [[implementation-pipeline]] — 開發流程、品質閘門、設定參數
- [[architecture-refactor]] — 2026-02 架構整頓紀錄（god-package 拆分）
- [[collaboration]] — 協作約定與設計邊界

## 技術棧速查

| 層 | 選擇 | 備註 |
|---|---|---|
| 後端語言 | Rust (stable) | axum 0.8, tokio, serde |
| HTTP/WebSocket | axum 內建 | 單程序雙協議 |
| 資料庫 | PostgreSQL | 唯一權威持久層 |
| AI | Ollama 本地 | reqwest HTTP API |
| 前端 | 原生 HTML/CSS/JS | 無框架，PWA |
| 部署 | systemd user service | PORT=1721, Cloudflare Tunnel |

## 核心架構決策

### 懶評估（Quantum Collapse）

90,000 NPC 不主動模擬。被觀測時才坍縮出狀態。事件日誌回溯重建歷史狀態——不儲存每 tick 快照，只儲存事件差分。

### 單一世界時鐘

所有 NPC 共用同一遊戲時間軸。無「個人時間線」。Tick 錯開由隨機初始延遲實現，非獨立時鐘。

### 持久層唯一性

新功能禁止只寫 JSON 不落庫。執行期不以 JSON 為真理——JSON 僅種子、靜態設定、可選備份。

## 相關決策紀錄

- ADR 004：技術選型
- ADR 005：房間＝格點＝視野
- ADR 006：登入與玩家模板
- ADR 008：架構整頓
- ADR 009：觀測分級與行程約束
- ADR 010：Go→Rust 遷移